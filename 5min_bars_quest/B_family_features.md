# Family B: Ordered Path-Shape Geometry

This family exists to do what Family A does not do.

Family A measures directional contribution totals, or at least tries to:

- how much support
- how much damage
- how much body conviction
- how much rejection burden

Family B measures arrangement:

- how that support and damage were ordered inside the recent window

That is the whole point of Family B.

Two windows can have identical Family A totals and completely different Family B structure.

## Example

### Window 1

- steady constructive progression
- one controlled pullback
- renewed push

### Window 2

- violent alternation
- late recovery
- same final displacement

Family A can look similar.

Family B must separate them cleanly.

So we will freeze Family B as the first true path-representation family.

# Purpose of Family B

For each side hypothesis `s` and time `t`, Family B should answer:

- Was recent progress orderly or chaotic?
- Was progress early, late, or evenly distributed?
- Did the side build ground and retain it?
- Were pullbacks shallow or destructive?
- Did the path advance in one coherent push or fragmented bursts?
- Did the path accelerate, stall, or decay inside the window?

This is still side-relative, but now explicitly ordered.

# Family B Global Conventions

## 2.1 Inputs

Use the frozen Family A primitives and conventions.

At each completed bar `t`:

- `scale[t]` is exactly the frozen 24-bar rolling median of mid true range in pips, floored at `0.25`
- `side_ret_1[s,t]` is exactly the frozen normalized side-relative close-to-close return
- all features use only completed bars less than or equal to `t`

## 2.2 Window set

Freeze the Family B window set as:

- `W_shape = {6, 12, 24}`

Not `3`.

Reason:

Family B is about ordered internal shape. A 3-bar window is too short to support most path-shape geometry robustly.

## 2.3 Side-relative cumulative path

For each side `s`, time `t`, and window `W in {6,12,24}`, define the side-relative incremental sequence:

```text
x_j[s,t,W] = side_ret_1[s, t-W+j]
for j = 1, 2, ..., W
```

So the first increment is the oldest bar in the window and the last increment is the current completed bar.

Define cumulative path points:

```text
cp_0[s,t,W] = 0
```

```text
cp_j[s,t,W] = sum from i=1 to j of x_i[s,t,W]
for j = 1, 2, ..., W
```

This cumulative path is the canonical Family B object.

Everything in Family B is derived from `x_j` and `cp_j`.

## 2.4 Validity

A Family B feature for window `W` is defined only when all bars in that window are valid and scale is valid for all those bars.

If not, return `NaN`.

- no forward-fill
- no zero-fill

# Family B Core Objects

Before listing features, freeze these helper objects.

## 3.1 Terminal displacement

```text
terminal_disp[s,t,W] = cp_W[s,t,W]
```

Meaning:

- net side-relative displacement across the window

## 3.2 Path length

```text
path_length[s,t,W] = sum from j=1 to W of abs(x_j[s,t,W])
```

Meaning:

- total signed movement turned into total traveled distance

## 3.3 Running peak

```text
run_peak_j[s,t,W] = max of cp_0 through cp_j
```

## 3.4 Running trough

```text
run_trough_j[s,t,W] = min of cp_0 through cp_j
```

## 3.5 Drawdown from running peak

```text
dd_from_peak_j[s,t,W] = run_peak_j[s,t,W] - cp_j[s,t,W]
```

## 3.6 Drawup from running trough

```text
du_from_trough_j[s,t,W] = cp_j[s,t,W] - run_trough_j[s,t,W]
```

These are side-relative, so they work symmetrically for up and down.

# Family B Frozen Features

## B1. Terminal displacement

```text
terminal_disp[s,t,W] = cp_W[s,t,W]
```

Meaning:

- net resolved progress for side `s` across the window

This is included in Family B even though it is simple, because the later ordered features are interpreted relative to it.

## B2. Path length

```text
path_length[s,t,W] = sum_j abs(x_j[s,t,W])
```

Meaning:

- gross traveled path, regardless of direction

## B3. Directional efficiency

```text
dir_eff[s,t,W] =
terminal_disp[s,t,W] / (path_length[s,t,W] + eps)
```

with `eps = 1e-8`

