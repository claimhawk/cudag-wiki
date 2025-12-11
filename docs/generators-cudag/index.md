# CUDAG - Computer Use Data Augmented Generation

## 1. Overview

CUDAG (Computer Use Data Augmented Generation) is a domain-specific framework for programmatically generating synthetic training data for Vision-Language Models (VLMs) that interact with computer interfaces. It provides a Rails-like MVC-inspired architecture to declaratively define UI screens, dynamically populate state, render visual outputs, and orchestrate interaction logic — all to simulate realistic user interactions with software applications.

The core problem CUDAG solves is the scarcity of high-quality, diverse, and annotated training data for VLMs that must understand and generate actions on graphical user interfaces (GUIs). Instead of manually curating datasets, CUDAG allows developers to define application “apps” (e.g., “claim-window-generator”, “wiki-agent”) that generate thousands of synthetic screenshots with corresponding interaction tasks, grounding the model in realistic UI contexts.

CUDAG abstracts away the complexity of image composition, coordinate mapping, and dynamic content generation, enabling rapid prototyping of data generators for any GUI domain — from medical claim forms to wiki editing interfaces.

---

## 2. Architecture

The codebase follows a modular, MVC-inspired structure with clear separation of concerns:

### Root Project Structure

```
cudag/
├── src/
│   └── cudag/
│       ├── __init__.py
│       ├── core/
│       │   ├── screen.py           # UI definition
│       │   ├── state.py            # Dataclass for dynamic state
│       │   ├── renderer.py         # Image generation engine
│       │   ├── task.py             # Interaction logic
│       │   ├── config.py           # App configuration loader
│       │   ├── models.py           # Domain models + generators
│       │   ├── coords.py           # Coordinate system utilities
│       │   ├── drawing.py          # Drawing primitives
│       │   ├── random.py           # Randomization utilities
│       │   ├── icon.py             # Icon rendering
│       │   ├── grid.py             # Grid-based layout
│       │   ├── scroll_task.py      # Scrollable UI interaction
│       │   ├── scrollable_grid.py  # Scrollable grid components
│       │   ├── iconlist_task.py    # Icon list interaction logic
│       │   └── __init__.py
├── tests/
│   ├── __init__.py
│   ├── test_screen.py
│   ├── test_task.py
│   ├── test_renderer.py
│   ├── test_models.py
│   ├── test_config.py
│   ├── test_coords.py
│   ├── test_drawing.py
│   ├── test_text.py
│   ├── test_fonts.py
│   ├── test_random.py
│   ├── test_cli.py
│   ├── test_utils.py
│   └── test_tools.py
├── scripts/
│   └── chandra_ocr_modal.py       # Example script using CUDAG
└── Makefile
```

### Key Directories

- **`src/cudag/core/`**: Core framework modules. Contains the MVC-like components:
  - `screen.py`: Defines the declarative UI structure using a Screen class.
  - `state.py`: Defines the state dataclass that holds dynamic values (e.g., patient name, claim amount).
  - `renderer.py`: The main image generation engine that composites base images, overlays, and text.
  - `task.py`: Orchestrates interaction logic (e.g., click, scroll, type) using task classes.
  - `models.py`: Domain-specific data models (e.g., `Patient`, `Provider`) with built-in generators.
  - `config.py`: Loads and validates app-specific configuration from `dataset.yaml`.
  - `coords.py`: Provides coordinate utilities for positioning UI elements.
  - `drawing.py`: Low-level drawing primitives (text, rectangles, icons).
  - `random.py`: Utilities for randomized generation of state values.

- **`tests/`**: Comprehensive test suite covering all core modules. Each test file targets a specific component (e.g., `test_screen.py` validates screen rendering, `test_task.py` tests interaction logic).

- **`scripts/`**: Example scripts demonstrating usage (e.g., `chandra_ocr_modal.py` generates OCR-modality training data).

- **`Makefile`**: Automation for development, testing, and formatting.

---

## 3. Key Components

### 3.1 Screen (UI Definition)

Defined in `src/cudag/core/screen.py`. Represents the declarative UI layout. It’s a class that defines regions, widgets, and their positions using a grid-based system.

```python
class Screen:
    def __init__(self, width, height):
        self.width = width
        self.height = height
        self.widgets = []  # List of Widget objects

    def add_widget(self, widget):
        self.widgets.append(widget)
```

### 3.2 State (Dynamic Data)

Defined in `src/cudag/core/state.py`. A dataclass that holds dynamic values used to populate the UI.

```python
@dataclass
class State:
    patient_name: str = field(default_factory=lambda: random_name())
    claim_amount: float = field(default_factory=lambda: random_float(100, 1000))
    provider_id: str = field(default_factory=lambda: random_provider_id())
```

### 3.3 Renderer (Image Generation)

Defined in `src/cudag/core/renderer.py`. The main engine that composites the final image by:
- Loading base image (`assets/base.png`)
- Overlaying dynamic text and icons
- Applying grid overlays (`assets/grid_blank.png`)
- Saving output to `datasets/`

```python
class Renderer:
    def __init__(self, screen, state):
        self.screen = screen
        self.state = state

    def render(self):
        image = Image.open("assets/base.png")
        for widget in self.screen.widgets:
            widget.render(image, self.state)
        image.save(f"datasets/output_{uuid.uuid4()}.png")
```

### 3.4 Task (Interaction Logic)

Defined in `src/cudag/core/task.py`. Defines interaction sequences (e.g., “click on button 3”, “scroll down 200px”).

```python
class Task:
    def __init__(self, screen, state):
        self.screen = screen
        self.state = state

    def execute(self):
        # Example: click on a button
        button = self.screen.get_widget("submit_button")
        self.click(button.x, button.y)
```

