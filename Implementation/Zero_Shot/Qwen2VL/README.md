# Zero-Shot: Qwen2-VL

Zero-shot inference with Qwen2-VL (2B is the default, 7B also works).

## Files

| File | Purpose |
|------|---------|
| `zero_shot_infer.py` | Main script. Runs the model on all candidate events of one rider |
| `scoring.py` | Scores predictions against gold labels (precision, recall, F1) |
| `prep_pilot_telemetry.py` | Prepares the sensor data used in the prompts |
| `extract_pilot_qualitative.py` | Pulls out model answers for manual review |
| `motor_zero_shot_infer.py` | Same idea for the other riders (GoPro sensor format) |
| `score_qwen.py` | Scores the other-rider predictions against their gold labels |

## How to run

```bash
# Prepare the sensor data
python prep_pilot_telemetry.py

# Run the model
python zero_shot_infer.py --rider R1 --model <model_name> \
    --condition video_only --model_path /path/to/Qwen2-VL-2B-Instruct

# Score the results
python scoring.py --pred <predictions_file> --gold <gold_labels_file> --out <output_file>
```

For the other riders, use `motor_zero_shot_infer.py` (options: `--model qwen2-2b` or `qwen25-3b`, `--condition`, `--model_path`) and set the rider name in the script.

## Sensor information

Choose it with `--condition`: `video_only`, `raw_telemetry`, `summarized_telemetry` or `telemetry_taxonomy`.
