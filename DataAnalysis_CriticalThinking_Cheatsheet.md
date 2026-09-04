# Aptitude Cheatsheet Part 2 — Data Analysis, Analytical Ability & Critical Thinking

---

# SECTION A: DATA ANALYSIS (Data Interpretation)

Tables, bar graphs, pie charts, line graphs — the skill tested is fast, accurate reading + calculation, not new math concepts. Speed comes from technique, not formulas.

## Core Techniques

### Reading the data correctly (do this FIRST, always)
- Before touching any question, spend 20-30 seconds scanning: what are the axes/columns? What units (₹, %, thousands, lakhs)? Is there a "total" row/column already given?
- **Common trap**: units mismatch — a bar graph in "units of 1000" means a bar at "45" represents 45,000, not 45. Missing this wrecks every calculation that follows.

### Pie Charts
- A pie chart's percentages must sum to 100% (or degrees sum to 360°)
- **Formula**: Value of a segment = (Segment's % / 100) × Total, OR (Segment's degrees / 360) × Total
- **Shortcut**: convert all segments to % first if given in degrees (° / 3.6 = %), then work in percentages — much faster mental math than degree fractions

### Bar Graphs
- Usually compares a quantity across categories (years, products, regions)
- **Common question types**: find the year/category with max value, find % change between two bars, find average across all bars
- **Shortcut for "find the average"**: eyeball-estimate first (sum roughly, divide by count) to sanity-check your calculated answer — catches arithmetic slips fast

### Line Graphs
- Usually shows a trend over time — watch for **rate of change** questions (steepest segment = fastest growth/decline), not just endpoint values
- **Shortcut**: "percentage increase between two points" uses the standard % change formula from the Quant sheet — DA doesn't need new formulas, just correctly reading the two values off the graph

### Tables (often the densest, most calculation-heavy)
- **Shortcut**: if a question asks about a ratio or comparison between two rows/columns, you often DON'T need to calculate each value fully — set up the ratio directly and simplify before computing (avoids unnecessary full calculations)

## Speed Rules for DA Specifically
- **Approximate first, calculate second**: for multi-step DA questions, round numbers to estimate which answer choice is in the right ballpark before doing exact math — this catches wrong-option traps fast in MCQ format
- **Never recalculate a value you can read directly** — if the table already shows the total, don't re-sum the column
- **Watch for "except" and "not" in DA question wording** — a classic trap is asking "which category does NOT satisfy X," and rushing candidates answer the opposite

## Practice Set

**1. (Easy) Pie Chart**
A company's expenses are divided as: Salaries 40%, Rent 20%, Marketing 15%, Utilities 10%, Others 15%. If total expenses are ₹50,00,000, find the amount spent on Marketing.
**Answer: ₹7,50,000** | 15% of 50,00,000 = 7,50,000

**2. (Medium) Bar Graph (described)**
A bar graph shows sales (in units of 1000) for 5 years: 2021:45, 2022:60, 2023:55, 2024:70, 2025:80. Find the % increase in sales from 2023 to 2024.
**Answer: ~27.27%** | Increase = 70-55=15. % increase = 15/55×100 = 27.27%

**3. (Hard) Table-based ratio**
A table shows Product A sold 1,200 units at ₹500 each, and Product B sold 800 units at ₹750 each. Find the ratio of total revenue from A to total revenue from B.
**Answer: 1:1** | Revenue A = 1200×500=6,00,000. Revenue B=800×750=6,00,000. Ratio=1:1 (set up as ratio directly: (1200×500):(800×750) = 6,00,000:6,00,000, simplify without needing decimal precision)

---

# SECTION B: ANALYTICAL ABILITY & PROBLEM SOLVING

This covers pattern recognition, logical sequences, puzzles, and structured reasoning — the goal is spotting the underlying rule fast.

## Core Techniques

### Number/Letter Series
- **Check these patterns in order** when you see a series: (1) simple arithmetic (+2, +3...), (2) simple multiplication/ratio (×2, ×3...), (3) alternating pattern (two interleaved series), (4) differences-of-differences (second-level pattern), (5) squares/cubes/primes embedded in the sequence
- **Shortcut**: write out the differences between consecutive terms first — this reveals arithmetic and difference-of-difference patterns immediately

### Coding-Decoding
- **Letter-shift codes**: identify the shift amount by comparing one letter pair (e.g., if CAT → DBU, each letter shifted +1)
- **Shortcut**: use the alphabet position number (A=1, B=2...Z=26) mentally rather than counting on fingers — faster for larger shifts

### Blood Relations
- **Shortcut**: draw a quick family tree diagram (even 3-4 symbols) rather than tracking relationships purely in your head — this is the single biggest speed/accuracy improvement for this question type
- Use "+" for male, "-" for female, arrows for parent→child, "=" for marriage, to keep the diagram fast to draw

