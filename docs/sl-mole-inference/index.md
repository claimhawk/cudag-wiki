# Project Overview: `sl/mole-inference`

## 1. Overview

`sl/mole-inference` is a **generic, configurable Mixture of Experts (MoE) inference server** designed to deploy and route multimodal tasks using **Qwen3-VL base models with LoRA adapters**. It enables dynamic, on-demand switching between multiple task-specific LoRA adapters — either via a router-based routing mechanism or by merging adapters into the base model for performance gains.

This project solves the problem of **operationalizing multiple LoRA adapters** for multimodal AI applications (e.g., vision-language tasks) without requiring separate model instances per adapter. It supports:

- **Runtime adapter switching** via dynamic loading
- **Automatic input routing** using a router LoRA
- **Merged model inference** for speedup (~10–20% faster)
- **Configurable deployment** via YAML or environment variables
- **Auto-discovery** of adapters from mounted volumes
- **HTTP API endpoints** for external integration

The system is built on **Modal**, a serverless platform for deploying machine learning applications, and uses **FastAPI** for the inference API. It is designed for **multi-tenant, multi-task, multimodal** deployment scenarios — particularly useful for research or production environments where task-specific LoRA adapters are frequently updated or deployed.

---

## 2. Architecture

The codebase is organized into logical layers:

### Core Directories:

- **`modal/`** — Core inference logic, model loading, merging, and routing. Contains the main inference engine and utility functions.
  - `inference.py` — Main inference handler for routing and adapter selection.
  - `merge.py` — Logic for merging LoRA weights into base model.
  - `merged_inference.py` — Inference engine for merged models.
  - `config.py` — Configuration loader and validator.
  - `deploy.py` — Deployment scripts for adapters and router.
  - `sam_inference.py`, `ocr_inference.py`, `sam_gradio.py`, `ocr_gradio.py` — Task-specific inference modules (SAM segmentation, OCR) — *Note: These appear to be experimental or legacy integrations, not core to MoE routing*.
  - `__init__.py` — Entry point for Modal app modules.

- **`apps/`** — Application-specific entry points for Modal deployment (e.g., `moe_app.py`, `sam_app.py`, `ocr_app.py`). These define the Modal app context and expose APIs.

- **`scripts/`** — CLI deployment and testing scripts.
  - `deploy.sh`, `deploy-adapter.sh`, `merge.sh`, `deploy-merged.sh`, `test.sh` — Scripts for deploying adapters, merging models, and running tests.

- **`config/`** — Configuration files.
  - `default.yaml`, `local.yaml` — Base and user-configurable config files. Define base model, adapter paths, labels, and Modal settings.

- **`tests/`** — Unit and integration tests.
  - `test_e2e_routing.py`, `test_config.py` — Test routing logic and config parsing.

- **`system.md`, `CLAUDE.md`** — Documentation for system setup or external integrations (possibly legacy or placeholder).

---

## 3. Key Components

### 1. **Router LoRA (`modal/inference.py`)**

The router is a LoRA adapter trained to classify inputs and route them to the correct expert adapter. It operates as a **classifier** that outputs a label (e.g., `0`, `1`) corresponding to an adapter.

- Uses `modal/inference.py` to load and apply the router LoRA.
- Routes input via `route_input()` — returns adapter label.
- Configurable via `config/local.yaml` under `adapters` with `label` and `path`.

### 2. **Dynamic Adapter Loader (`modal/inference.py`)**

- Dynamically loads LoRA adapters from mounted volumes (e.g., `/inference/loras/task-a/adapter`).
- Uses `HF Transformers` to load LoRA weights and merge them with the base model.
- Supports `mode: "dynamic"` — adapter is loaded on-demand, not merged.

### 3. **Merged Model Inference (`modal/merged_inference.py`)**

- Pre-merges LoRA weights into the base model for faster inference.
- Uses `modal/merge.py` to perform the merge.
- Optimized for latency-sensitive use cases.
- Configured via `mode: "merged"` in `config/local.yaml`.

### 4. **Modal App (`apps/moe_app.py`)**

