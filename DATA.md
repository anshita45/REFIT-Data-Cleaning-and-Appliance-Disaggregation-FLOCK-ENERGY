# DATA: Dataset Coverage, Quality Assessment, and Cleaning

## 1. Dataset Overview

This project uses the REFIT Electrical Load Measurements dataset provided by the University of Strathclyde. The dataset contains whole-house electricity measurements and appliance-level readings collected from UK households over an extended monitoring period.

| Property | Description |
|---|---|
| Dataset | REFIT Electrical Load Measurements |
| Provider | University of Strathclyde |
| Geographic scope | United Kingdom |
| Measurements | Whole-house aggregate and individual appliance power |
| Power units | Watts (W) |
| Nominal sampling interval | Approximately 8 seconds in the raw dataset |
| Raw-data inspection | House 1 |
| Cleaned-data analysis | Eligible households listed in the availability table |
| Analysis resolution | 1 minute |

The raw and cleaned REFIT files serve different purposes in this project. Raw House 1 data is used to examine data quality and demonstrate the cleaning pipeline. The official cleaned files are used for washing-machine exploratory data analysis and model evaluation across eligible households.

**Data provenance and coverage:** The dataset contains approximately 20 households monitored over roughly two years. The exact first and last timestamps, per-household coverage, and usable analysis periods must be verified from the downloaded files and generated availability tables.

**Official dataset resources:**

