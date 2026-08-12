# Omega Parser

A modern Python library for cleaning, parsing TNavigator simulation files, RSM and UNSMRY summary files with possible data extraction and comprehensive configuration support, CLI interface, event-driven scenario management.

## Features

- 🛢️ TNavigator simulation file cleaning and parsing (at least 66 keyword parsers: PVDG, WECON, DATES, GRUPTREE, VFPPROD, UDQ, WELSPECS, COMPDAT, WCONPROD, etc.)
- 📊 RSM summary file parsing with time-series data extraction
- 📊 UNSMRY summary file parsing with time-series data extraction
- 📊 Binary .res files parsing with time-series data extraction
- ⚡ ContextManager for sequential simulation DATA file data processing with state management 
- ⚙️ Pipeline1 - easy way clean, parse and preprocess both input and output TNavigator data via YAML configuration file
- 🛢️ TNavigator full reservoir numerical schema implementation - ready for `numpy` and `torch` arrays
- 🎯 Event-driven scenario management with parameterized queries (WARNING: needs a review)
- 🔧 Comprehensive YAML configuration management
- 🖥️ Full-featured CLI with Click (WARNING: needs an update)
- 🧪 Complete testing suite with pytest using real reservoir data (WARNING: needs an update)
- 📝 Type hints and validation with Pydantic
- 🌐 NetworkX support for graph-based operations
- 🎨 Code formatting with Black and isort
- 🔍 Linting with flake8 and type checking with mypy

## Installation

**Requirements**: Python 3.10.15 or higher

### From Source

```bash
git clone https://github.com/GitHub-User228/omega-parser.git
cd omega-parser

# Install dependencies
pip install -r requirements.txt

# Install package in development mode
pip install -e ".[dev]"
```

### Using pip (when published)

```bash
pip install omega-parser
```

### For Development

```bash
# Clone and setup development environment
git clone https://github.com/GitHub-User228/omega-parser.git
cd omega-parser

# Install all dependencies including development tools
pip install -r requirements.txt
pip install -e ".[dev]"

# Or use make
make install-dev
```

## Quick Start

### Python API

#### 1. `Cleaner` & `MainParser` Usage

```python
from pathlib import Path
from omega_parser.utils.common import save_pkl
from omega_parser import MainParser, Cleaner

# Clean TNavigator simulation files
cleaner = Cleaner()
cleaner.clean(data_dir=Path("data/"), output_dir=Path("data_cleaned/"))

# Parse cleaned TNavigator simulation files
parser = MainParser(data_dir=Path("data_cleaned/"), verbose=1)
content = parser.read(rel_file_path="simulation.data")

# Access parsed keywords by section
pvdg_data = content.PROPS.get("PVDG")  # Pressure-volume-viscosity data
dates_data = content.SCHEDULE.timestamps  # Schedule dates with timestamps

# Save parsed content as pkl file
save_pkl(path=Path("output/data.pkl"), data=content)
```

#### 2. `RSMParser` & `RSMParserParallel` & `RSMDataExtractor` Usage

```python
from pathlib import Path
from omega_parser import RSMParallel, RSMDataExtractor

# Parse RSM summary files using batched multiprocessing approach
parser = RSMParserParallel(data_dir=Path("data/"), verbose=True)
df = parser.read(rel_file_path="summary.rsm", batch_size=1000)

# Extract well data from parsed RSM
extractor = RSMDataExtractor()
keywords = ['WGPT', 'WBP']
df_well = extractor.extract_well_data(data=df, keywords=keywords)

# Extract perforation data from parsed RSM
keywords = ['CGPT', 'CBP']
df_perf = extractor.extract_perforation_data(data=df, keywords=keywords)
```

#### 3. `UNSMRYParser` & `UNSMRYDataExtractor` Usage

```python
from pathlib import Path
from omega_parser import UNSMRYParser, RSMParallel
from omega_parser.data.simple import DType, DataLevel

# Parse UNSMRY summary files for `perforation` and `well` levels
keywords = {
  DataLevel.perforation: ['CGPR', 'CGPT'],
  DataLevel.well: ['WGPR', 'WGPT'],
}
parser = UNSMRYParser(data_dir=Path("data/"), verbose=1)
data = parser.read(keywords=keywords, io_mode='stream')
# or you can directly specify path to UNSMRY and SMSPEC files
# data = parser.read(
#   rel_file_path_to_smspec="file.SMSPEC",
#   rel_file_path_to_unsmry="file.UNSMRY",
#   keywords=keywords,
#   io_mode='stream',
# )

# Extract data from parsed UNSMRY for `perforation` and `well` levels
keywords = {
  DataLevel.perforation: {'CGPR': DType.float32, 'CGPT': DType.float32},
  DataLevel.well: {'WGPR': DType.float32, 'WGPT': DType.float32},
}
extractor = UNSMRYDataExtractor()
data = extractor.extract_data(data=data, keywords=keywords, verbose=1)
```

