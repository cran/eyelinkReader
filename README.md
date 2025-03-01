# eyelinkReader <a href="https://alexander-pastukhov.github.io/eyelinkReader/"><img align="right" src="https://raw.githubusercontent.com/alexander-pastukhov/eyelinkReader/refs/heads/master/man/figures/logo.svg" alt="Logo" height="138" style="float:right; height:138px;"/></a>
<!-- badges: start -->
[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.7050482.svg)](https://doi.org/10.5281/zenodo.7050482)
[![CRAN status](https://www.r-pkg.org/badges/version/eyelinkReader)](https://cran.r-project.org/package=eyelinkReader)
<!-- badges: end -->

R package to import eye tracking recording generated by [SR Research](https://www.sr-research.com/) EyeLink eye tracker from  EDF-files. It includes options to import events and/or recorded samples and extract individual events such as saccades, fixations, blinks, and recorded variables.

## Installation

These instructions are also available as a vignette.

The library installation involves three easy (famous last words) steps.

### Install SR Research EyeLink Developers Kit
This package relies on _edfapi_ library that is as part of the _EyeLink Developers Kit_. Therefore, `read_edf()` function **will not work without it** but you will still be able to use utility functions. The _EyeLink Developers Kit_ can be downloaded from [www.sr-research.com/support](https://www.sr-research.com/support/) website. Note that you need to register and wait for your account to be activated. Next, follow instructions to install _EyeLink Developers Kit_ for your platform. The forum thread should be under _SR Support Forum › Downloads › EyeLink Developers Kit / API › Download: EyeLink Developers Kit / API Downloads (Windows, macOS, Linux)_.

### Configure R environment variables

The package needs to configure compiler flags for its dependency on EDF API library. Specifically, it needs to specify paths to include header files (_edf.h_, _edf_data.h_, and _edftypes.h_) and to the library itself. The package will try to compile using sensible defaults for each platform, i.e., default installation paths for _EyeLink Developers Kit v2.1.1_. However, these defaults may change in the future or you may wish to install the library to a non-standard location (relevant primarily for Windows).

If compilation with default paths fails, you need to define R environment variables as described below. These variables must be defined either in user or project [.Renviron](https://stat.ethz.ch/R-manual/R-devel/library/base/html/Startup.html) file. The simplest way to edit it is via [usethis](https://usethis.r-lib.org/) library and [edit_r_environ()](https://usethis.r-lib.org/reference/edit.html) function. Type `usethis::edit_r_environ()` for user and `usethis::edit_r_environ('project')` for projects environments (note that the latter shadows the former, read [documentation](https://usethis.r-lib.org/) for details). Note that in the case of Windows, you do not need to worry about forward vs. backward slashes as R will normalize strings for you. Once you define the variables, restart session and check them by typing `Sys.getenv()` (to see all variables) or `Sys.getenv("EDFAPI_INC")` to check a specific one.

#### Windows
For the package to work on Windows, you must install [Rtools](https://cran.r-project.org/bin/windows/Rtools/). Please note that as of 31.05.2024, you need [Rtool43](https://cran.r-project.org/bin/windows/Rtools/rtools43/rtools.html) even if your R version is 4.4.x, the compilation fails with Rtools44.

Default values assume that the EyeLink Developers Kit is installed in `c:/Program Files (x86)/SR Research/EyeLink` (default installation path).

* `EDFAPI_LIB` : path to `edfapi.dll` for **32-bit systems**. Defaults to `c:/Program Files (x86)/SR Research/EyeLink/libs`.
* `EDFAPI_LIB64` (optional): path to `edfapi64.dll` for **64-bit systems**. By default, the 64-bit library is in _x64_ subfolder, i.e., `c:/Program Files (x86)/SR Research/EyeLink/libs/x64`. This variable is optional, as the package will try to guess this by itself by appending `/x64` to `EDFAPI_LIB` path. However, you should specify this variable explicitly if 64-libraries are in a non-standard folder (or SR Research changed it, or you just want to be sure).
* `EDFAPI_INC` : path to C header files necessary for compilation. Specifically, the package requires _edf.h_, _edf_data.h_, and _edftypes.h_. Defaults to `c:/Program Files (x86)/SR Research/EyeLink/Includes/eyelink`.

Your `.Renviron` file should include lines similar to the ones below
```
EDFAPI_LIB="c:/Program Files (x86)/SR Research/EyeLink/libs"
EDFAPI_LIB64="c:/Program Files (x86)/SR Research/EyeLink/libs/x64"
EDFAPI_INC="c:/Program Files (x86)/SR Research/EyeLink/Includes/eyelink"
```

#### Linux
* `EDFAPI_INC` : path to C header files necessary for compilation. Specifically, the package requires _edf.h_, _edf_data.h_, and _edftypes.h_. Defaults to `/usr/include/EyeLink`.

Your `.Renviron` file should include a line like this
```
EDFAPI_INC="/usr/include/EyeLink"
```

#### Mac OS

* `EDFAPI_LIB`: path to EDF API framework. Defaults to `/Library/Frameworks`
* `EDFAPI_INC` : path to C header files necessary for compilation. Specifically, the package requires _edf.h_, _edf_data.h_, and _edftypes.h_. Defaults to `/Library/Frameworks/edfapi.framework/Headers`

Your `.Renviron` file should include lines similar to the ones below
```
EDFAPI_LIB="/Library/Frameworks"
EDFAPI_INC="/Library/Frameworks/edfapi.framework/Headers"
```

### Install the library

To install from CRAN

```r
install.packages("eyelinkReader")
```

To install from github
```r
library("devtools")
install_github("alexander-pastukhov/eyelinkReader", dependencies=TRUE)
```

## Usage
The main function is `read_edf()` that imports an EDF file (or throws an error, if EDF API is not installed). By default it will import all events and attempt to extract standard events: saccades, blinks, fixations, logged variables, etc.
```r
library(eyelinkReader)
gaze <- read_edf('eyelink-recording.edf')
```

Events and individual event types are stored as tables inside the `eyelinkRecording` object with `trial` column identifying individual trials. Trial 0 correspond to events (e.g, `DISPLAY_COORDS` message, etc.) sent to the eye tracker before the first trial. 
```r
View(gaze$saccades)
```

There is a basic utility for plotting saccades and fixations for an individual trial
```r
plot(gaze, trial = 1, show_fixations = TRUE, show_saccades = TRUE)
```

or across trials
```r
plot(gaze, trial = NULL, show_fixations = TRUE, show_saccades = FALSE)
```

The package also includes functions to parse non-standard events: recorded areas of interest (`extract_AOIs`) and trigger events that help to time events (`extract_triggers`). These need to be called separately.

```r
gaze <- extract_AOIs(gaze)
gaze <- extract_triggers(gaze)
```

To import samples, add `import_samples = TRUE` and, optionally, specify which sample attributes need to be imported, e.g., `sample_attributes = c('time', 'gx', 'gy')`. All attributes are imported if `sample_attributes` is not specified. See package and EDF API documentation for the full list of attribute names.

```r
# import samples with all attributes
gaze <- read_edf('eyelink-recording.edf', import_samples = TRUE)

# import samples with selected attributes
gaze <- read_edf('eyelink-recording.edf',
                 import_samples = TRUE,
                 sample_attributes = c('time', 'gx', 'gy'))
```

See also a vignette on package usage. Please refer to documentation on `eyelinkRecording` class for details on events and samples.


## Further information on EDF file content

I have attempted to document the package as thoroughly as I could. However, for any question about specific attributes please refer to the EDF API manual and EyeLink documentation, which is supplied by SR Research alongside the library.

