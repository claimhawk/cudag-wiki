```markdown
# Project Overview: `sl/generators/desktop-generator`

## 1. Overview

The `sl/generators/desktop-generator` project is a **CUDAG (Computer-Generated UI Data Augmentation Generator)** system designed to synthesize realistic desktop application screenshots for training machine learning models — particularly for tasks like UI element detection, screen grounding, and interaction prediction. It generates synthetic images by combining dynamic UI state definitions, domain-specific data models, and configurable interaction workflows.

This project solves the problem of **lack of diverse, annotated, and scalable UI training data** for desktop applications. Instead of manually annotating thousands of real screenshots, developers can define UI layouts, state models, and interaction tasks to generate high-fidelity, labeled synthetic datasets programmatically.

The system is modular, extensible, and designed for rapid prototyping and iteration — ideal for AI/ML teams building vision models that need to understand complex desktop interfaces.

---

## 2. Architecture

The codebase is organized around a **pipeline architecture** that separates concerns into:

- **UI Definition Layer** — Defines screen regions and layout rules.
- **State Modeling Layer** — Encapsulates dynamic data (e.g., patient info, provider names).
- **Rendering Layer** — Generates final images from screen definitions and state.
- **Task Layer** — Implements interaction sequences (e.g., clicking, waiting, scrolling).
- **Configuration Layer** — Specifies dataset parameters, workflows, and sample counts.
- **Domain Models Layer** — Defines entities like `Patient`, `Provider`, etc., used to populate UI state.

### Key Directories:

| Directory       | Purpose                                                                 |
|-----------------|-------------------------------------------------------------------------|
| `screen.py`     | Defines UI screen layout, regions, and rendering constraints.          |
| `state.py`      | Defines dataclass-based state models for dynamic UI content.           |
| `renderer.py`   | Core logic for rendering screen state into image output.               |
| `models/`       | Domain-specific data models (e.g., `Patient`, `Provider`, `Appointment`). |
| `tasks/`        | Implements interaction tasks (e.g., `grounding_task`, `wait_loading`). |
| `config/`       | YAML-based configuration files for dataset generation and workflows.   |
| `assets/`       | Static resources: base images, fonts, icons, and annotation manifests. |
| `scripts/`      | Utility scripts for verification, building, archiving, and previewing. |

---

## 3. Key Components

### 3.1. `screen.py`
Defines the **UI screen structure** using region definitions (e.g., buttons, text fields, headers). It uses coordinate systems and layout rules to position elements. This is the blueprint for what the UI looks like.

### 3.2. `state.py`
Defines **dataclass-based state models** that populate the UI. For example, a `PatientState` might contain `name`, `age`, `diagnosis`, etc. These are dynamically injected into the screen during rendering.

### 3.3. `renderer.py`
The **core rendering engine**. It takes a screen definition and state, composites them with base assets (fonts, icons, background images), and outputs a final PNG/JPEG image. It handles scaling, positioning, and visual layering.

### 3.4. `tasks/`
Implements **interaction workflows**. Each task class defines a step in a user interaction sequence. Examples:
- `grounding_task.py`: Generates images with bounding boxes for UI elements.
- `wait_loading.py`: Adds loading animations or overlays.
- `iconlist_task.py`: Generates screens with dynamic icon lists.

These tasks are orchestrated via `workflow.py`.

### 3.5. `workflow.py`
Manages the **execution pipeline**. It reads the workflow configuration (e.g., `config/workflow-test.yaml`) and sequences the tasks, applying state mutations and screen rendering in a defined order.

### 3.6. `generator.py`
The **main entry point** for dataset generation. It loads the config, initializes the workflow, and runs the task sequence to generate a batch of images. It also handles output directory management and metadata logging.

### 3.7. `api.py`
Defines the **public API** for external tools or integrations to trigger generation. It may expose functions like `generate_dataset(config_path)` or `render_screen(screen_def, state)`.

### 3.8. `config/`
Contains **YAML configuration files**:
- `dataset.yaml`: Defines sample counts, output paths, and task sequences.
- `dataset.test.yaml`: Test configuration with reduced sample size.
- `workflow-test.yaml`: Defines the sequence of tasks and their parameters.

---

## 4. Data Flow

The system follows a **pipeline-driven data flow**:

1. **Configuration Load** → `config/dataset.yaml` is parsed to define:
   - Number of samples
   - Task sequence
   - Output directory
   - Base assets

2. **State Initialization** → `state.py` dataclasses are instantiated with randomized or seeded values (e.g., `Patient(name="John Doe", age=35)`).

3. **Screen Definition** → `screen.py` defines the UI layout and regions.

4. **Task Execution** → `workflow.py` iterates over tasks in sequence:
   - Each task modifies state or screen layout.
   - Tasks may call `renderer.py` to generate an image.

5. **Image Rendering** → `renderer.py` composites screen + state + assets → outputs PNG/JPEG.

6. **Metadata & Output** → Each generated image is saved with metadata (e.g., task name, state values, timestamp).

7. **Validation & Verification** → Scripts like `scripts/verify.py` validate output integrity and annotation correctness.

---

## 5. Getting Started

### 5.1. Prerequisites

- Python 3.8+
- `pip` installed
- Basic familiarity with YAML and Python classes

### 5.2. Installation

```bash
# Install in editable mode
pip install -e .
```

### 5.3. First Run

```bash
# Generate dataset using default config
cudag generate --config config/dataset.yaml

# Or run directly
python generate.py --config config/dataset.yaml
```

### 5.4. Development Workflow

1. **Modify UI Layout** → Edit `screen.py` to add or adjust screen regions.
2. **Define Data Models** → Add new domain models in `models/` (e.g., `Appointment`, `Invoice`).
3. **Implement New Tasks** → Create a new task in `tasks/` (e.g., `tasks/new_task.py`).
4. **Configure Workflow** → Update `config/workflow-test.yaml` to include your new task.
5. **Generate Data** → Run `cudag generate` to produce new images.

### 5.5. Testing & Validation

- Run `scripts/verify.sh` to validate generated images and annotations.
- Use `scripts/preview.sh` to view generated images in-browser.
- Check `scripts/extract.sh` to extract metadata or annotations.

### 5.6. Code Quality

All contributions must pass:
- Syntax linting (`flake8`, `black`)
- Code formatting (`black`)
- Lexical complexity checks
- Copyright headers
- Tests (unit/integration)

AI-assisted code is allowed if accompanied by tests and quality checks.

---

## 6. Additional Resources

- **Contributing Guide**: `CONTRIBUTING.md`
- **Code Quality Standards**: `CODE_QUALITY.md`
- **System Architecture**: `system.md`
- **Agent Integration**: `agents.md`, `CLAUDE.md`
- **Manifests & Assets**: `manifest.json`, `assets/annotations/manifest.json`

---

> **Note**: This project is part of the CUDAG ecosystem. It is designed to be extended and integrated with other CUDAG components (e.g., `wiki-agent`, `agents.md`, `CLAUDE.md`), though those are not currently part of this repository’s scope.
```