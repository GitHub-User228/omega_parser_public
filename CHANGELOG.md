
# Changelog

## v0.1.0

- Initial release
- TNavigator simulation file parsing
- RSM summary file parsing with time-series extraction
- Support for major keywords: PVDG, WECON, DATES, GRUPTREE, VFPPROD
- YAML configuration support
- CLI interface
- Basic test suite with dummy simulation data

## v0.1.1

- Added support for keywords: WEFAC

## v0.1.2

- Added support for keywords: WELSPECS

## v0.1.3

- Reworked the parsing process
- Improved the time performance

## v0.1.4

- Added support for keywords: BRANPROP, COMPDAT, COMPLUMP, WCONPROD, WDFAC, WPIMULT

## v0.1.5

- Added support for keywords: WVFPDP

## v0.1.6

- Reworked the parsing process (kws, timestamp, vfpprod, etc.)

## v0.2.0

- Added ContextManager for sequential processing of TNavigator simulation data with proper state management
- Restructured the code architecture with parsers in dedicated modules
- Enhanced keyword parser support (now 18 parsers total)

## v0.2.1

- Added perforation data management support in ContextManager 

## v0.2.2

- Added support for keywords: DIMENS, SATNUM, SWFN, SGFN
- Added regions_specs, saturation_water_function and saturation_gas_function tables for ContextContent
- Added reset function for ContextManager

## v0.2.3

- Added support for keywords: WELOPEN, WCONHIST
- Improved ContextManager (added support for new keywords)

## v0.3.0

- Added comprehensive event-driven scenario management system
- Introduced QueryProcessor for Excel-based scenario queries
- Added Event, EventParameter, EventTemplate, and EventsRegistry models
- Implemented parameter validation with type checking and constraints
- Added Excel integration for scenario definition and query processing
- Introduced event templates with TNavigator keyword substitution support
- Fixed bugs in ContextManager
- Added support for PVTW keyword
- Added pvt_water_data to ContextContent

## v0.4.0

- Added Cleaner for cleaning TNavigator-model files from comments 
- Cleaned model files are saved in a separate folder with preserved relative structure
- Improved the performance of MainParser (if the parsing cleaned files)
- Now, it is REQUIRED to apply Cleaner before parsing model data (if you set do_cleaning=False in MainParser)
- Added support for parsing GEFAC keyword.


## v0.4.1

- Added `multistep` function to the ContextManager that can perform multiple steps at once
- Added `accumulate` function to the ContextManager that can accumulate well and perforation data for all timestamps

## v0.4.2

- Added UNSMRY file parser (yet to add this parser to CLI)
- Added support for keywords SWATINIT and GRAVITY

## v0.4.3

- Added support for keyword START
- Fixed invalid timestamp handling in SCHEDULE section (both in MainParser and ContextManager)

## v0.4.4

- Added support for keyword GCONPROD (not full yet)
- Fixed an issue with line cleaning code when expression for DEFINE keyword gets cut off after cleaning

## v0.4.5

- Fixed issue with MainParser when the data for the timestamp right before END is not collected
- Updated ContextManager, so that the default values for certain columns are applied + some updates in update method for certain keywords

## v0.4.6

- Added RSMDataExtractor which extracts well or perforation data from parsed RSM data

## v0.5.0

- Added support for keyword parsers related to blocks coordinates and their features (COORD, MAPAXES, ZCORN, ACTNUM, NTG)
- Added mapaxes and grid_specs sections to the data updated in the ContextManager. Grid data also contains info about volume and coordinates of each block (it is calculated) 
- Reworked RSMDataExtractor
- Reworked RSMParser so that it is compatitable with different RSM file formats

## v0.6.0

- Complete overhaul of the keyword parsers and DATAFILE main parser - the changes fix the issue when the keywords beyond schedule section can be processed not in the stated order (e.g. now the ParsedContent object have a field named keywords_order that holds keywords with their sections in the order of the appearance)
- ContextManager was adapted to the changes in the parser
- A few new keywords can be parsed now
- Method save of ParsedContent is disabled for now since it needs adaptation to the changes in the parsers.
- Infinite loop occuring in RSMParser has been removed

## v0.6.1

- Optimized RSMParser - now it works 40-50% faster
- Minor bug fixes in ContextManager and QueryProcessor
- Added generate_summary_file to utils/summary which can generate a summary.inc file with desired mnemonics (including block mnemonics)

## v0.6.2

- Added RSMParserParallel - faster version of RSMParser that utilizes batched multiprocessing approach (works roughly 2x faster than RSMParser)
- Bug fixes in generate_summary_file, MainParser and Cleaner 
- MainParser now ignores data in SUMMARY section
- Added missing libs to pyproject.toml

