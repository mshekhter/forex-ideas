## CATE sign estimation with DML and causal forests on bar-derived treatments

### Concept

This is the meta-method that turns the earlier feature mechanisms into a defensible entry-time sign decision.

Define treatments as structural events or continuous exposures computed from bars, such as:

- barrier breach
- rejection
- high-efficiency move
- similar bar-derived state events

Estimate the conditional effect:

$$
\tau_X(s) = \mathbb{E}[Y_{t+h} \mid do(X_t = 1), S = s] - \mathbb{E}[Y_{t+h} \mid do(X_t = 0), S = s]
$$

or the analogous effect for continuous \(X_t\).

In practice, this remains observational identification through back-door adjustment, so results must be interpreted cautiously.

Still, worth a try!!! and I got prerequisite data! Well, after I deal with the feature sets (in Olser we trust at the moment).

### Why DML and causal forests

DML provides orthogonalization and cross-fitting to reduce bias when flexible machine learning is used for nuisance functions.

Causal forests estimate heterogeneous treatment effects nonparametrically, with asymptotic inference under unconfoundedness and regularity conditions.

### Prerequisites

- Large sample sizes, which multiyear intraday data can support
- Strict timestamping
- Label construction with no overlap leakage

### Assumptions

These are the core causal bets:

1. Unconfoundedness given bar-derived controls:

$$
Y(x) \perp X \mid S
$$

This is strong and likely imperfect. The more the treatment can be viewed as a mechanical event after conditioning, such as a barrier-distance state, the more credible it becomes.

2. Overlap or positivity:

Both treatment states must occur with nontrivial probability across the relevant regions of the state space.

3. SUTVA-like small-trader assumption:

Your trading does not alter the historical price path.

### Step-by-step application

1. Choose horizon \(h\) and define the label, for example:

$$
Y_{t+h} = \ln\left(\frac{C_{t+h}}{C_t}\right)
$$

or use a net-of-cost proxy.

2. Define treatments for the chosen method, such as:

- barrier breach event, binary
- rejection event, binary
- continuous efficiency proxy thresholded into a high-efficiency treatment

3. Define controls \(S_t\), strictly from bars, including:

- volatility measures such as RS, GK, and RR
- semivariance
- path geometry
- barrier distances at other scales
- CLV

4. Fit the CATE model.

   For binary-treatment DML:

   - estimate the propensity model \(e(S)\)
   - estimate outcome regressions \(m_0(S)\) and \(m_1(S)\)
   - use an orthogonal score with cross-fitting

   For causal forests or generalized forests:

   - estimate \(\hat{\tau}(S_t)\)
   - use it directly as the conditional sign signal

5. At runtime:

   - compute \(S_t\)
   - compute the relevant \(X_t\)
   - set trade direction to the sign of \(\hat{\tau}(S_t)\)

Optionally moderate the decision using uncertainty and estimated transaction costs.

### Expected signal strength

Potentially medium when the treatment is a barrier or cascade state with strong structural grounding.

Lower when the treatment is only a generic geometry statistic.

The main benefit of CATE is that it avoids a global sign assumption and instead learns where the sign is stable and where it flips.

### Computational cost

- Moderate to high offline
- Low live

### Live suitability

High if you restrict the system to a few stable treatment definitions and enforce strong validation discipline.

Otherwise the overfitting risk is high.

### Failure modes

- Assumption-heavy observational identification
- Overlap failure for rare treatment states
- Leakage from overlapping labels if purging and embargo are not enforced
- Instability when too many treatment definitions are searched