# CUDAG - Computer Use Dataset Augmented Generation

A framework for generating synthetic training data for vision-language models
that perform computer use tasks.

## Overview

CUDAG provides tools to create high-quality synthetic datasets for training
AI agents to interact with graphical user interfaces. The framework generates
screenshots paired with action labels, enabling supervised learning of
computer use capabilities.

### Key Features

- **Synthetic Screen Generation** - Procedurally generate realistic UI screenshots
- **Action Annotation** - Automatic labeling of clickable regions and interactions
- **Coordinate System** - Resolution-independent coordinate normalization
- **Extensible Framework** - Easy to add new screen types and generators

## Quick Start

```bash
# Install CUDAG
pip install cudag

# Create a new generator project
cudag new my-generator

# Generate a dataset
cd my-generator
python generator.py
```

## Projects

| Project | Description |
|---------|-------------|
| [CUDAG Framework](generators-cudag/index.md) | Core framework and utilities |
| [Annotator](annotator/index.md) | Visual annotation tool |
| [Desktop Generator](generators-desktop-generator/index.md) | Example desktop screen generator |

## Coordinate System

All coordinates use **Resolution Units (RU)** normalized to [0, 1000]:

```
normalized = (pixel / image_dimension) * 1000
```

This ensures coordinates are resolution-independent and can be scaled to any
screen size.

## Tool Call Format

Generated datasets use a standardized tool call format:

```json
<tool_call>
{"name": "computer_use", "arguments": {"action": "left_click", "coordinate": [500, 300]}}
</tool_call>
```

---

*CUDAG is an open-source project for advancing computer use AI research.*
