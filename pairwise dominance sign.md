•  Freeze the entry object
Use all causal confirm_i entries as the raw universe. Do not use realized leg_dir. Do not filter on future stalled or failed outcomes. 
•  Freeze the label engine
From each confirm_i, run the same twin replay twice:
long from entry
short from entry
same horizon, same exits, same concurrency rules.
Define
S=pnl_long-pnl_short
•  Freeze the gray zone from train only
On the training split only, choose an ambiguity rate a, then set
δ=Q_a (∣S∣)
Label:
long if S>δ
short if S<-δ
ambiguous otherwise 
•  Build the sign feature matrix at confirm_i only
Implement these four feature blocks:
opposite-probe failure asymmetry
barrier-thickness asymmetry
template-fit differential
filtered state asymmetry 
•  Fit one scorer only
Train one linear pairwise dominance model on resolved rows only. No Bayesian, no dynamic state-space, no MLP yet. 
•  Validate by time split
Train on earlier data, test on later data.
Check:
resolved count
class balance
AUC
calibration
top-decile precision
stability by quarter/year 
•  Inspect failure modes
Look at false longs and false shorts separately.
Find which of the four feature blocks breaks first:
probe failure
barrier thickness
template fit
filtered state 
•  Only then decide next move
If performance is weak, change features first.
If ranking is good but uncertainty is poor, add Bayesian calibration later.
If coefficients drift hard over time, then consider dynamic logistic.
