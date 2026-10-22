# Fine-Tuning: PaliGemma

Fine-tuning of PaliGemma-3B on the rider 1 events.

## Files

| File | Purpose |
|------|---------|
| `FineTune_PaliGemma.py` | Main fine-tuning script |
| `train_rider1.py` | Training on R1 (data paths and epochs) |
| `evaluate_rider1.py` | Test on the R1 held-out events |
| `train_riderX.py` | Optional: train on another rider's data |
| `evaluate_riderX.py` | Test the R1-trained model on another rider |

In the `riderX` scripts, set `RIDER_NAME` to `Rider2_AZ`, `Rider3_VA` or `Rider4_UC`.

## Notes

- PaliGemma uses the SigLIP vision encoder.
- It uses the `PaliGemmaForConditionalGeneration` class from `transformers` (PyTorch).
- It is smaller than InternVL3 and Qwen2-VL and runs faster.
