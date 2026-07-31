# British Airways - Data Science Virtual Experience

A completed submission for the British Airways Data Science job simulation on Forage. The programme is framed around two problems the BA analytics team actually works on: forecasting lounge demand from a flying schedule, and predicting which enquiring customers will complete a booking. Both tasks are self-contained but share a common thread,  turning raw operational data into something the business can plan against.

---

## Task 1: Lounge Eligibility Modelling at Heathrow Terminal 3

**Business problem.** BA needs a way to estimate how many passengers will be eligible to use each lounge at Heathrow T3 for any given flight schedule. Eligibility depends on the passenger's cabin class and Executive Club / oneworld status, but Airport Planning doesn't have per-passenger data. hey have route and time information at the flight level. The task is to design a lookup structure that lets them go from a flying schedule to expected lounge headcount without needing individual booking records.

**Approach.**
- Defined three eligibility tiers (Tier 1 = First / Club World cabin; Tier 2 = Club Europe or Gold status; Tier 3 = Silver or oneworld Sapphire+)
- Grouped flights along two axes - **haul type** (Short / Long) and **time of day** (Morning / Lunchtime / Afternoon / Evening), producing 8 flight groupings, each with representative destinations
- Estimated the Tier 1 / 2 / 3 percentage split for each grouping based on the assumption that cabin allocation and loyalty-tier composition vary by route type and departure time
- Wrote a four-question justification covering the grouping method, why it makes sense for demand modelling, the assumptions taken, and how the model extends to schedule changes


**Key design choice.** Grouping by route type rather than by specific flight or destination means the same table works for any future schedule. New routes get slotted into an existing group by matching their haul length and departure window, so Airport Planning never has to rebuild the model when the schedule changes.

---

## Task 2: Predictive Modelling of Customer Bookings

**Business problem.** The customer buying cycle has shifted: by the time a customer arrives at the airport, the purchase decision is long made. BA wants to identify high-intent customers earlier in the funnel. At the enquiry stage so marketing and offer targeting can be proactive rather than reactive. The task is to build a classifier that predicts whether a customer will complete a booking, and to explain which variables drive that prediction.

**Dataset.** 50,000 customer enquiry records with 14 columns covering trip details (passengers, route, dates, duration), sales context (channel, trip type, booking origin), ancillary preferences (baggage, seat, meals), and the binary target `booking_complete`. The class balance is roughly 85% non-bookers to 15% bookers.

**Approach.**

1. **EDA**: verified no missing values, mapped column semantics, flagged the 85/15 class imbalance and the high-cardinality problem in `route` (799 unique values) and `booking_origin` (104)
2. **Feature engineering**:
   - Frequency-encoded `route` and `booking_origin` to collapse hundreds of sparse categoricals into single dense numeric features
   - One-hot encoded the low-cardinality categoricals (`sales_channel`, `trip_type`, `flight_day`) to keep individual categories interpretable
   - Created `ancillary_score` (sum of the three "wants" flags) as a stronger proxy for purchase intent than any single flag
   - Bucketed `purchase_lead` into last-minute / short / medium / long lead-time bands to capture non-linear booking behaviour
3. **Modelling**: trained a RandomForest classifier with `class_weight='balanced'` and stratified 5-fold cross-validation; benchmarked against Logistic Regression, Gradient Boosting, and XGBoost to check whether the model choice or the data was the binding constraint
4. **Diagnostics**: measured train-test AUC gap to quantify overfitting, and re-fit on the top 8 features only to test whether a pruned model matched full-feature performance
5. **Interpretation**: ranked feature importances and grouped them into thematic drivers (geography, timing, trip characteristics) for the business summary

**Results.**

| Model | Test AUC | Overfit Gap |
|---|---|---|
| XGBoost | 0.779 | 0.139 |
| Random Forest (full) | 0.769 | 0.231 |
| **Random Forest (top-8 pruned)** | **0.774** | — |
| Gradient Boosting | 0.773 | 0.002 |
| Logistic Regression | 0.681 | 0.001 |

The four tree-based models cluster within 0.01 AUC of each other, indicating the ceiling is set by the input data rather than the algorithm. The pruned RandomForest is the final model, comparable test AUC to the full-feature version, simpler, and faster to serve. The top drivers were `booking_origin_freq`, `purchase_lead`, `route_freq`, `length_of_stay`, `flight_hour`, `flight_duration`, `num_passengers`, and `ancillary_score`, dominated by geography and timing signals.

**Business finding.** Booking intent is driven by *who* the customer is and *when/where* they're booking, not by which channel or day of week. To meaningfully lift model performance beyond ~0.77 AUC, BA would need to capture upstream behavioural signals, session duration, browsing history, price interactions, which are not present in the current enquiry dataset.

**Deliverables.**
- `BA_Task2_Predictive_Modeling.ipynb` — the full analysis notebook, from EDA through pruning
- `BA_Task2_Summary.pptx` — single-slide summary for the manager, styled in BA's brand palette

---

## Repository Contents

| File | Task | Description |
|---|---|---|
| `Lounge_Eligibility_Lookup_Template_-_Task_1.xlsx` | 1 | Completed lookup table + written justification |
| `BA_Task2_Predictive_Modeling.ipynb` | 2 | End-to-end EDA, feature engineering, model comparison, and interpretation |
| `BA_Task2_Summary.pptx` | 2 | Single-slide business summary in BA brand style |
| `customer_booking.csv` | 2 | Source dataset (50,000 enquiry records) |
| `README.md` | — | This file |

## Tech Stack

Python 3.12 · pandas · numpy · scikit-learn (RandomForestClassifier, GradientBoostingClassifier, LogisticRegression, StratifiedKFold, StandardScaler) · XGBoost · matplotlib · seaborn · openpyxl · python-pptx

**Author.** Basim Shabir — Master of Data Science, Monash University
**Programme.** British Airways Data Science Virtual Experience (Forage)
