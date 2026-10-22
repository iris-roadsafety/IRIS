# Preprocessing

Finds candidate events (such as sudden braking or swerving) in the raw GoPro sensor files, helps people label them, and tests different filter and threshold settings.

## What it does

The raw export has three separate CSV files (accelerometer, gyroscope, GPS) with different sample rates. The scripts:

1. Merge the files onto one time axis
2. Filter the accelerometer signal to keep the motion of evasive actions
3. Find candidate events with rolling z-scores
4. Let people check the events and save the checked labels (gold labels)
5. Test different filter and threshold settings

## Files

| Script | Step | Purpose |
|--------|------|---------|
| `config.py` | - | **Edit this first.** Paths, thresholds and column names |
| `pipeline_utils.py` | - | Shared code: bandpass filter, z-score detector, temporal IoU |
| `step0_merge_raw_streams.py` | 0 | Merge the accelerometer, gyroscope and GPS files into one CSV |
| `step1_build_candidates.py` | 1 | Find candidate events and write `candidate_events.csv` for review |
| `step2_build_gold.py` | 2 | After review, write the fixed `gold_labels.csv` |
| `step3_filter_ablation.py` | 3 | Test different bandpass filter settings |
| `step4_threshold_ablation.py` | 4 | Test different z-score thresholds and window sizes |
| `step5_hitl_ablation.py` | 5 | Test how much the reviewer's time offset changes the results |
| `step6_grid_search.py` | 6 | Grid search over filter and threshold settings together |
| `step7_event_category_thresholds.py` | 7 | Separate thresholds for each event type (braking, lane change, acceleration) |
| `check_data_quality.py` | - | Plot signals and flag bad segments |

## How to run

```bash
# 1. Put the raw files in a data folder and set IRIS_DATA_DIR (default: ./data),
#    then edit config.py if your file names are different
export IRIS_DATA_DIR=/path/to/data

# 2. Merge the raw streams
python step0_merge_raw_streams.py

# 3. Find candidates, then review candidate_events.csv and fill in the label column
python step1_build_candidates.py

# 4. Save the gold labels after review
python step2_build_gold.py

# 5. Run the tests
python step3_filter_ablation.py
python step4_threshold_ablation.py
python step5_hitl_ablation.py
```

## Method details

- **Bandpass filter:** Butterworth filter, 0.3 to 15 Hz by default, order 4
- **Jerk:** the first difference of the filtered acceleration
- **Detector:** an event needs a high value on at least 2 axes (jerk X, Y or Z) within 1 second
- **Temporal IoU:** used to match detected events with gold events (match threshold 0.5)

## Output files

| File | Description |
|------|-------------|
| `candidate_events.csv` | All detected events with first-guess labels, for review |
| `gold_labels_v1_FROZEN.csv` | Labels checked by people. Do not edit |
| `filter_ablation_results.csv` | Results of the filter tests |
| `threshold_ablation_results.csv` | Results of the threshold tests |
| `grid_search_all_results.csv` | Results of the grid search |