### Seating Arrangement
- **Shortcut**: draw the seating positions first (circle or row) as empty slots, then place the MOST CONSTRAINED clue first (e.g., "X sits at one end") before less specific clues — placing constrained information first eliminates the most possibilities fastest
- For circular arrangements, remember "left" and "right" are relative to which direction the arrangement faces — misreading this is the most common error type

### Syllogisms
- **Draw Venn diagrams** — don't try to reason through "All A are B, No B is C" purely verbally; a 2-3 circle Venn diagram makes valid conclusions visually obvious
- **Key rule**: a valid conclusion must follow in ALL possible diagram configurations that satisfy the premises, not just one — if you can draw even one valid configuration where the conclusion fails, it's not a guaranteed conclusion

## Practice Set

**1. (Easy) Number Series**
Find the next number: 3, 6, 11, 18, 27, ?
**Answer: 38** | Differences: 3,5,7,9,11 (increasing by 2 each time) → next difference=11 → 27+11=38

**2. (Medium) Blood Relations**
Pointing to a photograph, Rahul said, "She is the daughter of my grandfather's only son." How is the woman related to Rahul?
**Answer: Sister** | Grandfather's only son = Rahul's father (since it's the ONLY son). Father's daughter = Rahul's sister.

**3. (Hard) Syllogism**
Premises: "All pens are pencils. Some pencils are erasers." Conclusion: "Some pens are erasers." Is this conclusion valid?
**Answer: Invalid (cannot be determined)** | Draw the Venn diagram: pens is fully inside pencils, but "some pencils are erasers" could refer to a part of the pencil circle that doesn't overlap with the pens sub-circle at all — so it's NOT guaranteed that any pen is an eraser.

---

# SECTION C: CRITICAL THINKING

Tests argument evaluation, assumptions, and inference — less about calculation, more about precise reading and logical rigor.

## Core Techniques

### Assumption Questions
- An assumption is something the argument's conclusion **depends on being true**, but which is NOT explicitly stated
- **The negation test (fastest technique)**: negate the proposed assumption — if the argument's conclusion falls apart when the assumption is false, it IS a necessary assumption. If the conclusion still holds even when negated, it's NOT a necessary assumption.

### Inference Questions
- An inference must follow **necessarily** from the given statements — it cannot introduce new information or rely on outside "common sense" beyond what's stated
- **Common trap**: an answer choice that sounds "reasonable" or "probably true in real life" but isn't strictly, logically guaranteed by the passage — these are designed to tempt you away from the strictly correct answer

### Strengthen/Weaken Arguments
- To weaken: find the answer that most directly attacks the *link* between the argument's evidence and its conclusion (not just anything vaguely negative)
- To strengthen: find the answer that most directly reinforces that same link
- **Shortcut**: identify the argument's core logical gap first (what unstated leap does the argument make?) — the correct strengthen/weaken answer almost always targets that exact gap, not peripheral details

### Cause-and-Effect Reasoning
- **Common flaw to watch for**: correlation presented as causation — "X and Y happened together, therefore X caused Y" ignores the possibility of (a) reverse causation, (b) a third factor causing both, or (c) simple coincidence
- Critical thinking questions often ask you to identify exactly this flaw, or to pick the answer that best exposes an alternative explanation

## Practice Set

**1. (Easy) Assumption**
Argument: "Since the new policy reduced traffic accidents by 30%, the government should implement it nationwide." What is the argument assuming?
**Answer: That the 30% reduction wasn't due to some other factor unique to the original location.** | Using the negation test: if some OTHER factor (like a temporary road closure) actually caused the reduction, extending the policy nationwide wouldn't replicate the result — so the argument depends on that not being the case.

**2. (Medium) Inference**
Statement: "All employees who completed the training scored above 80% on the assessment. Priya scored 85% on the assessment." Can we infer that Priya completed the training?
**Answer: No, cannot be inferred.** | The statement only tells us training→score above 80%, not the reverse. Priya could have scored 85% without completing the training — this is the classic "affirming the consequent" logical trap.

**3. (Hard) Weaken the argument**
Argument: "Ice cream sales and drowning incidents both rise in summer months. Therefore, buying ice cream causes an increased risk of drowning." Which statement most weakens this argument?
**Answer: A statement establishing a common third cause — e.g., "Both ice cream sales and swimming (hence drowning risk) increase due to hot weather, with no direct link between the two."** | This directly targets the argument's causal-vs-correlation flaw by supplying the real common cause (heat → both more swimming and more ice cream), which is a stronger weaken than simply saying "correlation isn't causation" in the abstract.

---

## Cross-Section Speed Notes

- **DA**: read-scan-estimate-calculate, in that order — never skip the estimate step, it's your error-catching mechanism under time pressure.
- **Analytical/Puzzles**: always externalize the problem (diagram, table, written-out differences) rather than holding it all mentally — this is the single highest-leverage habit across seating arrangements, blood relations, and syllogisms.
- **Critical Thinking**: the negation test for assumptions and the "does this NECESSARILY follow" filter for inference are your two most reusable tools — apply them by default, don't just go with gut feeling on tone/plausibility.
