# Metrics

Scripts that measure how well the models work. `run_all_metrics.sh` (V1) and `run_all_v2_metrics.sh` (V2) run the whole chain on a GPU node.

## Order of the scripts

1. `build_kb_and_mapping.py`: builds the clause lookup files
2. `run_inference.py` (V1) or `run_inference_v1_embed.py`: runs the model on the held-out events
3. `compute_standard_metrics.py`: accuracy, macro-F1, BLEU, ROUGE-L, CIDEr
4. `compute_chr.py`: citation hallucination rate
5. `compute_rhr.py`: reasoning hallucination rate
6. `compute_haa.py`: hallucination-adjusted accuracy
7. `compute_icdi.py`, `compute_idas.py`, `compute_crrs.py`: infrastructure and route risk scores
8. `compute_precision.py`: precision at k
9. `compute_hcas.py`: hierarchical clause alignment score (V2)

Other scripts: `cider_scorer.py` (CIDEr code), `measure_latency.py` (speed test), `rebuild_csvs_from_predictions.py` and `rebuild_csvs_from_v2_predictions.py` (turn a prediction file into the CSV files the metric scripts need), `config.py` (paths).

## What each metric means

**Standard metrics** (`compute_standard_metrics.py`). Computed for two parts of the answer: `road_surface` and `infrastructure`. Only answers whose JSON was read correctly are counted. Failed answers are reported separately.
- Accuracy: how often the predicted `presence` (Yes, No or Unclear) matches the gold label.
- Macro-F1: the average F1 over the three classes. It shows whether the model handles the rare classes, because most events are "No".
- BLEU, ROUGE-L and CIDEr: how close the generated reasoning text is to the gold reasoning text.

**Citation hallucination rate, CHR** (`compute_chr.py`). For every non-compliant verdict that cites a clause: was that clause among the clauses retrieved for that event? CHR is the share of citations that were not. A lower value is better.

**Reasoning hallucination rate, RHR** (`compute_rhr.py`). The text of the cited clause is the premise and the model's reasoning is the hypothesis. A natural language inference model (`cross-encoder/nli-deberta-v3-large`) labels the pair as entailment, neutral or contradiction. RHR = (neutral + contradiction) / total. A high value means the reasoning does not follow from the clause it cites.

**Hallucination-adjusted accuracy, HAA** (`compute_haa.py`). HAA = Accuracy x (1 - RHR).

**ICDI** (`compute_icdi.py`). For each row: `absence_confidence x severity x zone_risk x retrieval_score`. The rows of one event are averaged. Zone risk weights: School 1.5, Pedestrian Crossing 1.4, Residential 1.2, Highway 1.0, Unknown 1.0.

**IDAS** (`compute_idas.py`). The event's mean `absence_confidence` is put in a group: Absent (0.66 or more, weight 1.0), Unclear (0.33 to 0.66, weight 0.5) or Present (below 0.33, weight 0.1). `idas_score = ICDI x weight`. IDAS is the average over all events.

**CRRS** (`compute_crrs.py`). Events are sorted by start time and grouped. A new group starts when the gap to the previous event is more than `--gap` seconds (default 30), so one hazard that triggers several nearby events is not counted again and again. `frequency_weight` is the size of the group. `CRRS = sum(ICDI x frequency_weight) / number of events`. The grouping rule is our own choice. Change `--gap` or the weights if you want a different definition.

**Precision@k** (`compute_precision.py`). For each query: (relevant clauses in the top k) / k. It needs the `human_relevant` column in `retrieval_results.csv` to be filled in by hand. Rows that were not annotated are skipped.

**HCAS** (`compute_hcas.py`). Scores each cited clause on three levels: ILA (right IRC document), SLA (right section) and CLA (right clause). HCAS = (ILA + SLA + CLA) / 3, averaged over all citations.

**Latency** (`measure_latency.py`). Time per sample, tokens per second and peak GPU memory on a trained checkpoint, using greedy decoding.