#### 4. `OutsolTsMapsResParser` Usage

```python
from pathlib import Path
from omega_parser import OutsolTsMapsResParser

# Parse OutsolTsMaps results file with blocks data
# PS:
#   to determine values for `blocks`, `features`, `dimensions`
#   you must know what was specified in .DATA file for corresponding keywords
parser = OutsolTsMapsResParser(data_dir=Path("data/"), verbose=1)
data = parser.read(
    blocks=12, # OR np.ndarray of shape (n_blocks, 3) with coords OR Path to .xyz file with coords
    features=2, # or ['PRES', 'SWAT'],
    keep_features_order=True, 
    selected_features=None, # or ['PRES']
    selected_timestamps=None, # or like ['2022-10-01']
    output_format='dataframe', # or 'tensor',
    dimensions=None, # or like (10, 20, 33) when output_format = 'tensor'
)
```

#### 5. `RunStatResParser`, `WellResParser`, `ConnResParser`, `GroupResParser`, `NetworkResParser` Usage

Here is an example for `RunStatResParser`:

```python
from pathlib import Path
from omega_parser import RunStatResParser

parser = RunStatResParser(data_dir=Path("data/"), verbose=1)
data = parser.read(offset=10)
```

The same idea can be used for other parsers but with a different `offset` value. Check the type annotation for `offset` argument in each case in order to determine the possible values - they may be different depending on the number of `objects` (e.g. wells, connections, groups, ...)

## Configuration

The library uses YAML configuration files for parser setup:

```yaml
# config/config.yaml
MainParserConfig:
  end_keyword: END
  extensions:
    - ".vfp"
    - ".VFP"
    - ".inc"
    - ".INC"
    - ".grdecl"
    - ".GRDECL"
    - ".map"
    - ".MAP"
    - ".data"
    - ".DATA"
  parsers:
    - UDQParser
    - PVDGParser
    - WECONParser
    - WEFACParser
    - VFPPRODParser
    - INCLUDEParser
    - GRUPTREEParser
    - DATESParser
    - TSTEPParser
    - RESTARTParser
    - WELSPECSParser
    - BRANPROPParser
    - COMPDATParser
    - WPIMULTParser
    - WDFACParser
    - WCONPRODParser
    - COMPLUMPParser
    - WVFPDPParser

RSMParserConfig:
  extensions:
    - '.rsm'
    - '.RSM'
  end_of_file_offset: 1


RSMParserParallelConfig:
  extensions:
    - '.rsm'
    - '.RSM'

UNSMRYParserConfig:
  extensions:
    - '.smspec'
    - '.unsmry'
    
# Res file parsers configurations
OutsolTsMapsResParserConfig:
  resfile_name: outsol_ts_maps
RunStatResParserConfig:
  resfile_name: run_stat
WellResParserConfig:
  resfile_name: well
ConnResParserConfig:
  resfile_name: conn
GroupResParserConfig:
  resfile_name: group
NetworkResParserConfig:
  resfile_name: network

# Keyword configurations
PVDGParserConfig:
  keyword: PVDG
WECONParserConfig:  
  keyword: WECON
```

### Configuration Options

- `MainParserConfig.extensions`: Supported file extensions for TNavigator files
- `MainParserConfig.parsers`: List of keyword parsers to enable
- `RSMParserConfig`: Configuration for RSM summary file parsing
- `RSMParserParallelConfig`: Configuration for RSM summary file parsing in parallel way
- `UNSMRYParserConfig`: Configuration for UNSMRY summary file parsing
- `*ResParserConfig`: Individual configuration for a separate .res file parsing
- `*ParserConfig`: Individual keyword parser configurations
- Logger configuration in `config/logger.yaml`

## ContextManager

The ContextManager provides sequential processing capabilities for TNavigator simulation data, managing state transitions through simulation timestamps.

### Key Features

- **State Management**: Tracks simulation state across timestamps with proper RESTART handling
- **Well and Perforations Management**: Monitors well and perforation specifications, constraints, and production controls
- **Network Topology**: Maintains reservoir network structure using NetworkX DiGraph
- **VFP Tables**: Manages vertical flow performance tables throughout simulation
- **Data Validation**: Ensures consistency and validates data at each simulation step

