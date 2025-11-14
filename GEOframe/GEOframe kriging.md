---
owner: "Daniele Andreis"
created: 2025-11-08 08:43
updated: 2025-11-08
framework: GEOframe
component: kriging
tags: [hydrology, geoframe, kriging]
---


# GlobalParameterEvaluator

## Overview

This module processes spatial and temporal data from measurement stations to compute variogram parameters. The module reads station and distance data, calculates experimental variograms, and then evaluates theoretical variogram parameters. It supports both standard and detrended modes, with optional logarithmic data transformation. When the local variogram estimation does not fit the data well, the module may default to using global variogram parameters (with or without trend adjustments) computed in the package `org.geoframe.blogpost.kriging.variogram.theoretical`.

**Dependencies:**

The module relies on several supporting classes:

- **ExperimentalVariogram**: Computes the experimental variogram from station data.
- **VariogramParametersCalculator**: Evaluates the variogram parameters using the experimental variogram.
- **VariogramParamsEvaluator**: Processes and fits the variogram model.
- **ResidualsEvaluator**: Assesses residuals for detrended analysis.
- **OmsTimeSeriesIteratorReader/Writer**: Handle time series data input/output.
- Other utility classes for data transformation and logging.

---

## Input Parameters (@In)

| Field Name | Data Type | Description |
| --- | --- | --- |
| `inStations` | `SimpleFeatureCollection` | The features collection containing measurement points with station positions. |
| `fStationsid` | `String` | The field that identifies each station by its ID. |
| `fStationsZ` | `String` | The field that specifies the elevation of the stations. |
| `pm` | `IHMProgressMonitor` | A progress monitor for tracking execution status. |
| `doIncludeZero` | `boolean` | Flag to include zeros in computations (default is true). |
| `pSemivariogramType` | `String` | Type of theoretical semivariogram (e.g., exponential, gaussian, spherical, etc.). If provided only this variogram will be evaluated. |
| `doDetrended` | `boolean` | Switch to enable detrended mode. |
| `cutoffInput` | `double` | The cutoff value used in variogram analysis. |
| `cutoffDivide` | `int` | Number of bins to use in the analysis. |
| `inHValuesPath` | `String` | Input file path for the distance measurements data. |
| `tStart` | `String` | Start time for the analysis. |
| `tEnd` | `String` | End time for the analysis. |
| `tTimeStep` | `int` | Time step in seconds for the analysis (default is 60). |
| `doLogarithmic` | `boolean` | Flag to apply logarithmic transformation to the data. |
| `inTheoreticalVariogramFile` | `String` | Output file path for saving the computed variogram parameters. |
| `getExperimentalVariogramData` | `boolean` | Flag to output detailed experimental variogram data (e.g., distances, variances, number of pairs). If true 3 additional files are providfed in the same folder of the variogram file output |
| `fileNoValue` | `String` | Indicator for no-value entries (default is "-9999"). |

---

## Output

### Output File

The module produces an output file (typically in CSV format) specified by the `inTheoreticalVariogramFile` parameter. This file contains the computed variogram parameters organized in **8 columns**, where each column corresponds to a specific parameter as derived from the `toHashMap()` method. The structure is as follows:

- **Column 0**: **Nugget** – The nugget effect value.
- **Column 1**: **Sill** – The sill value of the variogram.
- **Column 2**: **Range** – The range over which the variogram increases.
- **Column 3**: **Local Flag** – A flag (1.0 if the variogram is computed locally, 0.0 otherwise). Locally means at each time step; in other words, the regression provides reliable results. Otherwise, the parameters evaluated over the entire series are used.
- **Column 4**: **Trend Flag** – A flag (1.0 if a trend is present, 0.0 otherwise).
- **Column 5**: **Variogram Code** – A numerical code representing the variogram model (obtained from `getVariogramCode(modelName)`).
- **Column 6**: **Trend Intercept** – The intercept of the trend (if applicable).
- **Column 7**: **Trend Slope** – The slope of the trend (if applicable).

