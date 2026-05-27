# RESEARCH PAPER ANALYSIS REPORT — DEEP CRITIQUE

---

**Title:** Streaming Video Question-Answering with In-Context Video KV-Cache Retrieval
**Authors:** Shangzhe Di, Zhelun Yu, Guanghao Zhang, Haoyuan Li, Tao Zhong, Hao Cheng, Bolin Li, Wanggui He, Fangxun Shu, Hao Jiang
**Affiliations:** Shanghai Jiao Tong University; Alibaba Group
**Venue:** ICLR 2025 (Conference Paper)
**Reading Mode:** Deep Critique
**Analysis Date:** May 27, 2026
**Domain Classification:** Interdisciplinary — Machine Learning / Systems Engineering / Video Understanding
**Overview:** This report applies the full 3-pass Deep Critique framework to the ReKV paper, analyzing its problem formulation, methodological validity, empirical soundness, and limitations. The parent disciplines engaged are: (1) ML/AI systems, (2) computer vision & video understanding, and (3) systems engineering (memory, I/O, latency). Each is evaluated by its own standards.

---

## OVERVIEW MAP

```
                    ┌─────────────────────────────┐
                    │     ReKV Core Contribution   │
                    │  Training-free StreamingVQA  │
                    └─────────────┬───────────────┘
                                  │
          ┌───────────────────────┼────────────────────────┐
          ▼                       ▼                        ▼
┌──────────────────┐   ┌──────────────────────┐  ┌──────────────────────┐
│  Video Encoding  │   │   KV-Cache Retrieval  │  │   VideoQA Inference  │
│  (Sliding-window │   │  External (CLIP-like) │  │  (Retrieved KV as    │
│   Attention)     │   │  Internal (Attention  │  │   context for LLM    │
│  → RAM/Disk off- │   │   weights, faster)    │  │   autoregressive     │
│    load KV cache │   │  → cosine similarity  │  │   generation)        │
└──────────────────┘   └──────────────────────┘  └──────────────────────┘
          │                       │                        │
          └───────────────────────┴────────────────────────┘
                                  │
                    ┌─────────────▼───────────────┐
                    │       Decoupled Processes    │
                    │  Video Enc. ≠ QA Process     │
                    │  (Separate GPU / async)      │
                    └─────────────────────────────┘

Key Claim: Real-time VQA with no re-encoding, stable latency, better accuracy than
           uniform sampling baselines across 7 long-form VideoQA benchmarks.
```

*Overview Map: ReKV's three-component pipeline — encoding, retrieval, QA — together enabling decoupled streaming video question-answering.*

---

## EXECUTIVE SUMMARY

### Problem Statement

Long video question-answering (VideoQA) using Large Language Models (LLMs) is computationally prohibitive because existing models must process entire videos before answering any query, and must repeat this process for each new question. When video content arrives as a continuous stream — as in surveillance, robotics, or live broadcasting — this offline-first paradigm fails entirely: the model cannot answer questions in real time, and sparse frame sampling causes critical visual information loss. No training-free framework existed to systematically decouple video encoding from question-answering while preserving fine-grained context over long durations.

### Methodology

ReKV modifies existing Video-LLMs (without retraining) by (1) encoding video streams incrementally with sliding-window causal attention, (2) offloading all resulting KV-Caches to RAM/disk, and (3) at query time, retrieving only the most query-relevant KV-Cache slices (via cosine similarity using either an external CLIP-like model or internal attention weights), then using those retrieved vectors as context for autoregressive answer generation.

### Key Results

- ReKV (internal retrieval) improves over the uniform sampling baseline across all 7 tested benchmarks, with notable gains: +3.8% on MLVU, +3.2% on QAEGO4D, +3.8% on ActivityNet-QA (with LLaVA-OV-7B).
- Internal retrieval achieves higher recall than external retrieval on QAEGO4D (70.5% vs. 58.1%) and better QA accuracy (56.0% vs. 54.2% for LLaVA-OV-7B).
- Latency and GPU memory usage remain stable as frame count increases (11 FPS encoding, ~3.3s response latency for LLaVA-OV-7B), while full-frame and uniform sampling approaches exceed GPU memory bounds (OOM) on long videos.
- LLaVA-OV-7B + ReKV surpasses Flash-VStream on RVS-Ego (63.7 vs. 57.3 accuracy) and RVS-Movie (54.4 vs. 53.1) with comparable latency.
- Oracle retrieval (100% recall) still outperforms internal retrieval on QAEGO4D (64.4 vs. 56.0 for 7B), revealing a measurable gap from the performance ceiling.

