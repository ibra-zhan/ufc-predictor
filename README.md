# ufc-predictor

Predicts the probability of each fighter winning a UFC fight, using only stats known before the fight. Logistic regression on ~7,200 fights (2010-2026).

## results

Test set is every fight after June 2023 (1,430 fights). Log loss is the main metric, lower is better.

| | log loss | accuracy |
|---|---|---|
| baseline (red's win rate for every fight) | 0.686 | 56.2% |
| v1 | 0.704 | 55.9% |
| v2 | 0.678 | 57.0% |
| v3 | 0.647 | 62.2% |
| v4 | 0.641 | 62.9% |
| v5 | 0.629 | 64.6% |
| betting odds* | 0.581 | 69.9% |

\*only the 1,207 test fights that have odds. v5 gets 0.621 on those.

## versions

- v1: age, losses, KO wins, sub wins
- v2: experience, height, reach, win streak, rankings, age
- v3: v2 with the age bug fixed (below)
- v4: rebuilt all the features from the raw columns, rates instead of counts
- v5: v4 + striking/grappling stats and a debut flag

## the age bug

v2 beat the baseline until 2020, then lost to it every year after. The dataset's `age_dif` column is red minus blue before mid 2020 and blue minus red after, so the model learned age backwards for all the recent fights. `loss_dif` and `lose_streak_dif` have the same problem. Fixing just the age column (v3) was the biggest jump in the whole project.

## odds

I didn't use betting odds as a feature, otherwise the model mostly just copies the market. I did try adding v5's features on top of the odds: it was slightly better than odds alone in 6 of 8 years, but not by enough to say it's real.

## running it

Download the [Ultimate UFC Dataset](https://www.kaggle.com/datasets/mdabbert/ultimate-ufc-dataset) and put `ufc-master.csv` and `upcoming.csv` in an `archive/` folder. Then:

```
pip install -r requirements.txt
jupyter lab ufc_predictor.ipynb
```

## todo

- try gradient boosting
- pre-UFC records (debut fighters look like beginners right now)
- predictions that don't depend on which corner a fighter is in
