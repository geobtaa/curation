# Curation

Python scripts replacing the original notebooks for the curation pipeline.

## Setup

```sh
uv python install 3.12
uv python pin 3.12
uv venv --python 3.12
source .venv/bin/activate
uv pip install -e .
```

Notes:
- Python 3.12 is recommended. Python 3.13+ (including 3.14) may try to build `fiona` from source and require a local GDAL install.

## Run (planned)

- `scripts/extract_metadata.py`
- `scripts/get_thumbs.py`
- `scripts/create_pm_tiles.py`
