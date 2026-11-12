# Mathematical Modelling Group Project (MAC F314)

## The problem

How can the same neuron settle to rest after a small disturbance, produce one spike after a larger disturbance, or fire repeatedly under sustained current? We use the FitzHugh–Nagumo model to study how the applied current and the neuron's starting state determine which response occurs.

## The FitzHugh–Nagumo model

The model describes one excitable cell with two coupled differential equations.

```text
dv/dt = v − v³/3 − w + I
dw/dt = ε(v + a − bw)
```

- `v` represents the membrane voltage, which changes rapidly during a spike.
- `w` represents slower recovery processes. As recovery increases, the `−w` term reduces the voltage slope.
- `I` is the applied current, held constant within each simulation.
- `a` and `b` shape recovery, while `ε` scales its rate. We start with `a = 0.7`, `b = 0.8`, and `ε = 0.08`.

The variables and time are expressed in scaled model units.

## What we investigate

1. **Rest and excitability.** At zero current, a small voltage nudge fades, while a larger nudge produces one spike before returning to the same equilibrium.
2. **Repeated firing and stability.** At sustained current `I = 0.50`, the model fires repeatedly. We use phase portraits, nullclines, and the Jacobian to explain how this relates to the stability of the equilibrium.
3. **Transitions and sensitivity.** Two subcritical Hopf bifurcations occur near `I = 0.331281` and `I = 1.418719`. The first Lyapunov coefficient is positive at both crossings, so the small periodic orbit near each crossing is unstable and lies on the locally stable side of the equilibrium. At `I = 0.50`, increasing `ε` from `0.06` to `0.10` shortens the mean firing period from `49.157` to `33.525` model time units.

4. **How sharp the threshold is, and how far we checked around rest.** Measured at zero current, with recovery held at its resting value and a peak above `v = 1` as the spike criterion, bisecting on the initial voltage puts the firing threshold at a displacement of `0.5555` from rest, i.e. `v = -0.6439`. That is `0.1424` units to the *right* of the middle-nullcline value `-0.7863`. At two test displacements, `0.001` below and `0.001` above the estimated threshold, the peak voltage is `-0.3341` and `+1.5887`. Solving the Lyapunov matrix equation `A'P + PA = -I` at the resting state and evaluating `Vdot` for the *full nonlinear* system over the sublevel set `V <= 0.2059` gives a **numerically checked region**: across 4x10^5 boundary points and 400 interior level sets the largest `Vdot` found is `-4.05e-6`. The grid checks the sign at the points sampled, not between them, so this is a numerical check rather than a guarantee. Along the voltage axis the region reaches a displacement of `0.43433`, which is `78.2%` of the measured threshold.
5. **How the period scales.** Widening the recovery sweep to `eps` in `[0.005, 0.10]` gives a fitted exponent of `0.909` in `T ~ eps^-a` at the small end but only `0.750` over `[0.06, 0.10]`. The slow–fast argument predicts `1` in the singular limit, so the `32%` drop reported above is a correct measurement taken before that regime begins.

The [analysis notebook](fhn_project.ipynb) contains the derivations, simulations, and figures. Sections 11–13 cover the threshold, the numerically checked region around the resting state and the scaling study; every number quoted above is printed by a cell there.

## Final report and bonus results

The [compiled report](report/report.pdf) covers the model assumptions and equations, equilibrium and stability analysis, phase portraits, the current sweep, and recovery sensitivity. It derives the first Lyapunov coefficient in unit-frequency coordinates and explains how its sign classifies both bifurcations. Its bonus section compares a physics-informed neural network (PINN) with SciPy least squares on the inverse problem of estimating the recovery parameter `a` from voltage readings alone.

The [PINN notebook](fhn_pinn_ml.ipynb) contains the executed original benchmark and controlled follow-up. With sparse, noisy synthetic observations, both fits recovered the true value `a = 0.700` within 1% and predicted the correct spike count on the held-out trajectory at `I = 0.90`.

| Method | Estimated `a` | Relative parameter error | Held-out voltage RMSE |
| --- | ---: | ---: | ---: |
| PINN | 0.704093 | 0.58% | 0.015570 |
| SciPy least squares | 0.701926 | 0.28% | 0.007343 |

Least squares gave the smaller parameter and voltage errors in this synthetic experiment.

### Controlled comparison of noise and sample count

The original comparison changed noise and sample count together. Step 12 uses four conditions: 100 clean readings, 40 clean readings, 100 noisy readings, and 40 noisy readings. Within each density, clean and noisy fits use the same observation times. Within each of five noise seeds, the sparse readings are copied directly from the corresponding dense readings. This lets us isolate each factor in paired comparisons.

| Noisy training readings | Median PINN voltage RMSE | Median least-squares voltage RMSE |
| --- | ---: | ---: |
| 100 | 0.018354 | 0.002744 |
| 40 | 0.022869 | 0.002029 |

Least squares gave lower parameter and held-out voltage errors in **all 10 noisy comparisons**. Noise increased test error for both methods at both sample counts in every matched trial. Reducing the noisy readings from 100 to 40 increased RMSE in four of five PINN pairs and three of five least-squares pairs. The result applies to this synthetic benchmark with known equations and a fixed network initialization.

The [trial CSV](report/pinn_controlled_trials.csv) contains all 24 results. The [LaTeX source](report/main.tex) includes the [bonus analysis](report/bonus.tex), [result table](report/pinn_controlled_results.tex), and [references](report/references.bib). The [figure folder](report/figures) contains seven figures in PDF and EPS.

## AI acknowledgment

ChatGPT assisted with topic planning and the design of the bonus experiment. OpenAI Codex assisted with drafting and organizing both notebooks, commenting the code, checking numerical results and notebook execution, drafting the LaTeX report, and verifying its compilation.
