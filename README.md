# CUDAG Documentation

Technical documentation for the CUDAG (Computer Use Data Augmented Generation) framework and related open-source tools.

## About CUDAG

CUDAG is a domain-specific framework for programmatically generating synthetic training data for Vision-Language Models (VLMs) that interact with computer interfaces. It provides a Rails-like MVC-inspired architecture to declaratively define UI screens, dynamically populate state, render visual outputs, and orchestrate interaction logic.

## Projects

This documentation covers the following open-source projects:

- **[CUDAG Framework](docs/generators-cudag/index.md)** - Core framework for synthetic data generation
- **[Annotator](docs/annotator/index.md)** - Visual annotation tool for training data
- **[Desktop Generator](docs/generators-desktop-generator/index.md)** - Example screen generator
- **[Dataset Viewer](docs/dataset-viewer/index.md)** - Tool for viewing generated datasets
- **[MoLE Trainer Server](docs/mole-trainer-server/index.md)** - Distributed training server
- **[Tensorboard Server](docs/tensorboard-server/index.md)** - Training visualization
- **[Visualizer](docs/visualizer/index.md)** - Attention heatmap visualization

## Local Development

```bash
# Install MkDocs
pip install mkdocs-material

# Serve locally
mkdocs serve

# Build static site
mkdocs build
```

## Links

- [CUDAG PyPI Package](https://pypi.org/project/cudag/)
- [GitHub Organization](https://github.com/claimhawk)

## License

MIT License - See individual project repositories for details.
