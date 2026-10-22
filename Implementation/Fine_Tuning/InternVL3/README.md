# Fine-Tuning: InternVL3

QLoRA fine-tuning of InternVL3-8B on the rider 1 events. This is the best model overall.

## Files

| File | Purpose |
|------|---------|
| `train.py` | Training with InternVL3-8B and QLoRA |
| `eval.py` | Test on R1, and optionally on the other riders |
| `dataset.py` | `PilotEventDataset`: loads the clips and gold labels |

The same `train.py` and `eval.py` are used for the other riders. Set `RIDER_NAME` (for example `Rider2_AZ`) in the scripts.

## How to run

```bash
python train.py 2>&1 | tee internvl3_r1_train.log
python eval.py
```

## Notes on the model

InternVL3 uses its own image tokenizer, so:
1. Load it with `trust_remote_code=True`.
2. Set `img_context_token_id` by hand after loading.
3. Normalise pixel values with ImageNet mean and standard deviation.
4. Keep the vision tower out of LoRA (a regex filter does this).
