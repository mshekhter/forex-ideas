# Freeze Family A Properly So It Is Code-Ready and Audit-Ready

## What I will do here

- freeze the live-safe conventions that Family A depends on
- freeze every Family A definition exactly
- resolve A6 and A8 fully
- state implementation notes so later audits can say Pass or Fail cleanly

# A. Global Conventions for Family A

## A.1 Bar timing convention

All features at bar `t` are computed using only completed bars with index less than or equal to `t`.

- no centered windows
- no future bars
- no global normalization
- no rank transforms using future sample information

So for any rolling window `W`, the feature uses bars:

- `t-W+1`
- `t-W+2`
- `...`
- `t`

and is defined only when `t >= W-1`.

## A.2 Frozen window set

For Family A, freeze the window set as:

- `W_set = {3, 6, 12, 24}`

No other windows belong to Family A version 1.

## A.3 Price inputs

Use mid prices for all Family A path geometry:

- `m_open[t]`
- `m_high[t]`
- `m_low[t]`
- `m_close[t]`

Use bid and ask only for spread context, not for Family A support and damage sums.

## A.4 Pip unit

- `PIP = 0.0001`

## A.5 Exact local scale definition

This must be frozen exactly.

Define mid true range in pips:

```text
tr_mid_pips[t] = max(
    (m_high[t] - m_low[t]) / PIP,
    abs(m_high[t] - m_close[t-1]) / PIP,
    abs(m_low[t] - m_close[t-1]) / PIP
)
```

for `t >= 1`.

For `t = 0`, `tr_mid_pips[0]` is undefined and all scale-dependent features are undefined.

Define rolling scale window:

- `W_scale = 24`

Define raw local scale:

```text
scale_raw[t] = median of tr_mid_pips[k] over k = t-W_scale+1 to t
```

using only available completed bars, so `scale_raw[t]` is defined only for `t >= W_scale`.

Define final scale:

```text
scale[t] = max(scale_raw[t], scale_floor_pips)
```

with

- `scale_floor_pips = 0.25`

Interpretation:

All normalized geometry is in units of recent robust bar scale, and zero or near-zero scale is prevented.

This is now frozen.

## A.6 Side convention

For side `s`:

- `dir[s] = +1` for `up`
- `dir[s] = -1` for `down`

Any side-relative quantity must be defined so that larger positive values mean more supportive for that side.

## A.7 One-bar normalized primitives

For `t >= 1`:

```text
side_ret_1[s,t] =
dir[s] * (m_close[t] - m_close[t-1]) / (PIP * scale[t])

side_body[s,t] =
dir[s] * (m_close[t] - m_open[t]) / (PIP * scale[t])

rng_norm[t] =
(m_high[t] - m_low[t]) / (PIP * scale[t])

upper_wick_norm[t] =
(m_high[t] - max(m_open[t], m_close[t])) / (PIP * scale[t])

lower_wick_norm[t] =
(min(m_open[t], m_close[t]) - m_low[t]) / (PIP * scale[t])
```

All Family A features depend on these primitives.

# B. Family A Frozen Definitions

For each side `s` and each `W in {3, 6, 12, 24}`, define the following using bars `k = t-W+1 to t`.

## B.1 Supportive flow

```text
support_flow[s,t,W] =
sum_k max(side_ret_1[s,k], 0)
```

Meaning:

- total favorable close-to-close progress for side `s` across the window

## B.2 Damage flow

```text
damage_flow[s,t,W] =
sum_k max(-side_ret_1[s,k], 0)
```

Meaning:

- total adverse close-to-close progress against side `s` across the window

## B.3 Support ratio

```text
support_ratio[s,t,W] =
support_flow[s,t,W] /
(
    support_flow[s,t,W] + damage_flow[s,t,W] + eps
)
```

with

- `eps = 1e-8`

Meaning:

- share of close-to-close directional flow that supported the side

Range:

- `0 to 1`

## B.4 Supportive body flow

```text
support_body_flow[s,t,W] =
sum_k max(side_body[s,k], 0)
```

Meaning:

- favorable body-based directional conviction for side `s`

## B.5 Damaging body flow

```text
damage_body_flow[s,t,W] =
sum_k max(-side_body[s,k], 0)
```

Meaning:

- body-level directional damage against side `s`

## B.6 Adverse rejection wick

This one must now be exact.

Principle:

Measure wick evidence that apparent directional progress for side `s` was not well accepted.

Define this bar by bar.

For side `up`:

```text
adverse_rejection_component[up,k] =
    1{m_close[k] >= m_open[k]} * upper_wick_norm[k]
    +
    1{m_close[k] < m_open[k]} * lower_wick_norm[k]
```

For side `down`:

```text
adverse_rejection_component[down,k] =
    1{m_close[k] <= m_open[k]} * lower_wick_norm[k]
    +
    1{m_close[k] > m_open[k]} * upper_wick_norm[k]
```

Then define:

```text
adverse_rejection_wick[s,t,W] =
sum_k adverse_rejection_component[s,k]
```

Interpretation:

For `up`:

- bullish bars with large upper wicks indicate upward progress that was rejected from above
- bearish bars with large lower wicks indicate failed downward extension and unstable acceptance structure around support battle

For `down`:

- bearish bars with large lower wicks indicate downward progress rejected from below
- bullish bars with large upper wicks indicate failed upward extension and unstable acceptance around resistance battle

This is a first-pass symmetric wick rejection rule. It is exact and code-ready.

## B.7 Net support balance

```text
net_support_balance[s,t,W] =
support_flow[s,t,W] - damage_flow[s,t,W]
```

Meaning:

- net close-to-close directional advantage for side `s` over the window

This is signed and unbounded above and below.

## B.8 Support efficiency

Freeze the numerator explicitly as summed side-relative net displacement across the window.

Define:

```text
net_side_disp[s,t,W] =
sum_k side_ret_1[s,k]
```

Then:

```text
support_efficiency[s,t,W] =
net_side_disp[s,t,W] /
(
    support_flow[s,t,W] + damage_flow[s,t,W] + eps
)
```

Since

```text
net_side_disp = support_flow - damage_flow
```

this is equivalently:

```text
support_efficiency[s,t,W] =
net_support_balance[s,t,W] /
(
    support_flow[s,t,W] + damage_flow[s,t,W] + eps
)
```

Range:

- `-1 to 1`

Meaning:

- whether gross directional movement resolved into one-sided progress for the hypothesis side

Interpretation:

- `+1` means all normalized close-to-close flow supported the side
- `0` means support and damage balanced out
- negative values mean net progress favored the opposite side

# C. Additional Family A Base Features That Should Be Included Now

The original Family A is workable, but two more base geometry features belong here because they complete the support-versus-damage layer without drifting into ordered shape.

## C.1 Body support ratio

```text
body_support_ratio[s,t,W] =
support_body_flow[s,t,W] /
(
    support_body_flow[s,t,W] + damage_body_flow[s,t,W] + eps
)
```

Meaning:

- share of body-based directional conviction that supported the side

## C.2 Wick rejection ratio

```text
wick_rejection_ratio[s,t,W] =
adverse_rejection_wick[s,t,W] /
(
    support_flow[s,t,W] + damage_flow[s,t,W] + eps
)
```

Meaning:

- rejection burden relative to total directional movement

This is useful because raw wick rejection can be large simply in high-activity windows.

# D. Edge Handling and Validity Rules

## D.1 Feature validity start

Because `scale[t]` requires `W_scale = 24` completed true-range values and true range itself needs `t-1` close, the earliest fully valid Family A feature is at:

- `t >= 24`

More conservatively in code:

- require all needed indices to exist explicitly

## D.2 Missing values

If any required ingredient is undefined, the Family A feature is `NaN`.

- do not forward-fill
- do not zero-fill

## D.3 Constant-price or ultra-low-range regions

Handled by:

```text
scale[t] = max(scale_raw[t], 0.25)
```

So division by zero or near-zero scale does not occur.

## D.4 No clipping

Do not clip Family A normalized values beyond what the scale floor already does.

If later modeling wants winsorization, that belongs in model preprocessing, not feature definition.

# E. Interpretation of Family A by Stage

This is still Family A, so it remains base geometry rather than developmental structure. But its meaning by stage should be explicit.

## For build

- `support_flow` rising with moderate `damage_flow` is constructive
- `body_support_ratio` rising matters more than raw `support_flow` alone
- `wick_rejection_ratio` should stay contained

## For release

- `support_flow` and `support_body_flow` should jump
- `support_efficiency` should rise sharply toward `1`
- `adverse_rejection_wick` should not jump proportionally

## For continuation

- `support_flow` should remain positive
- `damage_flow` can exist, but `support_ratio` and `support_efficiency` should remain favorable
- wick rejection should stay moderate

## For fragility

- `damage_flow` begins rising
- `support_ratio` decays
- `wick_rejection_ratio` rises even if net support remains positive

## For failure

- `damage_flow` dominates
- `net_support_balance` turns negative
- `support_efficiency` falls below `0`

# F. What Family A Does and Does Not Claim

## Family A does

- define side-relative supportive versus damaging movement
- separate close-to-close flow from body-based conviction
- add a first exact rejection-wick burden
- produce normalized, live-safe, rolling base geometry

## Family A does not

- encode ordered sequence structure inside the window
- identify build, release, continuation, fragility, or failure by itself
- represent active-episode anchoring

Those belong to later families.

# G. Frozen Audit Contract for Family A

A Family A implementation is `Pass` only if all of the following are true:

- `scale[t]` uses exactly 24-bar rolling median of mid true range in pips, floored at `0.25` pips
- every feature at `t` uses only completed bars `<= t`
- `W_set` is exactly `{3, 6, 12, 24}`
- side-relative sign convention is exactly: positive means supportive for the chosen side
- A1, A2, A4, A5 use sums of positive and negative parts exactly as defined
- A6 uses the exact bar-conditional wick logic frozen above
- A8 uses summed side-relative net displacement divided by total directional flow
- undefined early rows remain `NaN`