Range:

- `[-1, 1]`

Meaning:

- how efficiently the path translated gross movement into net side-relative progress

Interpretation:

- `1` means perfectly one-sided supportive progression
- `0` means no net resolution despite movement
- negative means the opposite side dominated overall

## B4. First-half displacement

For even `W` only, which holds for `6`, `12`, `24`:

```text
disp_first_half[s,t,W] = cp_{W/2}[s,t,W]
```

Meaning:

- side-relative progress achieved by the midpoint of the window

## B5. Second-half displacement contribution

```text
disp_second_half[s,t,W] =
terminal_disp[s,t,W] - disp_first_half[s,t,W]
```

Meaning:

- net progress added in the second half of the window

## B6. Progress tilt

```text
progress_tilt[s,t,W] =
disp_second_half[s,t,W] - disp_first_half[s,t,W]
```

Meaning:

- whether progress was back-loaded or front-loaded

Interpretation:

- positive means the path strengthened late
- negative means the path made early progress and then stalled or gave back ground

## B7. Quartile support profile

Split the `W`-bar window into four equal blocks.

- For `W = 24`, each quartile has `6` bars
- For `W = 12`, each quartile has `3` bars
- For `W = 6`, quartile splitting is too coarse, so do not define quartile profile for `W = 6`

Freeze this rule:

Quartile profile features exist only for `W in {12, 24}`.

For each quartile `q in {1,2,3,4}`, define:

```text
quart_support_q[s,t,W,q] =
sum over bars in quartile q of max(x_j[s,t,W], 0)
```

```text
quart_damage_q[s,t,W,q] =
sum over bars in quartile q of max(-x_j[s,t,W], 0)
```

```text
quart_balance_q[s,t,W,q] =
quart_support_q - quart_damage_q
```

Meaning:

- where inside the window the side actually built or lost control

## B8. Peak progress time index

```text
peak_idx[s,t,W] =
smallest j in {0,...,W} such that cp_j is maximal over the window
```

Normalize time location:

```text
peak_idx_norm[s,t,W] = peak_idx[s,t,W] / W
```

Meaning:

- when the best favorable level inside the window was first achieved

Interpretation:

- late peaks suggest recent strengthening
- early peaks followed by deterioration suggest stall or fragility

## B9. Trough time index

```text
trough_idx[s,t,W] =
smallest j such that cp_j is minimal over the window
```

```text
trough_idx_norm[s,t,W] = trough_idx[s,t,W] / W
```

Meaning:

- when the worst unfavorable point occurred inside the window

Useful together with peak timing.

## B10. Maximum pullback from running favorable peak

```text
max_pullback_from_peak[s,t,W] =
max over j=0..W of dd_from_peak_j[s,t,W]
```

Meaning:

- largest damage suffered after some favorable progress had already been achieved

This is one of the most important Family B features.

## B11. Final pullback from window peak

```text
final_pullback_from_peak[s,t,W] =
max over j of cp_j[s,t,W] - cp_W[s,t,W]
```

Equivalent:

- window peak minus current terminal level

Meaning:

- how much achieved favorable ground had been given back by the end of the window

Interpretation:

- small final pullback means good retention
- large final pullback means current state sits well below the best level reached

## B12. Pullback retention ratio

Let:

```text
peak_level[s,t,W] = max over j of cp_j[s,t,W]
```

Define:

```text
pullback_retention_ratio[s,t,W] =
cp_W[s,t,W] / (peak_level[s,t,W] + eps_pos)
```

where

- `eps_pos = 1e-8`

Important restriction:

This feature is only meaningful when `peak_level > 0`. If `peak_level <= 0`, set `NaN`.

Meaning:

- fraction of best favorable ground still retained at the end of the window

Range:

- can be negative if the path fully reversed after a positive peak, which is acceptable and informative

## B13. Maximum run-up from running trough

```text
max_runup_from_trough[s,t,W] =
max over j=0..W of du_from_trough_j[s,t,W]
```

Meaning:

- largest recovery or favorable drive from the worst point inside the window

This complements B10.

## B14. Monotone segment concentration

Define supportive runs on `x_j` as maximal contiguous sequences where `x_j > 0`.

