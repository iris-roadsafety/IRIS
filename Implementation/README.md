# Implementation

All code for IRIS, grouped by task.

## Folders

| Folder | What it does |
|--------|--------------|
| [`Preprocessing`](Preprocessing/) | Cleans the sensor data (filtering, jerk) and finds candidate events such as sudden braking or swerving |
| [`Zero_Shot`](Zero_Shot/) | Vision-language models run on the events without training |
| [`Fine_Tuning`](Fine_Tuning/) | QLoRA fine-tuning of vision-language models |
| [`TAGNet/TAGNet_V1`](TAGNet/TAGNet_V1/) | TAGNet V1: Qwen2.5-VL-3B-Instruct with QLoRA, a telemetry adapter and a context adapter |
| [`TAGNet/TAGNet_V2`](TAGNet/TAGNet_V2/) | TAGNet V2: Qwen2-VL-2B-Instruct with QLoRA, an adaptive modality gate and clause pointer attention |
| [`Knowledge_Graph`](Knowledge_Graph/) | Knowledge base built from IRC documents, and a check of video frames against it |
| [`Metrics`](Metrics/) | Scripts that measure model results |

## Order of work

```
Raw sensor files (accelerometer, gyroscope, GPS)
        |
        v
Preprocessing
  - step0: merge the sensor files and filter the signals
  - step1: find candidate events
  - step2: people check the events and the labels are saved as gold labels
  - step3 to step7: tests with different filter and threshold settings
        |
        +-----------------------------+
        |                             |
        v                             v
   Zero_Shot                     Fine_Tuning
   (no training)                 (train on rider 1,
                                  test on riders 2, 3 and 4)
        |                             |
        +--------------+--------------+
                       |
                       v
                    TAGNet
   V1: video + sensor tokens
   V2: V1 + modality gate + IRC clause pointer attention
                       |
                       v
                    Metrics
```

## Short description of each part

**Preprocessing.** The raw sensor files are merged and filtered with a bandpass filter (0.3 to 15 Hz). Events are found by comparing several sensor axes using z-scores. People then check the events and the checked labels become the fixed gold labels used by every later step.

**Zero-shot.** Each model gets 4 video frames from an event window and no training. Four ways of giving sensor information are compared: video only, raw sensor values, a short text summary of the sensors, and a structured JSON description.

**Fine-tuning.** Models are trained with QLoRA (rank 16, alpha 32, 4-bit) on the labelled events of rider 1 (about 130 events) and tested on riders 2, 3 and 4 to see how well they work on new riders.

**TAGNet V1.** Adds two small adapters to Qwen2.5-VL-3B-Instruct. The telemetry adapter turns the six sensor values (ax, ay, az, gx, gy, gz) into one token. The context adapter turns GPS information (speed, zone, location) into another token. Only the language part is trained.

**TAGNet V2.** Builds on V1 using Qwen2-VL-2B-Instruct. An adaptive modality gate learns how much to trust the sensor data compared with the video. Clause pointer attention picks which IRC clause applies, and it is used only when the model says a rule applies. Retrieval of IRC clauses is done with ChromaDB. The output is JSON with three parts: `evasive_action`, `road_compliance` and `infrastructure`.

## Folders used by the scripts

Many scripts read these environment variables. Set them before you run anything:

```bash
export IRIS_SCRATCH=/path/to/scratch   # fast working storage
export IRIS_HOME=/path/to/home         # permanent storage, keep checkpoints here
export HF_HOME=$IRIS_SCRATCH/hf_cache  # Hugging Face model cache
```

A few scripts use extra variables, which are listed in their folders:
`IRIS_DATA_DIR` (Preprocessing), `IRIS_DATA_ROOT` and `IRIS_OUTPUT_ROOT` (TAGNet V1 dataset builder), and `IRIS_RESULTS_ROOT` (TAGNet V1 metrics).

On a SLURM cluster, run heavy jobs with `sbatch` or `srun`, not on the login node.

## Setup

```bash
# 1. Install PyTorch for your CUDA version first
pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121

# 2. Install the other packages (run from the repository root)
pip install -r requirements.txt
```
