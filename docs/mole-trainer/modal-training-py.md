```markdown
## Purpose

This file defines a Modal-based training function to fine-tune Qwen3-VL-8B-Instruct with LoRA adapters on custom computer workflow datasets. It supports preprocessed data, early stopping, hyperparameter auto-tuning, and integrates with a shared training history volume. The system prompt dynamically injects available adapter names for routing.

## Key Components

- `_read_yaml_value`: Parses YAML config strings using regex to extract nested values.
- `_load_local_config_values`: Loads essential config values (e.g., volume names, base model) from local `adapters.yaml`.
- `_build_system_prompt`: Constructs a system prompt for the Mixture-of-Experts router using config.
- `train_qwen3vl_lora`: Main Modal function that orchestrates dataset loading, model setup, LoRA configuration, and training with Hugging Face `Trainer`.
- `app`, `image`, `volumes`: Modal app, Docker image, and volume definitions for persistent storage and GPU-accelerated training.

## Algorithm Notes

- Uses **LoRA** (Low-Rank Adaptation) for efficient fine-tuning of large vision-language models.
- Implements **early stopping** with configurable patience.
- Supports **auto-tuning** of hyperparameters (learning rate, batch size, etc.) based on dataset size.
- Uses **gradient accumulation** to simulate larger batch sizes without increasing VRAM.
- **Preprocessed data** path is prioritized for faster startup; raw data fallback is not implemented.

## Dependencies

- `modal`: For cloud orchestration and volume management.
- `torch`, `transformers`, `peft`, `datasets`, `huggingface-hub`, `Pillow`, `numpy`, `tqdm`, `pyyaml`: Core ML and training libraries.
- `flash-attn`: Optimized attention for CUDA acceleration (pre-compiled wheel).
- `huggingface-secret`: For model downloads (via Hugging Face API).

## Usage Example

```bash
modal run modal/training.py --run-name my_training_run --dataset-name routing_20251125_050207 --fast --patience 2
```

This trains a LoRA model on the specified dataset with fast mode enabled (higher LR, lower patience) and logs to `/moe-data/checkpoints/routing_20251125_050207/my_training_run`.
```