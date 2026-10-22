# Fine-Tuning: Qwen2-VL

QLoRA fine-tuning of Qwen2-VL-7B-Instruct (4-bit nf4) on the rider 1 events.

## Files

| File | Purpose |
|------|---------|
| `qwen2_rider1_train.py` | Training on R1, 5 epochs |
| `qwen2_rider1_train_7ep.py` | Same, 7 epochs |
| `evaluate_rider1.py` | Test on the R1 held-out events |
| `evaluate_rider1_7ep.py` | Test of the 7-epoch checkpoint |
| `qwen2_riderX_train.py` | Optional: train on another rider's data for comparison |
| `evaluate_riderX.py` | Test the R1-trained model on another rider |

In the `riderX` scripts, set `RIDER_NAME` to `Rider2_AZ`, `Rider3_VA` or `Rider4_UC`.

## How to run

```bash
python qwen2_rider1_train.py 2>&1 | tee train_r1.log
python evaluate_rider1.py
```

## Model

- Qwen2-VL-7B-Instruct, 4-bit nf4 with BitsAndBytes
- Reads video frames with `qwen_vl_utils.process_vision_info`
- LoRA on `q_proj`, `k_proj`, `v_proj`, `o_proj`, `gate_proj`, `up_proj` and `down_proj`
