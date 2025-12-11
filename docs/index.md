# ClaimHawk Platform Documentation

Internal technical documentation for the ClaimHawk computer use digital labor platform.

## Platform Overview

ClaimHawk uses a **Mixture of Experts (MoE)** architecture where a router model
classifies screenshots and routes them to specialized expert LoRA adapters trained
on specific screen types.

### Architecture

```
Screenshot → Router → Expert Selection → LoRA Adapter → Action Prediction
```

The platform consists of:

- **Screen Generators** - Synthetic data generation for each screen type
- **Training Infrastructure** - LoRA fine-tuning and MoE router training
- **Evaluation Environment** - RL-based agent evaluation on workflows
- **Supporting Tools** - Dataset viewing, annotation, and debugging

## Quick Links

| Category | Description |
|----------|-------------|
| [RL Projects](oracle-agent/index.md) | Reinforcement learning agents and evaluation |
| [Screen Generators](generators-calendar-generator/index.md) | Synthetic training data generation |
| [Training](lora-trainer/index.md) | LoRA and MoE training infrastructure |
| [Tools](dataset-viewer/index.md) | Development and debugging tools |

## Key Concepts

### Coordinate System
All coordinates use **Resolution Units (RU)** normalized to [0, 1000]:
```
normalized = (pixel / image_dimension) * 1000
```

### Tool Call Format
Training data uses the `<tool_call>` format:
```json
<tool_call>
{"name": "computer_use", "arguments": {"action": "left_click", "coordinate": [500, 300]}}
</tool_call>
```

### Dataset Naming
Uses `--` delimiter: `{expert}--{researcher}--{timestamp}`

---

<small>Auto-generated documentation. 25 projects
(8 public, 15 private), 607 source files.</small>
