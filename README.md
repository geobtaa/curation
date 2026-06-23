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

## Embed QGIS Metadata in GeoPackages

Use `scripts/embed_qgis_metadata.py` to write QGIS-style XML metadata into every
GeoPackage in a directory. The script reads one CSV row per GeoPackage, renders
`scripts/templates/qgis-metadata.xml` with values from that row, fills the CRS
and bounding box from the GeoPackage itself, and stores the XML in the
GeoPackage metadata extension tables.

For the Milwaukee urban base layers:

```sh
uv run python scripts/embed_qgis_metadata.py \
  mke-ubl \
  mke-ubl/b1g_55-53000_primary.csv
```

The third positional argument is optional and can point to a different XML
template:

```sh
uv run python scripts/embed_qgis_metadata.py \
  path/to/geopackages \
  path/to/metadata.csv \
  path/to/qgis-metadata.xml
```

### CSV Structure

The default match column is `filename`. Each value in that column must exactly
match a GeoPackage filename in the target directory, including the `.gpkg`
extension:

```csv
filename,Title,Description,ID,Date Range,Theme,Provenance,Rights,Source
mke_boundary_2026.gpkg,Municipal boundary [Wisconsin--Milwaukee] {2026},...,b1g_5XPUIjJ9q7Z8,2026-2026,Boundaries,...,...,...
```

The script accepts OpenGeoMetadata-style CSVs like
`mke-ubl/b1g_55-53000_primary.csv`. Extra columns are ignored unless the XML
template references them.

Use `--match-column` if the filename is stored in a different column:

```sh
uv run python scripts/embed_qgis_metadata.py \
  path/to/geopackages \
  path/to/metadata.csv \
  --match-column "Identifier"
```

The match column must be unique. Blank match values are ignored, and duplicate
values stop the run with an error.

### XML Template Tokens

The default template is `scripts/templates/qgis-metadata.xml`. Any text or
attribute value in the template can include tokens in braces. Tokens are
case-insensitive CSV column names:

```xml
<identifier>{ID}</identifier>
<title>{Title}</title>
<abstract>{Description}</abstract>
<rights>{Rights}</rights>
```

The default template currently uses these CSV columns:

- `ID`
- `Source`
- `Title`
- `Description`
- `Theme`
- `Provenance`
- `Rights`
- `Date Range`

Two special token forms are available for range-like fields:

```xml
<start>{Date Range first value}</start>
<end>{Date Range last value}</end>
```

For a value like `2024-2026`, the first token resolves to `2024` and the last
token resolves to `2026`. The resolver first looks for four-digit years. If no
years are present, it splits on `|`, `;`, `,`, or `/`.

The special `{now}` token resolves to the current date in `YYYY-MM-DD` format.

### GeoPackage Behavior

For each matched GeoPackage, the script:

- reads the first feature table in `gpkg_contents` to get the extent and SRS;
- replaces the template `<crs><spatialrefsys>` block with values from
  `gpkg_spatial_ref_sys`;
- replaces the template `<extent><spatial>` attributes with the GeoPackage
  bounding box;
- drops and recreates existing `gpkg_metadata` and
  `gpkg_metadata_reference` tables;
- inserts one dataset metadata record and references it from every feature table
  in the GeoPackage;
- refreshes `gpkg_extensions` rows for the GeoPackage metadata extension when
  `gpkg_extensions` exists.

Unmatched GeoPackages are skipped and left unchanged. Metadata rows that do not
match any GeoPackage are reported at the end of the run.

## Build PMTiles from GeoPackages

Use `build_pmtiles_from_gpkg.py` to recursively convert GeoPackage vector layers
to EPSG:4326 FlatGeoBuf files, then to PMTiles with Tippecanoe. The script
supports multi-layer GeoPackages, configurable field dropping, resumable runs,
and CSV or JSON reports.

Install the required command-line tools first:

```sh
brew install gdal tippecanoe
```

Start with a field inventory report so you can review large attribute tables:

```sh
python build_pmtiles_from_gpkg.py \
  --input-dir ./gpkg \
  --fgb-dir ./fgb \
  --pmtiles-dir ./pmtiles \
  --config pmtiles_config.json \
  --report pmtiles_build_report.csv \
  --field-report-only
```

Copy `pmtiles_config.sample.json` to `pmtiles_config.json`, then edit layer
rules and field keep/drop settings.

Run the conversion:

```sh
python build_pmtiles_from_gpkg.py \
  --input-dir ./gpkg \
  --fgb-dir ./fgb \
  --pmtiles-dir ./pmtiles \
  --config pmtiles_config.json \
  --report pmtiles_build_report.csv
```

Rerun without rebuilding completed outputs:

```sh
python build_pmtiles_from_gpkg.py \
  --input-dir ./gpkg \
  --fgb-dir ./fgb \
  --pmtiles-dir ./pmtiles \
  --config pmtiles_config.json \
  --report pmtiles_build_report.csv \
  --skip-existing
```
