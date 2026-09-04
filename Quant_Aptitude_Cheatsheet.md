# Quantitative Aptitude Cheatsheet — Formulas, Tricks & Practice

---

## 1. PERCENTAGE

### Formulas
- % change = (Change / Original) × 100
- New value after x% increase = Original × (1 + x/100)
- New value after x% decrease = Original × (1 - x/100)
- If A is x% more than B → B is [x/(100+x)]×100 % less than A
- If A is x% less than B → B is [x/(100-x)]×100 % more than A

### Shortcut Tricks
- **Fraction-percentage table (memorize)**: 1/2=50%, 1/3=33.33%, 1/4=25%, 1/5=20%, 1/6=16.67%, 1/8=12.5%, 1/10=10%, 1/12=8.33%, 1/16=6.25%, 1/20=5%
- **Successive % change**: two successive changes of a% and b% = a + b + (ab/100) — use negative values for decreases
- **x% of y = y% of x** — pick whichever is easier to compute mentally (e.g., 8% of 50 = 50% of 8 = 4)

### Practice Set
1. (Easy) A number increased by 20% becomes 180. Find the original number.
   **Answer: 150** | 180 = x × 1.2 → x = 150

2. (Medium) In an election, a candidate got 65% of votes and won by 4,200 votes. Find total votes.
   **Answer: 14,000** | Winning margin = 65% - 35% = 30% of total = 4,200 → total = 4,200/0.30 = 14,000

3. (Hard) The price of an item is increased by 25%, then decreased by 20%. Find net % change.
   **Answer: 0% (no change)** | Net = 25 - 20 + (25×-20)/100 = 25-20-5 = 0

---

## 2. PROFIT & LOSS

### Formulas
- Profit = SP - CP | Loss = CP - SP
- Profit% = (Profit/CP) × 100 | Loss% = (Loss/CP) × 100
- SP = CP × (100 + Profit%)/100
- SP = CP × (100 - Loss%)/100
- CP = SP × 100/(100 + Profit%) or SP × 100/(100 - Loss%)
- Discount% is always calculated on Marked Price (MP), not CP
- SP = MP × (100 - Discount%)/100

### Shortcut Tricks
- **Successive discounts**: two discounts of a% and b% = a + b - (ab/100) net discount (treat as negative % change combo)
- **False weight trick**: if a shopkeeper uses false weight of x grams instead of 1000g while claiming cost price profit%, actual profit% = [(1000-x)/x] × 100, when selling at CP
- **Equal SP, one profit one loss trick**: if two items sold at same SP, one at x% profit, other at x% loss → overall there's always a LOSS of (x/10)² %

### Practice Set
1. (Easy) CP = ₹800, SP = ₹960. Find profit%.
   **Answer: 20%** | Profit = 160, Profit% = 160/800×100 = 20%

2. (Medium) A shopkeeper marks goods 40% above CP and gives 15% discount. Find profit%.
   **Answer: 19%** | SP = CP×1.40×0.85 = CP×1.19 → 19% profit

3. (Hard) Two articles sold at ₹990 each, one at 10% profit, other at 10% loss. Find overall profit/loss.
   **Answer: Loss of 1%** | Using trick: loss% = (10/10)² = 1%. Overall CP = 990/1.1 + 990/0.9 = 900+1100=2000, SP=1980, Loss=20, Loss%=20/2000×100=1%

---

## 3. SIMPLE & COMPOUND INTEREST

### Formulas
- **Simple Interest**: SI = (P × R × T)/100
- **Amount (SI)**: A = P + SI = P(1 + RT/100)
- **Compound Interest**: A = P(1 + R/100)^T
- CI = A - P
- **CI compounded half-yearly**: A = P(1 + (R/2)/100)^(2T)
- **CI compounded quarterly**: A = P(1 + (R/4)/100)^(4T)
- **Difference between CI and SI for 2 years**: CI - SI = P(R/100)²
- **Difference between CI and SI for 3 years**: CI - SI = P(R/100)² × [(300+R)/100]

### Shortcut Tricks
- **Doubling time (Rule of 72)**: Years to double money ≈ 72/R (compound interest, approx)
- **For 2-year CI vs SI difference**: memorize CI-SI = P×(R/100)² directly — huge time-saver, avoids full CI calc
- **CI for 2 years, direct formula**: CI = P[(2R/100) + (R/100)²] — faster than computing A first

