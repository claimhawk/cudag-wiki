```markdown
1. **Purpose**  
This script merges multiple task datasets into a single JSONL file, enriching each record with adapter labels, source metadata, and resolved image paths. It supports filtering by task type, limiting row count, and optional shuffling of output.

2. **Key Components**  
- `SourceSpec`: Dataclass defining source configuration (label, path, tasks, limit, etc.).  
- `parse_source`: Parses CLI `--source` strings into `SourceSpec` objects.  
- `load_jsonl`: Streams JSON objects from a JSONL file.  
- `resolve_image`: Resolves relative image paths using multiple fallback directories.  
- `process_source`: Filters and enriches records from a source, adding adapter and image metadata.  
- `main`: CLI entry point that orchestrates merging, shuffling, and output.

3. **Algorithm Notes**  
- Uses streaming JSONL parsing for memory efficiency.  
- Image path resolution prioritizes user-specified root, JSONL parent, and grandparent (for calendar datasets).  
- Optional shuffling with fixed seed for reproducibility.

4. **Dependencies**  
- `argparse`, `json`, `random`, `sys` (standard library)  
- `dataclass`, `pathlib`, `typing` (Python standard)  

5. **Usage Example**  
```bash
python generate.py --source label=adapter1,jsonl=data/adapter1.jsonl,tasks=classification,limit=100 --source label=adapter2,jsonl=data/adapter2.jsonl,tasks=segmentation --output merged_routing.jsonl --shuffle --seed=123
```
```