## v0.6.3

- Opimized RSMParserParallel - now it works roughly 4x faster than RSMParser

## v0.6.4

- Reworked ContextManager so that possibly the same non-schedule data can be reused and loaded directly

## v0.6.5

- Added equil_data table to the ContextContent
- Added parsers for the following keywords (EQLNUM, EQUIL)
- ContextManager now can retrieve EQLNUM and EQUIL data from parsed data
- Expanded default lists with mnemonics for which the summary section file is generated via generate_summary_file

## v0.7.0

- Rework of _update method for all keyword parsers
- Rework of _update_persistent_keyword_data method in ContextManager
- Added parsers for the following keywords: ARR, ARITHMETIC
- Added ArithmeticEvaluator utility class that evaluates expressions specified within keyword ARITHMETIC
- Renamed coordinate-related columns in the tables of ContextContent to match the TNavigator notation
- Docstring update in certain places

## v0.7.1

- Rework of ContextManager:
  - Now it processes keywords in the given order for each timestamp
  - Fixed CDFAC calculation
  - Grid specs data not is added at the starting timestamp and not during collection process
  - Block size (DX, DY, DZ) can be calculated now.
  - Simplified initialization proccess with default values for schedule data
  - Some other code simplifications
- Rework of ContextContent
  - Created a separate class to store non-schedule data: NonScheduleContextContent
  - Created a separate class to store schedule data: ScheduleContextContent
  - ContextContent now stores data using classes above
- Code fixes due to the renaming (x -> I, y -> J, z -> K)
- Bug fixes in PVDG and PVTW parsers
- Documentaion fix in certain places

## v0.7.2

- Bug fix in RSMParserParallel
- Bug fix in ContextManager (missing data occured for SATNUM in perf data)
- Fixed CEFR calculation in ContextManager


## v0.7.3

- Take into account days when building a datetime index in `RSMDataExtractor`


## v0.7.4

- Formatted the code where necessary
- Added FIPNUM keyword parser
- Fixed a bug in `RSMParallelParser`
- Fixed a bug in `RSMDataExttactor`
- Added new extractor function `extract_region_data` to `RSMDataExttactor`

## v0.7.5

- Added a parser for binary data stored in outsol_ts_maps.res file

## v0.7.6 (contribution from @Vladislav0504)

- Improved the .inc file generator for summary section (summary.py)
- Improved `UNSMRYParser`

## v0.7.7

- Enhanced `OutsolTsMapsParser` so that you can parse data only for specific features and specific timestamps

## v0.7.8

- Fixed an issue in `OutsolTsMapsParser` where the timestamps for forecast data is not correctly extracted
- Added an option to select another the output format of the parsed data when using  `OutsolTsMapsParser`

## v0.7.9

- Added parsers for the following keywords: COMPOFF, GCONSUMP, GECON, NETBALAN, NETCOMPA, NODEPROP, NWATREM, VFPCHK, WORKLIM, WORKTHP, WTEST, WVFPEXP

## v0.8.0 (contribution from @Vladislav0504)

- Added `UNSMRYDataExtractor` which extracts data for requested levels from parsed UNSMRY data via `UNSMRYParser`

## v0.8.1

- Fixed a docstring typo in `OutsolTsMapsParser`
- Changed column naming in parsers: `well_cond` to `WSTAT`, `well_policy` to `WMCTL`, `perf_cond` to `CSTAT`
- Added `pipeline1` pipeline which parses and preprocesses TNavigator data.
- Added dtype adjustment postprocessing for most of keyword parsers (where it was neccessary)
- Reworked `ContextManager`:
  - added support for handling ground network related data: `gruptree_graph`, `branprop_graph`, `network_edge_data`, `network_node_data` 
  - added dtype adjustments when the schedule data is intialized
  - added dtype adjustments for processed data at the end of `.accumulate` method
  - now `grid_specs` data is not added to `perforation_data` dataframe (in order to save memory)
  - added `verbose` atrribute so that the logging level can be modified
  - changed the output of `.accumulate` method - now key is an attribute from `DataGroups`, not `string`
- Minor adjustments in the utilities that are used in `ContextManager`
- `ParsingUtils` now removes odd patterns defined in `ODD_PATTERNS` in `utils/constant.py`
- Added graph related utilities to `utils/graph.py`
- Added a parser for keyword `PVTNUM`
- Optimized `UNSMRYParser` - now it works faster

## v0.8.2