### Evaluation Note

In some cases, if the variogram computed from the current station data does not fit well (e.g., due to insufficient data or poor model fitting), the module may default to using the global variogram parameters. These can be either with trend adjustments or without, depending on the results of the evaluation. This fallback mechanism is implemented in the package `org.geoframe.blogpost.kriging.variogram.theoretical`, ensuring that the best available 

---

# Kriging

## Overview

The `Kriging` abstract class forms the foundation for spatial interpolation using the kriging algorithm. It extends the `HMModel` base class and provides a comprehensive framework to:

- Validate essential inputs.
- Initialize interpolation data providers.
- Select and process station data.
- Determine variogram parameters.
- Compute interpolation weights by solving a linear system.
- Calculate interpolated values and post-process the results.

The algorithm supports both standard and detrended modes, can apply logarithmic data transformation, and offers an option for parallel computation. The class is designed to be extended by concrete implementations (e.g., `KrigingPointCase`), which implement methods to store and output the final results.

**Dependencies:**

- **StationsSelection**: For selecting and filtering station data.
- **InterpolationDataProvider** (e.g., **VectorInterpolationProvider**): For extracting coordinates from input data.
- **TheoreticalVariogram**: For calculating covariance based on the variogram model.
- **StationProcessor**: For processing station data and computing residuals.
- **SimpleFeatureCollection** (GeoTools): To represent spatial features.
- **Coordinate** (JTS): For representing spatial coordinates.
- **HMModel**: The base model class.
- Other utilities such as linear system solvers (e.g., `SimpleLinearSystemSolverFactory`), and helper classes for post-processing.

---

## Input Parameters (@In)

| Field Name | Data Type | Description |
| --- | --- | --- |
| `inStations` | `SimpleFeatureCollection` | Shapefile containing measurement points with station positions. |
| `fStationsid` | `String` | Field in the station vector defining the station ID. |
| `fStationsZ` | `String` | Field in the station vector defining the elevation (required in detrended mode). |
| `pSemivariogramType` | `String` | Type of theoretical semivariogram (e.g., exponential, gaussian, spherical, etc.). |
| `inData` | `HashMap<Integer, double[]>` | Measured data to be interpolated, stored in a HashMap. |
| `pm` | `IHMProgressMonitor` | Progress monitor for tracking algorithm execution. |
| `doIncludeZero` | `boolean` | Flag to include zero values in computations. |
| `range` | `double` | Range parameter for the variogram (used in gaussian variogram). |
| `sill` | `double` | Sill parameter for the variogram (used in gaussian variogram). |
| `nugget` | `double` | Nugget parameter for the variogram (used in gaussian variogram). |
| `maxdist` | `double` | Maximum distance for considering neighboring stations. |
| `inNumCloserStations` | `int` | Number of closest stations to consider for interpolation. |
| `doDetrended` | `boolean` | Switch to enable detrended mode. |
| `linearSystemSolverType` | `String` | Specifies the type of solver for the linear system (default is "math3"). |
| `doLogarithmic` | `boolean` | Flag to apply logarithmic transformation to the data. |
| `boundedToZero` | `boolean` | Flag to bound negative interpolation results to zero. |
| `inTheoreticalVariogram` | `HashMap<Integer, double[]>` | Experimental variogram data used to build variogram parameters. |
| `fileNoValue` | `String` | Indicator for no-value entries (default is "-9999"). |
| `tStart` | `String` | Start time for the analysis. |
| `tTimeStep` | `int` | Time step in seconds for the analysis (default is 60). |
| `inIntercept` | `double` | Trend intercept for detrended mode. |
| `inSlope` | `double` | Trend slope for detrended mode. |
| `parallelComputation` | `boolean` | Flag to enable parallel computation of the kriging algorithm. |

---

## Output

The `Kriging` abstract class itself does not define a specific output field; however, its concrete implementations (e.g., `KrigingPointCase`) are responsible for storing and outputting the interpolation results—typically as a mapping from point IDs to interpolated values.

