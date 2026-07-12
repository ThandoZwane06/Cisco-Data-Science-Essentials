# SA Fraud Detection: Rule-Based Suspicion Flagging

A small, hands-on fraud detection project using a synthetic South African transaction dataset. The goal wasn't to build the most accurate model on day one. It was to start small enough to actually understand *why* a transaction gets flagged, before reaching for anything more complex.

This is part of an ongoing series of fintech-focused mini projects, building toward a comparison between rule-based detection and a simple ML model.

## Dataset

- 360 transactions across 45 customers, generated for this project
- Columns: `transaction_id`, `customer_id`, `timestamp`, `amount_zar`, `merchant_category`, `province`, `customer_home_province`, `is_fraud`
- ~10% fraud rate, with four fraud patterns deliberately built in:
  1. Large amount spikes
  2. Late-night transactions
  3. Out-of-province spending
  4. Rapid, back-to-back transactions

Keeping the dataset small and synthetic meant I could sanity-check my own logic against known ground truth, rather than guessing whether my rules made sense.

## Approach

Rather than jumping straight into a model, I deliberately started with manual, rule-based flagging. The idea: write suspicion rules based on what *should* look unusual, then test those assumptions against the real fraud labels to see what held up and what didn't.

### Rule 1: Late-Night Spending Spikes
Flag any transaction between 1:00 AM–5:00 AM where the amount is more than 2x that customer's historical average. Targets sudden, aggressive account draining during low-activity hours.

### Rule 2: Out-of-Province Restrictions
Flag any transaction where the merchant's province doesn't match the customer's home province, excluding the `Travel` category (since legitimate travel, such as flights, hotels, or car rentals, naturally crosses provinces).

### Rule 3: Rapid Consecutive Transactions
Flag any transaction occurring within 15 minutes of a previous transaction by the same customer. Targets card-cloning style behaviour: fast, repeated charges before a legitimate account holder can react.

### Rule 4: Amount Anomaly (Any Time)
Flag any transaction where the amount exceeds 3x the customer's historical average, regardless of time of day. This rule was added *after* evaluating Rule 1 in isolation; see Evaluation below for why.

## Evaluation

Rules were checked against the actual `is_fraud` labels using a confusion matrix.

**With Rules 1–3 only:**

| | Actual: Safe | Actual: Fraud |
|---|---|---|
| Predicted: Safe | 323 | 20 |
| Predicted: Suspicious | 1 | 16 |

- Precision: 94.1%
- Recall: 44.4%

**With Rule 4 added:**

| | Actual: Safe | Actual: Fraud |
|---|---|---|
| Predicted: Safe | 320 | 14 |
| Predicted: Suspicious | 4 | 22 |

- Precision: 84.6%
- Recall: 61.1%

### What I learned from digging into the misses

