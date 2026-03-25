# Episode Frame

At any completed 5 minute bar `t`, the model should evaluate the recent path under directional hypotheses, not as a generic bar snapshot.

For each side `s in {up, down}`, define an episode hypothesis `H_s(t)`:

- `H_up(t)`: the recent path may be organizing into or sustaining an upward directional episode
- `H_down(t)`: the recent path may be organizing into or sustaining a downward directional episode

A feature is only useful if it says something precise about one of these questions:

- Is a side building?
- Is it releasing?
- Is it continuing?
- Is it becoming fragile?
- Is it failing?

That means features must be side-relative and stage-aware.

# Base Notation

Let bars be indexed by `t`.

Use mid prices for path geometry:

- `m_open[t]`
- `m_high[t]`
- `m_low[t]`
- `m_close[t]`

Use bid and ask for execution and spread context:

- `b_close[t]`
- `a_close[t]`
- `spread[t] = a_close[t] - b_close[t]`

Define pip unit:

- `PIP = 0.0001`

Define a local scale over a rolling window `W_scale`, for example 24 bars:

- `scale[t] = robust local bar scale in pips`

This can be built from median true range or another robust local range estimator. The exact choice matters less than consistency.

All geometric features should be normalized by `scale[t]` or a closely related side-safe local scale.

Define normalized close-to-close return:

- `r1[t] = (m_close[t] - m_close[t-1]) / scale[t]`

Define normalized bar body:

- `body[t] = (m_close[t] - m_open[t]) / scale[t]`

Define normalized range:

- `rng[t] = (m_high[t] - m_low[t]) / scale[t]`

Define wick measures:

- `upper_wick[t] = (m_high[t] - max(m_open[t], m_close[t])) / scale[t]`
- `lower_wick[t] = (min(m_open[t], m_close[t]) - m_low[t]) / scale[t]`

Define close location within bar:

- `clv[t] = 0 if range is zero, else ((m_close[t] - m_low[t]) / (m_high[t] - m_low[t]))`

This is raw geometry only. It is not yet episode logic.

# Directional Hypothesis Frame

For each side, define the same path in side-relative coordinates.

For upward hypothesis:

- `dir = +1`

For downward hypothesis:

- `dir = -1`

Define signed side-relative quantities:

- `side_ret_1[s,t] = dir * (m_close[t] - m_close[t-1]) / scale[t]`
- `side_body[s,t] = dir * (m_close[t] - m_open[t]) / scale[t]`

For the upward side, positive values mean supportive movement.

For the downward side, the sign is flipped so positive values still mean supportive movement for that side.

This is essential. Otherwise you just have signed arithmetic, not side-relative structure.

Every episode feature below should be defined in side-relative coordinates.

# Feature Families

The feature system should have nine families:

- A. support and damage geometry
- B. ordered path-shape geometry
- C. build features
- D. release features
- E. continuation features
- F. fragility features
- G. failure and takeover features
- H. compression and readiness features
- I. context and execution feasibility features

These are not interchangeable. They answer different episode questions.

# Family A: Support and Damage Geometry

## Purpose

Measure how much the recent path has helped or harmed a side.

This is not generic momentum. It is side-relative support versus damage.

For a rolling window `W in {3, 6, 12, 24}` define:

### A1. Supportive Flow

- `support_flow[s,t,W] = sum over k=t-W+1 to t of max(side_ret_1[s,k], 0)`

Meaning:

- Total favorable close-to-close progress for side `s`.

### A2. Damage Flow

- `damage_flow[s,t,W] = sum over k=t-W+1 to t of max(-side_ret_1[s,k], 0)`

Meaning:

- Total adverse close-to-close movement against side `s`.

### A3. Support Ratio

- `support_ratio[s,t,W] = support_flow / (support_flow + damage_flow + eps)`

Meaning:

- Whether the path has mostly built in the side's favor or not.

### A4. Supportive Body Flow

- `support_body_flow[s,t,W] = sum max(side_body[s,k], 0)`

Meaning:

- Directional conviction expressed in bar bodies, not just closes.

### A5. Damaging Body Flow

- `damage_body_flow[s,t,W] = sum max(-side_body[s,k], 0)`

Meaning:

- Body-level damage against the hypothesis.

### A6. Wick Rejection Against Side

For upward hypothesis, upper rejections are less important than lower rejection failures. For downward, mirror.

