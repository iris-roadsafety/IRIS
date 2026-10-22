# TAGNet

TAGNet adds sensor information to a pre-trained vision-language model. It uses dashcam video and motion sensor (IMU) signals together to classify evasive actions.

## Design

```
Input:
  - Video frames          -> vision encoder    -> visual tokens
  - IMU sensor values     -> TelemetryAdapter  -> 1 telemetry token
  - GPS and context       -> ContextAdapter    -> 1 context token

Together:
  [telemetry token | context token | visual tokens | text tokens]
  -> language model (QLoRA fine-tuned) -> answer

Output: text with the action label, a confidence and a short explanation
```

## Versions

| Version | Backbone | What is added | Notes |
|---------|----------|---------------|-------|
| [V1](TAGNet_V1/) | Qwen2.5-VL-3B-Instruct | QLoRA, TelemetryAdapter, ContextAdapter | bf16, about 7 GB on an RTX 2080 Ti. Clauses retrieved with ChromaDB |
| [V2](TAGNet_V2/) | Qwen2-VL-2B-Instruct | QLoRA (4-bit), modality gate, clause pointer attention, multimodal retrieval | JSON output with three parts |

## How it differs from normal fine-tuning

1. **Sensor tokens:** IMU values (ax, ay, az, gx, gy, gz) go through a 2-layer MLP and are placed in front of the input as soft tokens.
2. **Context tokens:** GPS features (speed, zone, location) go through a separate MLP.
3. **Rules in the prompt:** clauses from IRC:35 and IRC:67 are retrieved with ChromaDB and added to the prompt.
4. **Frozen vision encoder:** LoRA is applied only to the language model (attention and MLP layers).

## Example

```bash
python TAGNet_V1/rider_pipeline.py --rider R1 --model V1
python TAGNet_V1/rider_pipeline.py --rider R2 --model FT
python TAGNet_V1/rider_pipeline.py --rider R1 --model ZS
```