### Practice Set
1. (Easy) Find SI on ₹5,000 at 8% p.a. for 3 years.
   **Answer: ₹1,200** | SI = (5000×8×3)/100 = 1200

2. (Medium) Find CI on ₹10,000 at 10% p.a. for 2 years.
   **Answer: ₹2,100** | CI = 10000×[(20/100)+(10/100)²] = 10000×[0.20+0.01] = 2100

3. (Hard) The difference between CI and SI on a sum for 2 years at 5% is ₹25. Find the sum.
   **Answer: ₹10,000** | 25 = P×(5/100)² = P×0.0025 → P = 25/0.0025 = 10,000

---

## 4. TIME & WORK

### Formulas
- If A does work in x days, A's 1-day work = 1/x
- Combined work rate = sum of individual 1-day work rates
- Time to complete together = 1/(sum of individual rates)
- **Work = Rate × Time** (same structure as Speed×Time=Distance)
- If A is k times as efficient as B, and B takes x days → A takes x/k days

### Shortcut Tricks
- **LCM method (fastest for OA)**: assume total work = LCM of all given days (avoids fractions entirely). E.g., A takes 10 days, B takes 15 days → total work = LCM(10,15)=30 units. A's rate=3/day, B's rate=2/day.
- **A+B together, then A leaves**: compute combined work done first, then remaining work ÷ remaining worker's rate
- **Wages division trick**: wages are divided in the ratio of work done (= ratio of 1-day work rates, i.e., inverse ratio of days taken alone)

### Practice Set
1. (Easy) A can do a work in 12 days, B in 18 days. In how many days can they finish together?
   **Answer: 7.2 days** | LCM=36. A's rate=3, B's rate=2, combined=5/day → 36/5=7.2 days

2. (Medium) A and B together can complete a work in 10 days. A alone can do it in 15 days. In how many days can B alone do it?
   **Answer: 30 days** | LCM=30. Combined rate=3/day. A's rate=2/day. B's rate=3-2=1/day → 30/1=30 days

3. (Hard) A can do a work in 20 days. B is 25% more efficient than A. Find B's time alone.
   **Answer: 16 days** | B's rate = A's rate × 1.25. A's rate=1/20. B's rate=1.25/20=1/16 → B takes 16 days

---

## 5. TIME, SPEED & DISTANCE

### Formulas
- Speed = Distance/Time | Distance = Speed×Time | Time = Distance/Speed
- **km/hr to m/s**: multiply by 5/18
- **m/s to km/hr**: multiply by 18/5
- **Average speed** (equal distances, two speeds a & b) = 2ab/(a+b) — NOT simple average
- **Relative speed (same direction)** = difference of speeds
- **Relative speed (opposite direction)** = sum of speeds
- **Train crossing a pole/point**: time = length of train / speed
- **Train crossing a platform**: time = (length of train + length of platform) / speed
- **Boats & streams**: Downstream speed = boat speed + stream speed; Upstream speed = boat speed - stream speed
- Boat speed in still water = (Downstream + Upstream)/2 | Stream speed = (Downstream - Upstream)/2

### Shortcut Tricks
- **5/18 and 18/5 conversion — memorize instantly**, this appears in nearly every speed problem
- **Average speed trap**: if a question gives two speeds for equal distances, NEVER simple-average — always use 2ab/(a+b)
- **Meeting point trick (opposite direction, same start time)**: time to meet = total distance / sum of speeds

### Practice Set
1. (Easy) Convert 72 km/hr to m/s.
   **Answer: 20 m/s** | 72 × 5/18 = 20

2. (Medium) A train 150m long crosses a pole in 15 seconds. Find its speed in km/hr.
   **Answer: 36 km/hr** | Speed = 150/15 = 10 m/s = 10×18/5 = 36 km/hr

3. (Hard) A man rows 30 km downstream in 3 hours and returns upstream in 5 hours. Find the speed of the boat in still water and the stream.
   **Answer: Boat = 8 km/hr, Stream = 2 km/hr** | Downstream speed=30/3=10, Upstream speed=30/5=6. Boat=(10+6)/2=8, Stream=(10-6)/2=2