### Usage Examples

```python
from omega_parser import ContextManager
from omega_parser.data.simple import DatafileGroups

# Create from parsed content
cm = ContextManager.from_datafile(
    data_dir=Path("data/"), 
    rel_file_path="simulation.data"
)

### Alternative ways to create

# cm = ContextManager.from_parsed_content(
#     content=Path("output/data.pkl"), 
# )
# ```
# IMPORTANT: manager must be reinitialized and content must be read again 
# whenever you want to apply manager again
# ```
# cm = ContextManager.from_parsed_content(
#     content=content, 
# )
# ```
# Specify context if you already have initialized and saved context with cleared schedule data
# ```
# cm = ContextManager.from_parsed_content(
#     content=content, 
#     context=Path("output/context.pkl"), 
# )

# Initialize schedule data
cm.init_schedule_data()

# Access current state
print(f"Current timestamp: {cm.context.schedule.timestamp}")
print(f"Grid specs data: {cm.context.forecast.grid_specs}")
print(f"Well data: {cm.context.schedule.well_data}")
print(f"Perforation data: {cm.context.schedule.perforation_data}")
print(f"UDQ data: {cm.context.schedule.udq_data}")
print(f"PVT data: {cm.context.forecast.pvt_gas_data}")
print(f"SWFN data: {cm.context.forecast.saturation_water_function}")
print(f"SGFN data: {cm.context.forecast.saturation_gas_function}")
print(f"VFP data: {cm.context.schedule.vfp_data}")

# Perform a single step
cm.step()

# Perform multiple steps at once
cm.multistep(end_timestamp='2024-01-01')

# Or perform steps up until the very end
cm.multistep()

# Accumulate specific data for all timestamps
# This requires the context to be intialized again
cm = ContextManager.from_datafile(
    data_dir=Path("data/"), 
    rel_file_path="simulation.data"
)
data = cm.accumulate()

# Access accumulated data
print(f"Well data: {data[DatafileGroups.well]}")
print(f"Perforation data: {data[DatafileGroups.perforation]}")
print(f"Group data: {data[DatafileGroups.group]}")
print(f"Edge data: {data[DatafileGroups.edge]}")
print(f"VFP data: {data[DatafileGroups.vfp]}")
print(f"UDQ data: {data[DatafileGroups.udq]}")
```

## Pipelines 

The Pipelines system helps to parse and process TNavigator files in a easy way with a simple config file.

In order to use a pipeline, execute the following command in console:

```bash
# Runs a pipeline named `pipeline1` using `config.yaml` configuration file
python3 -m omega_parser.pipelines.run -p pipeline1 -cf config.yaml
```

### Pipeline 1

Pipeline1 is designed to parse raw .DATA and RESULTS files for either one scenario or more if they are 
based on the same adapted history model. 

For each scenario the following files must exist:  
- `.DATA` files and all included files
- `result.UNSMRY` and `result.SMSPEC` files

Here is an example of `config.yaml` file:

```yaml
default:
  base_dir: /home/username/project1/data # where the processed is saved
  scenarios: # ids of scenarios
    - v1
    - v2
    - v3
  start_timestamp: 2024-01-01 # timestamp before which all processed data is omitted

