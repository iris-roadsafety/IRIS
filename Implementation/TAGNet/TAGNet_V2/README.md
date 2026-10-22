# TAGNet V2

- **Backbone:** Qwen2-VL-2B-Instruct
- **Adapters:** QLoRA (r=16, alpha=32, 4-bit nf4), TelemetryAdapter, ContextAdapter, **AMG** and **CGPA**
- **New compared with V1:** adaptive modality gate (AMG), clause-grounded pointer attention (CGPA) and retrieval of IRC clauses

## Design

```
Input:
  - Video frames           -> vision encoder   -> visual tokens
  - IMU sensor values      -> TelemetryAdapter -> telemetry token
  - GPS and context        -> ContextAdapter   -> context token
  - IRC clause embeddings  -> ClauseEncoder    -> clause tokens (found by retrieval)

Adaptive Modality Gate (AMG):
  Learns a weight g between 0 and 1 for each event:
    merged = g * telemetry_token + (1 - g) * visual_summary
  This stops the sensor data from dominating on events that are clear in the video, and the other way round.

Clause-Grounded Pointer Attention (CGPA):
  When the model says a rule applies (presence is "Yes" in the generated JSON),
  CGPA points to the clause that applies, among the retrieved IRC clauses.
  It is used only for positive events, because positives are rare (about 1 in 56).

Language model (QLoRA):
  [telemetry token | context token | AMG token | visual tokens | text tokens]
  -> JSON output with three parts:
     {"evasive_action": ..., "road_compliance": ..., "infrastructure": ...}
```

## Differences from V1

| | TAGNet V1 | TAGNet V2 |
|---|-----------|-----------|
| Backbone | Qwen2.5-VL-3B | Qwen2-VL-2B (smaller) |
| Joining the inputs | Fixed: tokens are added in front | **Adaptive modality gate** (learned) |
| Clause citation | From the prompt | **CGPA** pointer attention |
| Retrieval | ChromaDB text search | ChromaDB text search plus sign and plate image search |
| Output | Free text | JSON with three parts |
| Extra metrics | HCAS, CHR, IDAS | Also CRRS, RHR, CIDEr |

## Files

| File | Purpose |
|------|---------|
| `tagnet_v2_final.py` | **Main model file.** AMG, CGPA, the clause encoder and the HCAS metric |
| `train_tagnet_v2.py` | Training with a presence-gated pointer loss and oversampling |
| `build_v2_dataset.py` | Builds `v2_dataset.json` from the gold labels and retrieved clauses |
| `draw_tagnet_v2_architecture.py` | Draws the architecture diagram |

## Training

```bash
export IRIS_SCRATCH=/path/to/scratch
export IRIS_HOME=/path/to/home

# Build the dataset (the ChromaDB clause index must exist first)
python build_v2_dataset.py

# Quick test first
python train_tagnet_v2.py --smoke_test --adapter_mode embed

# Full training
python train_tagnet_v2.py --data v2_dataset.json --epochs 8 --adapter_mode embed
```

CGPA is trained with presence gating. The pointer loss is computed only on rows where the gold JSON says `"presence": "Yes"`. This way CGPA only has to learn which clause applies. The yes/no decision is made by the language model.