### Main Claim

ReKV is a training-free, plug-in framework that enables any existing Video-LLM to perform efficient streaming video QA — with real-time latency, bounded memory, and accuracy superior to sparse frame-sampling baselines — by preserving and selectively retrieving precomputed KV-Caches.

### Verdict: **SOLID**

ReKV presents a technically credible, practically motivated solution to a real problem. The training-free design, stable system performance, and consistent benchmark gains are genuine contributions. The primary limitation is the non-trivial gap to oracle performance (recall still tops out at ~70%), incomplete analysis of the information-theoretic cost of KV-Cache compression, and concerns around benchmark leakage and generalization to non-egocentric video domains.

---

## PASS 1 — ORIENTATION

### Paper Classification

| Dimension | Assessment |
|---|---|
| Paper Type | System/method paper with empirical evaluation |
| Primary discipline | ML/AI systems (Video-LLM engineering) |
| Secondary disciplines | Computer vision, information retrieval, systems engineering |
| Venue norms | ICLR — evaluated by ML/AI standards (benchmark accuracy, efficiency) |
| Dominant claim type | Empirical (benchmark results) + systems (latency/memory) |
| Theoretical contribution | Minimal — no formal analysis of retrieval quality bounds |

### Initial Questions from Pass 1

1. Does the KV-Cache compression from sliding-window encoding preserve enough temporal information for holistic questions (which require understanding the whole video)?
2. How does ReKV handle videos where relevant events are uniformly distributed (not localized), where retrieval precision is inherently limited?
3. Does decoupling video encoding from QA introduce staleness or consistency issues when the stream is live?

---

## PASS 2 — SECTION-BY-SECTION ANALYSIS

![Figure 1: StreamingVQA Task Overview and System Performance Comparison](page_2.png)
*Figure 1 (from paper): Overview of the StreamingVQA task and ReKV's performance across seven benchmarks, showing stable GPU memory and latency as frame count increases.*

---

### Introduction

**What This Section Says**

The introduction motivates StreamingVQA by contrasting it with offline VideoQA, identifies three core challenges (efficient encoding, context preservation, real-time response), and positions ReKV as a training-free solution bridging these gaps. The problem is well-framed: offline VideoQA is impractical for real-time streaming because it requires complete video access and reprocesses frames per question.

**Key Questions**
- The introduction claims StreamingVQA "cannot afford to revisit distant past frames." But ReKV explicitly reloads KV-Caches from disk/RAM. Is the distinction between "revisiting frames" vs. "reloading KV-Caches" genuinely meaningful computationally, or largely presentational?
- "Real-time" is used throughout but never formally defined. Is 3.3 seconds acceptable latency for real-time applications like robotics or surveillance?

**Red Flags**
Real-time response is cited as a core challenge, yet the latency results (Table 5) show 3.3s response time for the 7B model. This is not real-time by most deployed robotics or surveillance standards, where sub-second latency is often required. The paper should acknowledge this limitation explicitly.

---

### Task Formulation (Section 2)

**What This Section Says**

StreamingVQA is formalized: a model receives frames $\mathcal{V}_t = [v_1, ..., v_t]$ and must answer question $q_i$ using only frames up to time $t$. OfflineVQA is positioned as a special case where all questions come after the full video. The section argues that causal masking in LLM decoders naturally enables video encoding decoupled from QA.

**Key Questions**
- The formalization assumes questions come with known timestamps (annotated end times in RVS benchmarks). In real deployment, users ask spontaneous questions. Does this assumption weaken the real-world applicability claim?
- The causal decoupling argument is presented informally. Is there any analysis of what the sliding-window encoding loses in exchange for efficiency?

**Red Flags**
The claim that OfflineVQA is "a special case of StreamingVQA" is technically true but slightly misleading: offline models can re-attend to all frames freely, whereas StreamingVQA with sliding-window attention cannot. This is a fundamental difference in expressive power, not merely a difference in deployment scenario.

---

### Method (Section 3)

**What This Section Says**

ReKV has three components: (1) sliding-window causal attention for video encoding, (2) either external (CLIP-based) or internal (attention-weight-based) KV-Cache retrieval, and (3) autoregressive answer generation using retrieved key-value vectors. Positional encoding is handled by applying standard RoPE to retrieved tokens sequentially, discarding their original positions.

