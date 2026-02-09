# DPaaS - Data Prefiltering as a Service

A flexible client-server framework for building and executing data prefiltering pipelines with configurable filters and modality transformations.

## Overview

DPaaS enables distributed data prefiltering by separating data validation and filtering logic between client and server. It supports modular pipeline construction through configurable filters that can process various data modalities (filepaths, numpy arrays, etc.).

## Features

- **Client-Server Architecture**: Filter data remotely with automatic serialization/deserialization
- **Configurable Pipelines**: Define filtering stages via JSON configuration
- **Modality System**: Seamlessly transform data between formats (files, numpy arrays, etc.)
- **Filter Registry**: Extensible filter system with built-in validators and samplers
- **Batch Processing**: Efficient handling of multiple files with progress tracking
- **Compression Support**: Automatic gzip compression for network transfers

## Installation

```bash
pip install -r requirements.txt  # If available
# Or install dependencies manually:
pip install flask requests numpy opencv-python
```

## Quick Start

```bash
# Download test dataset
cd test/scannetpp
bash download.sh
cd ..

# Terminal 1 - Start the server
python3 demo.server.py

# Terminal 2 - Run the client (in a new terminal)
cd test
python3 demo.client.py
```

The demo will process MP4 videos through a multi-stage filtering pipeline and save results to `test/demo_output/`:
- `local_pipeline.txt` - Local pipeline configuration details
- `report.txt` - Filtering results and reports for each file

## Architecture

### Core Components

- **Pipeline**: Sequential execution of filters with modality propagation
- **Filter**: Individual filtering unit with input/output modality
- **Modality**: Data representation format (filepath, numpy array, etc.)
- **Client**: Handles local pipeline + server communication
- **Server**: Manages remote pipelines and task routing

### Data Flow

1. Client performs local validation/prefiltering
2. Data is serialized and sent to server
3. Server applies remote filtering stages
4. Results are returned to client with detailed reports

## Built-in Filters

- **MP4MetaChecker**: Validate video metadata (fps, duration, resolution)
- **MP4Sampler**: Sample frames from videos at specified intervals
- **RandomFilter**: Probabilistic filter for testing
- *(Extend by creating custom filters with `@dpaas_filter` decorator)*

## Project Structure

```
DPaaS/
├── dpaas/              # Core library
│   ├── client.py       # Client implementation
│   ├── server.py       # Flask server
│   ├── pipeline.py     # Pipeline orchestration
│   ├── filter.py       # Filter base class & registry
│   ├── modality.py     # Data modality definitions
│   └── utils.py        # Helper functions
└── test/               # Examples and tests
    ├── demo.server.py  # Server demo
    ├── demo.client.py  # Client demo
    ├── demo.local.json # Local pipeline config
    └── demo.remote.json # Remote pipeline config
```

## Example Use Case

The included demo processes MP4 videos through a multi-stage pipeline:

1. **Local Stage**: Validates video metadata (FPS, duration, resolution)
2. **Local Stage**: Samples frames from valid videos
3. **Remote Stage**: Applies additional filtering on sampled frames

Run the demo:
```bash
# Terminal 1 - Start server
cd test
python3 demo.server.py

# Terminal 2 - Run client
python3 demo.client.py
```

## Extending DPaaS

### Create a Custom Filter

```python
from dpaas.filter import Filter, dpaas_filter
from dpaas.modality import MODAL_FILEPATH

@dpaas_filter
class MyCustomFilter(Filter):
    def __init__(self, config, **kwargs):
        super().__init__(
            config,
            input_modality=MODAL_FILEPATH,
            output_modality=MODAL_FILEPATH,
            **kwargs
        )
        self.threshold = config.get("threshold", 0.5)
    
    def batch_validate(self, filenames, fileobjs):
        # Your filtering logic here
        reports = {}
        for fname, fobj in zip(filenames, fileobjs):
            # Return (confidence, reason)
            reports[fname] = (1.0, "passed")
        return reports
```

Then use it in your pipeline configuration:
```json
{
    "class": "MyCustomFilter",
    "config": {"threshold": 0.7}
}
```

## License

*Add your license here*

## Contributing

*Add contribution guidelines here*
