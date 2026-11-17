# UFC Fight Predictor

Predicts the probability that each fighter wins a UFC bout, using only information available **before** the fight. It's a logistic regression trained on about 7,200 UFC fights from 2010 to 2026, built in five versions and evaluated against a baseline, across time, and against the betting market.

## Results

All versions use the same model and are scored on the same 1,430 most recent fights (June 2023 to March 2026), held out until the end.

| Version | What changes | Test log loss (lower is better) | Test accuracy | Walk-forward years beating baseline |
|---|---|---|---|---|
| Baseline | "Red wins 58% of the time" for every fight | 0.686 | 56.2% (always pick Red) | — |
| v1 | Age, losses, KO wins, submission wins | 0.704 | 55.9% | 4 of 8 |
| v2 | Experience, height, reach, win streak, rankings, age | 0.678 | 57.0% | 4 of 8 |
| v3 | v2 with a sign bug in the dataset's age column fixed | 0.647 | 62.2% | 8 of 8 |
| v4 | Features rebuilt from raw columns: rates instead of counts, smoothed win rate, past-prime age | 0.641 | 62.9% | 8 of 8 |
| **v5** | v4 plus striking and grappling stats, and a debut flag | **0.629** | **64.6%** | **8 of 8** |

On the 1,207 test fights that have betting odds, the market scores **0.581** and **69.9%**, against v5's 0.621 and 64.8%.

v5 is also well calibrated: when it gives Red 60–70%, Red wins about 66% of the time.

## A bug in the dataset

v2 looked fine on the test set, but walk-forward validation showed it beating the baseline from 2016 to 2019 and **losing every year from 2020 on**.

The cause was in the data: the dataset's ready-made `age_dif`, `loss_dif` and `lose_streak_dif` columns are **Red minus Blue** for fights before mid-2020 and **Blue minus Red** after. Most training fights come from before 2020, so the model learned the old meaning and then applied it to fights where the sign had flipped.

Recalculating the age gap from the raw ages, and changing nothing else (v3), took v2 from beating the baseline in 4 of 8 years to all 8. That's the biggest single improvement in the project. From v4 on, every feature is built from the raw per-fighter columns.

## Approach

- **Label:** did the red-corner fighter win (1 or 0). Draws and no-contests are removed (8 fights).
- **Leakage rule:** a feature is only allowed if it was known before the fight. Columns describing the fight itself (finish method, round, fight time) are never used, and every statistic is a pre-fight career summary.
- **Features (v5):** Red-minus-Blue differences in experience, a smoothed win rate (so 1–0 doesn't count as 100%), KO and submission rates, win and lose streaks, age plus years past 32, reach, striking volume and accuracy, takedown volume and accuracy, submission attempts, debut status, plus who is better ranked.
- **Model:** a scikit-learn pipeline: median imputation → standard scaling → logistic regression. Every step is fit on training data only, and every version uses the same pipeline.
- **Evaluation:**
  - Time-based split: train on fights up to June 2023, test on everything after.
  - Walk-forward validation (train on all years before year *N*, score on year *N*, for 2016–2023) to compare versions without touching the test set.
  - Log loss as the main metric, since the goal is trustworthy probabilities, not just picking winners.
  - Comparisons against a constant baseline and against the betting market.
- **No betting odds as features.** This is deliberate: with odds included, the model mostly copies the market, which hides what fighter stats can predict on their own.

## Does it add anything to the market?

A separate experiment gives the model the market's odds and asks whether the fighter stats still help.

| Same 1,207 test fights | Log loss | Accuracy |
|---|---|---|
| Odds only | 0.579 | 70.5% |
| Odds + v5's features | 0.575 | 70.8% |

Adding the stats helps in 6 of 8 walk-forward years. But the test-set gain (0.0045) has a bootstrap 95% range of −0.002 to +0.011, so it isn't proven. **The market already prices in almost everything in this dataset.** This is not evidence of a betting edge.

## Run it yourself

1. Download the [Ultimate UFC Dataset](https://www.kaggle.com/datasets/mdabbert/ultimate-ufc-dataset) from Kaggle, and unzip it so the files are at `archive/ufc-master.csv` and `archive/upcoming.csv`.
2. Install the requirements (tested with Python 3.14):
   ```bash
   pip install -r requirements.txt
   ```
3. Open the notebook and run all cells:
   ```bash
   jupyter lab ufc_predictor.ipynb
   ```

The notebook ends with v5's predictions for the next event in `upcoming.csv`, shown next to the market's odds.

## Project structure

| File | Contents |
|---|---|
| `ufc_predictor.ipynb` | The full project: the data bug, features, all five versions under walk-forward validation, the final test, calibration, the market comparison and predictions |
| `requirements.txt` | Python packages |

## Limitations

- **UFC-only records.** A fighter making their UFC debut after going 15–0 elsewhere looks like a beginner here.
- **No information the market has:** injuries, camp changes, short-notice replacements, style matchups.
- **Corner dependence.** The red corner wins about 58% of fights, and the model uses that. For a matchup with no assigned corners, you'd need to predict both ways and average.
- **Missing odds:** about 16% of 2023 fights and 25% of 2024 fights have no odds in the dataset, so market comparisons use only the fights that do.

## What's next

- Tree models (random forest, gradient boosting) to capture effects a weighted sum can't, such as age mattering more after 35.
- Pre-UFC fight records.
- Corner-free predictions.

## Data

[Ultimate UFC Dataset](https://www.kaggle.com/datasets/mdabbert/ultimate-ufc-dataset) on Kaggle. The data isn't included in this repository; download it from the link above.
