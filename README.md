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

## Purdue Campus GeoTIFFs

Create the CSV, JSON, and STAC metadata inventories:

```sh
python3 scripts/inventory_geotiffs.py
```

This writes:

- `purdue-campus-metadata/geotiff_inventory.csv`
- `purdue-campus-metadata/geotiff_inventory.json`
- `purdue-campus-metadata/stac/*.json`

Benchmark selected rasters before processing the complete collection:

```sh
python3 scripts/create_geotiff_cogs.py \
  --profile balanced \
  --include 1893 1908 1962
```

Create full-resolution, JPEG-compressed COGs:

```sh
python3 scripts/create_geotiff_cogs.py --profile balanced
```

JPEG COGs use separate RGB bands instead of the more compact YCbCr JPEG
layout. This produces somewhat larger files but avoids incorrect color tints
in COG viewers that do not decode YCbCr TIFF imagery correctly.

Reduce rasters over 150 million pixels while creating COGs:

```sh
python3 scripts/create_geotiff_cogs.py \
  --profile reduced \
  --max-pixels 150000000
```

Create lossless, full-resolution ZSTD COGs:

```sh
python3 scripts/create_geotiff_cogs.py --profile archival
```

COGs are written to `purdue-campus-cogs/`. Processing results are written to
`purdue-campus-processing-report.csv` and `purdue-campus-processing-report.json`.
The scripts preserve the source directory and assign canonical `EPSG:3857`
metadata without reprojecting pixels. Use `--assign-srs none` to preserve the
source CRS definition instead.