# Defines all data processing component in the right order
components:

  # Cleans .DATA files and all included files keeping the relative tree order
  DatafileCleaningComponent: 
    save_dir: 'cleaned' # subdirectory relative to base_dir where that processed data is saved
    skip: False # whether to skip this component
    verbose: 0 # verbosity level
    data_dir: /home/username/project1/tnav_model_data # directory with .DATA file and all included files

  # Parses cleaned .DATA and included files, creating a separate .parquet 
  # or .pkl file for each data level (e.g. well, perforation)
  DatafileParsingComponent:
    save_dir: 'parsed'
    skip: False
    verbose: 1
    # Adjust the float precision - float32 or float64
    float_precision: float64
    # path to a .DATA file for each scenario relative to `DatafileCleaningComponent.data_dir`
    scenarios_rel_dir:  
      v1: scenarios/v1_scen.DATA
      v2: scenarios/v2_scen.DATA
      v3: scenarios/v3_scen.DATA
    # what mnemonics to parse for each data level (id columns are excluded 
    # here since they are parsed by default)
    mnemonics: 
      well:
        - well_parent
        - well_I
        - well_J
        - VFP_id
        - WSTAT
        - WMCTL
        - WEFAC
        - WDFAC
        - exists
        - n_opened
      perforation:
        - CDFAC
        - COMPLUMP
        - WPIMULT
        - CTFAC
        - CDIAM
        - CEKH
        - CSKIN
        - CEFR
        - CSTAT
        - exists
      network_node:
        - PRES
        - node_fitting_flag
        - alq_flag
        - node_fitting_group_id
        - group_id_for_source_or_sink_node
        - network_type
      network_edge:
        - VFP_id
        - ALQ

  # Parses UNSMRY and SMSPEC results files, creating a separate .parquet 
  # or .pkl file for each data level (e.g. well, perforation)
  ResultsParsingComponent:
    save_dir: 'parsed'
    skip: False
    # .UNSMRY file parsing mode
    io_mode: stream
    # Adjust the final float precision - float32 or float64 (WARNING: the real data have float32)
    float_precision: float32
    # Whether to delete autogenerated smpspec.csv on completion
    delete_parsed_smspec: True
    # Whether to process each block feature separetely 
    # (works when block data is parsed in a separate group - like in this example)
    separate_block_features: False
    # path to a directory with results files
    data_dir: /home/username/project1/tnav_model_data/scenarios/RESULTS 
    # path to a directory with results files for history relative to `data_dir`
    history_rel_dir: hist
    # path to a directory with results files for each scenario relative to `data_dir`
    scenarios_rel_dir:  
      v1: v1_scen
      v2: v2_scen
      v3: v3_scen
    # what mnemonics (with dtypes) to parse for each data level 
    # (id columns are excluded here since they are parsed by default)
    mnemonics: 
      perforation:
        CBP: float32
        CGPR: float32
        CGPT: float32
        CWPR: float32
        CWPT: float32
      well:
        WEFF: float32
      group:
        GEFF: float32    
      block:
        BSWAT: float32  
    # what data levels to parse at the same time
    mnemonics_groups: 
      - [perforation, well, group]
      - [block]

  # Preprocesses data from DatafileParsingComponent and ResultsParsingComponent
  # Optionally adds and keeps only certain timestamps - gaps can also be handled
  PreprocessingComponent:
    save_dir: 'preprocessed'
    skip: False
    # Define different processing pipelines
    pipelines:
      # Example pipeline
      block_pipeline:
        skip: False
        processors:
          - method: _read_data_source
            kwargs:
              table_name: block
              alias: block
              origin: RESULTS
              mnemonics: [BSWAT]
          - method: _add_timestamps
            kwargs:
              table_name: block
              id_cols: 
              timestamps_to_keep:
                start: 2022-10-01
                end: 2035-01-01
                freq: MS
          - method: _linear_interp
            kwargs:
              x: 
              table_name: block
              features: [BSWAT]
              grouping_cols: 
          - method: _apply_time_filtering
            kwargs:
              table_name: block
          - method: _save_table
            kwargs:
              table_name: block
```

### Dataset from Pipeline1

Use `Dataset` from `omega_parser.data.dataset` in order to read and transform data processed via `Pipeline1` into a vectorized numpy format - this way the data can be processed much faster (there is also a support for fast merge operations without groupby). 

You will need two YAML configuration files.

This config defined data schema for `block`, `perforation`, `well`, `group` and `edge` data groups. You can specify only some of the data groups if needed.


```yaml
block: 
  extra_metadata_features: [SWATINIT, SWL, SWU] # Those features are derived from ContextContext.pkl
  features: [BSWAT] 
perforation:
  extra_metadata_features: [I, J, K, well_id]
  features: [CBP, BSWAT, CGPR]
well:
  extra_metadata_features: [well_parent]
  features: [WEFF, well_id]
group: 
  extra_metadata_features: []
  features: [GEFF]
edge: 
  extra_metadata_features: []
  features: [VFP_id, ALQ]
```

This config file defines directories from which to read the data. The configuration is based on how the `Pipeline1` saves the data. You can specify only some of the data groups if needed.


```yaml
data_dir: /artifacts/data
sources_path_config:
  edge:
    origin: PREPROCESSED
    filename: edge.parquet
  group: 
    origin: PREPROCESSED
    filename: group.parquet
  well:
    origin: PREPROCESSED
    filename: well.parquet
  perforation: 
    origin: PREPROCESSED
    filename: perforation.parquet
  block: 
    origin: PREPROCESSED
    filename: block.pkl
  graph: 
    origin: DATA
    filename: graph.pkl
  vfp: 
    origin: DATA
    filename: vfp.pkl
  udq: 
    origin: DATA
    filename: udq.pkl