![Figure 2: ReKV Architecture Overview](page_4.png)
*Figure 2 (from paper): ReKV's three-component architecture — sliding-window encoding, KV-Cache retrieval via cosine similarity, and autoregressive QA using retrieved vectors.*

**Key Questions**
- **Positional encoding is a major design choice**: retrieved tokens are treated as if they are consecutive in sequence, despite potentially spanning hours of video. Does this temporal position discarding hurt performance on questions requiring understanding of event order or duration?
- The internal retrieval uses the *average* of key vectors across all heads. Multi-head attention is designed for different heads to specialize in different aspects. Averaging heads discards this specialization — is this justified?
- Sliding-window attention has size set at 15K tokens. How was this hyperparameter chosen, and how sensitive is performance to this value?
- The paper says KV-Caches can be offloaded to RAM/disk, but does not report the I/O read speed when loading from disk vs. RAM, which may dominate latency for very long videos.

**Red Flags**
The positional encoding choice — treating retrieved tokens as consecutively positioned — is a significant departure from original semantics. This is acknowledged but not rigorously ablated. The paper reports that "applying standard RoPE to retrieved video tokens leads to better performance," but the margin and statistical significance of this comparison are not reported. This is a methodological gap for a claim that directly affects architecture validity.

---

### Experiments (Section 4)

**What This Section Says**

Experiments cover 7 benchmarks: MLVU, QAEGO4D, EgoSchema, ActivityNet-QA (offline), and RVS-Ego, RVS-Movie, CGBench (streaming/long-form). Ablations test retrieval method, number of retrieved frames, and block size. Results consistently favor internal retrieval over external retrieval and both over uniform sampling.

![Table 2 and Ablation Figures (Page 7)](page_7.png)
*Figures 3 and Table 2 (from paper): Ablation study results showing the impact of retrieval method, number of retrieved frames, and block size on VideoQA accuracy.*

![Tables 4 and 5 (Page 8–9)](page_8.png)
*Tables 4 and 5 (from paper): Offline and streaming VideoQA comparison results across all benchmarks and system metrics.*

![Qualitative Results and Table 5 continued (Page 9)](page_9.png)
*Figure 4 (from paper): StreamingVQA qualitative examples from the QAEGO4D benchmark, showing the model's ability to retrieve context at the correct timestamps.*

**Key Questions**
- EgoSchema accuracy for LLaVA-OV-0.5B with ReKV is 31.0%, only +1.4% over baseline (29.6%). Given that random chance for 5-choice QA is 20%, this is only marginally above chance. Is this benchmark even appropriate to demonstrate the method's value on this model?
- All benchmarks are evaluated at 0.5 FPS. This is appropriate for slow-paced videos (cooking, activities) but may fail for fast-action content (sports, surveillance). Was this choice justified or just convenient?
- How is GPT-3.5-turbo-0613 used for ActivityNet-QA scoring? This judge model introduces its own variability. Are inter-rater reliability metrics reported? (They are not.)
- The oracle recall gap: on QAEGO4D, internal retrieval achieves 70.5% recall while oracle is 100%. This means ~30% of truly relevant frames are still missed. For questions requiring precise detail, this could be critical. Is this gap expected to shrink with more retrieved frames, or does it plateau?

**Red Flags**

**Critical concern:** EgoSchema results for 0.5B model (+1.4 points) and 7B model (+0.9 points) are extremely marginal. Without confidence intervals or statistical significance testing, these results cannot be confidently attributed to the method rather than variance. The paper reports no error bars, no repeated runs, no p-values anywhere in the paper. For benchmark comparisons this close to baseline, this is a notable methodological omission.

**Second concern:** CGBench results are mentioned in Figure 1's radar chart (showing ReKV outperforms baselines) but Table 4 does not include CGBench. The omission of CGBench from the main offline comparison table is unexplained, despite it being highlighted in the abstract and in the introduction.

---

### Related Work (Section 5)

**What This Section Says**

The related work covers LLMs for video understanding (sparse sampling models), long video understanding (memory-bank models), long-context LLM techniques (StreamingLLM, LM-Infinite, InfLLM), and RAG/in-context retrieval. The positioning is clear and generally fair.

**Key Questions**
- The paper compares favorably to Flash-VStream but does not compare to VideoLLM-Online (Chen et al., 2024), which is also a streaming approach. VideoLLM-Online is discussed in Related Work but excluded from Table 5. Why?
- InfLLM (Xiao et al., 2024a) is the closest prior work in mechanism (storing and retrieving KV-Caches for long text). The paper acknowledges this but minimizes the difference (video vs. text domain). What is the actual novel contribution over InfLLM applied to video?

