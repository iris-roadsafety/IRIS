# Fine-Tuning: LLaVA

Fine-tuning of LLaVA on the rider 1 events.

## Files

| File | Purpose |
|------|---------|
| `FineTune.py` | Fine-tuning script |
| `Evaluate.py` | Test on the R1 events |

## Notes

- LLaVA uses a CLIP vision encoder with a LLaMA-based language model.
- It is trained in a question-and-answer format that matches the evasive-action prompts.
- It did not do as well as InternVL3 or Qwen2-VL on this task.
