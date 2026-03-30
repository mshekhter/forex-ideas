## Excursion/continuation diagnostics from recent-window extremes

### Concept

Use the recent window’s internal excursion structure to estimate whether current direction is accepted (continuation) or rejected (reversal).

This is designed to be computable at entry time from the last \(k\) bars only, and it can be tied to barrier mechanics:

- rejection patterns near barriers are consistent with clustered take-profit and limit behavior
- acceleration after breach is consistent with stop-loss cascades

### Prerequisites

- A consistent definition of the window
- Stable ordering of bars
- Selection of \(k\) and \(h\) such that you have enough samples

### Assumptions

1. The within-window extreme geometry contains information about the latent pressure balance that persists over \(h\).

2. Normalizing by volatility or range makes the features more transportable across time.

### Step-by-step application

1. For each \(k\), compute:
   - window maximum price
   - window minimum price
   - end position \(p_t\)
   - the position-in-range feature \(A_t(k)\) from the formulas table

2. Construct asymmetry features, such as:

   Directional excursion ratio:

   $$
   DER_t(k) = \frac{p_t - \min_{i=t-k+1}^{t} p_i}{\max_{i=t-k+1}^{t} p_i - \min_{i=t-k+1}^{t} p_i + \varepsilon}
   $$

   Bullish if near 1.

   Excursion versus net displacement:

   $$
   \frac{|p_t - p_{t-k+1}|}{\max_{i=t-k+1}^{t} p_i - \min_{i=t-k+1}^{t} p_i + \varepsilon}
   $$

   High values indicate a more directed move.

3. Combine these with barrier proximity, including:
   - distance to support
   - distance to resistance
   - distance to round numbers

4. Learn the sign mapping using either:
   - a transparent rule set
   - a CATE model where treatment is "high DER in vicinity of barrier" versus not

### Expected signal strength

Medium in regimes where barrier and order clustering dominate. Otherwise low to medium.

The key point is that this method is synergistic with barrier logic, which has FX-specific empirical grounding.

### Computational cost

Low to moderate.

### Live suitability

High. All inputs are OHLC and rolling extrema.

### Failure modes

- Small changes in \(k\) can flip results, so sensitivity analysis across window scales is required.