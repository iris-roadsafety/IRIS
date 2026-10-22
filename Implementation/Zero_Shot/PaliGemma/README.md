# Zero-Shot: PaliGemma

Zero-shot inference with PaliGemma-3B and PaliGemma2-3B.

## Files

| File | Purpose |
|------|---------|
| `paligemma_full_run.py` | Full run with PaliGemma-3B |
| `paligemma2_full_run.py` | Full run with PaliGemma2-3B |
| `paligemma_control_test.py` | Quick test that the model loads and answers |
| `paligemma2_control_test.py` | The same quick test for PaliGemma2 on the other riders |

## How to run

```bash
python paligemma_full_run.py --condition video_only
python paligemma2_full_run.py --condition video_only
```

## Notes

- PaliGemma prepares images in a different way from Qwen2-VL and InternVL.
- Results are saved as JSON files named after the model and the condition.