### 3.5 Model (Domain Data + Generators)

Defined in `src/cudag/core/models.py`. Defines domain-specific data types with generators.

```python
class Patient:
    def __init__(self):
        self.name = generate_name()
        self.age = generate_age()
        self.condition = generate_condition()

    @staticmethod
    def generate():
        return Patient()
```

### 3.6 Config (App Configuration)

Defined in `src/cudag/core/config.py`. Loads `dataset.yaml` to configure:
- Number of samples
- Data generation rules
- Output directory
- Base image paths

```python
class Config:
    def __init__(self, config_path):
        self.data = yaml.safe_load(open(config_path))
        self.num_samples = self.data.get("num_samples", 1000)
        self.output_dir = self.data.get("output_dir", "datasets")
```

---

## 4. Data Flow

The data flow follows a pipeline:

1. **Configuration Load** (`config.py`): Load `dataset.yaml` to determine generation parameters.
2. **State Generation** (`models.py`): Instantiate domain models (e.g., `Patient`, `Provider`) and generate dynamic values.
3. **Screen Definition** (`screen.py`): Define UI layout with widgets (text fields, buttons, icons).
4. **Renderer Initialization** (`renderer.py`): Combine screen + state to generate image.
5. **Task Execution** (`task.py`): Define interaction sequence and execute (optional, for action-based datasets).
6. **Output** (`renderer.py`): Save image to `datasets/` directory.

Example flow:

```python
# 1. Load config
config = Config("config/dataset.yaml")

# 2. Generate state
state = State()
patient = Patient.generate()
state.patient_name = patient.name

# 3. Define screen
screen = Screen(1280, 720)
screen.add_widget(Button("Submit", (500, 500)))

# 4. Render
renderer = Renderer(screen, state)
renderer.render()

# 5. Execute task (optional)
task = Task(screen, state)
task.execute()
```

---

## 5. Getting Started

### 5.1 Prerequisites

- Python 3.9+
- `uvx` (for `pip install` commands)
- `make` (for automation)

### 5.2 Installation

```bash
# Create virtual environment
python3 -m venv .venv
source .venv/bin/activate

# Install CUDAG and dev dependencies
make install
make dev
```

### 5.3 Create a New Generator App

```bash
# Install CUDAG globally
uvx pip install cudag

# Create a new generator project
cudag new claim-window-generator

# Navigate into the project
cd claim-window-generator
```

This generates:

```
claim-window-generator/
├── assets/
│   ├── base.png
│   └── grid_blank.png
├── config/
│   └── dataset.yaml
├── models/
│   └── patient.py
├── tasks/
│   └── submit_task.py
├── screen.py
├── state.py
├── renderer.py
└── datasets/  # Output directory (gitignored)
```

### 5.4 Customize Your Generator

1. **Edit `screen.py`**: Define UI layout.
2. **Edit `state.py`**: Define dynamic data.
3. **Edit `models.py`**: Add domain models.
4. **Edit `tasks.py`**: Define interaction logic.
5. **Edit `config/dataset.yaml`**: Configure generation parameters.

### 5.5 Run the Generator

```bash
# Generate 1000 samples
make generate

# Run all quality checks
make check

# Format code
make format
```

### 5.6 Run Tests

```bash
# Run all tests
make test

# Run specific test
python -m pytest tests/test_screen.py
```

---

## 6. Example Use Case

Generate synthetic screenshots of a medical claim form with randomized patient data and interaction tasks:

```python
# In your app's main.py or generator script
from cudag.core.screen import Screen
from cudag.core.state import State
from cudag.core.models import Patient
from cudag.core.renderer import Renderer
from cudag.core.task import Task

# 1. Load config
config = Config("config/dataset.yaml")

# 2. Generate state
state = State()
patient = Patient.generate()
state.patient_name = patient.name
state.claim_amount = patient.claim_amount

# 3. Define screen
screen = Screen(1280, 720)
screen.add_widget(TextField("Patient Name", (50, 50), state.patient_name))
screen.add_widget(TextField("Claim Amount", (50, 100), f"${state.claim_amount}"))
screen.add_widget(Button("Submit", (500, 500)))

# 4. Render
renderer = Renderer(screen, state)
renderer.render()

# 5. Execute task (optional)
task = Task(screen, state)
task.execute()
```

This generates a synthetic screenshot with randomized patient data and saves it to `datasets/`.

---

## 7. Troubleshooting

### Error: `Lookup failed for Cls 'DocGenerator' from the 'wiki-agent' app: App 'wiki-agent' not found`

This error indicates that the app `wiki-agent` is referenced in tests or code but is not registered in the CUDAG app registry. Ensure:

- The app directory exists: `wiki-agent/`
- It contains `screen.py`, `state.py`, `renderer.py`, `task.py`, `models.py`
- It is registered in `config/dataset.yaml` or via CLI.

Example fix:

```yaml
# config/dataset.yaml
apps:
  - name: wiki-agent
    path: wiki-agent
    num_samples: 500
```

---

## 8. Contributing

Contributions are welcome! Follow these steps:

1. Fork the repo.
2. Create a feature branch.
3. Add tests for new features.
4. Run `make check` before submitting a PR.
5. Submit a PR with a clear description.

---

## 9. License

MIT License — see `LICENSE` for details.

---

## 10. Contact

For questions or support, open an issue on GitHub or contact the maintainers.

--- 

This overview provides a comprehensive technical reference for developers working with CUDAG. The framework is designed for rapid, scalable, and maintainable generation of synthetic GUI training data for VLMs.