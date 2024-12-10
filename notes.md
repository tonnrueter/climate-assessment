- From `MAGICC7_AR6.R`
  - What's `generateIIASASubmission` again?
  - `remindReportingFile`:
  - `climateAssessmentYaml`: from `piamInterfaces`
  - `climateAssessmentEmi`: Obtain from harmonization & infilling


# Understanding MAGICC7 using the `climate-assessment` toolchain

!! DO NOT RUN OF WINDOWS !!

## General `climate-assessment` stuff
- `climate-assessment`: Call chain `run_harm_inf` -> `harmonize_and_infill` -> `_harmonize_and_infill` -> `harmonization_and_infilling` (separate module)
- What files do we need and where do they come from?
  - Emission data! Example case comes with `climate-assessment`: `ar6_IPs_emissions.csv`
  - AR6 Probabilistic Distribution (comes with the MAGICC7 distribution!): `0fd0f62-derived-metrics-id-f023edb-drawnset.json`
  - CMIP6 infiller data base, also comes with `climate-assessment`: `cmip6-ssps-workflow-emissions_infillerdatabase_until2100.csv`
- Harmonized and infilled data dumped in C:\Users\tonnru\lab\climate-assessment\data\output-magicc-example-notebook
- Questions
  - What's `scmdata`'s job?

## Trial run `run-example-magicc.ipynb`
- Current show stopper

```
CalledProcessError: Command '['C:\\Users\\tonnru\\lab\\MAGICC7\\magicc-v7.5.3\\bin\\magicc.exe', '--version']' returned non-zero exit status 3221225595.
```

- When calling `.\magicc.exe --version` process does not throw an error but also does not return anything

- When running the whole thing on WSL ` ./magicc --version` yields `v7.5.3` as should be

### Log analysis
- 1st step: Harmonization. Cleaned up log:

```python
2024-12-06 14:36:39 climate_assessment.harmonization MainThread - INFO:  harmonization_year 2015
2024-12-06 14:36:39 climate_assessment.harmonization MainThread - INFO:  Stripping equivalent units for processing
2024-12-06 14:36:39 climate_assessment.harmonization MainThread - INFO:  Creating pd.DataFrame's for aneris
2024-12-06 14:36:39 climate_assessment.harmonization MainThread - INFO:  Adding 2015 values based on historical percentage offset from 2010
2024-12-06 14:36:39 climate_assessment.harmonization MainThread - INFO:  Interpolating onto output timesteps
2024-12-06 14:36:39 climate_assessment.harmonization MainThread - INFO:  Harmonisation overrides:
[..  prints long list of overrides and associated methods, e.g. constant_ratio, reduce_offset_2150_cov, etc. ..]
2024-12-06 14:36:44 climate_assessment.harmonization MainThread - INFO:  Hacking around some regression in aneris - pyam stack
2024-12-06 14:36:44 climate_assessment.harmonization MainThread - INFO:  Combining results
2024-12-06 14:36:44 climate_assessment.harmonization MainThread - INFO:  Putting equiv back in units
2024-12-06 14:36:44 climate_assessment.harmonization MainThread - INFO:  Converting to IamDataFrame
2024-12-06 14:36:44 climate_assessment.harmonization MainThread - INFO:  CHECK: if some emissions are zero in the harmonization year.
```


- 2nd step: Infilling. Group batches with shared interpolation (?) methods

```python
2024-12-06 14:36:44 climate_assessment.harmonization MainThread - INFO:  CHECK: if some emissions are zero in the harmonization year.
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Infilling database: <path to climate-assessment dev>\tests\test-data\cmip6-ssps-workflow-emissions_infillerdatabase_until2100.csv
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  CFC infilling database: <path to site-packages>\climate_assessment\infilling\cmip6-ssps-workflow-emissions.csv
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Loading infilling database
2024-12-06 14:36:44 scmdata.run MainThread - INFO:  Reading <path to climate-assessment dev>\tests\test-data\cmip6-ssps-workflow-emissions_infillerdatabase_until2100.csv
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Loading infilling database cfcs
2024-12-06 14:36:44 scmdata.run MainThread - INFO:  Reading <path to site-packages>\climate_assessment\infilling\cmip6-ssps-workflow-emissions.csv
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Preparing infilling databases
# [.. pyam.core complains about an empty df ..]
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Infilling using ['Emissions|CO2'] as the lead variable
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Nothing to infill
# [.. complains about empty dfs ..]
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Infilling using ['Emissions|CO2|Energy and Industrial Processes'] as the lead variable
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Interpolating data to infill
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Infilling using cruncher <class 'silicone.database_crunchers.quantile_rolling_windows.QuantileRollingWindows'>
# [.. no complaints here ..]
2024-12-06 14:36:44 climate_assessment.infilling MainThread - INFO:  Infilling ['Emissions|BC', 'Emissions|CH4', 'Emissions|CO2|AFOLU', 'Emissions|CO', 'Emissions|N2O', 'Emissions|NH3', 'Emissions|NOx', 'Emissions|OC', 'Emissions|Sulfur', 'Emissions|VOC']
2024-12-06 14:36:51 climate_assessment.infilling MainThread - INFO:  Infilling using cruncher <class 'silicone.database_crunchers.rms_closest.RMSClosest'>
# [.. new variables ..]
2024-12-06 14:36:50 climate_assessment.infilling MainThread - INFO:  Infilling ['Emissions|HFC|HFC134a', 'Emissions|HFC|HFC143a', 'Emissions|HFC|HFC227ea', 'Emissions|HFC|HFC23', 'Emissions|HFC|HFC32', 'Emissions|HFC|HFC43-10', 'Emissions|HFC|HFC245ca', 'Emissions|HFC|HFC125', 'Emissions|SF6', 'Emissions|PFC|CF4', 'Emissions|PFC|C2F6', 'Emissions|PFC|C6F14']
2024-12-06 14:36:50 silicone.database_crunchers.constant_ratio MainThread - INFO:  <class 'silicone.database_crunchers.constant_ratio.ConstantRatio'> won't use any information from the database
# NOTE: THE methods and the variables might not match up due to multi threading
# Random check prolly due to concurrent logging
2024-12-06 14:36:50 silicone.database_crunchers.constant_ratio MainThread - WARNING:  Note that the lead variable ['Emissions|CO2|Energy and Industrial Processes'] goes negative.
# [.. f gases go separate ..]
2024-12-06 14:36:51 climate_assessment.infilling MainThread - INFO:  Infilling ['Emissions|CCl4', ..lots of F gases here ,'Emissions|SO2F2']
2024-12-06 14:36:55 climate_assessment.infilling MainThread - INFO:  Converting HFC/PFC units back from CO2-equivalent
2024-12-06 14:36:57 climate_assessment.infilling MainThread - INFO:  Post-processing for climate models
# CHECKS
2024-12-06 14:36:57 climate_assessment.infilling MainThread - INFO:  Checking infilled results have required years and variables
2024-12-06 14:36:58 climate_assessment.infilling MainThread - INFO:  Check that there are no non-CO2 negatives introduced by infilling
```

- 3rd step: Dump to file

```python
2024-12-06 14:36:58 climate_assessment.harmonization_and_infilling MainThread - INFO:  Writing infilled data as csv to: ..\data\output-magicc-example-notebook\ar6_IPs_emissions_harmonized_infilled.csv
2024-12-06 14:36:58 climate_assessment.harmonization_and_infilling MainThread - INFO:  Writing infilled data as xlsx to: ..\data\output-magicc-example-notebook\ar6_IPs_emissions_harmonized_infilled.xlsx
```

