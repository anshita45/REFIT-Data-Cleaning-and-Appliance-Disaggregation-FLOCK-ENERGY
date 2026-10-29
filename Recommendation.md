# Recommendation: Washing-Machine Disaggregation for Utility Use


## 1. What the Model Does

The model estimates washing-machine power consumption at one-minute resolution using only the aggregate household electricity signal. It uses a `HistGradientBoostingRegressor` and is evaluated through leave-one-household-out validation, comparing performance with an always-off baseline and an hourly-profile baseline. Its purpose is to investigate appliance-level energy insights without requiring dedicated washer meters in every home.

## 2. Suitable Uses

Subject to the observed evaluation results, potential applications include:

- **Household usage insights:** estimating when washing machines are typically used and identifying broad usage patterns.
- **Portfolio-level energy analysis:** estimating washing-machine energy consumption across groups of households over several weeks.
- **Demand flexibility programmes:** identifying evening washing activity and targeting customers with information about shifting flexible loads outside peak periods.

These applications should be treated as estimates, with reliability assessed using the reported household-level and cycle-level metrics.

## 3. Limits of the Model

The model should not be used for individual customer billing, compliance enforcement, or other high-stakes decisions. Individual cycle-energy estimates may be unreliable where the evaluation shows large errors. Performance may also deteriorate for households, appliance types, or operating conditions that differ from the REFIT training population. Its current use of future readings also makes it suitable for offline analysis rather than strictly real-time prediction.

## 4. Practical Opportunity: Reduce Evening-Peak Washing

A utility could test a voluntary time-of-use nudge encouraging customers who frequently wash during the evening peak to use delayed-start settings or move washing to lower-demand periods. The programme could offer a reminder or a small incentive for shifting flexible consumption.

The analysis should first establish what proportion of washing energy occurs during the defined evening-peak window, using the household-level results in `RESULTS.md`. The opportunity is most promising if a meaningful share of washing energy occurs during that window; the size of the opportunity must be determined from the actual results.

**How to measure success:** run a randomised controlled trial with comparable households assigned to a nudge group or a control group. Compare the change in the proportion of washing energy consumed during peak hours between the two groups, while also measuring total washing energy. A successful intervention would produce a statistically credible reduction in peak-period washing without an unwanted increase in total energy consumption. Validate the disaggregation estimates against washer sub-meter readings from a sample of participating homes to distinguish real behavioural changes from model error.

## 5. Key Limitations

- **Overlapping appliance signatures:** simultaneous operation of a washer, kettle, dishwasher, or other high-power appliance can make their individual contributions difficult to distinguish from aggregate consumption.
- **Missing data:** long outages remain missing and provide no reliable estimate. Cycles affected by outages may be excluded, reducing the evidence available for evaluation.
- **Household differences:** appliance models, wash programmes, user habits, and background loads vary. Leave-one-household-out results indicate how performance changes across unseen homes.
- **UK-to-India transfer:** the model was developed using UK household data. Differences in washing-machine designs, heating behaviour, supply conditions, power interruptions, backup systems, and appliance usage may change the observed power signatures. The model should not be deployed directly in Indian homes without local data, retraining where necessary, and validation against washer sub-meters.

## 6. What Changes with 15- or 30-Minute Data?

At 15- or 30-minute resolution, individual washing cycles become harder to detect because each reading averages together different phases of operation, potentially hiding short heating peaks and other identifying patterns. Cycle start and end times become less precise, and washer consumption is harder to distinguish from overlapping appliances. Aggregate energy estimates and broad time-of-day usage patterns may remain useful, but individual-cycle detection and cycle-level energy estimates would require separate validation. The resolution study in `RESULTS.md` should be used to identify the point at which the model no longer improves on the hourly-profile baseline.

## Overall Recommendation

Use the current model as an exploratory tool for aggregate household energy insights and carefully evaluated demand-flexibility pilots—not as a definitive appliance meter. Before operational deployment, validate its energy estimates, cycle detection, and performance across diverse households, and assess whether the results remain useful at the smart-meter resolution available to the utility.

