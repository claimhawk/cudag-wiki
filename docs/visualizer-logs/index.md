# Project: visualizer/logs

## 1. Overview

The `visualizer/logs` project is a logging visualization and analysis tool designed to transform raw log data into interactive, time-series visualizations for debugging, performance monitoring, and operational insight. It solves the problem of log data being difficult to interpret in real-time or retrospectively — especially when logs are voluminous, heterogeneous, or lack structure.

This project ingests structured or semi-structured log entries (typically from application servers, microservices, or distributed systems), processes them into time-series metrics, and renders them in a web-based dashboard using modern visualization libraries. It supports filtering, aggregation, drill-down, and export capabilities.

The core value proposition is **real-time log insight without requiring manual log parsing or external tools** — all within a single, embeddable web interface.

---

## 2. Architecture

The codebase is organized into modular, decoupled components, following a layered architecture:

```
visualizer/logs/
├── app/                  # Main web application entry point
├── data/                 # Data processing and transformation logic
├── models/               # Data schemas and domain models
├── visualizations/       # Charting and rendering logic
├── utils/                # Utility functions (parsers, formatters, etc.)
├── config/               # Configuration files and loaders
├── static/               # Static assets (JS, CSS, images)
├── templates/            # HTML templates for the UI
├── tests/                # Unit and integration tests
└── main.py               # Entry point for CLI or server
```

### Key Directories:

- **`app/`**: Contains the Flask (or FastAPI) web server and route handlers. Entry point is `app.py`.
- **`data/`**: Houses ingestion pipelines and transformation logic. Key files:
  - `log_parser.py` — Parses raw log lines into structured events.
  - `aggregator.py` — Groups events by time window, service, or tag.
- **`models/`**: Defines Pydantic models for log events and metrics.
  - `log_event.py` — Represents a single log entry.
  - `metric_group.py` — Aggregated time-series data for charts.
- **`visualizations/`**: Chart rendering logic using Plotly or D3.js.
  - `line_chart.py` — Renders time-series line charts.
  - `heatmap.py` — Renders event density heatmaps.
- **`utils/`**: Utility functions for common tasks.
  - `time_utils.py` — Handles time zone conversion and windowing.
  - `log_formatter.py` — Formats logs for display or export.
- **`config/`**: Configuration management.
  - `config_loader.py` — Loads and validates config from YAML/JSON.
  - `default_config.yaml` — Default settings for log sources, visualization, etc.

---

## 3. Key Components

### `LogParser` (in `data/log_parser.py`)

This class is responsible for parsing raw log strings into structured `LogEvent` objects. It supports multiple log formats (e.g., JSON, syslog, custom formats) via pluggable parsers.

```python
class LogParser:
    def __init__(self, format_type: str):
        self.format_type = format_type

    def parse(self, raw_log: str) -> LogEvent:
        if self.format_type == "json":
            return self._parse_json(raw_log)
        elif self.format_type == "syslog":
            return self._parse_syslog(raw_log)
        ...
```

### `Aggregator` (in `data/aggregator.py`)

Aggregates log events into time-based buckets (e.g., 5-minute intervals) and computes metrics like count, error rate, latency percentiles.

```python
class Aggregator:
    def __init__(self, time_window: int = 300):  # 5 minutes
        self.time_window = time_window

    def group_by_time(self, events: List[LogEvent]) -> List[MetricGroup]:
        return [self._bucket_events(events, start_time) for start_time in self._get_time_buckets(events)]
```

### `LineChartRenderer` (in `visualizations/line_chart.py`)

Renders time-series data using Plotly. Accepts a list of `MetricGroup` objects and generates an interactive chart with hover tooltips and zoom.

```python
def render_chart(metric_groups: List[MetricGroup], title: str) -> plotly.graph_objs.Figure:
    fig = go.Figure()
    for group in metric_groups:
        fig.add_trace(go.Scatter(
            x=group.timestamps,
            y=group.values,
            name=group.label
        ))
    fig.update_layout(title=title)
    return fig
```

### `LogEvent` (in `models/log_event.py`)

Pydantic model representing a single log entry:

```python
from pydantic import BaseModel, Field

class LogEvent(BaseModel):
    timestamp: datetime.datetime
    level: str  # INFO, ERROR, WARN
    service: str
    message: str
    tags: Dict[str, str] = Field(default_factory=dict)
    metadata: Dict[str, Any] = Field(default_factory=dict)
```

---

## 4. Data Flow

The data flow follows a pipeline pattern:

1. **Ingestion** — Raw logs are read from file, socket, or API endpoint (via `data/log_parser.py`).
2. **Parsing** — Each log line is parsed into a `LogEvent` object.
3. **Aggregation** — Events are grouped by time window and service (via `data/aggregator.py`).
4. **Transformation** — Aggregated data is converted into `MetricGroup` objects suitable for visualization.
5. **Rendering** — `MetricGroup` objects are passed to `visualizations/line_chart.py` to generate Plotly charts.
6. **UI Display** — Charts are embedded in the Flask template (`templates/dashboard.html`) and served via `app.py`.

Example data flow in code:

```python
# In app.py
@app.route('/dashboard')
def dashboard():
    raw_logs = read_logs_from_source()  # e.g., from file or socket
    parsed_events = LogParser("json").parse_batch(raw_logs)
    aggregated = Aggregator().group_by_time(parsed_events)
    chart = LineChartRenderer().render_chart(aggregated, "Service Latency")
    return render_template("dashboard.html", chart=chart.to_html())
```

---

## 5. Getting Started

### Prerequisites

- Python 3.8+
- pip
- Node.js (for optional frontend build step, if using React/Vue components)

### Installation

```bash
git clone https://github.com/your-org/visualizer.git
cd visualizer/logs
pip install -r requirements.txt
```

### Configuration

Copy and edit `config/default_config.yaml`:

```yaml
sources:
  - type: file
    path: /var/log/app.log
    format: json
  - type: http
    url: http://localhost:8080/logs
    format: syslog

visualization:
  time_window: 300
  default_chart: line_chart
```

### Running the Server

```bash
python main.py --config config/default_config.yaml
```

This starts a Flask server at `http://localhost:5000`.

### Development Workflow

1. **Add a new log format** — Create a new parser in `data/log_parser.py` and register it in `config/log_formats.py`.
2. **Add a new visualization** — Implement a new renderer in `visualizations/` and register it in `app.py`.
3. **Test locally** — Use `pytest` to run unit tests in `tests/`.
4. **Deploy** — Use Docker (if available) or `gunicorn` for production.

### Example: Adding a New Log Format

Add a new parser method in `data/log_parser.py`:

```python
def _parse_custom_format(self, raw_log: str) -> LogEvent:
    # Custom parsing logic
    parts = raw_log.split(" | ")
    return LogEvent(
        timestamp=datetime.fromisoformat(parts[0]),
        level=parts[1],
        service=parts[2],
        message=" ".join(parts[3:]),
    )
```

Then register it in `config/log_formats.py`:

```python
LOG_FORMATS = {
    "custom": "data.log_parser.CustomParser",
}
```

---

## Notes

- The project assumes logs are either structured (JSON) or can be parsed with regex.
- Time zones are handled via `pytz` and `datetime.timezone`.
- Export functionality (CSV, PNG, PDF) is available via `visualizations/export.py`.
- Web UI is built with Jinja2 templates and Plotly.js for interactivity.

---

This project is designed for extensibility and ease of integration into existing logging infrastructures. Contributions are welcome — especially around new log formats, visualization types, and performance optimizations.