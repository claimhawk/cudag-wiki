# Mole-Trainer: Distributed MOLE Model Training Infrastructure

## 1. Overview

**Mole-Trainer** is a distributed training infrastructure designed to coordinate the fine-tuning of **Mixture of LoRA Experts (MOLE)** models — specifically, vision-language models like Qwen3-VL — across GPU clusters. The core problem it solves is the efficient, scalable, and reproducible training of multi-expert models where a learned router dynamically selects which LoRA adapter (expert) to activate for each input.

The system enables:
- **Distributed training** of multiple LoRA adapters (experts) in parallel.
- **Dynamic routing** via a router LoRA adapter that learns to assign inputs to the most appropriate expert.
- **End-to-end pipeline** from data preprocessing → routing dataset generation → router training → evaluation → history tracking.
- **Cloud-native deployment** via Modal, enabling scalable, containerized execution.

This infrastructure is particularly useful for training large-scale MoE models where expert specialization and routing efficiency are critical — e.g., for multimodal applications requiring diverse expert knowledge (e.g., OCR, image captioning, object detection, etc.).

---

## 2. Architecture

The codebase is organized into logical domains, with clear separation of concerns:

### Core Directories:

- **`modal/`** — Modal cloud deployment and execution logic. All training, preprocessing, and evaluation workflows are orchestrated via Modal’s containerized compute environment.
  - `modal/train_router.py` — Main router training loop.
  - `modal/preprocess.py` — Preprocesses Qwen3-VL datasets for router training.
  - `modal/generate.py` — Generates routing dataset via Modal app.
  - `modal/deploy.py` — Deploys trained router LoRA checkpoint to cloud storage.
  - `modal/eval_router.py` — Evaluates router performance on held-out data.
  - `modal/test.py` — Unit test evaluation of router LoRA.
  - `modal/history.py` — Persists and visualizes training history.
  - `modal/stacked_inference.py` — Implements inference for stacked MoE models.

- **`routing-generator/`** — Handles dataset merging and routing dataset generation.
  - `routing-generator/generate.py` — Merges task-specific datasets into a balanced JSONL routing dataset.
  - `routing-generator/README.md` — Describes dataset merging workflow.

- **`scripts/`** — Utility scripts for data preparation and history backfilling.
  - `scripts/backfill_history.py` — Retroactively saves training history for past router runs.
  - `scripts/generate_ocr_crops.py` — Generates OCR training data by cropping text-heavy regions.

- **`utils/`** — Shared utilities and code quality enforcement.
  - `utils/check_docstrings.py` — Enforces docstring standards across modules.

- **`config/`** — Configuration files for datasets, LoRA settings, and routing parameters.
  - `config/dataset.yaml` — Defines routing dataset generation parameters.
  - `config/loras.json` — Defines LoRA training settings for multiple experts.

- **`docs/`** — Architecture, implementation guides, and reference documentation.
  - `docs/moe-gpt.md` — Implementation guide for learned routers.
  - `docs/mole-paper-notes.md` — Insights from the MoLE paper.
  - `plans/router-lora-guide.md` — Guide for implementing router as LoRA adapter.

- **`agents.md`, `CLAUDE.md`, `CODING_STEPS.md`, `CONTRIBUTING.md`** — Project-specific workflows, contribution guidelines, and AI-assisted development protocols.

---

## 3. Key Components

### Router Training & Evaluation

- **`modal/train_router.py`** — The main training loop. Uses preprocessed expert datasets to train a router LoRA adapter. Supports logging, checkpointing, and early stopping.
- **`modal/eval_router.py`** — Evaluates router performance on held-out test/validation data. Computes metrics like accuracy, top-k accuracy, and routing consistency.
- **`modal/test.py`** — Unit test script for router LoRA evaluation on small held-out datasets.

### Data Generation & Preprocessing

- **`modal/preprocess.py`** — Preprocesses Qwen3-VL datasets (e.g., image-text pairs) for router training. Uses Modal’s distributed compute to parallelize preprocessing.
- **`modal/preprocess_ocr.py`** — Specialized preprocessing for OCR datasets, extracting text regions and generating OCR training data.
- **`routing-generator/generate.py`** — Merges multiple task datasets into a balanced routing dataset (JSONL format), ensuring each expert is represented proportionally.
- **`scripts/generate_ocr_crops.py`** — Generates OCR training crops from images, used to enrich routing dataset with text-heavy examples.