---

## Execution Flow

1. **Initialization and Verification:**
    
    The `execute()` method initiates the kriging process by verifying that all required inputs are present. In detrended mode, it ensures the presence of elevation fields.
    
2. **Interpolation Data Extraction:**
    
    The abstract method `initializeInterpolatorData()` is called to instantiate an `InterpolationDataProvider` that extracts coordinate data from the input.
    
3. **Station Selection and Variogram Determination:**
    
    A `StationsSelection` object is created and configured using the provided station data. Variogram parameters are determined either from the supplied experimental variogram (`inTheoreticalVariogram`) or computed using the input parameters (`pSemivariogramType`, `nugget`, `range`, `sill`).
    
4. **Kriging Computation:**
    
    For each interpolation point, the method:
    
    - Extracts the coordinates.
    - Selects neighboring stations (applying filtering based on maximum distance or number of stations).
    - Computes the covariance matrix and the known terms vector.
    - Solves the linear system to obtain interpolation weights.
    - Computes the interpolated value, including any detrended adjustment and post-processing (e.g., reversing logarithmic transformation and bounding values).
5. **Parallel Computation Option:**
    
    If `parallelComputation` is enabled, the interpolation points are processed concurrently using Java parallel streams, ensuring performance gains for large datasets.
    
6. **Result Storage:**
    
    The abstract method `storeResult(double[] result, HashMap<Integer, Coordinate> interpolatedCoordinatesMap)` must be implemented by subclasses to store the computed interpolated values.
    

---

## Methods Summary

- **execute():**
    
    Entry point that triggers the kriging process by selecting between sequential and parallel execution.
    
- **executeKriging() / executeKrigingParallel():**
    
    Implement the core logic of the kriging algorithm, handling initialization, variogram determination, station processing, interpolation, and result storage.
    
- **initializeKrigingParameters():**
    
    Constructs and returns the variogram parameters based on the user input or experimental variogram data.
    
- **createAndConfigureStationSelection():**
    
    Instantiates and configures a `StationsSelection` object using the input station data.
    
- **verifyInput():**
    
    Validates that all essential inputs are provided and correct, especially in detrended mode where elevation data is mandatory.
    
- **covMatrixCalculating() and knownTermsCalculation():**
    
    Compute the covariance matrix and known terms vector necessary for solving the kriging linear system.
    
- **postProcessResult():**
    
    Applies post-processing to the interpolated values, such as reversing logarithmic transformations and ensuring values are not negative if required.
    
- **storeResult(...):**
    
    Abstract methods to save the interpolation results; these must be implemented by concrete subclasses.
    
- **round(double value, int places):**
    
    Utility function for rounding values to a specified number of decimal places.
    

---

**2025-03-04 16:08:00**

**Confidence: 100/100**

Below is the Markdown documentation for the package **org.geoframe.blogpost.kriging.pointcase**.

---

# KrigingPointCase

## Overview

The **KrigingPointCase** module extends the abstract **Kriging** class to perform vector-based interpolation. It validates the required inputs, initializes an interpolation data provider, and stores the interpolation results. This class is used to compute interpolated values at specified spatial points and output the results as a HashMap. The algorithm supports both standard and detrended modes, ensuring that in detrended mode, elevation information is available for both stations and interpolation points.

**Dependencies:**

- **Kriging**: The base class providing common kriging functionality.variogram
- **InterpolationDataProvider** and **VectorInterpolationProvider**: Manage the preparation of input interpolation data.
- **SimpleFeatureCollection**: (GeoTools) A collection of spatial features representing the interpolation points.
- **Coordinate**: (JTS) Represents spatial coordinates for the interpolation process.
- Other supporting classes from the **org.geoframe.blogpost.kriging** package, which implement various aspects of the kriging algorithm.

---

## Input Parameters (@In)

