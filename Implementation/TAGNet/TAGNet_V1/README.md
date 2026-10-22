# TAGNet V1

- **Backbone:** Qwen2.5-VL-3B-Instruct
- **Adapters:** QLoRA (r=16, alpha=32, dropout=0.05), TelemetryAdapter and ContextAdapter
- **Adapter mode:** `embed`. The sensor and context tokens are added in front of the input embeddings.

## Files

| File | Purpose |
|------|---------|
| `rider_pipeline.py` | **Main inference script.** Modes: ZS (zero-shot), FT (fine-tuned) and V1 |
| `build_rider_datasets.py` | Builds the event dataset file for each rider |
| `rider_eval_full.py` | Full test: loads a checkpoint, runs all events and writes JSONL results |
| `rider_metrics.py` | Computes the metrics from the JSONL results |
| `rider_nlp_metrics.py` | Computes text metrics (BLEU, ROUGE, BERTScore) on the generated text |
| `motor_qwen25_train.py` | QLoRA training of Qwen2.5-VL-3B on the GoPro-format data |
| `motor_qwen25_eval.py` | Test after the Qwen2.5 fine-tuning |

## Folders used by these scripts

```bash
export IRIS_DATA_ROOT=/path/to/riders     # one sub-folder per rider (Rider1_NJ, Rider2_AZ, ...)
export IRIS_OUTPUT_ROOT=/path/to/outputs   # for build_rider_datasets.py
export IRIS_RESULTS_ROOT=/path/to/results  # for rider_metrics.py
```

## Running inference

```bash
python rider_pipeline.py --rider R1 --model V1

# All riders
for rider in R1 R2 R3 R4; do
    python rider_pipeline.py --rider $rider --model V1
done
```

## Training (Qwen2.5-VL-3B)

```bash
PYTHONUNBUFFERED=1 CUDA_VISIBLE_DEVICES=0 python motor_qwen25_train.py 2>&1 | tee motor_qwen25_train.log
```

## Settings

| Setting | Value |
|---------|-------|
| LoRA rank | 16 |
| LoRA alpha | 32 |
| LoRA dropout | 0.05 |
| Trained layers | attention and MLP of the language model (vision part frozen) |
| Precision | bf16 (no 4-bit; the 3B model needs about 7 GB) |
| Epochs | 4 for the final checkpoint |

## Adapters

```python
# TelemetryAdapter: 2-layer MLP, 6 sensor values -> hidden -> model embedding size
TelemetryAdapter(in_dim=6, hidden_dim=256, out_dim=model.embed_dim)

# ContextAdapter: GPS context (speed in km/h, zone, latitude, longitude)
ContextAdapter(in_dim=6, hidden_dim=128, out_dim=model.embed_dim)
```
