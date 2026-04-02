⚽ AsesorFutBot — Sports Analytics SaaS on Telegram

  A production-grade sports prediction platform I built and operated from scratch, combining machine learning, probabilistic modeling, and real-time data pipelines — delivered entirely through a Telegram bot interface.


• What This Is

  AsesorFutBot is a SaaS product I co-founded and led technically from 2023 to January 2026. It was a live system with paying subscribers, real predictions, and measurable outcomes across 10+ football leagues.
  This repository is a technical portfolio piece — evidence of what I can design, build, and operate end-to-end. No source code is shared here; the product remains proprietary.

• What the Product Does

AsesorFutBot was a Telegram-based subscription service giving users access to professional-grade statistical analysis for football matches. A user would select a match and receive:

  1X2 consensus probabilities aggregated from 10+ bookmakers

  Best available odds compared across Bet365, 1xBet, Unibet, Betano, and others
  Exact score predictions via Poisson distribution and Monte Carlo simulation (up to 20,000 iterations)
  30+ alternative betting markets ranked by confidence score (Over/Under lines, BTTS, team goal markets)
  ML-based predictions with explicit probability estimates per outcome

  Users configured simulation parameters directly inside Telegram via inline button menus — no commands needed.

• Product in Action

  Odds Comparator — 1X2
  Aggregates implied probabilities from 10+ bookmakers and surfaces the best available odds per outcome.
  
  
  Exact Score Prediction — Poisson + Monte Carlo
  Estimates expected goals (μH, μA) per team, then runs up to 20,000 simulations to produce a full probability distribution over scorelines. Each score shows its fair odds — the theoretical price at which that bet has zero edge.



  Alternative Markets — TOP 30
  
  Ranks 30+ betting markets by a composite confidence score combining ML probability, Poisson probability, and market consensus agreement.
  


  Market Consensus
  
  Computes fair odds from first principles by removing the bookmaker margin from 10 books, then averaging the resulting implied probabilities.



• System Architecture

  The system was built in four independent layers that composed into a single Telegram interface:
  Data Layer — Ingested and normalized historical match data across 10+ leagues. Custom feature engineering pipeline with rolling form windows, home/away splits, league-adjusted performance metrics, and head-to-head records.
  Prediction Engine — Two parallel models ran for every match request: a Poisson regression model to estimate expected goals per team, and an XGBoost classifier trained on engineered features. Monte Carlo simulation (up to 20,000 iterations) produced the full exact-score distribution from the Poisson estimates.
  Odds Engine — Scraped and normalized real-time odds from 10+ bookmakers. Computed market consensus probabilities by averaging implied probabilities across books after margin removal. Generated "cuota justa" (fair odds) as the model's theoretical price for each outcome.
  Bot Layer — Built on python-telegram-bot. Handled conversation state, tiered subscription gating, per-user rate limiting, and a full inline button menu system. Users navigated leagues, matches, and prediction types without typing any commands.

• Model Performance

  Models were trained and evaluated per geographic region to maximize sample size while preserving structural homogeneity. Train/test split was 80/20 time-ordered — no future leakage.
  
  
  Baseline context: A naive model that always predicts "home win" achieves ~45% accuracy. The 69% figure must be read against that baseline, not against 50%. (Ours si superior ro academic models and some of the market)


• Technical Decisions Worth Noting

  Why Poisson + MC instead of just the ML classifier?
  The ML classifier predicts 1X2 outcomes well but gives no information about goal distributions. The Poisson model produces a full probability distribution over every possible scoreline — necessary for exact score markets and for generating 30+ alternative market probabilities. Both models run in parallel and their outputs are blended for the final confidence scores.
  Why regional models instead of per-league?
  Per-league models had insufficient sample sizes (~400–600 matches per league). Grouping by region gave 2,000–4,000 samples while keeping structural characteristics consistent — average goals, home advantage factors, and team quality distributions are all more similar within a region than across them.
  Why Monte Carlo instead of analytical Poisson probabilities?
  For exact scores, both approaches converge. MC was chosen because it makes it trivial to compute arbitrary derived markets without writing closed-form expressions for each one — just count the simulations that satisfy the condition.
  On the tight CV variance (México 1X2: ±0.54%)
  This is the most meaningful number in the metrics table. Tight cross-validation variance across time windows means the model generalizes consistently — its performance isn't a product of a lucky test split or a specific season's characteristics.

Link to the webpage:
https://pronosfutbol-web.vercel.app/

![Screenshot_20251204_161009_Telegram](https://github.com/user-attachments/assets/526b2c1f-1e08-4424-9495-298ee5dcaed7)
![Screenshot_20251124_211204_WhatsApp](https://github.com/user-attachments/assets/8e6ac56c-df8b-416e-93a1-7203d88825d9)

![Screenshot_20251124_211258_WhatsApp](https://github.com/user-attachments/assets/9f3e191b-59e9-46d9-876a-7ec7178db48c)

![Screenshot_20251204_161009_Telegram](https://github.com/user-attachments/assets/9aaf7a13-94ab-4112-ae42-84b745d56ac3)
![Screenshot_20251124_211204_WhatsApp](https://github.com/user-attachments/assets/fb950448-6589-456f-8ace-e1df8704f85b)
![Screenshot_20251203_215518_Telegram](https://github.com/user-attachments/assets/d0acf727-a5af-449a-8a96-91660e67793f)
![Screenshot_20251123_123044_Telegram](https://github.com/user-attachments/assets/75a182e6-d27e-4328-b432-b8da6bdff1e2)
![Screenshot_20251123_115311_Telegram](https://github.com/user-attachments/assets/531da801-b0e3-4110-bd0c-58c5086ed7f0)


Stack
LayerTechnologyBot interfacePython, python-telegram-botML modelsscikit-learn, XGBoost, pandas, numpyProbabilisticCustom Poisson + Monte Carlo (numpy)Data storageSQLite + flat file store per leagueDeploymentLinux VPS, long-pollingOdds dataWeb scraping + public API aggregation
machine-learning  python  telegram-bot  sports-analytics  xgboost  poisson  monte-carlo  saas SQL for users database management and payments

👤 About
Built by Humberto G. Estrada
Interested in data, products, and systems that operate under real constraints.
LinkedIn https://www.linkedin.com/in/humberto-gonzalez-estrada-5b768b3a4/
