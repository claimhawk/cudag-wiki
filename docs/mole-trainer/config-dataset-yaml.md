```markdown
## Purpose
This configuration file defines parameters for generating a routing dataset used by the `mole-trainer-server`. It specifies adapter sources, sample counts, train/val splits, and output settings to produce a dataset for training a Mixture-of-Experts router.

## Key Components
- `name_prefix`: Prefix for generated dataset filenames.
- `seed`: Random seed for reproducible data splits.
- `splits`: Train/validation ratios applied per adapter.
- `test_samples_per_adapter`: Number of test samples per adapter.
- `adapters`: List of adapter sources with sample counts and optional source overrides.
- `output`: Configuration for output image format.
- `system_prompt`: System prompt used during routing training.

## Algorithm Notes
- Data is sampled from each adapter independently, with `count` samples pulled per adapter.
- Train/val splits are applied per adapter’s sample set.
- Test samples are drawn uniformly per adapter (not split), totaling 25 per adapter × 7 adapters = 175 total.

## Dependencies
None declared — this is a standalone YAML configuration file consumed by the `mole-trainer-server` dataset generator.

## Usage Example
```yaml
# This file is loaded by the dataset generator at runtime.
# No direct code invocation needed — used as input config.
```
```