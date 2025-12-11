```markdown
# Router as LoRA Adapter - Implementation Guide

## Purpose

This guide documents the implementation of a router LoRA adapter that outputs adapter names (e.g., "calendar", "claim-window") instead of direct tool calls, enabling a Mixture-of-Experts (MoE) architecture where routing selects the appropriate task-specific LoRA. The system includes end-to-end training, evaluation, and inference pipelines using Modal for distributed execution.

## Key Components

- **`generate_routing.py`**: Generates balanced routing datasets from source volumes.
- **`preprocess.py`**: Preprocesses dataset for training (e.g., tokenization, formatting).
- **`training.py`**: Trains router LoRA using SFTTrainer with LoRA fine-tuning.
- **`eval.py`**: Evaluates routing accuracy on held-out validation/evaluation splits.
- **`stacked_inference.py`**: Full inference pipeline: routes → loads task LoRA → generates output.
- **`loras.json`**: Configures paths to task-specific LoRA adapters.

## Algorithm Notes

- **Routing via LoRA**: The router LoRA outputs adapter names as text, not tool calls.
- **Fresh Base Model**: Task inference uses a fresh base model to avoid PeftModel conflicts when loading multiple adapters.
- **Balanced Dataset**: Routing datasets are split into train/val/eval with balanced class distribution.
- **Backward Compatibility**: Inference auto-detects `/final` subfolder in LoRA checkpoints.

## Dependencies

- `transformers`, `peft`, `datasets`, `accelerate`, `modal`
- `Qwen3VLForConditionalGeneration` (base model)
- `SFTTrainer` (for LoRA fine-tuning)
- `config/loras.json` (adapter path configuration)

## Usage Example

```bash
# Generate routing dataset
./scripts/generate_dataset.sh

# Train router LoRA
./scripts/train.sh --dataset-name datasets/routing_20251125_052946 --run-name my-router

# Evaluate
./scripts/eval.sh --run-name my-router --dataset-name datasets/routing_20251125_052946

# Run stacked inference
modal run modal/stacked_inference.py \
  --routing-run routing-20251125-052946 \
  --routing-dataset datasets/routing_20251125_052946 \
  --max-samples 10
```
```