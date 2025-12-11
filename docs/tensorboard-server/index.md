```markdown
# TensorBoard Server Project Overview

## 1. Overview

The `tensorboard-server` project provides a **standalone, deployable TensorBoard server** tailored for ClaimHawk’s distributed training infrastructure. It serves as a persistent, isolated monitoring endpoint for two distinct training workflows:

- **`lora-trainer`**: Runs TensorBoard at `/volume/tensorboard`, tracking LoRA (Low-Rank Adaptation) model training metrics.
- **`mole-trainer-server/router`**: Runs TensorBoard at `/moe-data/tb_logs`, tracking Mixture-of-Experts (MoE) router and trainer metrics.

This server is designed to **decouple TensorBoard from the training lifecycle** — meaning TensorBoard endpoints remain accessible even when training is paused, failed, or restarted. This ensures continuous monitoring without requiring re-deployment or re-initialization of TensorBoard instances.

The project leverages **Modal**, a serverless platform for running Python applications at scale, to host TensorBoard instances as isolated, scalable, and persistent services.

---

## 2. Architecture

The codebase is organized into two main directories:

- **`modal/`**: Contains the core server logic and deployment configuration.
  - `modal/tb_server.py`: The main server module that defines and deploys the TensorBoard endpoints.
- **`scripts/`**: Contains utility scripts for operational tasks.
  - `scripts/cleanup.py`: A command-line tool for pruning old TensorBoard log directories.

The project follows a **Modular Deployment Pattern**:
- Each TensorBoard endpoint is deployed as a separate Modal function or service.
- The server is configured to serve two distinct paths (`/tensorboard_lora` and `/tensorboard_router`) with separate log directories.
- Deployment is handled via Modal CLI (`modal deploy` or `modal serve`).

---

## 3. Key Components

### 3.1 `modal/tb_server.py`

This is the **core server module**. It defines:

- **TensorBoard Server Functions**: Two separate Modal functions (likely named `serve_lora_tensorboard` and `serve_router_tensorboard`) that:
  - Mount the respective log directories (`/volume/tensorboard` and `/moe-data/tb_logs`).
  - Launch TensorBoard in a background process using `tensorboard --logdir=...`.
  - Expose the server on a unique Modal-run URL.

- **Deployment Configuration**: Uses Modal’s `@modal.function` and `@modal.web_endpoint` decorators to define:
  - Persistent storage for logs.
  - Public HTTP endpoints.
  - Resource allocation (CPU, RAM, GPU if needed).

- **Environment Setup**: Likely includes:
  - Installing TensorBoard via `pip install tensorboard`.
  - Setting up environment variables for logging paths.
  - Configuring Modal’s `image` (e.g., `python:3.11` with TensorBoard pre-installed).

### 3.2 `scripts/cleanup.py`

This is a **CLI utility** for managing TensorBoard log directories. It:

- Accepts flags: `--trainer lora`, `--trainer router`, or `--trainer all`.
- Uses `os.walk` or `pathlib` to scan log directories for old TensorBoard logs (e.g., by checking timestamps or log file counts).
- Deletes directories older than a configurable threshold (e.g., 30 days).
- Logs cleanup actions for auditability.

Example usage:
```bash
modal run scripts/cleanup.py --trainer all
```

This script is designed to be run as a Modal task (`modal run`) to avoid blocking the main server process.

---

## 4. Data Flow

### 4.1 Training → TensorBoard Logs

- Training processes (LoRA or MoE) write metrics and logs to their respective directories:
  - LoRA: `/volume/tensorboard`
  - MoE: `/moe-data/tb_logs`

### 4.2 TensorBoard Server → TensorBoard UI

- The `tb_server.py` module mounts these directories and launches TensorBoard:
  ```python
  tensorboard --logdir=/volume/tensorboard --port=6006
  ```
- TensorBoard reads logs and exposes them via HTTP at:
  - `http://localhost:6006` (inside the Modal container)
  - Publicly accessible via Modal-run URLs:
    - `https://<workspace>--tensorboard-server-tensorboard-lora.modal.run`
    - `https://<workspace>--tensorboard-server-tensorboard-router.modal.run`

### 4.3 Cleanup → Log Pruning

- `scripts/cleanup.py` is invoked via `modal run` to scan and delete old log directories.
- It operates independently of the server, ensuring logs are cleaned without interrupting TensorBoard services.

---

## 5. Getting Started

### 5.1 Prerequisites

- Python 3.8+
- Modal CLI installed (`pip install modal`)
- Access to a Modal workspace (via `modal login`)

### 5.2 Setup

1. **Clone the repository** (if not already done).
2. **Install dependencies** (if not pre-installed in the Modal image):
   ```bash
   pip install tensorboard
   ```
3. **Verify the Modal image** (if custom):
   - Ensure `modal/tb_server.py` references the correct image (e.g., `modal run --image python:3.11`).

### 5.3 Deploy the Server

Deploy both endpoints (recommended for stable URLs):

```bash
modal deploy modal/tb_server.py
```

Or run temporarily (for testing):

```bash
modal serve modal/tb_server.py
```

> ✅ **Note**: The server will be accessible via the Modal dashboard or via the public URLs shown in the README.

### 5.4 Clean Up Old Logs

Run the cleanup tool:

```bash
# Clean up LoRA logs
modal run scripts/cleanup.py --trainer lora

# Clean up router logs
modal run scripts/cleanup.py --trainer router

# Clean up both
modal run scripts/cleanup.py --trainer all
```

> ⚠️ **Important**: The cleanup script must be run from the project root to locate `scripts/cleanup.py`.

### 5.5 Development Tips

- Modify `modal/tb_server.py` to adjust log paths, TensorBoard ports, or resource allocation.
- Add logging to `scripts/cleanup.py` to track what directories are deleted.
- Use `modal run --dry-run` to test deployment without actually deploying.
- Use `modal dashboard` to monitor running services and their logs.

---

## 6. Future Considerations

- Add automatic log rotation or retention policies.
- Integrate with external monitoring tools (e.g., Prometheus, Grafana).
- Add authentication or rate-limiting to TensorBoard endpoints.
- Support multiple TensorBoard versions or configurations.
- Add Dockerfile support for local development/testing.

---

## 7. Notes on File Structure

The project includes:

- `README.md`: Project documentation (as provided).
- `CLAUDE.md`: Likely contains AI-assisted documentation or prompts (not relevant for deployment).
- `scripts/cleanup.py`: CLI utility for log management.
- `modal/tb_server.py`: Core server logic.

> ⚠️ **Note**: The error message `Lookup failed for Cls 'DocGenerator' from the 'wiki-agent' app: App 'wiki-agent' not found` appears to be from an external tool (possibly a documentation generator) and is not part of the project’s codebase. It can be ignored.

---

This project enables reliable, scalable, and isolated TensorBoard monitoring for ClaimHawk’s training pipelines — essential for debugging, performance analysis, and model iteration.
```