**Red Flags**
The omission of VideoLLM-Online from the streaming benchmark comparison is a noticeable gap. The stated reason (ReKV is training-free, VideoLLM-Online requires training) is a design choice, not a reason to exclude it from an empirical comparison table. A fair comparison should include the trained baseline as an upper bound.

---

## DOMAIN-SPECIFIC DEEP ANALYSIS

### 1. Disciplinary Map

| Parent Discipline | Role in Paper | Standards Applied | Conclusions |
|---|---|---|---|
| ML / Deep Learning | Model design, training-free adaptation | Accuracy on benchmarks, ablation completeness | Generally met; lacks statistical testing |
| Computer Vision | Visual encoding, CLIP-based retrieval | Feature quality, retrieval recall | Partially met; CLIP encoder choice not ablated |
| Systems Engineering | Latency, memory, I/O efficiency | FPS throughput, GPU memory, disk offload rate | Well-measured; disk I/O latency not reported |
| Information Retrieval | KV-Cache retrieval mechanism | Recall, precision, ranking quality | Partially met; no precision metric reported |

---

### 2. Contribution Taxonomy (ML/AI Paper)

| Category | Prior State | ReKV Contribution | Evaluation Standard |
|---|---|---|---|
| Dataset | Existing benchmarks (MLVU, EgoSchema, etc.) | Two streaming benchmarks (RVS) borrowed from prior work | Uses others' benchmarks; no new dataset |
| Architecture | Full-frame or sparse-sampling Video-LLMs | Sliding-window encoding + KV-Cache retrieval added as plug-in wrapper | Ablated; marginal improvement on some benchmarks |
| Training | Most streaming methods require training | **Training-free** adaptation — genuine strength | Confirmed across multiple Video-LLM backbones |
| Evaluation | Offline VideoQA benchmarks | Tests on 7 benchmarks including streaming-specific ones | Comprehensive, but lacks error bars |
| Baselines | Memory-based streaming (Flash-VStream) | Outperforms on streaming benchmarks | Fair comparison; VideoLLM-Online missing |

---

### 3. Per-Discipline Analysis

#### ML Component — Retrieval Design

The internal retrieval mechanism (averaging key vectors across attention heads, computing cosine similarity with averaged query vectors) is a pragmatic and computationally light choice. It is inspired by InfLLM (Xiao et al., 2024a) and SnapKV (Li et al., 2024d). The novelty here is applying this to *video* KV-Caches specifically and showing it outperforms external CLIP-based retrieval.

The key question is whether head-averaged keys lose information. Multi-head attention in Transformers is theoretically structured so different heads capture different relationships (syntactic, semantic, etc.). For video, different heads may capture motion, appearance, or temporal structure differently. Averaging them into a single vector for retrieval conflates these dimensions. No ablation of head-selection strategies is provided (e.g., using only a subset of heads, or attention-head-wise retrieval).

**Assessment:** Pragmatic and effective, but under-justified theoretically. The choice is not ablated.

#### Systems Component — Memory and I/O

Table 5 reports KV-Cache offload rates: 18.8 GB/h for the 7B model at 0.5 FPS. For a 60-minute video, this is ~18.8 GB of disk writes. The paper reports disk/RAM offloading is possible, but does not report read latency from disk vs. RAM during retrieval. For very long videos (>2 hours), disk retrieval latency could dominate response time.

The 11 FPS encoding speed for LLaVA-OV-7B is reported as adequate. At 0.5 FPS sampling, the encoder is effectively running well within its capacity. But at higher sampling rates (1–2 FPS for faster-paced content), the encoder may bottleneck.

**Assessment:** System performance is well-measured under the tested conditions. Generalization to higher sampling rates and disk-heavy scenarios is unknown.

#### Information Retrieval Component

The retrieval uses block-level cosine similarity. For b=1 (default), this is frame-level retrieval. Recall at 70.5% (internal retrieval, 7B model, QAEGO4D) is promising but leaves a 30% gap to oracle.

The paper reports **recall** but not **precision**. Precision matters: retrieving 64 frames includes both relevant and irrelevant frames, and the paper's MLVU ablation shows that retrieving more frames eventually *hurts* accuracy due to "unnecessary distractions." This means the system is not precision-optimal, and there is an uncharacterized trade-off between recall and noise injection.