### Deployment & Inference

- **`modal/deploy.py`** — Deploys trained router LoRA checkpoint to cloud storage (e.g., GCS, S3) for use in inference pipelines.
- **`modal/stacked_inference.py`** — Implements stacked inference for MoE models: router selects expert, expert generates output, and results are aggregated.

### History & Monitoring

- **`modal/history.py`** — Persists training metrics and logs to a database or file system. Includes visualization utilities for tracking router performance over time.

---

## 4. Data Flow

The system follows a pipeline-driven data flow:

1. **Data Ingestion** — Raw datasets (e.g., image-text pairs for OCR, object detection, etc.) are ingested into the system.
2. **Preprocessing** — `modal/preprocess.py` and `modal/preprocess_ocr.py` preprocess datasets into a format suitable for router training.
3. **Routing Dataset Generation** — `routing-generator/generate.py` merges preprocessed datasets into a balanced JSONL routing dataset.
4. **Router Training** — `modal/train_router.py` trains the router LoRA adapter using the routing dataset.
5. **Evaluation** — `modal/eval_router.py` and `modal/test.py` evaluate router performance on held-out data.
6. **Deployment** — `modal/deploy.py` deploys the trained router LoRA checkpoint to cloud storage.
7. **Inference** — `modal/stacked_inference.py` implements inference for stacked MoE models using the router and expert LoRAs.
8. **History Tracking** — `modal/history.py` logs and visualizes training metrics for monitoring and debugging.

> **Note**: All steps are orchestrated via Modal, enabling distributed, scalable, and reproducible execution.

---

## 5. Getting Started

### Prerequisites

- Python 3.9+
- Modal CLI installed (`pip install modal`)
- GPU-enabled cloud environment (e.g., Modal, GCP, AWS)
- Access to datasets (e.g., Hugging Face, local directories)

### Setup

```bash
# Install dependencies
pip install -r requirements.txt

# Install Modal CLI (if not already installed)
pip install modal

# Initialize Modal (if needed)
modal login
```

### Initialize Project

```bash
# Clone the repository
git clone https://github.com/your-org/mole-trainer.git
cd mole-trainer

# Set up environment variables (optional, for cloud deployment)
export MODAL_API_KEY=your_api_key
```

### Run a Sample Workflow

#### 1. Generate Routing Dataset

```bash
python routing-generator/generate.py --config config/dataset.yaml --output routing_data.jsonl
```

#### 2. Preprocess Data

```bash
python modal/preprocess.py --input dataset_path --output processed_data/
```

#### 3. Train Router

```bash
modal run modal/train_router.py --dataset-path processed_data/ --config config/loras.json
```

#### 4. Evaluate Router

```bash
modal run modal/eval_router.py --checkpoint checkpoint_path --test-data test_data.jsonl
```

#### 5. Deploy Router

```bash
modal run modal/deploy.py --checkpoint checkpoint_path --bucket your-bucket-name
```

#### 6. Visualize Training History

```bash
modal run modal/history.py --checkpoint checkpoint_path --output-dir history/
```

### Development Workflow

1. **Fork** the repository.
2. **Create a feature branch**.
3. **Implement changes**:
   - Avoid hardcoded values — use config files.
   - Add tests (e.g., unit tests for router evaluation).
   - Ensure code passes `utils/check_docstrings.py` and linter checks.
4. **Submit a PR**.

> **Code Quality**: All code must pass:
> - Lexical complexity checks
> - Syntax linting (Black, Flake8)
> - Code formatting (Black)
> - Copyright headers
> See `CODE_QUALITY.md` for full standards.

---

## Additional Notes

- **AI-Assisted Development**: Welcome, provided code includes tests and passes all quality checks.
- **License**: MIT (see `LICENSE` or `README.md`).
- **Documentation**: See `docs/` for architecture, implementation guides, and MoLE paper notes.

This project is designed for scalability, reproducibility, and ease of integration into existing ML pipelines. It is particularly suited for teams training large-scale MoE models with heterogeneous expert knowledge.