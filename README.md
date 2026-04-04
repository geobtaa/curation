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
- GDAL is required for `osgeo` bindings and for CLI tools like `ogr2ogr`.

### System dependencies

Install GDAL (includes `ogr2ogr`) before installing Python deps:

```sh
# macOS
brew install gdal

# Ubuntu/Debian
sudo apt-get install -y gdal-bin libgdal-dev
```

Python GDAL bindings must match the system GDAL version. If `pip`/`uv` fails to build `gdal`, install a compatible GDAL version first.

## Run (planned)

- `scripts/extract_metadata.py`
- `scripts/get_thumbs.py`
- `scripts/create_pm_tiles.py`
- `scripts/export_gdb_feature_classes_to_gpkg.py`
- `scripts/merge_csvs.py`

Example:

```sh
python scripts/merge_csvs.py people.csv scores.csv id -o combined.csv
```

The merged output includes `match_status` and `unmatched_source` columns so rows that only exist in one CSV are still written to the combined file.
Use `--ignore-key-case` when values like `ABC123` and `abc123` should be treated as the same key. Columns that are blank in every output row are omitted automatically.