Rule 1 caught 0 fraud cases on its own initially. Investigating why revealed that requiring *both* "late-night" and "amount spike" in a single condition was too strict. Nearly half of my missed fraud cases were amount anomalies happening at completely normal hours (a customer's usual R1,700 average spiking to R6,200 at 1:00 PM, for example). Separating the amount check from the time check (Rule 4) closed a large part of that gap, lifting recall from 44% to 61% with only a small drop in precision.

I also questioned whether excluding `Travel` from Rule 2 was the right call, since it created a blind spot: one missed fraud case was a Travel transaction that would have been caught without the exclusion. I chose to keep the exclusion rather than remove it, since legitimate travel is genuinely cross-province and removing it would likely increase false positives across the board. A sharper version of this rule (flagging Travel only in a province the customer has never transacted in before) is a natural next iteration, rather than a blanket category exclusion.

## Limitations of rule-based flagging

- **Precision-recall tradeoff is manual and blunt.** Every threshold I chose (2x, 3x, 15 minutes) was a judgment call, not something learned from the data. Small changes to these thresholds shift results in ways that are easy to test but hard to optimise by hand.
- **Rules can't learn interactions between features.** A model could potentially learn that "medium amount + slightly unusual hour + new merchant category" together are suspicious, even if no single factor crosses a threshold on its own. My rules can only catch what I explicitly thought to check for.
- **Even after adding Rule 4, ~39% of fraud cases were still missed.** Some of these likely don't trip any single obvious signal (a normal amount, normal hour, home province, and no rapid repeats), which by definition, rule-based flagging can't catch.
- **Rules don't scale well to new fraud patterns.** If a new type of fraud emerged that didn't match any of the four patterns built into this dataset, none of my rules would catch it, since they were designed against known patterns rather than learned from the data itself.

## Model comparison: logistic regression

After evaluating the rules, I built a logistic regression model to see whether a statistical approach could improve on manually chosen thresholds, and to compare two different ways of framing the same problem.

### Features

Reused the same signals the rules were built on, converted into numeric form so the model could use them: `amount_zar`, `amount_over_avg` (amount relative to that customer's average), `hour`, `is_midnight`, `is_out_of_province`, `rapid_transx`, and `period_between_transx` (raw minutes since the customer's previous transaction).

Deliberately included both the raw and derived versions of some features (`amount_zar` alongside `amount_over_avg`, `hour` alongside `is_midnight`) as a first pass, rather than pre-selecting only one version of each signal. This introduces multicollinearity (correlated features competing for the same signal), which shows up later in the coefficient results.

### Setup

- 70/30 train/test split, stratified on `is_fraud` so both sets kept roughly the same ~10% fraud rate
- Missing values in `period_between_transx` (45 rows, each customer's first transaction with no prior gap) filled with a large placeholder (9999) rather than dropped, to avoid losing already-scarce fraud examples
- Features scaled with `StandardScaler`, fit on the training set only and applied to the test set, to avoid data leakage and to prevent large-magnitude features like `amount_zar` from dominating over binary features like `is_midnight`
- Trained two versions: one with default settings, one with `class_weight='balanced'`, to compare how that setting shifts the precision/recall tradeoff

### Results (evaluated on the same held-out test set, 108 transactions, 11 actual fraud)

| | Rules | Logistic Regression (default) | Logistic Regression (balanced) |
|---|---|---|---|
| Recall | 45% (5/11) | 73% (8/11) | 91% (10/11) |
| Precision | 100% | 100% | 53% |
| False positives | 0 | 0 | 9 |

The default logistic regression model matched the rules' perfect precision while catching more fraud, likely because it learned precise numeric boundaries from the data rather than relying on hand-picked round-number thresholds (2x, 3x, 15 minutes). The balanced version pushed recall up further, but at real cost: 9 legitimate transactions incorrectly flagged.

**Which is "better" depends entirely on what happens after a flag.** If a flagged transaction gets blocked outright, false positives are expensive and the default (or rules-based) model is safer. If a flag just routes to a human analyst for review, the balanced model's extra recall may be worth the added review volume. This project doesn't define that downstream step, so I'm treating it as a deliberate open question rather than picking a winner.

### Coefficients

Across both model versions, `rapid_transx` and `is_out_of_province` came out as the two strongest predictors of fraud, independently confirming what I'd already found by hand: Rules 2 and 3 were consistently my best-performing manual rules. Two completely different approaches (manual reasoning and statistical fitting) converging on the same features is a strong signal that those two patterns are real, not coincidental.

The multicollinearity from including both raw and derived amount/time features showed up clearly here: `amount_over_avg` and `hour` both ended up with small, less interpretable coefficients, since their information overlapped with `amount_zar` and `is_midnight`. In the balanced model, `period_between_transx` (the raw, redundant version of `rapid_transx`) was pushed to a coefficient of exactly 0, the model effectively dropped it once forced to work harder on the minority class. A future iteration could drop the raw duplicates and keep only the derived features to get cleaner, more interpretable coefficients.

## What I learned this week

- **Working with datetime in pandas**: converting a raw timestamp column into proper datetime format, then using the `.dt` accessor to extract properties like the hour of a transaction.
- **`.diff()`** for calculating the time gap between a customer's consecutive transactions, one of the strongest fraud signals in this project.
- **Renaming columns** with `df.rename(columns={'old': 'new'})`, and the distinction between a Series and a DataFrame: a Series (like the output of a `.groupby()` aggregation) has a single `.name` attribute instead of column names, so it doesn't accept a `columns=` argument the way a DataFrame does.
- **Merging on index** with `left_index=True, right_index=True`: this tells pandas to align rows by their underlying row position rather than matching on a shared column, useful when merging a derived Series back onto its original DataFrame.
- **f-strings**: putting an `f` before the quotation marks tells Python to scan for `{ }` and substitute in the variable's value, made debugging and printing results far cleaner.
- **Why models can't handle missing data**: `.fit()` relies on algebraic formulas to adjust feature weights, and any calculation involving NaN returns NaN. If a prediction resolves to NaN, the model has no error to learn from, so scikit-learn throws a hard error rather than silently continuing. Missing values have to be filled or dropped before training, there's no way around it.
- **Why train/test splitting matters**: training and evaluating on the same data only tests whether the model memorized answers it already saw, not whether it generalizes. Splitting, and never letting the model see the test set during training, is what makes precision/recall numbers meaningful.
- **`class_weight='balanced'` is not a "better settings" switch, it's a scenario-based tradeoff.** The default (leaving it blank, treating every row equally) makes sense when classes are naturally balanced, or when the cost of a false alarm is high. There's no universally correct setting, it depends on what a false positive versus a false negative actually costs in the real system the model feeds into.
- Spent extra time this week also reinforcing Cisco Networking Academy's Data Science Essentials with Python material, particularly around data cleaning: `.info()`, `.astype()`, and string splitting.

## Limitations of rule-based flagging

- **Precision-recall tradeoff is manual and blunt.** Every threshold I chose (2x, 3x, 15 minutes) was a judgment call, not something learned from the data. Small changes to these thresholds shift results in ways that are easy to test but hard to optimise by hand.
- **Rules can't learn interactions between features.** A model could potentially learn that "medium amount + slightly unusual hour + new merchant category" together are suspicious, even if no single factor crosses a threshold on its own. My rules can only catch what I explicitly thought to check for.
- **Even after adding Rule 4, ~39% of fraud cases were still missed.** Some of these likely don't trip any single obvious signal, a normal amount, normal hour, home province, and no rapid repeats, which by definition, rule-based flagging can't catch.
- **Rules don't scale well to new fraud patterns.** If a new type of fraud emerged that didn't match any of the four patterns built into this dataset, none of my rules would catch it, since they were designed against known patterns rather than learned from the data itself.



## Tools

Python, pandas, scikit-learn
