# F1 Race Strategy Analysis — 2025 Season

Pre-race and post-race Formula 1 strategy modelling, built from real timing data with [FastF1](https://docs.fastf1.dev/), self-directed across the full 2025 season (Australia → Abu Dhabi). The pipeline estimates tyre degradation from historical laps, builds free-air strategy scenarios under each race's tyre rules, and then stress-tests one-stop vs. two-stop plans with a Monte Carlo simulation of overtaking, traffic, and safety-car events.

This repository is a representative sample of that work, not the full archive. It contains two complete races and the shared tyre-degradation engine — chosen deliberately to show how the methodology evolved (see [How the project evolved](#how-the-project-evolved)).

---

## What's in here

| Folder / file | What it is |
| --- | --- |
| `tyre_deg.ipynb` | The data-driven core: pulls real race laps via FastF1, fuel-corrects them, filters out-laps and mistakes, and fits per-stint degradation with linear regression. |
| `australia-gp/` | Australian GP analysis — an **earlier** iteration, built on hand-tuned tyre models (linear and exponential concept curves). |
| `abu-dhabi-gp/` | Abu Dhabi GP analysis — the **final** iteration, with a full 10,000-run Monte Carlo race simulator and an overtaking model. |

Australia was one of the first races I analysed; Abu Dhabi 2025 was the last. Reading them side by side is the point.

---

## The pre-race workflow

For each round, published on LinkedIn the Thursday before Free Practice 1:

1. **Ingest data.** Load the previous three editions of the same circuit via FastF1 — kept within a single regulation cycle so the cars are comparable.
2. **Clean.** Restrict to accurate green-flag laps, drop in/out laps, and apply a 107%-of-fastest filter per compound to remove traffic-affected and mistake laps.
3. **Fuel-correct.** Subtract a fuel-load term (≈0.03 s/kg, 110 kg start load) so degradation isn't confounded by the car getting lighter.
4. **Model degradation.** Estimate per-compound wear, first as concept curves (linear and exponential) and later by regressing fuel-corrected lap times against tyre age (with R² reported as a fit check).
5. **Build free-air strategies.** Construct candidate stint plans under that race's specific dry-tyre rules, and find the optimal one-stop and two-stop in clean air.
6. **Simulate reality.** Run the one-stop and two-stop through a Monte Carlo simulator using circuit-specific estimates for overtaking delta, per-lap overtaking probability, SC/VSC probability and duration, pit loss under green vs. safety-car conditions, and time lost to lapped cars.
7. **Pick the real optimum.** The free-air winner often isn't the real-world winner. (Singapore was a clean example: two-stop is faster in free air, but track position makes the one-stop the stronger race plan.)

## The post-race workflow

After the race I compared the simulator's predicted race time against the **observed** result and looked at where the model was right, where it was off, and why — which is what drove the changes between early and late rounds.

---

## How the project evolved

The two races in this repo bracket a real methodological shift, and I've kept the rough early version on purpose.

- **Early (Australia):** tyre degradation came from *assumed* coefficients — hand-tuned linear and exponential curves. The notebooks say so in the comments. The strategy comparison was a straight free-air lap-time sum.
- **Mid-season (`tyre_deg.ipynb`):** I replaced those guesses with degradation rates *regressed from real lap data*, with outlier filtering and an R² sanity check. This is the single biggest jump in the project.
- **Late (Abu Dhabi):** on top of data-driven pace, I added a 10,000-iteration Monte Carlo layer — Poisson-based safety-car/VSC modelling, an overtaking model (DRS range, dirty-air loss, per-lap pass probability), traffic loss on pit rejoin, and a direct comparison of predicted vs. observed race time.

So the arc is: *assumed parameters → measured parameters → probabilistic race simulation.* The early code is left intact because the improvement is the most honest thing the repository can show.

---

## Tools

`Python` · `FastF1` · `pandas` · `NumPy` · `scikit-learn` (linear regression) · `Matplotlib` / `Plotly` · `Jupyter`

## Running it

These notebooks pull live timing data through FastF1, which caches large files locally (excluded from this repo via `.gitignore`).

```bash
pip install fastf1 pandas numpy scikit-learn matplotlib plotly
```

On first run FastF1 will create a `cache/` folder and download session data; subsequent runs read from the cache. Open any notebook in Jupyter and run top to bottom.

---

## Assumptions & limitations

Stated plainly, because a strategy model is only as good as what it admits to:

- Degradation is modelled per stint and assumes reasonably representative running; very short or heavily traffic-affected stints are filtered rather than corrected.
- The fuel-correction rate and several Monte Carlo inputs (overtaking delta, SC/VSC rates, pit losses) are circuit-specific estimates, not measured constants.
- The race simulation uses a reference-pace competitor rather than a full multi-car field, so traffic is approximated.
- The earlier (Australia) notebooks use hand-tuned tyre coefficients and are kept for comparison, not as a current best method.

---

## Background

I ran this as a self-directed, extracurricular project from January to December 2025, alongside finishing my BSc and starting an MSc in Artificial Intelligence and Data Analytics. I used Jupyter to teach myself Python through the project, publishing each pre-race analysis on LinkedIn. The series ended after Abu Dhabi 2025, as the new regulation cycle left no comparable historical data to model from.

The work was reviewed and critiqued throughout by industry experts, whose feedback shaped much of the methodology above.
