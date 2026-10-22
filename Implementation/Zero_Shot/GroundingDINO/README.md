# Zero-Shot: Grounding DINO

Object detection from text prompts with Grounding DINO on frames from the candidate events.

## Files

| File | Purpose |
|------|---------|
| `run_grounding_dino.py` | Runs Grounding DINO on the event frames with road-related text prompts |
| `run_grounding_dino.sh` | SLURM job script |

## How to run

```bash
sbatch run_grounding_dino.sh

# Or directly
python run_grounding_dino.py
```

## Text prompt

```
vehicle . car . truck . motorcycle . pedestrian . pothole . obstacle
```

## Output

A CSV file with the event id, the frame path, the detected objects, the boxes and the confidence scores.
