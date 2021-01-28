# TESAsamSim
## Closed-loop simulation package for salmon CUs for 2021 TESA workshop

-----  

**Authors: Carrie Holt, adapted from samSim by Cameron Freshwater & revised by Kendra Holt**  
**Date: 2021-01-27 (ONGOING)**

-----



### Summary
This repository contains the necessary files to run stochastic closed-loop simulations parameterized with Pacific salmon stock-recruitment data. The principal function, `genericRecoverySimulator()`, is intended to simulate the population dynamics for multiple populations of Pacific salmon and then evaluate the performance of different management procedures (broadly a mix of harvest control rules and assessment methods) across operating models representing distinct ecological hypotheses. A suite of performance metrics are generated that allow analysts to evaluate different management procedures ability to achieve multiple, interacting conservation- and catch-based objectives. In short, the model is intended to provide a framework for the quantitative component of a management strategy evaluation. This simulation model is a more generic, simplifed version of 'samSim' initially developed to evaluate recovery strategies applied to Fraser River sockeye and Nass chum salmon.

The focal unit of the simulated dynamics are conservation units (CUs) - genetically distinct aggregates of one or more spawning populations that are unlikely to recolonize in the event of extirpation. Under Canada's Wild Salmon Policy these will be the target of future rebuilding strategies. Thus the `TESAsamSim` package (and the original `samSim` package) are well suited for evaluating management procedures for CUs in a mixed-stock context, but should not be used to evaluate the dynamics of subpopulations within CUs and should only be used to evaluate multiple management units (distinct aggregates of CUs managed quasi-independently) with care. Although this model simulates multiple CUs within an aggregate simulataneously, for the purposes of this TESA workshop, we will be using time-series from only 1 CU. In particular, for this workshop we will simulate the dynamics for 5 CUs of coho salmon from the interior Fraser River as documented in Arbeider et al. (2020), but will explore performance of time-varying assessment models on only 1 CU. Future work can incorporate multi-CU dynamics.

A summary of relevant files and how to run a simulation are provided below. Most functions contain relatively detailed documentation (and sometimes functioning examples). Details on the operating model (biological dynamics and fishery interactions) and the management procedures (harvest control rule and assessment process) will be provided in a vignette to come. The following documentation is adapted from that for the `samSim` package.