- Entry point for Modal deployment.
- Defines the Modal app context, mounts volumes, loads config, and exposes the inference API.
- Uses `FastAPI` to serve endpoints like `/infer_web`.

### 5. **Deployment Scripts (`scripts/deploy-adapter.sh`, `scripts/deploy.sh`)**

- `deploy-adapter.sh` — Deploys a LoRA adapter or router to Modal.
- `deploy.sh` — Deploys the full MoE inference server.
- Uses `modal run` to execute deployment scripts.

### 6. **Configuration (`config/default.yaml`, `config/local.yaml`)**

- Defines:
  - Modal app name, Hugging Face secret
  - Base model (e.g., `Qwen/Qwen3-VL-8B-Instruct`)
  - Adapter list with `name`, `label`, `path`, `mode`
  - Volume mount path
- Supports environment variable overrides.

---

## 4. Data Flow

1. **Client Request** → `POST /infer_web` (via FastAPI endpoint in `apps/moe_app.py`)
2. **Request Payload** — Contains `image_b64` and `prompt`.
3. **Router LoRA** — If `mode: "dynamic"` is configured, the request is routed via `modal/inference.py` → `route_input()` → returns label (e.g., `0`).
4. **Adapter Selection** — Based on label, the correct LoRA adapter is loaded from `/inference/loras/...`.
5. **Inference** — The base model + selected LoRA adapter (or merged model) processes the input.
6. **Response** — Result returned to client.

> **Note**: If `mode: "merged"` is configured, the router is skipped, and the merged model is used directly.

---

## 5. Getting Started

### Step 1: Set Up Environment

Install dependencies (assumed via `requirements.txt` or `pyproject.toml`):

```bash
pip install -r requirements.txt
```

### Step 2: Configure

Copy and edit the default config:

```bash
cp config/default.yaml config/local.yaml
```

Edit `config/local.yaml` to define:

- `modal.app_name`
- `models.base_vlm`
- `volumes.inference.mount_path`
- `adapters` — list of LoRA adapters with `name`, `label`, `path`, `mode`

Example:

```yaml
adapters:
  - name: "task-a"
    label: 0
    path: "/inference/loras/task-a/adapter"
    mode: "dynamic"

  - name: "task-b"
    label: 1
    path: "/inference/loras/task-b/adapter"
    mode: "dynamic"
```

### Step 3: Deploy Adapters

Deploy task adapters and router using `scripts/deploy-adapter.sh`:

```bash
./scripts/deploy-adapter.sh --adapter-name task-a --checkpoint /path/to/lora --accuracy 0.95
./scripts/deploy-adapter.sh --router --checkpoint /path/to/router
```

Set up label mapping:

```bash
modal run modal/deploy.py --save-labels '{"0": "task-a", "1": "task-b"}'
```

### Step 4: Deploy Server

```bash
./scripts/deploy.sh
```

This will:

- Mount volumes
- Load config
- Deploy Modal app via `apps/moe_app.py`
- Start FastAPI server

### Step 5: Use API

Send POST request to inference endpoint:

```bash
curl -X POST https://your-app--moe-inference.modal.run/infer_web \
  -H "Content-Type: application/json" \
  -d '{
    "image_b64": "data:image/jpeg;base64,...",
    "prompt": "What action should ..."
  }'
```

---

## Notes

- **Legacy/Experimental Modules**: `sam_inference.py`, `ocr_inference.py`, `sam_gradio.py`, `ocr_gradio.py` appear to be task-specific interfaces (e.g., for SAM segmentation or OCR) — likely not part of the core MoE routing logic.
- **Testing**: Run `./scripts/test.sh` to execute unit/integration tests.
- **Merging**: Use `scripts/merge.sh` or `modal/merge.py` to pre-merge LoRA weights into base model for performance.
- **Error Handling**: The repeated `Lookup failed for Cls 'DocGenerator' from the 'wiki-agent' app` errors in file listings are likely **false positives** or **legacy artifacts** — they do not appear to be part of the MoE inference logic and may be from a different project or test framework.

---

This project provides a **production-ready, configurable, and extensible** MoE inference server for multimodal LoRA adapters — ideal for research teams or enterprises deploying multiple task-specific models on a single base model.