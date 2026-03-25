# Family B Addon

# Purpose of the Addon

The original reduced Family B subset captured core path shape:

- net resolution
- pullback
- fragmentation
- alternation
- stall
- recovery

What it still does not capture cleanly enough for Family C is:

- where support versus damage sits inside the window
- whether the path is strengthening into the present
- whether the cumulative path bends toward late release or toward stall

That is why Build Family C started drifting. Those are ordered-path objects and should live in B, not be improvised inside C.

So this addon adds only the missing ordered-path objects that Build needs.

# Family B Addon Scope

Freeze only these additions:

- `B23 half-window support balance`
- `B24 quartile support balance`
- `B25 fixed-third cumulative-path slopes`
- `B26 bend metrics from third-slopes`
- `B27 late-path dominance ratio`

That is enough.

# Family B Addon Conventions

Use all previously frozen Family A and Family B conventions unchanged:

- completed bars only, indices `<= t`
- frozen `scale[t]`
- frozen `side_ret_1[s,t]`
- side-relative ordered increments
- cumulative path `cp_j` from oldest to newest within the window

Addon window rules:

- for `B23`: `W in {6, 12, 24}`
- for `B24`: `W in {12, 24}` only
- for `B25` and `B26`: `W in {6, 12, 24}`
- for `B27`: `W in {6, 12, 24}`

No other window choices belong to this addon.

# Supporting Object: Window Increment Sequence

For direction `s`, time `t`, and window `W`, reuse:

```text
x_j[s,t,W] = side_ret_1[s, t-W+j]
for j = 1,...,W
```

with oldest bar at `j = 1` and current bar at `j = W`.

# B23 Half-Window Support Balance

## Purpose

Measure whether directional balance improves in the second half of the window relative to the first half.

## Definition

Split the `W`-bar window into two equal halves.

First half:

- `j = 1,...,W/2`

Second half:

- `j = W/2 + 1,...,W`

Define for each half `h`:

```text
half_support[h] =
sum of max(x_j, 0) over bars in that half
```

```text
half_damage[h] =
sum of max(-x_j, 0) over bars in that half
```

```text
half_balance[h] =
half_support[h] - half_damage[h]
```

Then define:

```text
half_balance_first[s,t,W] = half_balance[first]
half_balance_second[s,t,W] = half_balance[second]
```

```text
half_progressive_balance[s,t,W] =
half_balance_second[s,t,W] - half_balance_first[s,t,W]
```

## Meaning

Whether the direction-relative net balance is strengthening into the present.

## Interpretation

- positive: more favorable balance late than early
- negative: better balance early, weaker late

This is the cleanest first progression object.

# B24 Quartile Support Balance

## Purpose

Provide a finer-grained ordered balance profile inside the window.

## Definition

This exists only for `W in {12, 24}`.

For `W = 12`:

- quartiles are 4 blocks of 3 bars each

For `W = 24`:

- quartiles are 4 blocks of 6 bars each

For quartile `q in {1,2,3,4}`, define:

```text
quart_support_q[s,t,W,q] =
sum of max(x_j, 0) over bars in quartile q
```

```text
quart_damage_q[s,t,W,q] =
sum of max(-x_j, 0) over bars in quartile q
```

```text
quart_balance_q[s,t,W,q] =
quart_support_q[s,t,W,q] - quart_damage_q[s,t,W,q]
```

Then define two derived features:

```text
quart_progressive_balance[s,t,W] =
(quart_balance_3 + quart_balance_4) - (quart_balance_1 + quart_balance_2)
```

```text
quart_late_vs_early_ratio[s,t,W] =
(quart_support_3 + quart_support_4 + eps) /
(quart_support_1 + quart_support_2 + eps)
```

with:

- `eps = 1e-8`

## Meaning

Where supportive structure was concentrated inside the window.

## Interpretation

- high `quart_progressive_balance`: the direction is strengthening late
- high `quart_late_vs_early_ratio`: supportive activity is concentrated in the back half

# B25 Fixed-Third Cumulative-Path Slopes

## Purpose

Replace vague curvature with interpretable path-bending structure.

## Definition

For each `W in {6, 12, 24}`, split the cumulative path `cp_j` into three equal segments.

For `W = 6`:

- thirds are `j = 0..2, 2..4, 4..6`

For `W = 12`:

- thirds are `j = 0..4, 4..8, 8..12`

For `W = 24`:

- thirds are `j = 0..8, 8..16, 16..24`

Freeze this overlap convention:

The boundary point is shared geometrically, but each slope is fit on its own segment endpoints only.

Define endpoint slopes:

```text
slope_1[s,t,W] =
(cp_{W/3} - cp_0) / (W/3)
```

```text
slope_2[s,t,W] =
(cp_{2W/3} - cp_{W/3}) / (W/3)
```

```text
slope_3[s,t,W] =
(cp_W - cp_{2W/3}) / (W/3)
```

These are average per-bar direction-relative slopes over each third.

## Meaning

How fast the direction-relative cumulative path advanced in the early, middle, and late parts of the window.

This is cleaner than fitting OLS lines for version 1.

# B26 Bend Metrics from Third-Slopes

## Purpose

Capture whether the path is strengthening, decelerating, or rolling over.

## Definition

Using B25:

```text
bend_early[s,t,W] =
slope_2[s,t,W] - slope_1[s,t,W]
```

```text
bend_late[s,t,W] =
slope_3[s,t,W] - slope_2[s,t,W]
```

```text
bend_total[s,t,W] =
slope_3[s,t,W] - slope_1[s,t,W]
```

## Meaning

- `bend_early`: did the path strengthen or weaken from early to middle segment
- `bend_late`: did the path strengthen or weaken from middle to late segment
- `bend_total`: is the path stronger now than it was at the start of the window

## Interpretation

- positive `bend_late`: late acceleration or strengthening into the present
- negative `bend_late`: late weakening, stall, or rollover
- positive `bend_total`: overall strengthening across the window

This is the main addon object Family C needs for late strengthening.

# B27 Late-Path Dominance Ratio

## Purpose

Measure whether recent supportive control is dominating the path more strongly near the end than across the window as a whole.

## Definition

Use the last third of the window.

Let:

```text
late_support[s,t,W] =
sum of max(x_j, 0) over j in the final third
```

```text
late_damage[s,t,W] =
sum of max(-x_j, 0) over j in the final third
```

Define:

```text
late_support_ratio[s,t,W] =
late_support / (late_support + late_damage + eps)
```

Then compare this with full-window support ratio built directly from `x_j` over the same `W`:

```text
window_support =
sum over all j of max(x_j, 0)
```

```text
window_damage =
sum over all j of max(-x_j, 0)
```

```text
window_support_ratio_local[s,t,W] =
window_support / (window_support + window_damage + eps)
```

Finally define:

```text
late_path_dominance_ratio[s,t,W] =
late_support_ratio[s,t,W] - window_support_ratio_local[s,t,W]
```

## Meaning

Whether the direction is more dominant in the late part of the window than it was across the full window.

## Interpretation

- positive: supportive dominance is improving into the present
- negative: late path is weaker than the overall window

This is useful because it isolates recent strengthening without relying only on cumulative slopes.

# Why These Belong in Family B, Not C

These are still pure ordered trailing-window geometry.

They do not require:

- explicit build labeling
- explicit release logic
- anchor-relative episode state
- compression state assumptions

They simply describe where support and damage sit in order, and how the cumulative path bends.

That makes them proper Family B objects.