For each supportive run `r`, define `run_support[r]` as the sum of `x_j` over the run.

Then define:

```text
monotone_segment_concentration[s,t,W] =
max_r run_support[r] / (sum_r run_support[r] + eps)
```

If there are no supportive runs, return `0`.

Meaning:

- how much of all supportive progress came from a single coherent push

Interpretation:

- high values mean one dominant impulse
- lower values mean fragmented supportive progress

## B15. Supportive run count

```text
supportive_run_count[s,t,W] =
number of maximal contiguous sequences with x_j > 0
```

Meaning:

- how many times the side had to restart supportive movement

## B16. Damaging interruption count

Define a damaging interruption as a maximal contiguous run where `x_j < 0` and the total absolute damaging sum across that run is at least `theta_interrupt`.

Freeze:

- `theta_interrupt = 0.25`

in normalized scale units.

Then:

```text
damaging_interruption_count[s,t,W] =
number of damaging runs meeting that threshold
```

Meaning:

- number of material interruptions against the side

This is better than raw sign-change count because it ignores tiny noise flickers.

## B17. Sign-change density

Define `sign_j = sign(x_j)`, where:

- `+1` if `x_j > theta_sign`
- `-1` if `x_j < -theta_sign`
- `0` otherwise

Freeze:

- `theta_sign = 0.05`

Then count sign changes between adjacent nonzero signs only.

```text
sign_change_density[s,t,W] =
(number of sign changes among filtered signs) / max(number of valid sign comparisons, 1)
```

Range:

- `[0,1]`

Meaning:

- alternation intensity after ignoring tiny near-zero bars

## B18. Mean local curvature

Define first differences on cumulative path as `x_j`.

Define second differences:

```text
curv_j[s,t,W] = x_j[s,t,W] - x_{j-1}[s,t,W]
for j = 2,...,W
```

Then:

```text
curvature_mean[s,t,W] =
mean of curv_j
```

```text
curvature_std[s,t,W] =
standard deviation of curv_j
```

```text
curvature_last[s,t,W] =
curv_W[s,t,W]
```

Meaning:

- how abruptly the path is accelerating, decelerating, or jerking between successive bars

These are coarse but precise.

## B19. Late-window slope versus early-window slope

Fit ordinary least squares slope of `cp_j` against `j` on:

- first half of the window
- second half of the window

Define:

- `slope_first_half[s,t,W]`
- `slope_second_half[s,t,W]`

Then:

```text
slope_tilt[s,t,W] =
slope_second_half - slope_first_half
```

Meaning:

- whether the path is accelerating into the present or decaying into the present

This is more robust than raw half displacement when the internal path is noisy.

## B20. New-ground frequency

Define a bar `j` as making new favorable ground if:

```text
cp_j[s,t,W] > max(cp_0, ..., cp_{j-1})
```

Then:

```text
new_ground_count[s,t,W] =
number of such j
```

```text
new_ground_frequency[s,t,W] =
new_ground_count / W
```

Meaning:

- how often the side actually extended the favorable frontier

A path can have positive terminal displacement with very low new-ground frequency if most movement was wasted in pullbacks.

## B21. Stalled-tail length

Define:

```text
last_peak_idx[s,t,W] =
largest j such that cp_j is maximal over the window
```

Then:

```text
stalled_tail_length[s,t,W] =
W - last_peak_idx[s,t,W]
```

Meaning:

- how many bars have passed since the most recent best favorable level

Interpretation:

- large values indicate the path has not extended recently

## B22. Recovery after deepest pullback

Let `j_star` be the first index where `dd_from_peak_j` attains `max_pullback_from_peak`.

Define post-pullback recovery as:

```text
recovery_after_deepest_pullback[s,t,W] =
cp_W[s,t,W] - cp_{j_star}[s,t,W]
```

Optionally normalize by `max_pullback_from_peak`:

```text
recovery_after_deepest_pullback_ratio[s,t,W] =
recovery_after_deepest_pullback / (max_pullback_from_peak + eps)
```

Meaning:

- whether the path absorbed the deepest damage and repaired it before the end of the window

# Family B Implementation Restrictions

