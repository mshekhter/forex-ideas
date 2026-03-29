# Pairwise Dominance Sign Model

## Objective

Train a sign scorer at the causal entry point using a frozen entry universe, a frozen twin-replay label engine, and four signal-bar feature blocks. The first model is a linear pairwise dominance model only.

## 1. Freeze the entry object

Use all causal `confirm_i` entries from causal replay (cell 7) as the raw universe.
Turning point rule (causal)in cell 7:
- Track running extreme in current direction
- Confirm reversal when excursion from extreme >= k * s_t
- s_t is EWMA(|Δprice|) computed causally 

Rules:

- Use every causal `confirm_i` produced there
- Do **not** use realized `leg_dir`
- Do **not** filter on future stalled outcomes
- Do **not** filter on future failed outcomes

This entry object is the fixed universe for labeling and feature construction.

## 2. Freeze the label engine

For each `confirm_i`, run the same twin replay twice:

- long from entry
- short from entry

Use the same:

- horizon
- exits
- concurrency rules

Define the pairwise dominance score as:

$$
S = \mathrm{pnl}_{\text{long}} - \mathrm{pnl}_{\text{short}}
$$

Where:

- $\mathrm{pnl}_{\text{long}}$ is the replay PnL from entering long at `confirm_i`
- $\mathrm{pnl}_{\text{short}}$ is the replay PnL from entering short at `confirm_i`

## 3. Freeze the gray zone from train only

Determine ambiguity from the training split only.

Choose an ambiguity rate $a$ on the training set, then define:

$$
\delta = Q_a \left( |S| \right)
$$

Where:

- $Q_a(|S|)$ is the $a$-quantile of the absolute dominance magnitude on the training split

Apply labels as follows:

$$
y =
\begin{cases}
\text{long}, & S > \delta \\
\text{short}, & S < -\delta \\
\text{ambiguous}, & |S| \le \delta
\end{cases}
$$

Operationally:

- label `long` if $S > \delta$
- label `short` if $S < -\delta$
- label `ambiguous` otherwise

Important freeze:

- $\delta$ is estimated on train only
- the same frozen $\delta$ is then applied to the later test split

## 4. Build the sign feature matrix at `confirm_i` only

Construct the feature matrix strictly at `confirm_i`.

Do not use future path information in features.

Implement exactly these four feature blocks:

1. opposite-probe failure asymmetry  
2. barrier-thickness asymmetry  
3. template-fit differential  
4. filtered state asymmetry  

This is the frozen first-pass feature scope.

## 5. Fit one scorer only

Train one model only:

- linear pairwise dominance model
- resolved rows only

Resolved rows are those labeled:

- `long`
- `short`

Exclude:

- `ambiguous`

Do not introduce any other model class at this stage.

Explicit exclusions for this phase:

- no Bayesian model yet
- no dynamic state-space model yet
- no MLP yet

## 6. Validate by time split

Use a forward time split:

- train on earlier data
- test on later data

Evaluate the following:

- resolved count
- class balance
- AUC
- calibration
- top-decile precision
- stability by quarter
- stability by year

## 7. Inspect failure modes

Review errors separately:

- false longs
- false shorts

Then determine which feature block breaks first:

- probe failure
- barrier thickness
- template fit
- filtered state

This failure analysis comes before model expansion.

## 8. Decision rule for next steps

Only decide the next move after the first-pass evaluation and failure analysis.

Decision logic:

- If performance is weak, change features first
- If ranking is good but uncertainty is poor, add Bayesian calibration later
- If coefficients drift hard over time, consider dynamic logistic later

## Frozen sequence of work

1. Freeze entry universe from cell 7 causal `confirm_i`
2. Freeze twin-replay label engine
3. Compute $S$
4. Estimate $\delta$ from training only
5. Label `long`, `short`, `ambiguous`
6. Build four-block feature matrix at `confirm_i`
7. Fit one linear model on resolved rows only
8. Validate by forward time split
9. Inspect false-long and false-short failure modes
10. Decide whether the next change is feature refinement, Bayesian calibration, or dynamic logistic

## Minimal formula summary

Dominance score:

$$
S = \mathrm{pnl}_{\text{long}} - \mathrm{pnl}_{\text{short}}
$$

Gray-zone threshold from train only:

$$
\delta = Q_a \left( |S| \right)
$$

Label rule:

$$
\text{long if } S > \delta
$$

$$
\text{short if } S < -\delta
$$

$$
\text{ambiguous otherwise}
$$

