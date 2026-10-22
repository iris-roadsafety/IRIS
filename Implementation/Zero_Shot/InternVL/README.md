# Zero-Shot: InternVL

Zero-shot inference with InternVL2-8B and InternVL3-8B. The model is loaded with `trust_remote_code=True`.

## Files

| File | Purpose |
|------|---------|
| `InternVL_zeroshot.py` | Zero-shot inference with the different sensor-information options |
| `ablation.py` | Runs all four sensor-information options across the events |
| `run_server_all_models.py` | Runs all the models one after another on a GPU node |

## How to run

```bash
export IRIS_SCRATCH=/path/to/scratch
export IRIS_HOME=/path/to/home
python InternVL_zeroshot.py
```

The model files are expected under `$IRIS_SCRATCH` (for example `InternVL2-8B`). If they are not there, the script downloads them from Hugging Face.
