```markdown
## Purpose

This document outlines how to implement a learned router for a Mixture-of-Experts (MoE) system atop Qwen3-VL, enabling dynamic selection of LoRA adapters based on input request and screen context. It provides a step-by-step guide for router architecture, training strategy, integration, and practical considerations to avoid common pitfalls.

## Key Components

- **Router Architecture**: Small MLP that takes concatenated text/vision features and outputs softmax-weighted gating over LoRA adapters.
- **Soft/Hard Routing**: Mechanisms to blend or select one adapter per input, with training tricks for stability.
- **Training Stages**: Supervised classification → end-to-end task loss → optional joint LoRA/router fine-tuning.
- **Feature Extraction**: Text (first token/mean-pooled) and vision (image token pool) embeddings for router input.
- **Loss Functions**: Task loss, entropy penalty, KL divergence to supervised labels for router regularization.

## Algorithm Notes

- **Gating**: Router outputs `g(x) ∈ ℝ^K` (softmax) to weight LoRA contributions.
- **LoRA Fusion**: Weighted sum of LoRA deltas applied in activation space: `y = Wx + α·Σg_k·ΔW^(k)`.
- **Training Phases**: Freeze LoRAs initially to train router as task classifier, then co-train with task loss.
- **Hard Routing**: Use Gumbel-Softmax or straight-through estimator during training; argmax at inference.
- **Per-Example Global Routing**: Default routing granularity; avoid per-layer/token routing unless needed.

## Dependencies

- Qwen3-VL base model (with LoRA adapters already trained)
- LoRA adapter weights (`ΔW^(k) = A^(k)B^(k)`)
- Vision transformer and text encoder for feature extraction
- Standard PyTorch/transformers libraries for MLP, softmax, loss functions

## Usage Example

```python
# Pseudo-code for integrating router into Qwen3-VL

# 1. Extract features
text_feat = model.text_encoder.get_first_token_hidden(prompt)  # shape: (B, d)
vision_feat = model.vision_encoder.get_pooled_image_tokens(image)  # shape: (B, d)
route_input = torch.cat([text_feat, vision_feat], dim=-1)
route_input = LayerNorm(route_input)

# 2. Router forward
gating_weights = router(route_input)  # shape: (B, K)

# 3. Soft routing: blend LoRA outputs
lora_outputs = []
for k in range(K):
    lora_output = lora_adapters[k](hidden_states)  # shape: (B, d)
    lora_outputs.append(lora_output)
fused_output = torch.sum(gating_weights * torch.stack(lora_outputs, dim=1), dim=1)

# 4. Final output
output = model.base_model(hidden_states) + alpha * fused_output
```
```