# WaniKani Personalized Review Recommender

Predicting which Japanese kanji cards will be hard **for me**, using my own study data from the
[WaniKani](https://www.wanikani.com/) API.

I study Japanese (JLPT N3) with WaniKani, a spaced-repetition app. Some cards keep falling back down
the SRS levels. This project learns which kinds of cards are hard for me and ranks the cards I'm
still learning, so I know where extra review time pays off.

## What's inside

| Step | What happens |
|---|---|
| Data | Pulls subjects, review statistics and assignments from the WaniKani API v2 (paginated, cached locally) |
| Features | Card properties only: type, level, length, components, meanings, readings, visually similar kanji, how long I've studied it |
| Label | `isHard`: card's error rate is at or above my median |
| Models | Baseline vs Logistic Regression vs Random Forest, 5-fold cross-validation + held-out test set |
| Evaluation | ROC-AUC, confusion matrix, precision / recall, feature importance |
| Output | Ranked list of cards in progress, most likely to be hard first |

## Avoiding target leakage

The first version of this notebook used my error rates as features **and** built the label from the same
error rates. The model scored perfectly (every top card had probability 1.0) because it was simply
recomputing the label. I removed every feature derived from my answers (error rates, current SRS stage,
number of reviews) and kept only properties of the card itself. The scores are lower now, but they are honest.

## Results

| Model | CV ROC-AUC (5-fold) | Test ROC-AUC |
|---|---|---|
| Baseline (most frequent class) | 0.500 | 0.500 |
| Logistic Regression | 0.724 ± 0.037 | 0.715 |
| Random Forest | 0.717 ± 0.043 | 0.717 |

Both models clearly beat the baseline and perform about the same: the difference
is smaller than the cross-validation spread. I chose Logistic Regression because it is
simpler and its weights are easy to interpret.

On the held-out test set (351 cards) Logistic Regression reaches accuracy 0.66.
It finds 61% of the hard cards (recall), and 67% of the cards it flags are actually
hard (precision). CV and test scores are close, so the model is not overfitting.

Key findings:
- Vocabulary is the easiest card type for me; kanji and radicals are harder.
  Words probably benefit from context.
- Cards built from more components, and cards from higher WaniKani levels,
  are the hardest.
- Cards that appear inside many other cards are easier: I keep seeing them
  in other words, which works like extra review.
- Surprisingly, visually similar kanji have only a small effect.

Caveats: `subjectType` is encoded as a number (0/1/2), so the model cannot compare
radicals and kanji separately; one-hot encoding is a planned improvement. Features
are correlated (e.g. radicals have no components but are used in many cards),
so weights show associations, not causes.

## Run it

```bash
pip install -r requirements.txt
export WANIKANI_TOKEN="your-read-only-token"   # Windows PowerShell: $env:WANIKANI_TOKEN="..."
jupyter notebook wanikani_ds.ipynb
```

Get a read-only token at WaniKani → Settings → API Tokens. Data is cached in `wanikani_cache/`
(git-ignored), so the API is called only on the first run.

## Limitations and next steps

- The label describes the past. Next step: a time-based target. Save a snapshot of my statistics today,
  download again in two weeks, and predict which cards get **new** mistakes in between
  (the notebook already saves dated snapshots).
- One learner, ~1,700 cards, so the patterns are personal.
- Planned: a small Streamlit app showing the ranked review queue.

## Stack

Python · pandas · NumPy · scikit-learn · matplotlib · seaborn · Jupyter · REST API
