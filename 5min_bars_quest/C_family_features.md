# Family C -  measures build

Build means:

- supportive participation is recurring
- support is clustering rather than scattering
- damage is acceptable rather than destructive
- directional balance is improving into the present
- the path is separating from the opposite anchor
- the path is strengthening, but not yet fully in release

That is enough. It does not need to do everything.

# Family C Frozen Dependency Chain

Family C may use only these upstream frozen objects.

## From Family A

- `support_flow[s,t,W]`
- `damage_flow[s,t,W]`
- `support_ratio[s,t,W]`
- `support_body_flow[s,t,W]`
- `damage_body_flow[s,t,W]`
- `body_support_ratio[s,t,W]`
- build primitives from `side_ret_1[s,t]`
- current bar and rolling bar geometry from `m_open, m_high, m_low, m_close`
- `rng_norm[t]`

## From Family B Core

- `terminal_disp[s,t,W]`
- `dir_eff[s,t,W]`
- `max_pullback_from_peak[s,t,W]`
- `final_pullback_from_peak[s,t,W]`
- `pullback_retention_ratio[s,t,W]`
- `monotone_segment_concentration[s,t,W]`
- `damaging_interruption_count[s,t,W]`
- `sign_change_density[s,t,W]`
- `new_ground_frequency[s,t,W]`
- `stalled_tail_length[s,t,W]`
- `recovery_after_deepest_pullback_ratio[s,t,W]`

## From Family B Addon

- `half_progressive_balance[s,t,W]`
- `quart_progressive_balance[s,t,W]` where available
- `slope_1[s,t,W]`
- `slope_2[s,t,W]`
- `slope_3[s,t,W]`
- `bend_late[s,t,W]`
- `bend_total[s,t,W]`
- `late_path_dominance_ratio[s,t,W]`

Nothing else.

# Family C Window Set

Freeze:

- `W_build = {6, 12, 24}`

Family C features must be defined separately for each `W` in this set unless noted otherwise.

# Family C Subset to Freeze Now

Now that Family B addon exists, the clean Family C set can be:

- `C1 build_support_persistence`
- `C2 build_support_clustering`
- `C3 build_anchor_separation`
- `C4 build_pullback_acceptability`
- `C5 build_progressive_balance`
- `C6 build_late_strengthening`
- `C7 build_late_dominance`
- `C8 build_pre_release_efficiency`
- `C9 build_structure_quality` optional cumulative bs, discarded

This is a strong but still manageable build family.

# Exact Definitions

## C1. Build support persistence

Purpose:

Recurring supportive participation.

Use the same threshold as before.

Freeze:

- `theta_build_support = 0.10`

Define:

```text
build_support_flag[s,k] = 1 if side_ret_1[s,k] > theta_build_support, else 0
```

Then:

```text
build_support_persistence[s,t,W] =
(sum from k=t-W+1 to t of build_support_flag[s,k]) / W
```

Range:

- `[0,1]`

Meaning:

- fraction of recent bars making meaningful supportive contribution

## C2. Build support clustering

Purpose:

Supportive bars should organize into coherent local clusters.

Use the same supportive-run rule:

A supportive build run is a maximal contiguous sequence where `side_ret_1[s,k] > theta_build_support`.

Let `max_run_len` be the maximum supportive run length in the window.

Then:

```text
build_support_clustering[s,t,W] = max_run_len / W
```

If no supportive runs exist, return `0`.

Range:

- `[0,1]`

Meaning:

- how concentrated supportive participation is into coherent bursts

## C3. Build anchor separation

Purpose:

A build should separate from the opposite recent anchor.

For side `up`:

```text
opp_anchor[up,t,W] = min of m_low[k] over k=t-W+1 to t
```

For side `down`:

```text
opp_anchor[down,t,W] = max of m_high[k] over k=t-W+1 to t
```

Then:

```text
build_anchor_separation[s,t,W] =
dir[s] * (m_close[t] - opp_anchor[s,t,W]) / (PIP * scale[t])
```

Meaning:

- distance of the current close from the recent opposite-side anchor in local scale units

## C4. Build pullback acceptability

Purpose:

Interruptions should remain acceptable relative to supportive pushes.

Use the same frozen logic.

Supportive bars:

- `side_ret_1[s,k] > theta_build_support`

Damaging bars:

- `side_ret_1[s,k] < -theta_build_support`

Define:

```text
mean_support_mag[s,t,W] =
mean of side_ret_1[s,k] over supportive bars
```

```text
mean_damage_mag[s,t,W] =
mean of abs(side_ret_1[s,k]) over damaging bars
```

Edge rules:

- if no supportive bars, return `NaN`
- if supportive bars exist and no damaging bars, return `1`

Else:

```text
build_pullback_acceptability[s,t,W] =
1 - min(mean_damage_mag[s,t,W] / (mean_support_mag[s,t,W] + eps), 2.0) / 2.0
```

with `eps = 1e-8`

Range:

- `[0,1]`

Meaning:

- whether pullbacks are modest relative to supportive pushes

## C5. Build progressive balance

Purpose:

Build should strengthen into the present.

For `W in {12,24}`, use the quartile version:

```text
build_progressive_balance[s,t,W] =
quart_progressive_balance[s,t,W]
```

For `W = 6`, use the half version:

```text
build_progressive_balance[s,t,6] =
half_progressive_balance[s,t,6]
```

Meaning:

- whether side-relative support-minus-damage balance is improving in later parts of the window

Interpretation:

- positive means build is strengthening into the present

This is now fully frozen because the B addon froze the required objects.

## C6. Build late strengthening

Purpose:

The cumulative path should steepen into the present if build is intensifying.

Use the B addon bend metric directly.

Define:

```text
build_late_strengthening[s,t,W] =
bend_late[s,t,W]
```

Meaning:

- change in cumulative-path slope from middle third to final third

Interpretation:

- positive means the path is strengthening late
- negative means late weakening or stall

## C7. Build late dominance

Purpose:

Recent supportive dominance should exceed full-window average dominance if build is maturing.

Define:

```text
build_late_dominance[s,t,W] =
late_path_dominance_ratio[s,t,W]
```

Meaning:

- difference between support ratio in the last third and support ratio over the full window

Interpretation:

- positive means recent bars are more side-dominant than the window as a whole

## C8. Build pre-release efficiency

Purpose:

Build should be directional enough to matter, but still pre-release rather than fully explosive.

This should not use any new unfrozen compression object yet. Keep it simple and code-ready.

Define:

```text
build_pre_release_efficiency[s,t,W] =
dir_eff[s,t,W] * build_support_persistence[s,t,W]
```

Range:

- approximately `[-1,1]`, though usually useful in positive region

Meaning:

- persistent support that actually resolves into directional progress

Interpretation:

- high values mean the side is not only active, but constructively resolving that activity

This is intentionally lean and avoids weak compression proxies for now.

# Family C Interpretation by Stage

## For build

- `C1` should be positive and moderate to high
- `C2` should be positive
- `C3` should be increasing
- `C4` should remain favorable
- `C5`, `C6`, `C7` should usually be positive
- `C8` should be positive

## For release

- `C5` and `C6` may peak right before or at early release
- `C3` often keeps rising
- `C8` may remain high, but Family D should take over

## For continuation

- C features become less central
- `C3` can remain high, but `C5` and `C6` often flatten

## For fragility

- `C4` deteriorates
- `C5` weakens
- `C6` turns down
- `C7` weakens or goes negative

## For failure

- most C features collapse or reverse

# Family C Audit Contract

A Family C implementation is `Pass` only if all of the following are true:

- it uses only frozen Family A and Family B objects
- `W_build` is exactly `{6, 12, 24}`
- `theta_build_support` is exactly `0.10`
- `C1` uses the fraction of bars with `side_ret_1 > 0.10` exactly
- `C2` uses maximal contiguous supportive runs exactly
- `C3` uses current close versus opposite recent anchor exactly
- `C4` compares mean damage magnitude to mean support magnitude exactly, with the frozen edge rules
- `C5` uses `quart_progressive_balance` for `W in {12,24}`
- `C5` uses `half_progressive_balance` for `W = 6`
- `C6` equals `bend_late` exactly
- `C7` equals `late_path_dominance_ratio` exactly
- `C8` equals `dir_eff` times `build_support_persistence` exactly
- undefined rows remain `NaN`