**Assessment:** Recall is measured; precision is not. This is a genuine gap in the evaluation of the retrieval component.

---

### 4. Bridge Assessment

The bridge between the three disciplines in this paper is largely **pragmatic and sound**, but the connections are loose in places:

**Strongest bridge:** The systems engineering and ML components are tightly integrated. KV-Cache offloading is a natural consequence of Transformer inference, and the retrieval mechanism directly reuses the same cached computations — this is elegant and principled.

**Most strained bridge:** The information retrieval component is the weakest link. The retrieval design (cosine similarity over averaged keys) is borrowed from text-domain RAG (InfLLM) and applied to video without a principled analysis of whether cosine similarity over averaged attention keys is the best similarity measure for visual content. Video frames have different statistical properties than text tokens; the suitability of this metric is assumed rather than justified.

**Unaddressed bridge:** Positional encoding across retrieved non-consecutive frames — the paper treats retrieved tokens as consecutive (standard RoPE), effectively erasing temporal ordering information between retrieved clips. For questions about event order, duration, or causality, this is a structural limitation that the paper does not address or ablate rigorously.

---

### 5. Open Questions at the Interface

These are questions that require expertise spanning multiple fields to answer and that this paper leaves open:

**Q1:** For video content with high information density (sports, news broadcasts), is 0.5 FPS sampling sufficient, or does ReKV require higher-rate sampling that would change all efficiency numbers?

**Q2:** The internal retrieval aggregates across attention layers (each layer retrieves independently). Do different layers retrieve systematically different frame types (e.g., early layers retrieving low-level visual features, later layers retrieving semantic content)? If so, does this improve or degrade holistic/multi-detail QA?

**Q3:** When the video stream is live (not pre-recorded), the encoder runs concurrently with QA. Are there race conditions or consistency guarantees when a question arrives mid-encoding of a relevant segment?

**Q4:** What is the theoretical upper bound on retrieval recall achievable with internal attention-based similarity, given the sliding-window encoding constraint (distant frames never attended to during encoding)?

---

## PAPER-SPECIFIC QUESTIONS

**Question 1**

**Q:** Is the sliding-window attention during video encoding fundamentally limiting for holistic questions (those requiring understanding of the entire video), since distant frames never interact during encoding?

**Analysis:** The paper uses a window of 15K tokens during encoding. Tokens outside this window are stored but never attend to each other during the encoding phase. The MLVU "Holistic" subtask results show internal retrieval improving over uniform sampling (+2.5%), but the paper provides no analysis of *why* holistic tasks improve if distant frames were never jointly encoded.

**Implication:** The accuracy gain on holistic tasks may be largely driven by the strong base model (LLaVA-OV-7B) rather than the retrieval mechanism itself. The encoding mechanism structurally limits cross-frame reasoning at encoding time, which should theoretically hurt holistic tasks.

---

**Question 2**

**Q:** Does the choice of 0.5 FPS sampling rate predetermine the performance of the retrieval mechanism, and does varying this rate change the conclusions?

**Analysis:** The paper fixes sampling at 0.5 FPS throughout to "align with GPT-4o's testing on MLVU." This is a reasonable standardization choice, but it means the experiments are inseparably bound to a specific sampling density. At 0.5 FPS, a 60-minute video yields 1,800 frames. At 2 FPS (more appropriate for dynamic content), this rises to 7,200 frames, dramatically changing memory and retrieval costs.

**Implication:** The efficiency numbers (FPS, GB/h, latency) are strongly dependent on the 0.5 FPS assumption. Papers citing these numbers should be aware that doubling the sampling rate could quadruple KV-Cache storage costs and retrieval overhead.

---

**Question 3**

**Q:** Why is CGBench included in Figure 1's radar chart but excluded from Table 4's main offline comparison?

**Analysis:** CGBench (27 min average video length, 12,129 QA pairs) is highlighted in the introduction as "an ideal testbed for ReKV" due to its clue-grounded design. Figure 1(b) shows ReKV outperforming baselines on CGBench visually. Yet Table 4 does not include CGBench results with numbers. The paper mentions only that experiments were conducted "with Yikun Liu" in the acknowledgments.

**Implication:** This selective reporting is a concern. Including a benchmark in a visual summary without providing it in the quantitative comparison table is unusual and potentially misleading. Full CGBench numbers should be in Table 4 for transparency.

---

**Question 4**

