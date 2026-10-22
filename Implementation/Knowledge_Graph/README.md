# Knowledge Graph and IRC Compliance Check

Checks dashcam and two-wheeler video frames against two IRC documents:

- **IRC:35-2015** (road markings)
- **IRC:67-2022** (road signs)

Each document is kept as its own knowledge base. The code flags frames that break a rule, names the clause that applies, and builds a knowledge graph of the whole run. It also writes the files needed to compute the retrieval and compliance metrics (Precision@k, ICDI, CRRS, IDAS and CHR).

## What is a trained model and what is a simple rule

| Part | How it works |
|------|--------------|
| Clause extraction | `pdfplumber` reads the PDFs and splits them by clause number (for example `4.6`, `11.6.2`). Only the chosen sections are kept |
| Clause citation | TF-IDF search inside the knowledge base of the matching document |
| Lane and marking detection | Simple image processing: colour masks, shape filters and Hough lines on the road area. Not a trained model |
| Sign detection | Simple image processing: colour and shape (circle, triangle, rectangle) matched to IRC:67 sign groups. Not a trained model |
| Vehicles, people and animals | Pre-trained YOLOv8n (COCO classes) |
| `absence_confidence` | A rough score from the detector signal at the moment a rule fires |
| `zone_type` | A rough guess from the object counts. There is no zone classifier yet |

Every detector is a separate module (`marking_detector.py`, `sign_shape_detector.py`, `object_detector.py`), so a trained model can replace it later without other changes.

## Sections that are loaded

- **IRC:35-2015:** sections 3, 4, 6.1, 6.2, 7, 8, 11
- **IRC:67-2022:** sections 3, 11, 13, 14, 15, 16, 17, 24, 25, 26

The page ranges of each section are in `config.py`. To add another document, add a block for it to `IRC_DOCS` in `config.py`.

Put the IRC PDF files in a `docs` folder next to the code (`docs/IRC35_2015.pdf`, `docs/IRC67_2022.pdf`), or change the paths in `config.py`.

## Data flow

```
IRC35_2015.pdf, IRC67_2022.pdf -> irc_kb_builder.py (one knowledge base per PDF)

video or frames -> main_pipeline.py -> marking_detector.py
                                       sign_shape_detector.py   -> compliance check -> violations and cited clauses
                                       object_detector.py

                                    -> knowledge_graph.py -> GraphML, JSON and HTML graph
                                    -> annotated frame images

frame_reports.csv + retrieval_results.csv + verdicts.csv
        -> merge_events.py (with events.csv from Preprocessing) -> event_compliance.csv
        -> ICDI -> CRRS and IDAS
        -> Precision@k, CHR
```

## Setup

```bash
pip install -r ../../requirements.txt
```

The first run downloads the small YOLOv8n weights (about 6 MB).

## How to run

```bash
# A whole video
python main_pipeline.py --video /path/to/ride.mp4 --sample-every 15 --video-name "ride_01"

# One image
python main_pipeline.py --image /path/to/frame.jpg

# A folder of frames
python main_pipeline.py --image-dir /path/to/frames_folder

# Merge with the events found in Preprocessing
python merge_events.py --events events.csv \
    --frame-reports outputs/frame_reports.csv \
    --output outputs/event_compliance.csv
```

`--sample-every N` skips N frames between checks. 15 is about every 0.5 seconds at 30 fps.

The scripts that compute ICDI, CRRS, IDAS, CHR and Precision@k are in the `Metrics` folder.

## Outputs

Main outputs in `outputs/` (and `vis/` for images):

| File | Made by | Contents |
|------|---------|----------|
| `frame_reports.json` and `.csv` | `main_pipeline.py` | One row per violation with the detections and the cited clause |
| `retrieval_results.csv` | `main_pipeline.py` | Retrieved clauses for each query. The `human_relevant` column is blank for you to fill in |
| `verdicts.csv` | `main_pipeline.py` | Verdict, cited clause and reasoning for each frame |
| `event_compliance.csv` | `merge_events.py` | Compliance rows joined with the events |
| `irc_knowledge_graph.html`, `.graphml`, `.json` | `knowledge_graph.py` | The knowledge graph |
| `vis/annotated_<frame_id>.jpg` | `main_pipeline.py` | Frames with boxes and verdicts |

Columns of `frame_reports.csv` that matter for the metrics:

| Column | Meaning |
|--------|---------|
| `retrieval_score` | TF-IDF similarity of the cited clause |
| `retrieval_rank` | Rank of the cited clause among the candidates |
| `query_text` | The text used for the search |
| `absence_confidence` | Rough confidence (0 to 1) that the needed infrastructure is missing |
| `zone_type` | `School`, `Residential`, `Highway`, `Pedestrian Crossing` or `Unknown` |
| `infrastructure_element` | For example `Lane Marking`, `Warning Sign`, `Regulatory Sign` or `Informatory Sign` |
| `severity` | 1 = low, 2 = medium, 3 = high, 4 = critical |

Note on `event_id`: files made by `main_pipeline.py` use a frame-level id like `<frame_id>_v<index>`. `event_compliance.csv` and later files use a sensor-event id like `<clip_id>_evt<index>`. These are different ids. Do not join files across them.

## Graph structure

```
Video -[HAS_FRAME]-> Frame -[HAS_DETECTION]-> Detection
Detection -[RESULTS_IN]-> Violation -[CITES]-> Clause -[PART_OF]-> Section -[PART_OF]-> IRCDocument
Detection -[COMPLIES_WITH]-> Compliant
```

`IRC35_2015` and `IRC67_2022` stay as separate document nodes with their own sections and clauses.

## Files

| File | Purpose |
|------|---------|
| `config.py` | Paths, section page ranges and chosen sections |
| `irc_kb_builder.py` | Reads each PDF into a clause-level JSON knowledge base |
| `marking_detector.py` | Road marking detection (IRC:35) |
| `sign_shape_detector.py` | Sign shape detection (IRC:67) |
| `object_detector.py` | YOLOv8n wrapper for vehicles, people and animals |
| `knowledge_graph.py` | Builds and exports the knowledge graph |
| `greyspot_algorithm.py` | Finds risky road locations by clustering GPS points of events (DBSCAN) and scoring each cluster |
| `main_pipeline.py` | Command-line entry point |
| `merge_events.py` | Joins sensor events with the frame reports |