```

Then follow the python script below in order to initialize the `Dataset`:

```python
from pathlib import Path

from omega_parser.data.dataset import Dataset

dataset = Dataset.from_parsed_data(
  schema_path=Path('path/to/first/yaml/file'),
  path_config_path=Path('path/to/seconds/yaml/file'),
)
dataset.add_scenario('v1')
dataset.add_scenario('v2')
```

Additionally, `Dataset` has `persistent_data` which is derived from `ContextContent.pkl` stored in `data_dir`

## TNavigator Full Reservoir Numerical Schema

Use the implementation of a full reservoir numerical schema in `reservoir/equation.py`.

The schema is fully described below. The equation below works for each block `i` and timestamp `t`:

<img src="docs/TNavSchema.png" alt="Logo" width="1000">

## Query Processing and Event Management (WARNING - needs a review and an updated)

The Query Processing system enables event-driven scenario management for TNavigator simulations, allowing you to modify simulation behavior through parameterized events defined in Excel queries.

### Key Features

- **Event Templates**: Define reusable simulation events with TNavigator keyword data
- **Parameter Validation**: Type-safe parameter handling with constraints and validation
- **Excel Integration**: Query-based scenario definition through Excel spreadsheets  
- **Date Validation**: Event scheduling with date range constraints
- **Context Integration**: Seamless integration with baseline simulation contexts

### Usage Examples

```python
from pathlib import Path
from omega_parser.query import QueryProcessor

# Initialize query processor with event schemas
processor = QueryProcessor(paths=[
    Path("schemas/field1_events.yaml"),
    Path("schemas/field2_events.yaml")
])

# Process scenario from Excel file
scenario_context = processor.get_scenario_data(
    context=baseline_context,  # ParsedContent from main parser
    field_alias="FIELD_A", 
    path=Path("scenarios/scenario_1.xlsx")
)

# Or process DataFrame directly  
query_data = processor.process_df(
    field_alias="FIELD_A",
    query=query_df  # DataFrame with event specifications
)

# Integrate with baseline context
updated_context = processor.add_data_to_baseline_context(
    context=baseline_context,
    query_data=query_data
)
```

### Event Schema Configuration

Events are defined in YAML schema files with the following structure:

```yaml
# field_events.yaml
field_alias: "FIELD_A"
events:
  well_constraint_change:
    name: "Well Constraint Modification"
    description: "Modify well production constraints"
    file: "well_constraint_template.data"  # TNavigator template file
    ffill: true  # Forward-fill through simulation
    date_min: "2020-01-01"
    date_max: "2025-12-31"
    parameters:
      oil_rate_max:
        keyword: "WCONPROD"
        param_name: "oil_rate_max"
        param_type: "float"
        param_min: 0.0
        param_max: 10000.0
      well_cond:
        keyword: "WCONPROD"
        param_name: "well_cond"
        param_type: "str"
        param_values: ["OPEN", "SHUT", "STOP"]
```

### Excel Query Format

Scenario queries are defined in Excel files with the following columns:

| Column | Description | Type | Required |
|--------|-------------|------|----------|
| `event_id` | Event identifier from schema | str | Yes |
| `param_name` | Parameter to modify | str | Optional |
| `param_value` | Parameter value | any | Optional |
| `event_date` | Event execution date | str/datetime | Yes |
| `is_enabled` | Enable/disable event | bool | Yes |

Example Excel content:
```
event_id                | param_name     | param_value | event_date | is_enabled
well_constraint_change  | oil_rate_max   | 5000.0      | 2023-06-01 | TRUE
well_constraint_change  | well_cond      | SHUT        | 2023-12-01 | TRUE
```

### Event Template Files

Event templates are standard TNavigator input files that serve as templates for event data:

```TNavigator
-- well_constraint_template.data
WCONPROD
  'WELL_01' 'OPEN' 'ORAT' 2.0 4* /
/
```

Parameters like `oil_rate_max` get substituted with values from the query.

### Validation and Error Handling

The system provides comprehensive validation:

- **Type Checking**: Parameter values validated against defined types
- **Range Validation**: Numeric parameters checked against min/max constraints  
- **Value Sets**: String parameters validated against allowed value lists
- **Date Constraints**: Events validated against date range limits
- **Template Consistency**: Parameter names verified in template files


## Development

### Setup Development Environment

```bash
# Install development dependencies
make install-dev

