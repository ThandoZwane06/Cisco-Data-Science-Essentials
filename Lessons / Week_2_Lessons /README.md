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

## Next steps

- Compare this rule-based approach against a simple ML model (Isolation Forest or logistic regression) to see whether it can catch fraud patterns that rules structurally can't.
- Investigate the remaining false negatives to see if there's a shared, catchable pattern among them.

## Tools

Python, pandas
