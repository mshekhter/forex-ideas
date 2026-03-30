## Bar-path geometry as a proxy for latent pressure sign

### Concept

Without order flow, you attempt to infer whether recent bars reflect directed pressure versus mean-reverting churn.

This is not causal identification in the strict do-calculus sense. It is proxy inference within an SCM where latent pressure causes both path shape and future returns.

I am still dancing around the subject with zigzags, etc. This is worth looking into further. Structure detection methods - i got it. Sign/direction needs more work.

If I am to publish code, it is probably early failures, dead ends.

### Prerequisites

- Clean, gap-handled bar series
- Stable log-return computation
- Consistent handling of weekend gaps

### Assumptions

1. Certain path-shape statistics, such as efficiency, zigzag density, and run-length distribution, correlate with latent pressure and are stable enough to exploit.

2. Conditioning on volatility and range reduces the major confounding from volatility clustering, as supported by the realized-volatility literature.

### Step-by-step application

1. Compute multi-scale path geometry features:
   - $E_t(k)$
   - $Z_t(k)$
   - run-length summaries
   - CLV
   - window-position metrics

2. Define a continuous pressure proxy $X_t$, for example:

$$
X_t = \operatorname{sign}\left(\sum_{i=t-k+1}^{t} r_i\right) \cdot E_t(k) \cdot \left(1 - Z_t(k)\right)
$$

This is:

`direction × efficiency × persistence`

3. Estimate:

$$
\mathbb{E}[Y_{t+h} \mid X_t, S_t]
$$

with controls $S_t$ capturing:

- volatility, such as RS, GK, and RR
- range
- barrier distances

Use DML or orthogonalization if you treat $X_t$ as a treatment-like exposure.

4. At runtime, direction is the sign of the predicted conditional mean return, with thresholding by cost proxy and uncertainty.

### Expected signal strength

Low to medium in general.

It may be higher at short horizons during clean directional phases and lower in highly efficient markets where residual drift is tiny.

The limitation is structural. If order flow is the real driver, proxies will only partially recover it.

### Computational cost

- Low live
- Moderate offline if you estimate nonlinear models across many $k$ values

### Live suitability

Moderate to high as part of an ensemble with barrier features and volatility normalization.

It is weaker as a standalone sign engine because of instability across time.

### Failure modes

- Geometry overfitting

  Path metrics can fit noise. Use PBO, deflated Sharpe, and reality checks.