**Q:** The paper uses GPT-3.5-turbo-0613 as a judge for ActivityNet-QA open-ended evaluation. How stable are these scores across judge model versions?

**Analysis:** GPT-3.5-turbo-0613 is a specific, dated model version. Other papers in this space use different judge models. The 0.23-point improvement in "Score" (3.29 → 3.52) for ActivityNet-QA with ReKV is meaningful only if the judge's scoring distribution is stable across runs and conditions. No inter-rater reliability or judge calibration is reported.

**Implication:** Open-ended VideoQA scores judged by LLMs are difficult to compare across papers using different judge models or versions. The claim of +0.23 improvement should be treated with caution without repeatability data.

---

**Question 5**

**Q:** Does ReKV's internal retrieval generalize to Video-LLMs other than LLaVA-OV (the primary evaluation model)?

**Analysis:** The paper states "we conduct experiments with several other Video-LLMs" in the Appendix but does not provide details in the main paper. The main body exclusively evaluates LLaVA-OV-0.5B and LLaVA-OV-7B. The architecture of the attention mechanism (Transformer decoder, RoPE embeddings) may vary significantly across Video-LLMs, affecting whether the internal key-averaging retrieval generalizes.

**Implication:** The training-free plug-in claim is strong, but without main-paper evidence across architectures, the generalizability claim rests on appendix evidence that most readers will not scrutinize.

---

## CRITIQUE & VALIDITY ANALYSIS

### Per-Claim Assessment

**Claim 1:** "ReKV maintains stable latency and GPU memory usage as frames increase."

*Validity:* Well-supported. Figure 1(b) demonstrates stable GPU memory (~38 GB for 7B model) and latency (~3.3s) over increasing frame counts, compared to OOM errors in full-frame baselines.

*Concerns:* The stability is evaluated on a single 1-hour video (RVS-Ego) under controlled conditions. It is not tested on videos with varying complexity or at frame rates higher than 0.5 FPS. Disk I/O latency (when KV-Caches exceed RAM) is not measured.

---

**Claim 2:** "Internal retrieval outperforms external CLIP-based retrieval."

*Validity:* Well-supported by Tables 2, 3, and 5. Internal retrieval consistently achieves higher recall and accuracy than external retrieval across benchmarks and model sizes.

*Concerns:* The external retrieval uses SigLIP-SO400M. Other CLIP variants (e.g., OpenCLIP ViT-L/14) are not tested. The conclusion that internal retrieval is superior may be specific to the SigLIP choice. Additionally, external retrieval is slower (5.8s vs. 3.3s latency) partly because it requires a separate inference pass — this is a systems overhead, not a fundamental retrieval quality difference.

---

**Claim 3:** "ReKV enhances accuracy without additional training."

*Validity:* Supported across 7 benchmarks for both 0.5B and 7B models. The improvements are consistent in direction.

*Concerns:* Improvement magnitude varies widely: from +0.9% (EgoSchema, 7B) to +7.4% (QAEGO4D, 0.5B). Without statistical significance testing, the smaller improvements cannot be reliably attributed to the method. The paper provides no error bars, confidence intervals, or multiple run averages for any result.

---

**Claim 4:** "ReKV outperforms Flash-VStream on streaming benchmarks."

*Validity:* Supported by Table 5 (63.7 vs. 57.3 on RVS-Ego; 54.4 vs. 53.1 on RVS-Movie for 7B internal retrieval).

*Concerns:* Flash-VStream uses a fixed 681-token memory. The paper correctly identifies this as a structural limitation for long videos. However, the comparison is between a 681-token memory system and ReKV retrieving 64 frames (each with many tokens) — the effective "working memory" size is vastly different. This is not an apples-to-apples comparison of retrieval strategies; it is comparing a compressed-memory system to a full-cache retrieval system. The difference in accuracy may primarily reflect this capacity difference rather than algorithmic superiority.

---

**Claim 5:** "ReKV preserves complete visual information via KV-Cache storage."

*Validity:* Technically true — all KV-Caches are stored.

*Concerns:* This claim is misleading in a subtle way. The KV-Caches stored are computed under *sliding-window attention*, meaning each frame's keys and values reflect only local (15K token window) context during encoding. These are not the same KV-Caches that would result from full-attention encoding. The stored caches encode locally-contextualized representations, not globally-contextualized ones. For questions requiring long-range temporal reasoning, the locally-encoded KV-Caches may contain systematically less information than full-attention encodings would.

---