[After installing the package development prerequisites](https://support.rstudio.com/hc/en-us/articles/200486498) install `samSim` with:

```{r}
devtools::install_github("Pacific-salmon-assess/TESAsamSim")
```

-----

### Files
All files are stored in the following directories:

#### data

Includes data input files. For the 2021 TESA workshop, we will be using example data from Interior Fraser River coho salmon in the directory, *IFCohoPars*. See Input Files section below.

#### man
Includes the .rd files used to populate help files for each function. Created automatically via `roxygen`.

#### outputs
Directory generated automatically by running `genericRecoverySimulator()`. Contains a *diagnostics* directory that includes diagnostic plots and *simData* that includes output data files summarizing performance. Note that in practice this specific output directory may not be used by the analyst because its contents are only populated when the source code is ran or when the function is run within the `TESAsamSim` package, which is not necessarily recommended. **However** an equivalent directory is automatically generated by `TESAsamSim` when it is first run in a new working directory (see *Running a simulation* below for clarification).

#### R
Contains `genericRecoverySimulator()` function as well as necessary helper and post-processing functions.

#### Rmd
Includes Rmarkdown files that include an example simulation run, as well as descriptions model structure and parameterization.

#### src
Includes scripts necessary for several helper C++ functions.

------

### Running a simulation

Simulations are run by installing the samSim package and using the `genericRecoverySimulator()` function. Generally this should occur in a fresh working directory (e.g. a new `.Rproj`), which will automatically generate an `outputs` directory and necessary subdirectories. Parameter values are passed to the function using a series of .csv files with the `simPar` and `cuPar` arguments being most critical. `simPar` contains parameter values that are shared among CUs and define a given scenario (e.g. species, simulation length, OM and MP characteristics). `cuPar` contains parameters that are CU-specific including at a minimum names and SR model type, but typically stock-recruit parameters as well. See *Input file details* below. Details of how to pass a suite of scenarios to the simulation model are provided in `Rmd/exampleSimRun.Rmd`.

------

### Input file details 

#### `cohoSimPar`
`cohoSimPar` is a .csv file that contains the input parameters that characterize a specific simulation run, but which are *shared* among CUs. Each row represents a unique scenario (i.e. combination of operating model and management procedure). Generally it is easiest to create multiple `simPar` input files, each of which contain a coherent analysis (e.g. one input focusing on the effects of different harvest control rules across changing productivity regimes, a second input examining the effects of survey effort), but this is not strictly necessary. Contents include:
  
  - `scenario` - scenario name
  - `nameOM` - operating model name
  - `nameMP` - management procedure name
  - `keyVar` - focal variable of the analysis; subjective since typically multiple variables will differ among scenarios, but should be a focal point of main figures. Currently can be one of the following arguments: `prodRegime`, `synch`, `expRate`, `ppnMix`, `sigma`, `endYear`, `adjustAge`, `mixOUSig`, `adjustForecast`, `adjustEnRoute`, `obsSig`, `obsMixCatch` (**NOTE, to be defined explicitly**)
  - `plotOrder` - order in which grouped scenarios will be plotted (useful when keyVar is not an ordinal or numeric variable)
  - `species` - lower case species name (chum and sockeye have been tested robustly; pink and coho have not; chinook should be used with extreme caution since most stocks do not meet assumptions of the model)
  - `simYears` - length of the simulation period (excluding priming period)
  - `harvContRule` - harvest control rule (`TAM`, `fixedER`, `genPA`)
  - `benchmark` - biological benchmark used to assess conservation status (`stockRecruit`, `percentile`)
  - `canER` - total Canadian exploitation rate
  - `usER` - American exploitation rate (note can also be supplied as CU-specific value in `cuPars`)
  - `propMixHigh` - proportion of Canadian catch allocated to mixed-stock fisheries (can range from 0 to 1)
  - `enRouteMortality` - on/off switch for en route mortality
  - `constrain` - if `TRUE` and harvest control rule is TAM then mixed stock fisheries are constrained
  - `singleHCR` - single stock harvest control rule (`FALSE`, `retro`, `forecast`)
  - `moveTAC` - if `TRUE` and single stock quota from low-abundance CUs is re-allocated to other CUs
  - `prodRegime` - productivity regime (`low`, `lowStudT`, `med`, `studT`, `skew`, `skewT`, `decline`, `divergent`, `oneUp`,  `oneDown`, `high`, `scalar`, `increase`). The regime `med` represents a stable value based on the median value. The regime `linear` represents a linear change in productivitiy over the length of the trend period followed by stable values, where the propotional change over the length of the trend is specified by `prodPpnChange`.The regimes `decline` and `increase` represent declines to 65% and increases to 135% of current productivity estimates over the length of the trend, followed by stable levels. 
  - `prodPpnChange` - the proportional change in productivity over the trend period when the `prodRegime` is linear.
  - `trendLength` - indicates the number of years over which there is a trend (if `prodRegime == "decline"` or `"increase"`, or `capRegime == "decline"` or `"increase"`).
  - `capRegime` - capacity regime (`med`, `linear`, `decline`, `increase`). The regime `med` represents a stable value based on the median value. The regime `linear`represents a linear change in capacity over the length of the trend period followed by stable values, where the propotional change over the length of the trend is specified by `capPpnChange`. The regimes `decline` and `increase` represent declines to 65% and increases to 135% of current capacity estimates over the length of the trend, followed by stable levels.
  - `capPpnChange` - the proportional change in capacity over the trend period when the `capRegime` is linear.
  - `rho` - temporal autocorrelation coefficient in recruitment deviations
  - `arSigTransform` - if `TRUE` estimates of sigma from input are transformed so that they account for temporal autocorrelation
  - `correlCU` - the correlation among CUs in recruitment deviations
  - `corrMat` - if `TRUE` a custom correlation matrix is required to be passed as an input and is used to specify the covariance matrix for recruitment deviations
  - `tauCatch` - logistic variation in CU-specific catches
  - `obsSig` - log-normal variation in spawner observation error 
  - `mixOUSig` - beta-distributed variation in mixed-stock fishery outcome uncertainty; input parameter represents the standard deviation used to calculate the location parameter
  - `singOUSig` - beta-distributed variation in single-stock fishery outcome uncertainty; input parameter represents the standard deviation used to calculate the location parameter
  - `obsMixCatch` - log-normal variation in mixed-stock catch observation error
  - `obsSingCatch` - log-normal variation in single-stock catch observation error
  - `obsAgeErr` - logistic variation in observed age error
  - `lowCatchThresh` - lower aggregate catch target (used as a performance metric)
  - `highCatchThresh` - upper aggregate catch target (used as a performance metric)
  - `extinctThresh` - quasi-extinction threshold
  - `adjustSig` - scalar on CU-specific sigmas (recruitment deviations)
  - `adjustAge` - scalar on `tauCatch`
  - `adjustEnRouteSig` - scalar on en route mortality rates


#### `cohoCUPars`
`cohoCUPars` is a .csv file that contain CU-specific input parameters. Note that these parameters should *not* vary among simulation runs. Differences in operating models that involve CU-specific traits (e.g. population dynamics) can typically be introduced via options in the `cohoSimPar` file. Each row represents a specific CU. 

Mandatory contents include:

  - `manUnit` - management unit
  - `stkName` - CU name
  - `stk` - CU identification number (can be assigned arbitrarily or based on previous modeling exercises)
  - `model` - stock-recruit model used to forward simulate dynamics (`ricker`, `larkin`)
  - `minER` - minimum Canadian exploitation rate
  - `alpha` - productivity parameter for Ricker models
  - `beta0` - density-dependence parameter for Ricker models
  - `sigma` - recruitment variation for Ricker models
  - `meanRec2` - mean proportion of age-2 recruits
  - `meanRec3` - mean proportion of age-3 recruits
  - `meanRec4` - mean proportion of age-4 recruits
  - `meanRec5` - mean proportion of age-5 recruits
  - `meanRec6` - mean proportion of age-6 recruits
  - `medianRec` - median historical recruitment
  - `lowQRec` - 25th percentile historical recruitment
  - `highQRec` - 75th percentile historical recruitment

Optional contents include:
  
  - Necessary if modeling cyclic stocks
    - `domCycle` - integer to identify the dominant cycle line in Larkin stocks (`1, 2, 3, 4` or `NA`)
    - `tauCycAge` - logistinc variation in age structure
    - `larkAlpha` - productivity parameter for Larkin models
    - `larkBeta0` - density-dependence parameter for Larkin models
    - `larkBeta1` - lag-1 density-dependence parameter for Larkin models
    - `larkBeta2` - lag-2 density-dependence parameter for Larkin models
    - `larkBeta3` - lag-3 density-dependence parameter for Larkin models
    - `larkSigma` - recruitment variation for Larkin models
  - Necessary if American exploitation differs among CUs
    - `usER` - American exploitation rate
  - Necessary if modeling en route mortality
    - `meanDBE` - mean difference between estimates (a proxy for en route mrotality)
    - `sdDBE` - interannual standard deviation of difference between estimates
  - Necessary if modeling TAM harvest control rule
    - `medMA` - median mortality adjustment used in TAM harvest control rule
  - Necessary if modeling forecast process
    - `meanForecast` - mean forecast relative to observed
    - `sdForecast` - interannual standard deviation of forecast

#### `cohoRecDatTrm`

`cohoRecDatTrm`  is a .csv file that contains the historical spawner and recruitment data for plotting purposes. The column labels are as follows:
  - `stk` - CU identification number (can be assigned arbitrarily or based on previous modeling exercises)
  - `yr` - brood year
  - `ets` - effective total spawner numbers, accounting for proportional spawning success
  - `totalSpwn` - total spawner numbers. For interior Fraser Coho, this is equal to ets.
  - `rec2` - abundance of adult recruitment at age 2, aligned by brood year
  - `rec3` - abundance of adult recruitment at age 3, aligned by brood year  
  - `rec4` - abundance of adult recruitment at age 4, aligned by brood year
  - `rec5` - abundance of adult recruitment at age 5, aligned by brood year
  - `rec6` - abundance of adult recruitment at age 6, aligned by brood year
  
### Changes from samSim package 

1) Removing species-specific code references. Life history types will be specified through new fields in the CUPars.csv input file (e.g., firstAgeRec, maxAgeRec, obsBYlag)
2) Removing performance measures specific to the TAM harvest control rule option used for Fraser sockeye
3) Adding a new SR model option that includes a marine survival co-variate (needed in the short-term for Interior Fraser Coho)
4) Adding performance measures needed to calculate LRPs
