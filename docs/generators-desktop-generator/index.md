# Project Overview: `generators/desktop-generator`

---

## 1. Overview

The `generators/desktop-generator` project is a **CUDAG (Computer-Generated UI Data Augmentation Generator)** system designed to synthesize realistic desktop application screenshots for training machine learning models — particularly vision models, UI interaction models, or screen parsing systems. It generates synthetic UI images by combining domain-specific data models (e.g., Patient, Provider) with dynamic UI state definitions and rendering logic.

This system solves the problem of **data scarcity in UI-centric AI applications** by programmatically generating diverse, labeled, and reproducible desktop UI screenshots. It supports configurable workflows, task-based interactions, and customizable screen layouts, enabling researchers and developers to create large-scale synthetic datasets without manual screenshot capture.

The project is built around a modular architecture that separates UI layout definition, state management, rendering logic, domain models, and task-driven generation workflows.

---

## 2. Architecture

The codebase is organized into logical, decoupled components to support extensibility and maintainability.

### Key Directories:

- **`models/`** — Defines domain-specific data classes (e.g., `Patient`, `Provider`, `Appointment`) used to populate UI state. These are the “data sources” for synthetic UI generation.
- **`tasks/`** — Contains task implementations that define UI interaction sequences (e.g., `wait_loading`, `iconlist_task`). Each task represents a specific UI behavior or state transition.
- **`config/`** — Houses configuration files (`dataset.yaml`, `workflow-test.yaml`) that define generation parameters, sample counts, task sequences, and screen layouts.
- **`screen.py`** — Defines the UI layout structure, including regions, widgets, and positioning logic. This is the “blueprint” for each generated screen.
- **`state.py`** — Defines a dataclass-based state model that holds dynamic values (e.g., patient names, timestamps, button states) to be injected into the UI layout.
- **`renderer.py`** — Implements the core image rendering logic, combining layout definitions, state data, and assets to produce final PNG/JPG images.
- **`generator.py`** — The main orchestration module that coordinates the entire generation pipeline: loads config, initializes state, runs tasks, and invokes the renderer.
- **`workflow.py`** — Manages the sequence of tasks and screen transitions, ensuring generation follows a defined workflow (e.g., “login → dashboard → view patient → exit”).
- **`scripts/`** — Contains utility scripts for preprocessing, verifying, building, uploading, and generating datasets. These are typically invoked via CLI.
- **`assets/`** — Holds base images, fonts, icons, and other static resources used during rendering.

---

## 3. Key Components

### `screen.py`
Defines the UI layout structure using region-based definitions. It specifies:
- Screen dimensions
- Widget positions and sizes
- Layering and z-ordering
- Placeholder regions for dynamic content

Example: A “Patient Dashboard” screen may define regions for “Header”, “Patient Info Panel”, “Appointment List”, and “Footer”.

### `state.py`
Defines a dataclass (e.g., `ScreenState`, `PatientState`) that holds dynamic values to be injected into the UI. These values are populated by domain models and tasks.

Example:
```python
class PatientState:
    name: str
    age: int
    appointment_time: datetime
    status: str  # e.g., "Scheduled", "Cancelled"
```

### `renderer.py`
Implements the core rendering engine. It:
- Takes a screen layout (`screen.py`) and state data (`state.py`)
- Applies transformations (scaling, positioning, overlays)
- Combines base assets (from `assets/`) with dynamic content
- Outputs final image to disk

### `generator.py`
The main entry point. It:
- Parses `config/dataset.yaml`
- Initializes the workflow (`workflow.py`)
- Runs task sequences
- Invokes renderer for each screen/state combination
- Outputs generated images to a configured directory

### `workflow.py`
Manages the sequence of tasks and screen transitions. It:
- Loads task definitions from `tasks/`
- Executes tasks in order (e.g., `login_task`, `wait_loading`, `iconlist_task`)
- Updates state between tasks
- Can be configured via YAML to define branching or loops

### `tasks/`
Contains individual task modules that represent UI interactions:
- `wait_loading.py` — Simulates loading states with animated progress bars
- `iconlist_task.py` — Generates screens with dynamic icon lists (e.g., patient icons, menu items)
- Each task modifies the state and may trigger screen transitions

---

## 4. Data Flow

The system follows a **task-driven, stateful generation pipeline**:

1. **Configuration Load** — `generator.py` reads `config/dataset.yaml` to determine:
   - Number of samples per screen/task
   - Task sequence
   - Asset paths
   - Output directory

2. **State Initialization** — `state.py` defines base state models. Domain models (`models/`) populate these with realistic data (e.g., generate 100 unique patient records).

3. **Screen Definition** — `screen.py` defines the layout for each screen type (e.g., “Login”, “Dashboard”, “Patient Detail”).

4. **Task Execution** — `workflow.py` orchestrates task execution:
   - For each task, it:
     - Applies state modifications
     - May trigger screen transitions
     - May generate intermediate UI states

5. **Rendering** — `renderer.py` is called for each screen/state combination:
   - Combines layout, state, and assets
   - Renders to image
   - Saves to output directory

6. **Output** — Generated images are saved in a structured directory (e.g., `output/screens/dashboard/`), with metadata (e.g., `patient_id`, `task_sequence`) optionally embedded.

---

## 5. Getting Started

### Prerequisites

- Python 3.8+
- `pip` installed
- Basic familiarity with Python OOP and CLI tools

### Setup

```bash
# Install project in editable mode
pip install -e .

# Install additional dependencies (if any)
pip install -r requirements.txt  # if exists
```

### Generate Dataset

```bash
# Using CLI wrapper
cudag generate --config config/dataset.yaml

# Or run directly
python generate.py --config config/dataset.yaml
```

### Development Workflow

1. **Define UI Layout** — Edit `screen.py` to add new screen regions or modify existing ones.
2. **Define Data Models** — Add new domain models in `models/` (e.g., `models/doctor.py`) to populate state.
3. **Add Tasks** — Implement new UI interaction tasks in `tasks/` (e.g., `tasks/search_task.py`).
4. **Configure Workflow** — Edit `config/dataset.yaml` to define sample counts, task sequences, and screen transitions.
5. **Run Generation** — Execute `cudag generate` or `python generate.py` to produce images.

### Testing & Validation

- Run `scripts/verify.py` to validate generated dataset structure and metadata.
- Use `scripts/pre-commit.sh` to ensure code quality checks pass before commits.
- Add unit tests for new tasks or renderers (see `CODE_QUALITY.md` for requirements).

---

## Additional Notes

- **Code Quality**: All contributions must pass:
  - Syntax linting (e.g., `flake8`)
  - Code formatting (e.g., `black`)
  - Lexical complexity checks
  - Copyright headers
- **AI-Assisted Code**: Welcome, provided it includes tests and passes all quality checks.
- **Contributing**: See `CONTRIBUTING.md` for guidelines on forking, branching, and PR submission.

---

This project is designed to be **extensible, modular, and configurable**, making it suitable for generating large-scale, diverse UI datasets for training and evaluation in computer vision, UI automation, and screen parsing applications.