# Or manually
pip install -e ".[dev,docs]"
pre-commit install
```

### Code Quality

```bash
# Run all checks
make check-all

# Individual commands
make test          # Run tests (WARNING - needs a review)
make test-cov      # Run tests with coverage (WARNING - needs a review)
make lint          # Run linting
make format        # Format code
make type-check    # Run type checking
```

### Testing (WARNING - needs a review)

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=omega_parser --cov-report=html

# Run specific test markers
pytest -m "not slow"           # Skip slow tests
pytest -m integration          # Run only integration tests

# Test MainParser with dummy simulation data
pytest tests/test_main_parser.py -v

# Test RSM parser with sample data
pytest tests/test_rsm_parser.py -v
```

### Building and Publishing

```bash
# Build package
make build

# Upload to PyPI (requires authentication)
make upload
```

## Project Structure

```
omega-parser/
├── src/omega_parser/             # Main package
│   ├── __init__.py               # Package initialization
│   ├── cli.py                    # CLI implementation
│   ├── errors.py                 # Custom exceptions
│   ├── logger.py                 # Logging utilities
│   ├── parsers/                  # Parser modules
│   │   ├── __init__.py
│   │   ├── base.py               # Base parser classes
│   │   ├── main_parser.py        # TNavigator file parser
│   │   ├── settings.py           # Configuration management
│   │   ├── keyword_parsers/      # TNavigator keyword parsers (18 parsers)
│   │   │   ├── __init__.py
│   │   │   ├── pvdg.py           # PVDG data parser
│   │   │   ├── wecon.py          # Well constraints parser
│   │   │   ├── dates.py          # Schedule dates parser
│   │   │   ├── welspecs.py       # Well specifications
│   │   │   ├── compdat.py        # Completion data
│   │   │   ├── wconprod.py       # Well production control
│   │   │   └── ...               # Other keyword parsers
│   │   ├── extra_parsers/        # Additional file parsers
│   │   │   └── rsm.py            # RSM summary file parser
│   │   │   └── rsm_parallel.py   # RSM summary file parser with batched multiprocessing apporoach
│   │   │   └── unsmry.py         # UNSMRY summary file parser
│   │   └── resfile_parsers/      # RESULTS files parsers
│   │       ├── base.py           # Base parser
│   │       ├── outsol_ts_maps.py # OutsolTsMaps binary file parser
│   │       ├── run_stat.py       # RunStat binary file parser
│   │       ├── conn.py           # Conn binary file parser
│   │       ├── well.py           # Well binary file parser
│   │       ├── group.py          # Group binary file parser
│   │       └── network.py        # Network binary file parser
│   ├── context/                  # Context management
│   ├── cleaner/                  # TNavigator model data cleaning
│   ├── query/                    # Query processing and event management
│   │   ├── __init__.py           # Query module initialization
│   │   └── process.py            # Query processor implementation
│   ├── config/                   # Configuration files
│   │   ├── config.yaml           # Parser configurations
│   │   └── logger.yaml           # Logging configuration
│   ├── utils/                    # Utility functions
│   ├── package_data/             # Package data
│   │   └── keyword_section.json # Keyword-section mappings
│   ├── data/                     # Data models and structures
│   │   ├── event.py              # Event management models
│   │   └── query.py              # Query data models
│   │   └── ...                   # Other data structures
│   └── pipelines/                # Data parsing and processing pipelines
│   │   ├── __init__.py               
│   │   ├── run.py                # Script to execute a pipeline from console
│   │   └── pipeline1/            # Pipeline for parsing and processing .DATA and RESULTS files
│   └── reservoir/                # TNavigator full reservoir numerical schema implementation
├── tests/                        # Test suite
│   ├── test_main_parser.py       # MainParser tests
│   ├── test_rsm_parser.py        # RSM parser tests
│   └── ...                       # Other tests
├── data/                         # Test and sample data
│   ├── test_data/                # TNavigator simulation test files
├── .github/workflows/            # GitHub Actions CI/CD
├── pyproject.toml                # Project configuration
├── Makefile                      # Development commands
├── requirements.txt              # Dependencies
└── CHANGELOG.md                  # Lists notable changes in all versions of the parser
└── README.md                     # This file
```

## Code Style

This project uses:
- **Black** for code formatting
- **isort** for import sorting
- **flake8** for linting
- **mypy** for type checking
- **pre-commit** hooks for automated checks

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Changelog

All major changes between versions are listed in [CHANGELOG.md](CHANGELOG.md) 