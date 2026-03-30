1. Freeze the entry object

Use all causal `confirm_i` entries as the raw universe.

- Do not use realized `leg_dir`
- Do not filter on future stalled or failed outcomes

2. Freeze the label engine

From each `confirm_i`, run the same twin replay twice:

- long from entry
- short from entry

Use the same horizon, same exits, and same concurrency rules.

Define:

`S = pnl_long - pnl_short`

3. Freeze the gray zone from train only

On the training split only, choose an ambiguity rate `a`, then set:

`δ = Q_a(|S|)`

Label as follows:

- long if `S > δ`
- short if `S < -δ`
- ambiguous otherwise

4. Build the sign feature matrix at `confirm_i` only

Implement these four feature blocks:

- opposite-probe failure asymmetry
- barrier-thickness asymmetry
- template-fit differential
- filtered state asymmetry

5. Fit one scorer only

Train one linear pairwise dominance model on resolved rows only.

- No Bayesian
- No dynamic state-space
- No MLP yet

6. Validate by time split

Train on earlier data and test on later data.

Check:

- resolved count
- class balance
- AUC
- calibration
- top-decile precision
- stability by quarter/year

7. Inspect failure modes

Look at false longs and false shorts separately.

Find which of the four feature blocks breaks first:

- probe failure
- barrier thickness
- template fit
- filtered state

8. Only then decide next move

- If performance is weak, change features first
- If ranking is good but uncertainty is poor, add Bayesian calibration later
- If coefficients drift hard over time, then consider dynamic logistic
