# IRIS: Infrastructure-aware Multimodal Reasoning for Road Safety Auditing with Regulatory Compliance

**Dataset:** [huggingface.co/datasets/VedulaAbhishek/IRIS](https://huggingface.co/datasets/VedulaAbhishek/IRIS)

IRIS is a low-cost way to support road safety audits. It uses video, motion sensor (IMU) data, GPS and road rules from the Indian Roads Congress (IRC). The aim is to link events seen on the road (such as sudden braking or swerving) to the road condition, the place, and the IRC rule that applies. The result is a structured, evidence-based audit record that engineers can check.

The data was recorded with a camera and a phone on a two-wheeler during real rides.

## Data

The recorded data (GoPro video and phone sensor files) is on Hugging Face:
https://huggingface.co/datasets/VedulaAbhishek/IRIS

## What is in this repository

All code is in the `Implementation` folder:

| Folder | What it does |
|--------|--------------|
| `Preprocessing` | Cleans the sensor data and finds events such as sudden braking or swerving |
| `Zero_Shot` | Runs vision-language models on the events without any training |
| `Fine_Tuning` | Fine-tunes vision-language models on the labelled events |
| `TAGNet` | Our model (V1 and V2) that uses video, sensor data and IRC rules together |
| `Knowledge_Graph` | Builds a knowledge base from the IRC documents and checks video frames against it |
| `Metrics` | Scripts that measure how well the models do |

See `Implementation/README.md` for the order in which to run things.

## Setup

1. Install PyTorch for your CUDA version. For example:
   ```bash
   pip install torch torchvision --index-url https://download.pytorch.org/whl/cu121
   ```
2. Install the other packages:
   ```bash
   pip install -r requirements.txt
   ```
3. Set the folders used by the scripts (see `Implementation/README.md`).

Tested with Python 3.10 to 3.12 and CUDA 12.1.