### Methodology Critique

The paper's methodology is sound at a high level but has three notable gaps:

**Gap 1 — No statistical testing.** All benchmark comparisons are reported as point estimates. For improvements of <2%, no conclusion can be drawn without confidence intervals. This is standard practice in ML evaluation papers and is conspicuously absent here.

**Gap 2 — Positional encoding not rigorously ablated.** The decision to discard original positions for retrieved tokens (treating them as consecutive) is described as better than the alternative tested (static positions from InfLLM). But no full ablation is provided comparing this to other positional schemes (e.g., keeping original absolute positions, using relative position distances). For video — a modality where temporal structure matters — this is a significant choice that deserves more thorough evaluation.

**Gap 3 — Retrieval quality incompletely evaluated.** Recall is reported; precision is not. The paper shows that adding more frames eventually hurts accuracy (MLVU plateau), implying precision degrades at higher retrieval counts. The precision-recall curve for the retrieval mechanism is never characterized. This is a gap for a paper whose core contribution is retrieval.

---

### Reproducibility Score: **6/10**

**Justification:**

*Positive:* Code and implementation details sufficient for replication are partially provided. The use of publicly available base models (LLaVA-OV) and open benchmarks is good practice. The method is training-free, which simplifies reproduction.

*Negative:* Key hyperparameters (local window size = 15K, 0.5 FPS, r=64 frames) are stated but their sensitivity is only partially ablated. Disk/RAM management specifics are not documented in sufficient detail for implementation. The appendix is referenced multiple times but not included in the reviewed version. No code repository is linked in the paper. The CGBench evaluation details are incomplete.

---

### Comparison to Prior Work

| Method | Type | Memory | Streaming | Training Required |
|---|---|---|---|---|
| Flash-VStream | Memory-bank (compressed) | 681 tokens fixed | Yes | Yes |
| VideoStreaming | Memory compression | Compressed | Yes | Yes |
| VideoLLM-Online | Frame tokenization | Single token/frame | Yes | Yes |
| LLaVA-OV (baseline) | Sparse sampling | N/A | No | N/A |
| InfLLM | KV-Cache retrieval (text) | KV-Cache | Text only | No |
| **ReKV (this paper)** | **KV-Cache retrieval (video)** | **Full KV-Cache (offloaded)** | **Yes** | **No** |

ReKV's closest prior work is InfLLM — the application of KV-Cache retrieval to video is the primary novelty. The paper's positioning as a training-free approach is its clearest differentiator. The accuracy improvements over memory-bank methods (Flash-VStream, VideoStreaming) are real but partly attributable to the much larger effective context (64 retrieved frames with full token representations vs. 681 compressed tokens). Whether the accuracy gain justifies the storage cost (18.8 GB/h at 0.5 FPS for a 7B model) is application-dependent.

---

## FINAL VERDICT

### Overall Assessment

ReKV is a **solid, practically motivated contribution** to the emerging StreamingVQA problem. The core idea — preserve all KV-Caches from video encoding and retrieve relevant slices at query time — is elegant and technically coherent. The training-free design is a genuine strength that allows immediate integration with any Transformer-based Video-LLM. The systems-level results (stable memory and latency at scale) are convincing and important.

However, the paper falls short of being a *strong* contribution due to three compounding issues: (1) the absence of statistical significance testing for benchmark results, some of which are marginal; (2) incomplete evaluation of the retrieval mechanism (no precision metric, no positional encoding ablation); and (3) the gap to oracle performance (~30%) that suggests the retrieval mechanism still has significant headroom, but no roadmap is given for closing it.

The paper is best understood as a pragmatic systems paper that opens a new formulation (StreamingVQA) and provides an initial solid baseline — not a theoretically deep contribution that advances our understanding of why retrieval works for video or when it will fail.

---

### Strengths

- **Training-free plug-in design** is genuinely flexible and allows direct comparison with an unmodified base model (LLaVA-OV) — this is methodologically clean.
- **Comprehensive benchmark coverage** across 7 diverse benchmarks (egocentric, movie, general activity, multi-detail, holistic) is thorough for an ML systems paper.
- **Internal retrieval** outperforming external CLIP retrieval is a non-obvious finding with practical implications — using the model's own attention weights for retrieval avoids the overhead of a separate encoder.
- **Decoupled encoding/QA processes** enable parallel computation on separate GPUs — this is a meaningful systems engineering contribution with deployment implications.
- **Memory management** (RAM/disk offloading with manageable KV-Cache rates) is well-characterized and practically useful for deployment.