---

## 6. RATIO & PROPORTION

### Formulas
- a:b = a/b | If a:b = c:d, then ad = bc (cross-multiplication / "product of extremes = product of means")
- **Dividing a quantity in ratio a:b:c**: parts are [a/(a+b+c)]×Total, [b/(a+b+c)]×Total, [c/(a+b+c)]×Total
- **Compounded ratio**: (a:b) and (c:d) compounded = ac:bd
- **Duplicate ratio** of a:b = a²:b²
- If A:B = x:y and B:C = p:q, then A:B:C = xp : yp : yq (make B's value common)

### Shortcut Tricks
- **Combining two ratios (fastest method)**: to combine A:B=x:y and B:C=p:q into A:B:C, multiply first ratio by p and second ratio by y (making B equal in both) → A:B:C = xp:yp:yq
- **Mixture/Alligation shortcut**: for mixing two quantities at different rates to get a mean rate — (Cheaper qty)/(Dearer qty) = (Dearer rate - Mean rate)/(Mean rate - Cheaper rate)

### Practice Set
1. (Easy) Divide ₹1,500 in the ratio 2:3.
   **Answer: ₹600 and ₹900** | Total parts=5. 2/5×1500=600, 3/5×1500=900

2. (Medium) If A:B = 3:4 and B:C = 8:9, find A:B:C.
   **Answer: 6:8:9** | Multiply A:B by 8, B:C by 4 → A:B=24:32, B:C=32:36 → A:B:C=24:32:36 = 6:8:9 (divide by 4)

3. (Hard) In what ratio must rice at ₹40/kg be mixed with rice at ₹60/kg to get a mixture worth ₹52/kg?
   **Answer: 2:3** | Alligation: (60-52)/(52-40) = 8/12 = 2:3 (cheaper:dearer)

---

## 7. NUMBER SYSTEMS

### Formulas
- Sum of first n natural numbers = n(n+1)/2
- Sum of squares of first n natural numbers = n(n+1)(2n+1)/6
- Sum of cubes of first n natural numbers = [n(n+1)/2]²
- Sum of first n odd numbers = n²
- Sum of first n even numbers = n(n+1)
- **Divisibility rules**: by 3 → digit sum divisible by 3; by 4 → last 2 digits divisible by 4; by 9 → digit sum divisible by 9; by 11 → (sum of digits at odd places) - (sum at even places) divisible by 11 (including 0)
- HCF × LCM = Product of two numbers (only valid for exactly two numbers)

### Shortcut Tricks
- **Finding last digit of a^b**: last digits of powers cycle in patterns of length 4 (for most digits) — find b mod 4 (careful: if remainder is 0, use 4th position, not 0th), then match to the cyclicity pattern of that digit
- **HCF via Euclid's algorithm (fastest)**: repeatedly replace (larger, smaller) with (smaller, larger mod smaller) until remainder is 0 — the last non-zero divisor is the HCF
- **LCM shortcut once HCF is known**: LCM = (Product of numbers)/HCF — avoids separately factoring for LCM

### Practice Set
1. (Easy) Find the sum of the first 20 natural numbers.
   **Answer: 210** | 20×21/2 = 210

2. (Medium) Find the HCF and LCM of 24 and 36.
   **Answer: HCF=12, LCM=72** | Euclid: 36=24×1+12, 24=12×2+0 → HCF=12. LCM=(24×36)/12=72

3. (Hard) Find the last digit of 7^45.
   **Answer: 7** | Cycle of 7: 7,9,3,1 (length 4). 45 mod 4 = 1 → 1st position in cycle = 7

---

## Speed Notes for OA/Timed Conditions

- **Percentage & Ratio problems**: almost always solvable faster via the fraction-equivalence table than via direct % formulas — memorize section 1's table cold.
- **Time & Work**: the LCM method eliminates fraction arithmetic entirely — always default to it over rate-equation setups.
- **Speed problems**: the 5/18 and 18/5 conversions should be instant, zero-hesitation recall — these appear in nearly every question in this category.
- **When stuck on a hard question**: check if it's a disguised version of a simpler pattern (e.g., "two successive % changes" and "two successive discounts" use the *identical* formula) — recognizing the underlying pattern is often faster than deriving from scratch.