- [Raw REFIT dataset](https://pureportal.strath.ac.uk/files/62090177/REFIT_RAW_081116.7z)
- [Cleaned REFIT dataset](https://pureportal.strath.ac.uk/files/62090184/CLEAN_REFIT_081116.7z)
- [Dataset readme](https://pureportal.strath.ac.uk/ws/portalfiles/portal/62090183/CLEAN_READ_ME_081116.txt)
- [Official dataset page](https://pureportal.strath.ac.uk/en/datasets/refit-electrical-loadmeasurements/)

## 2. Section 1: Raw House 1 Data Quality Assessment

The raw House 1 dataset is inspected before cleaning to identify timestamp problems, missing readings, duplicate timestamps, irregular sampling intervals, invalid power values, and inconsistencies between aggregate and appliance measurements.

### 2.1 Overall Quality Checks

<!-- TABLE:raw_quality -->
<!-- /TABLE -->

The summary table reports the quality indicators calculated by the pipeline. These measurements provide a baseline for evaluating the effect of preprocessing and identifying data limitations that may influence subsequent analysis.

### 2.2 Channel-Level Quality Checks

The following table presents quality checks for individual channels, including missing readings, negative values, extreme values, and instances where appliance power exceeds aggregate power.

<!-- TABLE:raw_channels -->
<!-- /TABLE -->

### 2.3 Interpretation of Raw-Data Findings

Use the generated tables above to document the findings from the actual run:

- **Timestamp validity and ordering:** Report whether timestamps could be parsed successfully and whether readings were chronologically ordered before preprocessing.
- **Duplicate timestamps:** State how many duplicate timestamps were identified and whether repeated observations showed conflicting power values.
- **Sampling irregularity:** Describe the observed interval distribution and the extent to which the readings departed from the nominal sampling interval.
- **Missing readings and gaps:** Summarize missing observations and any prolonged gaps that could affect daily usage estimates or washing-cycle detection.
- **Extreme or invalid values:** Report the number of negative readings and values exceeding the configured power limits.
- **Aggregate–appliance inconsistencies:** Describe how often individual appliance readings exceeded the aggregate and explain that timing differences or sensor calibration can contribute to such inconsistencies.

Only include findings supported by the generated quality tables. These checks distinguish actual data-quality problems from normal variability in household electricity demand.

## 3. Cleaning and Preprocessing Pipeline

The raw House 1 data is processed through the following stages.

### Step 1: Timestamp parsing

Timestamps are parsed into a consistent datetime representation. Rows with invalid timestamps are dropped and counted so that their effect on data quality remains visible.

### Step 2: Invalid-value handling

Negative power readings and raw values above the configured 25 kW limit are marked as missing rather than clipped. This avoids treating implausible values as genuine measurements while preserving the distinction between observed and missing data.

The limit is a project-level quality-control rule and should be reviewed if the dataset, electrical supply conditions, or study population changes.

### Step 3: Duplicate-timestamp handling

Readings sharing the same timestamp are aggregated using their mean. This provides a consistent value for each timestamp without arbitrarily selecting one of several observations.

### Step 4: Chronological ordering

Observations are sorted by timestamp before resampling, ensuring that subsequent temporal processing uses the correct order.

### Step 5: One-minute resampling

Readings are resampled to one-minute mean power. Minutes without valid observations remain missing unless they qualify for the short-gap handling rule.

This produces a consistent temporal resolution for washing-machine analysis and model evaluation.

### Step 6: Short-gap interpolation

Short internal gaps are filled by linear interpolation according to the configured maximum gap length. Interpolated values are estimates rather than real meter readings and should be treated accordingly during interpretation.

The exact threshold must match `MAX_FILL_GAP_MIN` in `src/config.py`. It should not be described as five minutes unless that is the value used in the current implementation.

### Step 7: Long-gap and outage handling

Longer gaps remain missing and are flagged as outages. They are not filled with artificial values, since doing so could conceal prolonged periods of unavailable measurements or create misleading appliance patterns.

Affected observations are excluded from model training and scoring where required by the pipeline.

### 3.1 Before-and-After Comparison

![Raw House 1 data before and after cleaning](results/figures/01_raw_before_after.png)

The figure compares selected power channels before and after preprocessing. Use it to verify whether implausible spikes have been masked, duplicate and irregular readings have been consolidated, and prolonged gaps remain visible rather than being artificially filled.

The plot should be interpreted alongside the raw quality tables because a visual comparison alone cannot establish the number of corrected observations or the proportion of data affected.

## 4. Sections 2–3: Cleaned REFIT Data

The washing-machine EDA and disaggregation experiments use the official cleaned REFIT files for the eligible households. The data is then processed to the project's one-minute resolution, with the configured short-gap handling applied and long outages preserved as missing.

Rows marked with `Issues = 1` in the official cleaned files are masked when `MASK_ISSUES` is enabled.

### 4.1 Household and Washer-Channel Availability

<!-- TABLE:washer_availability -->
<!-- /TABLE -->

This table records the usable observation periods for the selected washing-machine channels. Differences in coverage affect the amount of evidence available for detecting cycles, estimating energy, and evaluating performance across households.

The reported date ranges and coverage statistics should be used to identify the actual analysis period for each household rather than assuming that every household has a complete two-year record.

### 4.2 Excluded Households

<!-- TABLE:excluded_houses -->
<!-- /TABLE -->

Households that fail the washing-machine channel sanity checks are excluded from the relevant analysis. These checks use configured criteria, including peak-power and cycle-count requirements, to identify channels that may not represent plausible washing-machine behaviour.

House 4 is excluded from the current study because it contains two washing machines, whereas the analysis assumes a single target washing-machine channel per household.

Exclusion decisions reduce the risk of evaluating the model against unsuitable target channels, but they also limit the population to which the findings can be generalized.

## 5. Remaining Data-Quality Concerns

Despite preprocessing, several sources of uncertainty remain.

- **Timestamp time zone:** The pipeline assumes timestamps represent UTC/GMT and converts them to `Europe/London` for time-of-day analysis. An incorrect assumption can shift usage patterns, particularly during British Summer Time.
- **Appliance-channel mapping:** Channel assignments are based on the dataset documentation and checked using simple power and cycle criteria. These checks do not guarantee that every channel is correctly identified.
- **Interpolated observations:** Short-gap interpolation introduces estimated values rather than measured consumption. Its influence should be assessed using the reported gap statistics.
- **Extended outages:** Missing periods reduce the amount of valid observation time. Cycle frequency and energy summaries therefore describe usable periods rather than necessarily representing the complete calendar period.
- **Aggregate–sub-meter mismatch:** Appliance power may occasionally exceed aggregate power because of sensor timing differences, calibration differences, or measurement noise.
- **Household representativeness:** REFIT consists of monitored UK households and does not necessarily represent other household populations, appliances, electricity systems, or usage habits.
- **Cycle-boundary uncertainty:** A washing cycle intersecting a missing-data gap may be incomplete. Such cycles should not be interpreted as fully observed events.
- **Resolution and feature limitations:** One-minute resampling and the handling of missing values affect the temporal information available to downstream models.

## 6. Reproducibility and Data Handling

The cleaning and analysis pipeline is controlled through `src/config.py`. Important settings include the resampling interval, maximum fillable gap, invalid-value limits, issue masking, time-zone assumptions, and washing-machine channel mappings.

To reproduce the analysis, install the dependencies listed in `requirements.txt`, obtain the REFIT data from the official source, and follow the execution instructions in `README.md`.

The raw and cleaned dataset archives are not included in this repository. Users should obtain the data from the provider and follow the applicable access, attribution, and redistribution terms.

## Summary

The preprocessing pipeline establishes a consistent one-minute representation of household electricity demand while making data-quality problems explicit. Invalid values are masked, duplicate timestamps are aggregated, short gaps are handled according to a configured rule, and extended outages remain missing.

These decisions support reproducible washing-machine EDA and model evaluation, but they do not eliminate uncertainty caused by missing observations, sensor inconsistencies, channel-mapping errors, and differences between households. The generated quality and availability tables should therefore be considered part of the evidence required to interpret the results responsibly.
