# Mole-Trainer-Server: Distributed MOLE Model Training Infrastructure

## 1. Overview

**Mole-Trainer-Server** is a distributed training infrastructure designed to coordinate the fine-tuning of **Mixture of LoRA Experts (MOLE)** models — a variant of Mixture-of-Experts (MoE) architectures where each expert is a LoRA (Low-Rank Adaptation) module. The system enables efficient, scalable, and fault-tolerant training across multi-GPU clusters, dynamically routing inputs to the most appropriate expert based on learned routing policies.

This project solves the problem of managing complex, distributed training workflows for large-scale vision-language models with heterogeneous expert modules. It abstracts away the complexity of model parallelism, data sharding, expert routing, and cloud deployment — allowing researchers and engineers to focus on model design and training objectives.

The system supports:
- Dynamic expert routing via learned router models
- Distributed training across GPU clusters
- Modular expert management (LoRA-based)
- Cloud deployment via Modal (a serverless Python framework)
- Data preprocessing, history backfilling, and evaluation pipelines

## 2. Architecture

The codebase is organized into logical modules that separate concerns: training orchestration, data processing, routing logic, cloud deployment, and utilities.

### Key Directories

- **`modal/`** — Cloud deployment and distributed training orchestration using [Modal](https://modal.com). Contains functions for launching training jobs, managing expert data, preprocessing, routing generation, and evaluation. All Modal functions are designed to be serverless and scalable.

- **`config/`** — Configuration files for datasets, LoRA expert definitions, and training hyperparameters. Includes:
  - `dataset.yaml` — Defines data sources, splits, and preprocessing pipelines.
  - `loras.json` — Specifies LoRA expert configurations (rank, scaling, layer targets, etc.).

- **`scripts/`** — Standalone utility scripts for data preparation and post-processing:
  - `backfill_history.py` — Reconstructs training history from partial logs.
  - `generate_ocr_crops.py` — Generates OCR crop data for vision-language alignment.
  - `generate_routing.py` — Generates routing data for training router models.

- **`routing-generator/`** — Dedicated module for generating expert routing data. Contains:
  - `generate.py` — Core routing data generation logic, likely using dataset splits and expert assignments to create routing supervision signals.

- **`utils/`** — Shared utilities including:
  - `check_docstrings.py` — Enforces docstring standards across the codebase.
  - `preprocess_router.py` — Preprocesses data for router training (e.g., tokenization, expert assignment encoding).

- **`docs/`** — Technical documentation including:
  - `moe-gpt.md` — Overview of MoE architectures and their variants.
  - `mole-paper-notes.md` — Notes and summaries from the original MOLE paper.
  - `router-lora-guide.md` — Step-by-step guide for configuring and training router + LoRA experts.

- **`agents.md`, `CLAUDE.md`, `CODING_STEPS.md`, `CONTRIBUTING.md`, `CODE_QUALITY.md`** — Developer-facing guides, contribution policies, and code quality standards.

## 3. Key Components

### Core Training Orchestration

- **`modal/train_router.py`** — Trains the expert routing model using generated routing data. Uses `modal` to distribute training across GPUs.
- **`modal/training.py`** — Orchestrates the main training loop for MOLE models, coordinating expert updates and routing policy updates.
- **`modal/eval_router.py`** — Evaluates the routing model’s performance on held-out data.

### Data & Routing Generation

- **`routing-generator/generate.py`** — Generates routing supervision data by assigning examples to experts based on content or metadata. Outputs data in a format consumable by `train_router.py`.
- **`scripts/backfill_history.py`** — Recovers or reconstructs training history from partial logs, useful for debugging or resuming training.
- **`scripts/generate_ocr_crops.py`** — Generates OCR crops from images to align visual and textual data for training.

### Cloud Deployment & Preprocessing

- **`modal/deploy.py`** — Deploys the entire training infrastructure as a Modal service, including expert models, router, and data loaders.
- **`modal/preprocess.py`** — Preprocesses training data for model input (e.g., tokenization, padding, expert assignment encoding).
- **`modal/preprocess_ocr.py`** — Preprocesses OCR crop data for vision-language alignment.
- **`modal/link_router_data.py`** — Links generated routing data to corresponding training batches for efficient routing model training.

### Evaluation & Inference

- **`modal/stacked_inference.py`** — Runs inference using the trained MOLE model, combining outputs from multiple experts.
- **`modal/history.py`** — Manages and logs training history for monitoring and debugging.

## 4. Data Flow

The system follows a pipeline-based data flow:

1. **Data Ingestion & Preprocessing**  
   - Raw data (images, text, OCR crops) is ingested via `scripts/generate_ocr_crops.py` or from dataset sources defined in `config/dataset.yaml`.
   - Preprocessing is handled by `modal/preprocess.py` and `modal/preprocess_ocr.py`, which tokenize, align, and encode data for model input.

2. **Routing Data Generation**  
   - `routing-generator/generate.py` generates routing supervision signals by assigning examples to experts based on content or metadata.
   - This data is stored and later consumed by `modal/train_router.py`.

3. **Router Training**  
   - `modal/train_router.py` trains the routing model using the generated data. The router learns to assign incoming examples to the most appropriate LoRA expert.

4. **Expert Training**  
   - `modal/training.py` orchestrates the training of LoRA experts. It uses the router’s output to route examples to experts and updates them via gradient descent.
   - Expert updates are stored in `config/loras.json`, which defines LoRA configurations.

5. **Evaluation & Inference**  
   - `modal/eval_router.py` evaluates routing performance.
   - `modal/stacked_inference.py` combines expert outputs for final predictions.

6. **Deployment & Monitoring**  
   - `modal/deploy.py` packages and deploys the entire system as a Modal service.
   - `modal/history.py` logs training progress for monitoring.

## 5. Getting Started

### Prerequisites

- Python 3.8+
- `pip` and `virtualenv` (or `conda`)
- GPU access (for training) or cloud access (for Modal deployment)
- Modal CLI installed: `pip install modal`

### Setup

```bash
git clone https://github.com/your-org/mole-trainer-server.git
cd mole-trainer-server
pip install -r requirements.txt
```

### Initialize Configuration

Copy and modify the sample config files:

```bash
cp config/dataset.yaml.example config/dataset.yaml
cp config/loras.json.example config/loras.json
```

Edit `config/dataset.yaml` to specify your data sources, splits, and preprocessing parameters.

### Generate Routing Data

Run the routing data generator:

```bash
python routing-generator/generate.py --output_dir ./routing_data
```

This generates routing supervision data for training the router.

### Train the Router

Deploy and train the router using Modal:

```bash
modal deploy modal/deploy.py
modal run train_router --routing_data ./routing_data
```

### Train the MOLE Model

Once the router is trained, deploy and run the main training loop:

```bash
modal run training --router_model_path ./router_model
```

### Generate OCR Crops (Optional)

If you’re working with vision-language data:

```bash
python scripts/generate_ocr_crops.py --input_dir ./images --output_dir ./ocr_crops
```

### Evaluate and Inference

After training, evaluate the router:

```bash
modal run eval_router --test_data ./test_data
```

Run inference on new data:

```bash
modal run stacked_inference --input_data ./input_data
```

### Documentation & Contribution

- Read `docs/mole-paper-notes.md` for theoretical background.
- Review `CODING_STEPS.md` for development workflows.
- Contribute by following `CONTRIBUTING.md` and `CODE_QUALITY.md`.

---

> **Note**: All Modal functions are designed to be serverless and scalable. You can run them locally for development, or deploy them to the cloud for production training.

This project is maintained by Tylt LLC under the MIT License (see `LICENSE`).