Define:

- `adverse_rejection_wick[s,t,W] = sum over window of wick mass that rejects the side's advance`

For up:

- `adverse_rejection_wick[up,t,W] = sum upper_wick on bullish-supportive bars plus lower_wick on bearish bars that pierce support badly`

This needs precise coding, but the principle is:

- capture bar-level evidence that apparent progress was not well accepted.

### A7. Net Support Balance

- `net_support_balance[s,t,W] = support_flow - damage_flow`

### A8. Support Efficiency

- `support_efficiency[s,t,W] = net side displacement over W / (support_flow + damage_flow + eps)`

Meaning:

- Whether gross movement actually resolved into one-sided progress.

These are the base side-relative geometry measures.

# Family B: Ordered Path-Shape Geometry

## Purpose

Encode the arrangement of movement inside the recent window. This is where bag-of-aggregates must be avoided.

Use the last `W` bars, typically `W = 12` or `24`.

Construct side-relative cumulative path:

- `cp[s,t,j]` for `j = 0..W`
- `cp at 0 = 0`
- each next point adds `side_ret_1`

From this side-relative path define:

### B1. Terminal Displacement

- `terminal_disp[s,t,W] = cp[s,t,W]`

### B2. Cumulative Absolute Path Length

- `path_length[s,t,W] = sum absolute side_ret_1 across the window`

### B3. Directional Efficiency

- `dir_eff[s,t,W] = terminal_disp / (path_length + eps)`

Meaning:

- Clean one-way progression versus noisy wandering.

### B4. First Half Displacement

- `disp_first_half[s,t,W]`

### B5. Second Half Displacement

- `disp_second_half[s,t,W]`

### B6. Back-Loaded Versus Front-Loaded Progress

- `progress_tilt[s,t,W] = disp_second_half - disp_first_half`

Meaning:

- Whether the move is accelerating late or was mostly early and then stalled.

### B7. Ordered Support Profile

Split the window into quartiles. Compute supportive minus damaging flow in each quartile.

- `support_profile_q1`
- `support_profile_q2`
- `support_profile_q3`
- `support_profile_q4`

Meaning:

- The sequence of pressure matters. Build followed by collapse is different from steady strengthening.

### B8. Pullback Depth from Running Favorable Peak

Within the side-relative cumulative path, compute the largest drawdown from the running maximum.

- `max_pullback_from_peak[s,t,W]`

For downward hypothesis, since coordinates are already side-relative, this works symmetrically.

Meaning:

- Damage to an in-progress side after favorable progress was achieved.

### B9. Recovery After Pullback

After the maximum pullback point, how much of the prior favorable progress was recovered before `t`?

- `pullback_recovery_ratio[s,t,W]`

Meaning:

- Whether damage was absorbed or remained unresolved.

### B10. Monotone Segment Concentration

Fraction of total supportive flow contributed by the single strongest contiguous supportive segment.

Meaning:

- Whether progress came in one coherent push or many fragments.

### B11. Interruption Count

Count the number of supportive runs interrupted by damaging bars above a minimum magnitude.

Meaning:

- A side that requires constant restarting is weaker than one clean sequence.

### B12. Sign-Change Density

Count sign changes in `side_ret_1` above a minimum absolute threshold.

Meaning:

- Alternation intensity.

### B13. Local Curvature

Use second differences of the side-relative cumulative path.

- `curvature_mean`
- `curvature_std`
- `curvature_last`

Meaning:

- Whether the path is smoothly bending, abruptly jerking, or flattening.

### B14. Path Convexity Toward Side

Fit a low-order polynomial or use a three-segment approximation on `cp`. Estimate whether the path is accelerating in the side's favor or decelerating.

This family preserves ordering inside the window.

# Family C: Build Features

## Purpose

Measure whether a side is organizing before release.

Build is not just positive drift. It is the structured accumulation of directional control without full expansion yet.

These features should be computed before any explicit release trigger.

### C1. Low-Amplitude Support Accumulation

- `build_support_low_amp[s,t,W] = supportive flow occurring while per-bar range is below the local median range`

Meaning:

- Quiet directional organization.

### C2. Damage Containment During Build

- `build_damage_containment[s,t,W] = 1 - damage_flow during low-range bars / total low-range movement`

Meaning:

- Whether the side is gaining without repeated adverse intrusion.

### C3. Build Persistence

