# ProScout

**The best way to improve your team** — football analytics powered by match data, player stats, and machine learning.

ProScout is a full pipeline for **scouting and predicting team performance**: it collects match and player data, aggregates team-level statistics, and trains models to predict **Expected Goals (xG)** and **Expected Goals Against (xGA)** from in-game metrics.

---

## What's inside

| Component | Description |
|-----------|-------------|
| **Data pipeline** | Scrape match IDs and lineups from HTML, fetch player stats from SofaScore, normalize team names (La Liga & more). |
| **Datasets** | Match-level stats (560+ rows, 80+ features), player-level stats, team aggregates, and stat coefficients. |
| **Notebook** | End-to-end ML workflow: feature engineering, Multi-Output XGBoost, RMSE evaluation, feature importance, and SHAP explainability. |

---

## Project structure

```
ProScout/
├── matchDictsL.py          # Build match_id → {home, away, round} from HTML
├── statsPerHalfL.py        # Extract match IDs from round HTML files
├── playerStatsL.py         # Fetch & process SofaScore player stats → CSV
├── FinalProject.ipynb      # Load data, train xG/xGA model, SHAP & plots
├── Datasets/
│   ├── matches_ALL_numeric (2).csv   # Match-level stats (teams, xG, referee, etc.)
│   ├── player_stats_ALL.csv          # Player-level stats per match
│   ├── final_team_aggregates (1).csv # Team aggregates per match
│   └── coefficients.csv              # Weights/coefficients for stats
└── README.md
```

---

## Data pipeline (scripts)

1. **`statsPerHalfL.py`**  
   Reads HTML round files (e.g. `round1.txt` … `round26.txt`), parses with BeautifulSoup, and collects all `data-id` values (match IDs).

2. **`matchDictsL.py`**  
   For each round HTML, builds a dictionary: `match_id → { "home", "away", "round" }` with normalized team slugs (Barcelona, Real Madrid, Atlético, Valencia, etc.). Merges rounds 1–26 into `all_match_dict`.

3. **`playerStatsL.py`**  
   For each match ID in `data_ids`, calls the SofaScore lineups API, processes home/away player statistics (ratings, goals, passes, duels, etc.), and appends to a combined list. Writes everything to a CSV (e.g. `player_statsBundesliga.csv`). Uses `matchDictsL.all_match_dict` for round/side context.

**Note:** Scripts use hardcoded paths (e.g. `/Users/jd/.../Bundesliga/round{i}.txt`). Point these to your own HTML and output paths as needed.

---

## Modeling (notebook)

`FinalProject.ipynb` does the following:

- **Load** `Datasets/matches_ALL_numeric (2).csv` and rename `Expected goals` → `xG`.
- **Pair matches** so that for each team, xGA = opponent’s xG.
- **Targets:** `xG` (offensive) and `xGA` (defensive).
- **Features:** All numeric columns (passes, possession, shots, duels, referee dummies, etc.) after dropping `match_number`, `team`, `opponent`, `RESULT`.
- **Model:** `MultiOutputRegressor(XGBRegressor)` predicting both xG and xGA.
- **Evaluation:** RMSE on a 80/20 train/test split (e.g. xG ~0.44, xGA ~0.73).
- **Interpretation:** XGBoost feature importance and SHAP summary plots for both outputs.

---

## Tech stack

- **Python:** BeautifulSoup, requests (curl_cffi), pandas, numpy  
- **ML:** scikit-learn (train_test_split, MultiOutputRegressor), XGBoost  
- **Explainability:** SHAP, matplotlib  

---

## Quick start

1. **Data:** Place your round HTML files where the scripts expect them (or edit paths in `statsPerHalfL.py` and `matchDictsL.py`). Run `statsPerHalfL.py` → `matchDictsL.py` → `playerStatsL.py` to refresh match and player CSVs.

2. **Notebook:** Open `FinalProject.ipynb`, point the CSV path to `Datasets/matches_ALL_numeric (2).csv` (and optionally mount Google Drive if you use Colab). Run all cells to train the model and generate plots.

---

## Datasets at a glance

| File | Rows | Description |
|------|------|-------------|
| `matches_ALL_numeric (2).csv` | 560 | One row per team per match: result, xG, passes, shots, duels, referee dummies. |
| `player_stats_ALL.csv` | ~37k+ | One row per player per match: ratings, goals, assists, passes, tackles, etc. |
| `final_team_aggregates (1).csv` | ~2k+ | Aggregated team stats per match. |
| `coefficients.csv` | 49 | Stat names and coefficients (e.g. for weighting metrics). |

---

## License

Use and adapt as you like. If you use SofaScore data, respect their terms of service and rate limits.

---

*ProScout — from raw match HTML to xG/xGA predictions.*
