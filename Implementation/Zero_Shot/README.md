# Zero-Shot

Vision-language models are asked about each event without any training. They get video frames and, sometimes, sensor information.

## Folders

| Folder | Model | Notes |
|--------|-------|-------|
| [`Qwen2VL`](Qwen2VL/) | Qwen2-VL 2B and 7B | Also has the scripts for scoring and for the other riders |
| [`InternVL`](InternVL/) | InternVL2 and InternVL3 (8B) | Includes the sensor-information tests |
| [`PaliGemma`](PaliGemma/) | PaliGemma 3B and PaliGemma2 3B | |
| [`Florence2`](Florence2/) | Florence-2 | Object detection only |
| [`GroundingDINO`](GroundingDINO/) | Grounding DINO | Detection from text prompts |

## Models

| Model | Size | Notes |
|-------|------|-------|
| Qwen2-VL | 2B, 7B | 2B is the fast baseline |
| InternVL2 | 8B | Good at visual question answering |
| InternVL3 | 8B | Best zero-shot results |
| PaliGemma, PaliGemma2 | 3B | Small and fast |
| Florence-2 | 0.7B | Object detection |
| Grounding DINO | - | Object detection from text prompts |

## Ways of giving sensor information

1. **Video only:** frames and the question, no sensor data
2. **Raw telemetry:** raw ax, ay, az values added to the prompt
3. **Summarized telemetry:** a short text summary of the sensor signals
4. **Telemetry taxonomy:** a structured JSON description of the event

## Other riders

Scripts whose names end in `_riderX` or start with `motor_` use the data format of the other riders. They are shared by riders 2, 3 and 4. Set the rider name in the script (for example `RIDER_NAME = "Rider2_AZ"`) or pass it as an option. The same model files are used for every rider, with no retraining.

## Example

```bash
python Qwen2VL/zero_shot_infer.py --rider R1 --model <model_name> \
    --condition video_only --model_path /path/to/model
python InternVL/InternVL_zeroshot.py
```

See each folder for details.
