# Zero-Shot: Florence-2

Object detection and scene description with Microsoft Florence-2 on frames from the candidate events.

## Files

| File | Purpose |
|------|---------|
| `run_florence2.py` | Cuts frames from the event clips and runs Florence-2 |
| `run_florence2.sh` | SLURM job script |

## How to run

```bash
# As a SLURM job
sbatch run_florence2.sh

# Or directly on a GPU node
python run_florence2.py
```

## Tasks used

- `<OD>`: object detection (vehicles, people, obstacles)
- `<CAPTION>`: short description of the scene
- `<GROUNDING_CAPTION>`: description with bounding boxes

## Output

A CSV file with the event id, the frame path, the detections and the caption.