| Field Name | Data Type | Description |
| --- | --- | --- |
| `inInterpolate` | `SimpleFeatureCollection` | The vector of points at which interpolation must be performed. |
| `fInterpolateid` | `String` | The field in the interpolated vector that defines the point ID. |
| `fPointZ` | `String` | The field in the interpolated vector that defines the elevation. (Required in detrended mode.) |

---

## Output Parameters (@Out)

| Field Name | Data Type | Description |
| --- | --- | --- |
| `outData` | `HashMap<Integer, double[]>` | A HashMap containing the interpolation results. Each key corresponds to a point ID and the associated value is an array of interpolated values. |

---

## Methods Overview

- **verifyInput()**
    
    This method validates that all essential input parameters are provided. It checks that the interpolation ID field (`fInterpolateid`) is not null, and if detrended mode is enabled, it also verifies that the elevation field (`fPointZ`) exists in the schema of `inInterpolate`.
    
- **initializeInterpolatorData()**
    
    This method initializes the interpolation data provider by creating a new instance of **VectorInterpolationProvider** using the provided interpolation points, ID, and elevation fields. This prepares the data for the interpolation process.
    
- **storeResult(double[] result, int[] id)**
    
    A private helper method that stores the interpolation results in the `outData` HashMap. It maps each interpolated value from the `result` array to the corresponding point ID provided in the `id` array.
    
- **storeResult(double[] result, HashMap<Integer, Coordinate> interpolatedCoordinatesMap)**
    
    An overridden method that first converts the key set of the provided coordinates map into an integer array and then calls the private `storeResult(double[], int[])` method to save the interpolation results.
    

---

## Use in OMS console

To use OMS console, first, ensure that the **lib** folder contains only one **kriging library**. Otherwise, classes with the same signature may conflict, making it unclear which one is being used.

The basic usage (which don’t include variable transformation, which are in charge of otehr library) are substantiali 2 case:

- fixed theoretical variogram.
- a theoretical variogram for each time step.

In the second case the first step are use  GlobalParameterEvaluator to get the theoretical variogram values as [[GEOframe kriging]] here an example of sim file.

Then  we use this file as input in the KrigingPointCase, here an example.

```jsx
import static oms3.SimBuilder.instance as OMS3

def home = oms_prj

OMS3.sim() {
    resource "$oms_prj/lib"

    model() {
        components {
            "vreader_station"   "org.hortonmachine.gears.io.shapefile.OmsShapefileFeatureReader"
            "gParamEvaluator"   "org.geoframe.blogpost.kriging.variogram.theoretical.GlobalParameterEvaluator"
        }

        parameter {
            "vreader_station.file"                "${home}/data/meteo_data/stations_tot.shp"
            // Name of the field with the name of station in the shapefile of meteo station
            "gParamEvaluator.fStationsid"         "ID"
            "gParamEvaluator.fStationsZ"          "quota"
            "gParamEvaluator.tStart"              "1990-01-01 01:00"
            "gParamEvaluator.tTimeStep"           "60"
            // Name of the field of subbasin in the shapefile of interpolation point (subbasin centroids)
            "gParamEvaluator.doDetrended"         "True"
            "gParamEvaluator.doIncludeZero"       "True"
            "gParamEvaluator.doLogarithmic"       "False"
            "gParamEvaluator.cutoffDivide"        "10"
            "gParamEvaluator.inHValuesPath"       "${home}/data/meteo_data/temperature_gf_2.csv"
            // Parameter of the writing component
            "gParamEvaluator.inTheoreticalVariogramFile" "${home}/data/kriging/params/temperature_param.csv"
        }

        connect {
            "vreader_station.geodata"  "gParamEvaluator.inStations"
        }
    }
}

```

Why don’t we directly connect **GlobalParameterEvaluator** with **KrigingPointCase**?

Because, in our application, we need a CSV file for each **hydrologic response unit (HRU)**. This means we have a separate **.sim** file for each HRU. If we were to directly connect the two modules, the same values would be computed multiple times, leading to unnecessary redundancy.