To keep coding and audits clean, freeze the following.

## 5.1 No smoothing of x_j or cp_j

Use raw `side_ret_1` increments exactly as defined.

## 5.2 No pivot extraction inside Family B

Family B uses cumulative-path geometry directly. Swing-point or landmark compression belongs later if we decide to add a subfamily.

## 5.3 No event anchor dependence

Family B is ordered path geometry over a fixed trailing window only. It is not yet anchored to an active episode start. That anchor-relative layer comes later.

## 5.4 Thresholds are frozen

Use exactly:

- `theta_interrupt = 0.25`
- `theta_sign = 0.05`

These are in normalized scale units.

# Family B Interpretation by Stage

This family is still not stage classification by itself, but its features have stage meanings.

## For build

- `dir_eff` should be positive
- `monotone_segment_concentration` often moderate to high
- `damaging_interruption_count` should stay low to moderate
- `progress_tilt` often positive if build is strengthening into the present
- `new_ground_frequency` should be steady, not necessarily explosive

## For release

- `slope_tilt` should turn sharply positive
- `new_ground_frequency` should rise
- `peak_idx_norm` should move late in the window
- `terminal_disp` should expand quickly
- `max_pullback_from_peak` should stay contained initially

## For continuation

- `pullback_retention_ratio` should stay high
- `stalled_tail_length` should stay low
- `new_ground_frequency` can moderate but should remain healthy
- `damaging_interruption_count` can exist, but `recovery_after_deepest_pullback` should remain positive and meaningful

## For fragility

- `final_pullback_from_peak` rises
- `pullback_retention_ratio` decays
- `stalled_tail_length` grows
- `peak_idx_norm` shifts earlier than current time
- `damaging_interruption_count` rises
- `slope_tilt` weakens or turns negative

## For failure

- `terminal_disp` collapses or turns negative
- `dir_eff` falls sharply
- `max_pullback_from_peak` becomes large relative to peak progress
- `recovery_after_deepest_pullback` becomes weak
- `new_ground_frequency` becomes very low

# Family B Audit Contract

A Family B implementation is `Pass` only if all of the following are true:

- it uses the frozen Family A scale and side-relative primitives exactly
- it uses only completed bars less than or equal to `t`
- `W_shape` is exactly `{6, 12, 24}`
- it constructs the ordered side-relative increment sequence `x_j` from oldest to newest bar in the window
- it constructs cumulative path `cp_j` exactly as the cumulative sum of `x_j` with `cp_0 = 0`
- B10, B11, B12 are computed from cumulative-path peaks and pullbacks, not from raw closes directly
- B14 and B16 are based on maximal contiguous runs, not arbitrary sign counts
- B17 uses the frozen `theta_sign = 0.05` rule
- B18 uses second differences of the ordered side-relative increment sequence
- undefined rows remain `NaN`

# Conceptual Caution

Family B is the first place where feature explosion can happen.

The purpose is to capture a few distinct path facts:

- net resolution
- efficiency
- time placement of progress
- pullback severity
- fragmentation
- alternation
- recent extension versus stall
- recovery after damage

That is enough for a first frozen version.

# Recommended First Coding Subset

For a practical first-pass Family B before expanding to all 22, freeze this subset as the required core:

- B1 `terminal_disp`
- B2 `path_length`
- B3 `dir_eff`
- B6 `progress_tilt`
- B10 `max_pullback_from_peak`
- B11 `final_pullback_from_peak`
- B12 `pullback_retention_ratio`
- B14 `monotone_segment_concentration`
- B16 `damaging_interruption_count`
- B17 `sign_change_density`
- B19 `slope_tilt`
- B20 `new_ground_frequency`
- B21 `stalled_tail_length`
- B22 `recovery_after_deepest_pullback_ratio`

That subset already captures most of Family B’s intended geometry cleanly.

# B18 Revision Note

`B18` is weak.

So instead of:

- `curvature_mean`
- `curvature_std`
- `curvature_last`

use:

- `slope_1`
- `slope_2`
- `slope_3`
- `bend_early`
- `bend_late`
- `bend_total`

The leanest version:

- `bend_late`
- `bend_total`

Those are much stronger than raw second differences.