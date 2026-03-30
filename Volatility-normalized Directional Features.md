## Volatility-normalized features and objective technical levels

### Concept

Many directional features become unstable because they are not scale-normalized.

Range-based volatility estimators, such as Parkinson, Garman-Klass, Rogers-Satchell, Yang-Zhang, and realized-range approaches, provide a principled way to normalize bar-path and barrier distances by a local volatility proxy.

Objective technical levels, such as rolling highs, rolling lows, and channels, are simply barrier constructs with clear formulas, avoiding discretionary pattern labels.

This looks prmising to experiment with, although coding the statistical concepts are new and look like a "deep Kimchi" category for me, a heavier lift.

### Prerequisites

- Ability to compute local volatility windows
- Ability to compute rolling maxima and minima for multiple scales

### Assumptions

1. Local volatility is estimable from OHLC with acceptable bias. OHLC estimators are derived under stylized diffusion assumptions, so robustness checks are required.

2. Normalization reduces nonstationarity in feature distributions.

### Step-by-step application

1. Choose a volatility proxy $\sigma_t(k)$.

   Common choices:
   - Rogers-Satchell
   - Garman-Klass
   - Realized range for additional robustness to microstructure effects

2. Normalize all distance-like features by $\sigma_t(k)$, including:
   - barrier distance
   - net displacement
   - excursion magnitudes

3. Use normalized breakouts, for example:

$$
\mathrm{breakout}_t(k) = 1\left[\frac{C_t - R_t(k)}{\sigma_t(k) + \varepsilon} > \theta\right]
$$

and similarly for downside breakouts.

4. Learn the sign effect of these objective levels with back-door controls.

### Expected signal strength

Medium only when combined with barrier logic or path-geometry logic.

Normalization is primarily a robustness tool rather than a standalone directional engine.

### Computational cost

Low.

### Live suitability

High.

### Failure modes

- Volatility-estimator mismatch to bar frequency can distort normalization.

  A proxy that behaves reasonably at 1-minute bars may behave differently at 10-minute bars.

- Sensitivity must always be reported across estimators, especially:
  - Parkinson
  - Rogers-Satchell
  - Garman-Klass