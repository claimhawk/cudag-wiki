```markdown
1. **Purpose**  
This script trains a router model using preprocessed expert datasets stored in `.pt` files, with labels remapped at runtime to route inputs to appropriate experts. It leverages LoRA fine-tuning on a Qwen3-VL base model and uses Modal for distributed training with GPU acceleration and persistent volumes.

2. **Key Components**  
- `train_router()`: Main training function with configurable hyperparameters.  
- `LinkedRouterDataset`: Loads expert samples and remaps labels to routing tokens.  
- `DataCollatorForRouter`: Pads and batches sequences for training.  
- `LoraConfig` + `get_peft_model`: Applies LoRA to selected model layers.  
- `TrainingArguments`: Configures training loop, logging, and checkpointing.

3. **Algorithm Notes**  
- Labels are remapped dynamically at inference time using tokenized label IDs (0–6).  
- Input sequences are truncated before the label, then appended with the new label token.  
- Validation and training samples are sampled per expert, with held-out test indices excluded from training.  
- Batch size is reduced to 1 to avoid OOM, compensated by gradient accumulation steps.

4. **Dependencies**  
- `torch`, `transformers`, `peft`, `datasets`, `tqdm`, `Pillow`, `qwen-vl-utils`  
- `flash-attn` for optimized attention  
- `modal` for distributed execution  
- Hugging Face `huggingface-secret` for model access

5. **Usage Example**  
```bash
modal run modal/train_router.py --manifest router_linked_20251209_204654 --run_name my_run --num_epochs 3 --batch_size 1 --learning_rate 2e-4 --lora_rank 64
```
```