- Reworks and fixes `ContextManager`:
  - Removed unrelated import
  - Now `.accumulate` returns perforation data only for active blocks
  - Switched from `np.uint8` to `pd.UInt8Dtype` (and similarly for other uint types) for certain columns in certain dataframes
  - Dtype optimizations
  - Now `.accumulate` returns data with no NaNs - they are replaced with 0 - this is how it is done in TNavigator
- Dtype optimizations for certain keyword parsers (like `np.uint8` -> `pd.UInt8Dtype`)
- Fixed docstring typos in `DatafileParsingComponent`
- Changed column naming in parsers: `perf_I` to `I`, `perf_J` to `J`, `perf_K` to `K`, `perf_K_end` to `K_end`
- Reworks in `ResultsParsingComponent` of `pipeline1`:
  - `well_IJK` column is added to `perforation` data
  - blocks coordinates are now parsed and saved into a single .pkl file
- Added a new component to `pipeline1`: `PreprocessingComponent` - it is desinged to merge and preprocess data from `ResultsParsingComponent` and `DatafileParsingComponent`
- Now `UNSMRYDataExtractor` drops duplicated timestamps for `block` level data

## v0.9.0

- Added dataclasses to accumulate and store `vfp` and `udq` data for all timestamps: `VFPData` and `UDQData`
- Reworks and fixes `ContextManager`:
  - Network data now processed differently via `edge`, `group`, `history_graph` and `forecast_graph` data structures
  - `vfp` and `udq` data is processed differently - data is accumulated into `VFPData` and `UDQData`
  - Removed network graph from the output of `.accumulate` method
  - Added `vfp`, `udq`, `group` and `edge` data structures to the output of `.accumulate`
- Some updates and fixes to keywords parsers
- Added a bunch of utility functions to deal with a `graph` to `utils/graph.py`
- Added a bunch of utility functions to deal with numpy arrays to `utils/math.py`
- Reworks and fixes in `pipeline1`:
  - `DatafileParsingComponent`: Names of groups, wells, ... can be truncated via `name_max_length` parameter of the config
  - A complete overhaul of `PreprocessingComponent` - now you can easily define several complex processing pipelines out of available processing functions in order to modify and save data from `ResultsParsingComponent` and `DatafileParsingComponent`
- Added `Dataset` dataclass that stores data for all levels (from `group` to `block` + `vfp`, `udq` and `graph` data) in completely vectorized numpy format - this way the data can be processed much faster (there is also a support for fast merge operations without groupby). This dataclass can be initialized from the data processed via `pipeline1`.
- Moved change logs from `README.md` to `CHANGELOG.md`

## v0.9.1

- Reworked `Pipeline1`:
  - Improved `_merge` function in `PreprocessingComponent` of `Pipeline1` so that you can specify how to handle common columns in the right and left tables
  - `DatafileParsingComponent` and `ResultsParsingComponent` can now handle history data
- Added an error message to indicate that some of the keywords passed to `UNSMRYParser` do not exist
- Reworked `Dataset`:
  - Added `cumsum` `agg_type` option to a list of functions that can be applied for mergers from `get_merger`
  - Added region mapping features for `perforation` and `block` level data
  - `features` now store data as `List[NDArray]` instead of `Dict[str, NDArray]` - basically, the features can be splitted into several groups and stored together in the same array. `features_idx` and `features_names` attributes are created in order to get the location of the features by their names
  - Added `group_to_scenarios_mask` to `metadata` which helps to mark only valid scenarios for each perforation (i.e. not all perfortaions might exist for some scenarios)
  - `add_scenario` function is added - it can be used to add a new scenario to the `Dataset`
  - `add_extra_data` function is added - it can be used after adding scenarios data to add extra data to the dataset (like merging functions or region mappers)
- `UNSMRYDataExtractor` now correctly returns extracted data if `keywords` is specified as `List[str]`


## v0.9.2

- Added parsers for the following keywords: KRG, KRGR, KRW, KRWR, SGCR, SGL, SGU, SWCR, SWU
- Changed column naming in the `WVFPDPParser`: `BHP_delta` to `WVFPDPBD`, `BHP_factor` to `WTPLSF`
- Added a possibility to adjust the float precision when parsing .DATA file via `MainParser` - use `float_precision` argument when initializing the parser (two available options: `float32` and `float64`)
- Improved `UNSMRYParser`:
  - You can now auto-delete auto-generated `smspec.csv` file - use `delete_parsed_smspec` argument when initializing the parser
  - You can now specify time bounds via `start_timestamp` and `end_timestamp` within which the data is kept (in `read` and `_read_unsmry`). If specified, the binary data outside the time bounds is ignored and not loaded into RAM at all
  - Some code quality improvments in `_read_unsmry`
