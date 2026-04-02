# Model Evaluation — AsesorFutBot Prediction Engine

## Overview

Two prediction tasks were modeled independently for every match:

- **1X2** — classify match outcome as Home Win, Draw, or Away Win (3-class problem)
- **O/U** — binary classification: total goals above or below 2.5

Models were trained per geographic region, not per individual league, to maximize sample size while preserving structural homogeneity in play styles and competition dynamics.

---

## Data

### Sources
Historical match results from 10+ leagues (2018–2025). All features are derived entirely from historical match data — no external data dependencies at inference time.

### Regions & Sample Sizes

| Region      | Leagues | Total Samples | Train  | Test |
|-------------|---------|---------------|--------|------|
| México      | 5       | 3,856         | 3,084  | 772  |
| Sudamérica  | 5       | 2,369         | 1,895  | 474  |

Train/test split is 80/20, **time-ordered**. The test set is always chronologically after the training data — no future leakage at any point in the pipeline.

---

## Feature Engineering

All features are computed from rolling historical windows to prevent data leakage. Key feature groups:

**Team form (rolling 5-match window)**
Win rate, goals scored per game, and goals conceded per game for both the home and away team, computed separately over their last 5 matches before the fixture date.

**Venue-specific performance**
Home win rate for the home team, away win rate for the away team, computed over the full available history. This captures structural home/away tendencies beyond recent form.

**Head-to-head record**
Win counts and average total goals across the last 10 meetings between the two specific teams.

**League context**
Seasonal average goals per match in the specific league, used to normalize team-level metrics relative to the competitive environment.

**Match metadata**
Current standings position difference, days of rest since each team's last match, and season stage (early / mid / late / playoffs).

---

## Model: XGBoost Classifier

XGBoost was selected over logistic regression and random forest based on cross-validation performance. Hyperparameters were tuned via randomized search with 5-fold time-series cross-validation.

Key regularization choices: moderate tree depth (max 4), conservative learning rate (0.05), and both L1 and L2 regularization to prevent overfitting on the relatively small regional datasets.

Class imbalance (home wins are ~45% of outcomes in most leagues) was handled via class weighting during training, preventing the classifier from simply learning to over-predict home wins.

---

## Results

### México — 5 Leagues

```
Samples totales: 3,856
Samples train:   3,084
Samples test:      772

Accuracy 1X2:   69.30%
Accuracy O/U:   67.36%

CV 1X2:  65.89%  (±0.54%)
CV O/U:  67.22%  (±2.01%)

Target range: 68–73%
```

### Sudamérica — 5 Leagues

```
Samples totales: 2,369
Samples train:   1,895
Samples test:      474

Accuracy 1X2:   60.76%
Accuracy O/U:   70.68%

CV 1X2:  60.79%  (±1.72%)
CV O/U:  68.60%  (±1.16%)
```

---

## Interpretation

### On the baseline
In most football leagues, home wins occur roughly 45% of the time. A naive classifier that always predicts "home win" achieves ~45% accuracy on 1X2. The 69.30% figure for México must be read against that baseline — the model adds ~24 percentage points over naive prediction.

### On calibration
Raw XGBoost probabilities are not inherently calibrated. A Platt scaling step was applied post-training to align predicted probabilities with observed empirical frequencies. This matters for the "cuota justa" (fair odds) output — if the model outputs 40% probability, that should reflect approximately 40% real-world frequency on structurally similar matches.

### On cross-validation variance
The tight CV variance for México 1X2 (±0.54%) is the most meaningful number in the evaluation. It indicates that model performance is stable across different time windows — not a product of a lucky test split or a specific season. The higher O/U variance for México (±2.01%) reflects genuine seasonal unpredictability in goal volumes, particularly across early vs. late season stages.

### On the performance gap between regions
México (69%) outperforms Sudamérica (61%) on 1X2. This reflects both data volume differences and structural factors: Mexican leagues have more consistent team quality metrics season-over-season; South American leagues show higher variance driven by factors like mid-season Copa Libertadora peaking and larger talent gaps between clubs. Notably, O/U performance is comparable between regions (~67–70%), suggesting that goal-scoring volumes are more structurally predictable than match outcomes.

### On draw prediction
The draw class is systematically the hardest to predict — typically 35–40% recall regardless of model architecture. This is consistent with academic literature on football prediction and reflects the genuine difficulty of distinguishing "match where neither team dominates" from a structural signal.

---

## Probabilistic Layer — Poisson + Monte Carlo

Parallel to the ML classifier, a Poisson regression model estimates expected goals (μH, μA) for each match. This powers the **Marcadores Exactos** and **Apuestas Alternativas** features.

### Goal rate estimation
Each team's expected goals are estimated as a product of their attack strength, their opponent's defensive weakness, and the league baseline — all computed from recent rolling windows and normalized to the league average. This approach follows the Dixon-Coles framework for football prediction.

### Monte Carlo simulation
Given the expected goal rates (μH, μA), the match is simulated N times by drawing independently from two Poisson distributions. At N=20,000, probability estimates for the top scorelines are stable to within ±0.3%.

### Fair odds
Each scoreline's fair odds are computed as the reciprocal of its simulated probability. This gives users a direct comparison point against bookmaker prices — if a book offers 10.0 on a scoreline the model prices at 7.49 (13.35% probability), that implies a theoretical positive edge.

### Derived markets
All 30+ alternative markets (Over/Under thresholds, BTTS, team-specific goal lines) are derived by counting the fraction of simulations that satisfy each condition. This approach makes it trivial to add new markets without deriving closed-form expressions — the simulation naturally handles any countable condition.

---

## Known Limitations


**No live data** — models use only pre-match features. In-game events (red cards, injuries) are not incorporated.

**Transfer lag** — a team that signed key players mid-season won't reflect that in rolling form windows until sufficient post-transfer matches have been played.

**Small-sample leagues** — leagues with fewer than 200 historical matches are underrepresented; model confidence is lower for those competitions.

**Draw prediction** — systematically harder across all configurations, consistent with the broader literature.



[Evidencia_Exito_FutbolBot.xlsx](https://github.com/user-attachments/files/26444015/Evidencia_Exito_FutbolBot.xlsx)

Algunas de las ultimas predcciones hechas con el bot y los modelos propios.