- Fraction of the last `W` bars where `side_ret_1` is positive and above a small threshold.

### C4. Build Clustering

- Longest contiguous run of supportive bars in the last `W` bars.

### C5. Build Staircase Score

Count patterns like:

- supportive bar
- small pause
- supportive bar
- small pause
- supportive bar

with higher lows for up or lower highs for down in side-relative terms.

This should be explicitly pattern-based, not hand-wavy.

### C6. Micro Pullback Acceptability

- Average damage-bar magnitude divided by average supportive-bar magnitude within the build.

Small controlled pullbacks are acceptable. Large abrupt pullbacks damage build quality.

### C7. Build Compression Coexistence

A strong build is often inside partial compression.

Define:

- `build_in_compression[s,t,W] = support_ratio high while average range percentile remains low`

### C8. Latent Directional Imbalance

Use side-relative close location and body dominance to detect that bars increasingly finish in the supportive portion of their ranges without full release yet.

### C9. Structural Distance to Recent Opposite Extreme

For upward hypothesis:

- distance from current close to recent `W`-bar low

For downward:

- distance to recent `W`-bar high

Normalized by scale.

Meaning:

- Has the side already established separation from the opposite anchor?

# Family D: Release Features

## Purpose

Measure whether build has transitioned into decisive expansion.

Release is not just a big bar. It is build resolving into directional expansion.

### D1. Expansion Shock

- `release_expansion[s,t] = current side-supportive body or range divided by median prior compressed range`

Meaning:

- How decisively the path has expanded from prior local conditions.

### D2. First Decisive Break from Compressed Envelope

Compute a rolling local envelope over prior bars. Measure penetration beyond the envelope in side direction.

### D3. Release Acceptance

After a large supportive bar, measure whether close location, wick structure, and next-bar follow-through indicate acceptance rather than rejection.

### D4. Release Cleanliness

- `release_cleanliness[s,t,H] = supportive excursion over the next H bars divided by supportive plus damaging excursion`

with `H` small, like `2` or `3` bars for research labeling, not live feature generation.

For live features, replace this with immediate bar-close proxies:

- current expansion bar close near favorable extreme
- limited adverse wick
- low overlap with prior bar

### D5. Release Displacement Relative to Prior Build

- `release_disp_vs_build[s,t,W] = current supportive displacement / cumulative supportive build over prior W bars`

Meaning:

- Whether release is meaningful relative to what was being built.

### D6. Multi-Bar Release Coherence

Across the last `2` to `3` bars:

- is there a sequence of expansion, hold, and second push
- rather than a single isolated burst

### D7. Release Asymmetry

- supportive expansion magnitude relative to simultaneous adverse wick and adverse overlap

### D8. Release Location Within Recent Path

Did release occur after prolonged compression, after steady build, or after already extended movement?

This matters because late release after full extension is weaker.

# Family E: Continuation Features

## Purpose

Measure whether a released move remains usable.

Continuation is not the same as release. It is the ability to extend while absorbing damage.

### E1. Post-Release Support Maintenance

- Supportive flow over the last `W_c` bars after a release marker.

### E2. Continuation Damage Burden

- Damage flow over the same post-release window.

### E3. Continuation Retention Ratio

- `retention[s,t,W_c] = current side-relative displacement since release / best side-relative displacement since release`

Meaning:

- How much of the achieved move has been retained.

### E4. Continuation Impulse Spacing

- Average number of bars between supportive impulse bars after release.

Meaning:

- Healthy continuation often shows renewed pushes before damage accumulates too much.

### E5. Continuation Overlap Burden

- Fraction of post-release bars that significantly overlap the previous bar.

Too much overlap suggests digestion or stall.

### E6. Continuation Pullback Depth

- Largest side-relative drawdown since release from best achieved point.

### E7. Continuation Recovery Speed

After the largest post-release pullback, number of bars required to recover a specified fraction.

### E8. Continuation Channel Quality

Approximate the post-release path with a local channel. Measure slope, residual variance, and adverse excursions below the lower continuation band for up, reversed for down.

### E9. Continuation Breath

Count how many post-release bars actually contribute new favorable ground versus just oscillate inside achieved territory.

# Family F: Fragility Features

## Purpose

Detect that a move still exists but has become vulnerable.

Fragility is not failure. It is the degradation stage before failure.

### F1. Deterioration Rate

