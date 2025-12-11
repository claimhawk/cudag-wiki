```markdown
1. **Purpose**  
This script preprocesses OCR images from expert datasets for router training, converting them into tokenized inputs with labels for fine-tuning a Mixture of Experts router. It aggregates images from specified `ocr/` subdirectories, splits them into train/val sets, and saves them as PyTorch tensors with metadata for downstream use.

2. **Key Components**  
- `preprocess_ocr()`: Main function that collects, preprocesses, and saves OCR image data with router labels.  
- `preprocess_image()`: Processes a single image into model inputs (input_ids, attention_mask, pixel_values, labels).  
- `@app.function`: Modal function that runs the preprocessing with GPU, volumes, and secrets.  
- `@app.local_entrypoint`: CLI entrypoint to trigger remote preprocessing.  
- `metadata.json`: Output file storing dataset config and split counts.

3. **Algorithm Notes**  
- Uses `Qwen/Qwen3-VL-8B-Instruct` processor to encode image+text prompts.  
- Labels are masked to ignore input tokens, training only the router’s response (OCR label `2`).  
- Random shuffling and split ratio ensure balanced training/val sets.  
- Uses `transformers` and `qwen-vl-utils` for vision-language processing.

4. **Dependencies**  
- Python 3.11  
- PyTorch 2.4.0 + torchvision 0.19.0 (CUDA 12.1)  
- Transformers >=4.57.0, qwen-vl-utils, Pillow, tqdm  
- Hugging Face `AutoProcessor` and `process_vision_info`  
- Modal (for distributed execution)  
- `modal.Secret.from_name("huggingface-secret")` for HF token access

5. **Usage Example**  
```bash
modal run modal/preprocess_ocr.py --output_name "ocr--test--20250101_120000" --val_split 0.1 --max_samples 1000
```
This generates a dataset named `ocr--test--20250101_120000` with 1000 samples, 10% validation split, saved to `/expert-data/preprocessed/`.
```