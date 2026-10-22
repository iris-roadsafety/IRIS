# Fine-Tuning

QLoRA fine-tuning of vision-language models on the labelled events. Models are trained on the gold labels of rider 1 (R1) and then tested on riders 2, 3 and 4 to see how well they work on new riders.

## Folders

| Folder | Model |
|--------|-------|
| [`Qwen2VL`](Qwen2VL/) | Qwen2-VL-7B |
| [`InternVL2`](InternVL2/) | InternVL2-8B |
| [`InternVL3`](InternVL3/) | InternVL3-8B (best overall) |
| [`PaliGemma`](PaliGemma/) | PaliGemma-3B |
| [`LLaVA`](LLaVA/) | LLaVA |
| [`DeepSeek`](DeepSeek/) | DeepSeek-VL |

## Training settings

All models use the same LoRA settings so the results can be compared:

| Setting | Value |
|---------|-------|
| LoRA rank | 16 |
| LoRA alpha | 32 |
| LoRA dropout | 0.05 |
| Trained layers | attention and MLP layers (vision part is frozen) |
| Epochs | 5 to 7 |
| Batch size | 4 (gradient accumulation 4, so 16 in effect) |
| Learning rate | 2e-4 |

## Rider names in the scripts

- `rider1` or `Rider1_NJ` means R1, the training rider.
- `riderX` means any other rider (R2, R3 or R4). In these scripts, set `RIDER_NAME` to `Rider2_AZ`, `Rider3_VA` or `Rider4_UC`.

## Typical steps

```bash
# 1. Train on R1
python Qwen2VL/qwen2_rider1_train.py

# 2. Test on R1
python Qwen2VL/evaluate_rider1.py

# 3. Test on another rider (set RIDER_NAME in the script first)
python Qwen2VL/evaluate_riderX.py
```
