```markdown
1. **Purpose**  
This file captures key insights and design cues from the MoLE paper (arXiv:2404.13628), documenting the method for dynamically gating LoRA experts per transformer layer to enable selective, trainable composition without degrading base model performance.

2. **Key Components**  
- `MoLEGate`: Learnable per-layer gate that scores LoRA outputs via softmax with temperature τ.  
- `LoRAExpert`: Represents a trained LoRA applied at a specific layer.  
- `LayerFuser`: Combines weighted LoRA outputs with frozen base block output.  
- `GatingImbalancePlotter`: Visualizes gate distribution across LoRAs (for debugging/analysis).

3. **Algorithm Notes**  
- Uses **per-layer gating** with **temperature annealing** to control selectivity (sharpens gate distribution).  
- **Layer-wise specialization** is critical — gating is applied independently per layer, not globally.  
- **Base model remains frozen**; LoRA experts are optionally unfrozen after gate initialization to preserve quality.

4. **Dependencies**  
- `torch` (for tensor operations, softmax, gradients)  
- `matplotlib` (for plotting gating imbalance)  
- `references/mole-paper/` (local assets: PDFs, figures, LaTeX) — referenced for visuals and context.

5. **Usage Example**  
```python
from mole_paper_notes import MoLEGate, LoRAExpert, LayerFuser

# Initialize gate and experts
gate = MoLEGate(num_experts=3, temperature=1.0)
experts = [LoRAExpert(lora_weights) for lora_weights in trained_loras]
fuser = LayerFuser()

# Forward pass per layer
for layer in transformer_layers:
    lora_outputs = [expert(layer_input) for expert in experts]
    gate_logits = gate(torch.cat(lora_outputs, dim=-1))
    gate_probs = torch.softmax(gate_logits / gate.temperature, dim=-1)
    fused_output = fuser(torch.stack(lora_outputs), gate_probs)
    layer_output = base_block_output + fused_output
```
```