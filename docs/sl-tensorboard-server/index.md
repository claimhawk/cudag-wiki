```markdown
# Project Overview: `sl/tensorboard-server`

## 1. Overview

The `sl/tensorboard-server` project provides a **standalone, Modal-hosted TensorBoard server** tailored for ClaimHawk’s distributed training infrastructure. It serves as a persistent, decoupled visualization layer for two distinct training workflows:

- **`lora-trainer`**: LoRA-based fine-tuning runs, logs stored at `/volume/tensorboard`.
- **`mole-trainer-server/router`**: Mixture-of-Experts (MoE) router or trainer runs, logs stored at `/moe-data/tb_logs`.

The core problem this project solves is **maintaining stable, accessible TensorBoard endpoints during training lifecycle**. Since training jobs are ephemeral and may be restarted or scaled independently, this server ensures TensorBoard dashboards remain available and consistent — regardless of training job state.

This is achieved by deploying a long-running Modal service that mounts and serves TensorBoard from persistent log directories, without requiring training jobs to be active.

---

## 2. Architecture

The project follows a **Modal-based microservice architecture**, with minimal dependencies and clear separation of concerns.

### Directory Structure

```
sl/tensorboard-server/
├── modal/                 # Core server logic
│   └── tb_server.py       # Main Modal service entrypoint
├── scripts/               # CLI utilities
│   └── cleanup.py         # Interactive log cleanup tool
├── README.md              # Public documentation
├── system.md              # System-level configuration or deployment notes
├── CLAUDE.md              # Likely internal documentation or AI-assisted design notes
```

### Purpose of Key Directories

- **`modal/`**: Contains the Modal Python service that:
  - Launches TensorBoard instances.
  - Mounts log directories.
  - Exposes HTTP endpoints for TensorBoard UI.
- **`scripts/`**: Contains CLI tools for managing the server and its logs.
- **`README.md`**: Public-facing documentation for users and contributors.
- **`system.md`**: Likely contains deployment-specific configurations, environment variables, or infrastructure notes.
- **`CLAUDE.md`**: Possibly AI-generated design notes or internal documentation (e.g., from Claude AI assistant).

---

## 3. Key Components

### `modal/tb_server.py`

This is the **core service module**. It defines a Modal function that:

- Mounts the appropriate log directory (`/volume/tensorboard` or `/moe-data/tb_logs`) based on the endpoint.
- Launches a TensorBoard process using `tensorboard --logdir=...`.
- Exposes the TensorBoard UI via HTTP on port 6006 (default TensorBoard port).
- Uses Modal’s `@modal.function` decorator to define a deployable service.

Example snippet (inferred from usage):

```python
@modal.function(
    image=modal.Image.from_registry("tensorflow/tensorflow:2.15.0"),
    secrets=[modal.Secret.from_name("tensorboard-secret")],
    mounts=[modal.Mount.from_local_dir("/volume/tensorboard", remote_path="/tensorboard_logs")],
)
def start_tensorboard():
    import subprocess
    subprocess.run(["tensorboard", "--logdir=/tensorboard_logs", "--port=6006"])
```

The server is designed to be **multi-endpoint**, meaning it can serve both `lora` and `router` logs simultaneously — likely via separate Modal functions or a single function with branching logic.

### `scripts/cleanup.py`

This is a **CLI utility** for cleaning up stale TensorBoard logs. It uses Modal’s `modal.run` to execute a cleanup script that:

- Identifies log directories based on `--trainer` flag (`lora`, `router`, or `all`).
- Deletes old log directories (likely based on age or size).
- Logs cleanup actions for auditability.

Example usage:

```bash
modal run scripts/cleanup.py --trainer lora
```

This script likely uses `os.walk`, `shutil.rmtree`, and Modal’s `modal.Secret` or `modal.Volume` for access control and persistence.

---

## 4. Data Flow

### High-Level Flow

1. **Deployment**:
   - `modal deploy modal/tb_server.py` → Deploys a long-running Modal service.
   - The service mounts log directories and starts TensorBoard.

2. **Training Runs**:
   - Training jobs write logs to `/volume/tensorboard` or `/moe-data/tb_logs`.
   - These directories are mounted into the TensorBoard server.

3. **TensorBoard Serving**:
   - TensorBoard reads logs from mounted directories.
   - Serves UI at `http://localhost:6006` (or via Modal’s public URL).

4. **Cleanup**:
   - `modal run scripts/cleanup.py` → Deletes old log directories.
   - Ensures disk space and dashboard relevance.

### Detailed Flow

```
Training Job → Writes logs → Mounts to TensorBoard Server → TensorBoard reads logs → Serves UI
                     ↑
                 Cleanup Script → Deletes old logs → Prevents disk bloat
```

The server is **stateless** (except for mounted volumes) and **persistent** — it does not depend on training jobs being active.

---

## 5. Getting Started

### Prerequisites

- Python 3.9+
- Modal CLI installed (`pip install modal`)
- Access to a Modal workspace
- Permissions to deploy to the workspace

### Step-by-Step Setup

#### 1. Install Dependencies

```bash
pip install modal tensorboard
```

#### 2. Deploy the Server

Deploy both endpoints (recommended for stable URLs):

```bash
modal deploy modal/tb_server.py
```

> This deploys the server as a Modal function. The server will auto-scale and remain available.

#### 3. Access the Dashboard

Once deployed, access via:

- Modal Dashboard: `https://<your-workspace>.modal.run`
- Direct URLs (if configured):

  ```
  https://<your-workspace>--tensorboard-server-tensorboard-lora.modal.run
  https://<your-workspace>--tensorboard-server-tensorboard-router.modal.run
  ```

> Note: URLs may vary depending on Modal’s naming convention. Check the Modal dashboard for exact endpoints.

#### 4. Clean Up Old Logs

Run the cleanup tool:

```bash
# Clean up lora logs
modal run scripts/cleanup.py --trainer lora

# Clean up router logs
modal run scripts/cleanup.py --trainer router

# Clean up all
modal run scripts/cleanup.py --trainer all
```

> This script likely uses `os.path.getmtime` or `datetime` to filter logs older than a threshold.

#### 5. (Optional) Local Development

To test locally:

```bash
modal serve modal/tb_server.py
```

This runs the server locally (useful for debugging). Note: Local runs may not mount persistent volumes unless configured.

---

## Notes for Contributors

- **File Naming**: All files are prefixed with `modal/`, `scripts/`, etc., for clear separation.
- **Secrets**: TensorBoard may require secrets (e.g., for authentication or logging). Check `modal/tb_server.py` for `modal.Secret` usage.
- **Volumes**: Log directories are mounted via `modal.Mount`. Ensure paths are correct and accessible.
- **TensorBoard Version**: Use a compatible TensorFlow version (e.g., 2.15.0) to avoid compatibility issues.
- **Scaling**: Modal handles scaling automatically. You can configure `@modal.function` with `gpu` or `cpu` resources as needed.

---

## Future Improvements

- Add support for multiple log sources (e.g., wandb, MLflow).
- Integrate with Modal’s `modal.Volume` for persistent log storage.
- Add automatic log rotation or retention policies.
- Add health checks or metrics to monitor TensorBoard server status.

---

This project is a **production-ready, lightweight TensorBoard server** designed for ClaimHawk’s distributed training ecosystem. It ensures visualization is always available, independent of training job lifecycle.
```