---

### Weaknesses

- **No statistical significance testing** — improvements under 2% cannot be claimed without error bars or p-values.
- **Oracle gap** of ~30% in retrieval recall (QAEGO4D) is large and unaddressed; the method's ceiling is well below oracle performance.
- **Positional encoding ablation is incomplete** — treating retrieved non-consecutive frames as consecutive is a significant choice with implications for temporal reasoning, inadequately justified.
- **CGBench results** appear in Figure 1 but are absent from Table 4 — a consistency issue that suggests selective result reporting.
- **3.3-second response latency** is presented as efficient but is not "real-time" for robotics/surveillance applications, which are prominently cited as motivations.
- **Storage costs** (18.8 GB/h at 0.5 FPS, 7B model) scale poorly for multi-camera or higher frame-rate deployments.
- **VideoLLM-Online omitted** from streaming benchmark comparisons despite being a directly relevant baseline.

---

### Who Should Read This Paper

**Primary audience:** Researchers and engineers working on Video-LLMs, long-context inference optimization, or deployment of video understanding systems. The paper provides a clean formulation of StreamingVQA, a working implementation strategy, and comprehensive benchmark baselines that will serve as reference points for future work.

**Secondary audience:** ML systems researchers interested in KV-Cache management and retrieval for long-context inference. ReKV's approach to video is an interesting extension of InfLLM-style text KV-Cache retrieval, and the video domain adds new challenges (frame granularity, temporal structure, retrieval latency).

**Not the primary audience:** Researchers looking for theoretical analysis of retrieval quality, formal guarantees on streaming QA performance, or work on fast-paced video content understanding.

---

### Follow-Up Work Worth Reading

1. **InfLLM (Xiao et al., ICML Workshop 2024):** The direct predecessor for KV-Cache retrieval in text LLMs. Reading InfLLM provides the conceptual foundation for ReKV's retrieval mechanism.

2. **Flash-VStream (Zhang et al., 2024):** The primary competing baseline. Reading both papers together clarifies the design trade-off between fixed compressed memory (Flash-VStream) and full-cache retrieval (ReKV).

3. **VideoLLM-Online (Chen et al., CVPR 2024):** The trained streaming baseline omitted from comparison tables. Provides an upper bound for what training-based streaming models can achieve, and frames the training-free vs. trained trade-off.

4. **SnapKV (Li et al., NeurIPS 2024):** Work on using attention patterns for selective KV-Cache retention. Closely related to ReKV's internal retrieval mechanism; provides complementary analysis of which KV vectors are most informative for QA.

5. **LM-Infinite (Han et al., 2023):** The sliding-window attention framework that ReKV directly builds on for video encoding. Essential reading for understanding the positional encoding design in ReKV.

---

## REFERENCE LIST — KEY PAPERS FROM THIS WORK

**Li et al. (2024a). "LLaVA-OneVision: Easy Visual Task Transfer."** *arXiv 2408.03326.*
The primary base model used in all ReKV experiments. Understanding LLaVA-OV's architecture is prerequisite to understanding ReKV's modifications.

**Xiao et al. (2024a). "InfLLM: Training-Free Long-Context Extrapolation for LLMs with an Efficient Context Memory."** *ICML Workshop.*
The text-domain predecessor to ReKV's KV-Cache retrieval approach.

**Zhang et al. (2024a). "Flash-VStream: Memory-Based Real-Time Understanding for Long Video Streams."** *arXiv 2406.08085.*
Primary streaming baseline and source of RVS-Ego and RVS-Movie benchmark data.

**Han et al. (2023). "LM-Infinite: Simple On-the-Fly Length Generalization for Large Language Models."** *arXiv 2308.16137.*
The sliding-window attention framework underpinning ReKV's video encoding design.

**Zhou et al. (2024a). "MLVU: A Comprehensive Benchmark for Multi-Task Long Video Understanding."** *arXiv 2406.04264.*
The most comprehensive offline long-form VideoQA benchmark used in the paper.

**Di & Xie (2024). "Grounded Question-Answering in Long Egocentric Videos."** *CVPR 2024.*
Source of QAEGO4D benchmark and Oracle Retrieval ground truth annotations.

---

*Analysis prepared using the Research Paper Reader skill — Deep Critique mode. Domain: Interdisciplinary (ML/AI + Systems Engineering + Computer Vision). Paper: ICLR 2025.*