- Change in support ratio over recent short windows.

If support ratio is falling while price still sits near favorable territory, fragility is rising.

### F2. Stalled Extension

- Number of bars since the last meaningful new favorable excursion.

### F3. Damage-to-New-Ground Ratio

In the last `W_f` bars:

- damage flow divided by new favorable ground achieved

Meaning:

- Is the move paying too much adverse cost for too little extension?

### F4. Repeated Failed Pushes

- Count attempts to make new ground that were immediately rejected by next-bar damage.

### F5. Late Overlap Density

- Overlap fraction rises late in the move.

### F6. Wick-Based Rejection Against the Side

A move that still closes favorably but prints repeated adverse rejection wicks is fragile.

### F7. Retention Erosion Speed

- Rate of decline of retention ratio over the last few bars.

### F8. Directional Efficiency Collapse

- `dir_eff` measured over short trailing windows after a continuation period.

Collapse from high efficiency to low efficiency is a core fragility signal.

### F9. Opposite-Side Interruption Quality

Not just that interruptions exist, but whether interruptions are becoming larger, more body-driven, and less recoverable.

# Family G: Failure and Takeover Features

## Purpose

Measure whether one side has lost control and the opposite side is taking over.

Failure is episode-relative. It depends on prior control.

These features must only be computed meaningfully when a side had prior build, release, or continuation.

For side `s`:

### G1. Failure from Achieved Favorable Peak

- `failure_drawdown[s,t] = drawdown from best side-relative cumulative level achieved during the current episode`

### G2. Support Collapse

- Recent support ratio falls below a threshold after having been high.

### G3. Opposite-Side Impulse Intrusion

- Count and size of opposite-side impulse bars entering the recent path.

### G4. Unrecovered Damage

- Cumulative damage since last favorable peak that has not been retraced.

### G5. Takeover Speed

- Bars required for opposite-side supportive flow to exceed current-side supportive flow after fragility starts.

### G6. Opposite-Side Acceptance

- Opposite-side impulses now close well and hold rather than being faded immediately.

### G7. Failed Recapture Attempts

- Current side attempts to regain control but prints shallow, quickly rejected supportive bars.

### G8. Regime Inversion Marker

A compact feature that indicates:

- support for one side has become support for the other

In practice this means:

- opposite-side flow, efficiency, close quality, and extension are all simultaneously improving while the prior side deteriorates

# Family H: Compression and Readiness Features

## Purpose

Measure whether the path is in a low-energy state from which a move can emerge.

Compression must be structured, not just low volatility.

### H1. Range Compression Percentile

- Current average range relative to the last `N` bars.

### H2. Body Compression Percentile

- Current average body relative to the last `N` bars.

### H3. Overlap Compression

- Fraction of bars whose ranges substantially overlap prior ranges.

### H4. Path Cancellation

- Low terminal displacement relative to high path length.

Meaning:

- Lots of movement but little net progress.

### H5. Alternating Micro-Control

- High sign-change density in a low-range environment.

### H6. Envelope Tightening

- Rolling max-high minus rolling min-low over shorter and shorter trailing windows.

### H7. Compression with Latent Side Build

Compression is more interesting when one side is quietly accumulating. So pair `H` features with build features.

### H8. Readiness Asymmetry

Compression is not neutral if close quality, body quality, or minor support flow is increasingly one-sided.

Readiness asymmetry detects which side is more likely to own the break.

# Family I: Context and Execution Feasibility

## Purpose

The episode model still needs context and realism.

### I1. Spread Burden

- spread relative to scale

### I2. Spread Percentile in Recent Context

### I3. Session State

- Asia
- London
- overlap
- New York late
- rollover

### I4. Time Since Session Transition

### I5. Local Liquidity Proxy

Can be approximated from spread stability and range behavior if no tick data is used.

### I6. Day-of-Week Block

### I7. Recent Scale Regime

- low
- medium
- high realized scale regime

These are not episode features, but they constrain whether an episode is executable.

# Ordered-Shape Features Using Landmark Segmentation

To avoid losing sequence arrangement, add a landmark representation inside the recent window.

For the last 24 bars, construct the side-relative cumulative path and extract 5 landmarks:

- `L0 = start`
- `L1 = first local extreme of material size`
- `L2 = next local opposite extreme`
- `L3 = best favorable level`
- `L4 = current level`

