```markdown
## Purpose

This file implements a stacked inference system for a Mixture-of-Experts (MoE) model architecture, where a routing LoRA selects between task-specific LoRAs or a dedicated OCR model (Chandra). It loads all models into memory at startup for fast inference and formats outputs according to tool-call conventions for computer-use tasks.

## Key Components

- **`discover_deployed_loras()`**: Auto-discover task LoRA adapters from mounted inference volume.
- **`format_ocr_tool_call()`**: Formats Chandra OCR output as a tool_call for downstream processing.
- **`run_chandra_ocr()`**: Executes OCR inference using the Chandra model on an image.
- **`MoEInferenceServer`**: Modal App class that loads and maintains all models (base, router, task LoRAs, Chandra) in memory for fast inference.
- **`LABEL_TO_ADAPTER`**: Maps router integer outputs to adapter names (e.g., 0→"calendar", 2→"ocr").

## Algorithm Notes

- **Routing-based expert selection**: The router LoRA classifies input into one of 7 expert categories (e.g., calendar, OCR, desktop), directing inference to the appropriate adapter.
- **Tool-call output format**: For OCR tasks, outputs are formatted as `computer_use` tool calls to integrate with agent workflows.
- **Warm-start server**: Models are pre-loaded at container startup to minimize inference latency.

## Dependencies

- `modal` (for serverless deployment)
- `torch`, `transformers`, `peft`, `qwen-vl-utils`, `Pillow`, `chandra-ocr`, `fastapi`
- SDK modules (`sdk.modal_compat`, `sdk.adapters`) for config, or fallbacks for remote execution
- Hugging Face `AutoModel`, `AutoProcessor`, and `PeftModel` for model loading

## Usage Example

```python
# Run locally or via Modal CLI
modal run modal/stacked_inference.py --test
# or
modal run modal/stacked_inference.py --sample-idx 0
```

The server is designed to be invoked via Modal’s remote execution system, with models pre-loaded and inference routed dynamically based on input context.
```