- Improved `UNSMRYDataExtractor`:
  - Added a possibility to adjust the float precision (WARNING: this is purely a cosmetic thing since the max float precition of the real data is `float32`) - use `float_precision` argument when calling `extract_data` (two available options: `float32` and `float64`)
  - Rewroked the code of `extract_data` so that it properly handles the case when the passed parsed data is empty
- Improved `ContextManager`
  - The manager can now retrieve the float precision from the passed parsed data and follow that precision in all float-related calculations
  - Now the manager calculates geometric transmissibility between blocks in all directions for all blocks (`TRANx+`, ..., `TRANz-`) - this new data is added to `context.GRID` dataframe. You can check new corresponding functions that are used to calculate in `utils/grid.py`: `calculate_contact_squares`, `calculate_contact_centres`, `calculate_contact_normals`, `calculate_geometric_transmissibility`
- Improved `Pipeline1`
  - Added `end_timestamp` basic configuration parameter
  - `DatafileParsingComponent` supports float precition adjustment via the configuration
  - `ResultsParsingComponent` supports float precition adjustment via the configuration
  - `ResultsParsingComponent` correctly handles a case when an empty data is passed through `_add_well_IJK`
  - `ResultsParsingComponent` supports a new configuration parameter `delete_parsed_smspec` (see changes in `UNSMRYParser`)
  - `ResultsParsingComponent` supports a new configuration parameter `separate_block_features` which tells the component to parse and save the data for each block level feature separately
  - `PreprocessingComponent` supports `VFPData` and `UDQData` for several functions
  - Added `_time_weighted_avg` function to `PreprocessingComponent` - calculates a timedelta averaged feature for new timestamps added to `BlockData` (`_add_timestamps` must be applied beforehand for `BlockData`)
  - Quality of code improvments in `_merge` and `_read_data_source` function of `PreprocessingComponent`


## v0.9.3

- The error message is logged and returned for raising via a single function `error` from `utils.common`
- Improvments in `Dataset`:
  - `Dataset` can be initialized from a specified `ContextContent` binary pkl file via `cc_filename` argument added to `add_scenarios` and `add_scenario` functions
  - Added `perforation` to `block` merger to default mergers (if necessary id data exists for both levels)
- Changes in `UNSMRYDataExtractor`:
  - Creates empty columns for non-existing keywords if `create_empty_column_if_not_exists` is `True`. Value in `empty_value` is used as a placeholder value. Those two arguments can be changed in `extract_data` function.
- Changes in `UNSMRYParser`:
  - Use `error_if_keyword_not_exist` argument in `_read_unsmry` or `read` functions in order to not raise an error if some of the requested keywords do not exist. By default an error is raised.
- Changes in keywords parsers:
  - Added a new update function `_update_template3` to the `BaseKeywordParser` - it is now used for `WPIMULTParser` instead of `_update_template2`
  - `upper` is applied for values in `WSTAT` column by default in `WCONPRODParser`
- Changes in `pipeline1`:
  - Added previously mentioned new arguments `error_if_keyword_not_exist`, `create_empty_column_if_not_exists` and `empty_value` to `ResultsParsingComponentConfig`
  - `force_original_bounds` argument can now be specified for function `_add_timestamps` in `PreprocessingComponent`
  - Initial pressure data is now automatically added to `ContextContent` file when `ResultsParsingComponent` is called - a new file is created called `ContextContentNew.pkl` with this feature added to a forecast `GRID` dataframe. Also the history `GRID` is removed from the new file and data only for the active blocks is kept in the forecast `GRID`. `ContextContentNew.pkl` filename is used by default in the `Dataset` during initialization.
- Added a new code section with an implementaion of a numerical schema that is used in the `reservoir` in the `TNavigator`. Check `reservoir.equation.right_side`, `reservoir.equation.left_side` and `reservoir.equation.residual` in order to compute schema related data for all blocks. The functions are desgined to work with either `numpy` or `torch` arrays for either `2D` or `3D` case.

## v0.9.4

- Reworked and extended the parsers for binary .res files:
  - Improved `BaseResParser` to include the common logic behind parsing any .res file
  - Renamed `OutsolTsMapsParser` to `OutsolTsMapsResParser`
  - Reworked `OutsolTsMapsResParser` taking into account the changes in `BaseResParser`
  - Added a parser for the following .res files:
    - `conn.res`: `ConnResParser`
    - `well.res`: `WellResParser`
    - `group.res`: `GroupResParser`
    - `network.res`: `NetworkResParser`
    - `run_stat.res`: `RunStatResParser`
- Several updates in `README.md`