From these define:

- first leg size
- first pullback size
- second leg size
- ratio of second leg to first
- pullback fraction of first leg
- current retention versus best favorable level
- bars spent in each segment

This provides an explicit mini-episode geometry inside the window.

It is much closer to path representation than rolling means.

# Episode-Relative Anchoring

Some features only make sense relative to an active or hypothesized episode anchor.

For each side, define:

- `episode_start_candidate[s,t]`
- `episode_peak[s,t]`
- `episode_last_push[s,t]`

Then compute anchor-relative features:

- displacement since episode start
- supportive flow since episode start
- damage since episode start
- retention from episode peak
- bars since last new favorable extreme
- bars since last supportive impulse
- damage since last push
- recovery since last damage burst

This is how you stop confusing raw opposite flow with actual episode damage.

# Stage-Aware Feature Subsets

Not every feature should be active for every stage.

## For Build Assessment, Emphasize

- low-amplitude support accumulation
- support ratio
- build persistence
- build clustering
- compression coexistence
- readiness asymmetry

## For Release Assessment, Emphasize

- expansion shock
- envelope penetration
- release acceptance
- multi-bar coherence
- release asymmetry

## For Continuation Assessment, Emphasize

- retention ratio
- continuation damage burden
- pullback depth
- recovery speed
- continuation overlap burden
- new-ground frequency

## For Fragility Assessment, Emphasize

- deterioration rate
- stalled extension
- failed push count
- retention erosion
- opposite interruption quality

## For Failure Assessment, Emphasize

- failure drawdown
- support collapse
- opposite impulse intrusion
- takeover speed
- failed recapture attempts

That is what stage-aware means in practice.

# Sequence-Preserving Model Inputs

The strongest implementation is not to flatten everything immediately.

For the last 24 bars, keep a per-bar tensor with these columns:

## Per-Bar Side-Relative Channels

- `side_ret_1`
- `side_body`
- supportive contribution
- damaging contribution
- range
- upper wick
- lower wick
- close location
- overlap with prior bar
- bar makes new favorable ground flag
- bar makes damaging intrusion flag

Then add cumulative channels across the sequence:

- running side-relative cumulative path
- running best favorable level
- running drawdown from peak
- running support ratio
- running interruption count

This gives the model ordered geometry directly.

Then the aggregate features described earlier become auxiliary summary channels, not the only representation.

# Strict First-Pass Feature Table

If freezing a first implementation, use this exact structure.

## Per-Bar Sequence Channels Over Last 24 Bars

- 18 columns per side, plus shared context

### Shared Per-Bar

- normalized range
- spread burden
- overlap fraction with previous bar
- session encoding

### Per-Side Per-Bar

- `side_ret_1`
- `side_body`
- supportive piece
- damaging piece
- side-supportive wick quality
- side-adverse wick quality
- new favorable ground flag
- damaging intrusion flag
- running cumulative side path
- running peak
- running drawdown
- running support ratio

## Window Summaries Over 3, 6, 12, 24 Bars

- support_flow
- damage_flow
- support_ratio
- support_efficiency
- interruption_count
- sign_change_density
- max_pullback_from_peak
- pullback_recovery_ratio
- progress_tilt
- build_persistence
- build_clustering
- overlap_density
- readiness_asymmetry
- stalled_extension
- failure_drawdown

## Landmark Geometry Over Last 24 Bars

- first leg size
- first pullback size
- second leg size
- pullback fraction
- best favorable level
- current retention
- bars in build segment
- bars in pullback segment
- bars in renewed push segment

# What This Design Prevents

This design prevents the common drift described earlier because:

- it does not replace path representation with rolling summaries
- it does define build, release, continuation, fragility, and failure separately
- it does preserve order inside the window
- it does make damage relative to a side under hypothesis
- it does compute features relative to directional control
- it does require precise meanings for push, interruption, pullback, compression, persistence, and failure

# Final Design Principle

Every feature must be testable against one sentence:

> "If this feature rises, what exact change in episode condition does it mean?"

If that sentence is vague, the feature is not ready.

A good feature has one clear meaning, such as:

- upward supportive flow increased
- upward move retained less of its achieved ground
- downward interruption intensity rose inside an upward continuation
- compression became more asymmetric toward upward release
- upward release was accepted rather than rejected
- upward episode entered fragility without full failure yet

That is the level of precision required.