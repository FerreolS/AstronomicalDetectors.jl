# AstronomicalDetectors [![Build Status](https://travis-ci.com/emmt/AstronomicalDetectors.jl.svg?branch=main)](https://travis-ci.com/emmt/AstronomicalDetectors.jl) [![Build Status](https://ci.appveyor.com/api/projects/status/github/emmt/AstronomicalDetectors.jl?svg=true)](https://ci.appveyor.com/project/emmt/AstronomicalDetectors-jl) [![Coverage](https://codecov.io/gh/emmt/AstronomicalDetectors.jl/branch/main/graph/badge.svg)](https://codecov.io/gh/emmt/AstronomicalDetectors.jl)

`AstronomicalDetectors` is a Julia package to deal with the calibration files
of astronomical detectors like those of the VLT/Sphere instrument.

Example of usage:

```julia
using AstronomicalDetectors, Glob
list = scan_calibrations(glob("SPHER.2015-12-2*", dir))
data = read(CalibrationData{Float64}, list; roi=(501:580,601:650))
calib = ReducedCalibration(data)
```

## YAML configuration file

To deal with the numerous configuration of the different instruments, all calibrations files and keywords can be described in a YAML configuration file (see in the [config zoo folder](zoo)).

Usage:

```julia
using AstronomicalDetectors, ScientificDetectors
data = ReadCalibrationFiles("ymlfile.yml"; dir="path/to/calib/folder")
calib = ReducedCalibration(data)
```

A YAML file should be as follow :

```yaml
title: Detector calibration

suffixes: [.fits, .fits.gz,.fits.Z]
include subdirectory: true
exclude files: M.

exptime: "ESO DET1 SEQ1 DIT"

categories:
  DARK:
    ESO DPR TYPE: DARK
    sources: dark

  FLAT:
    ESO DPR TYPE: FLAT
    ESO ANOTHER KEYWORD: VALUE
    dir: my/flat/folder
    sources: dark + flat

  WAVE:
    exptime: "ESO OTHER DIT"
    ESO DPR TYPE:  ["LAMP,WAVE","WAVE,LAMP"]
    sources: dark + wave
```

The mandatory keywords are:

- `exptime` : the FITS keyword containing the integration time
- `categories` : lists all calibration categories (e.g. FLAT, DARK,...)
- `sources` : the sources corresponding to the parent category (mandatory for each category).

In this example, the calibration files are identified by their `ESO DPR TYPE` keyword.  If several values are allowed, they can be given in a array (as for the `WAVE` category in this example). If several keywords are given (as for the `FLAT` category) all of them must be valid.

Optional keywords are:

- `dir` :  the folder containing the calibration files (default `pwd`),
- `files` : list of files or pattern,
- `suffixes` : suffixes of the files  (default `[.fits, .fits.gz,.fits.Z]`),
- `include subdirectory` : if `true` parse all subdirectories
- `exclude files` : patterns of files that must be excluded
- `hdu` : name of the FITS HDU that contains the data (default `primary`).

All the keywords given in the root of the file are set for all the categories (as `suffixes` in the example) but this can be overiden by keywords in each category (as `exptime` in the example).

### Night ranges

to specify a range for the `DATE-OBS` keyword:

```yaml
DATE-OBS:
    min: 2021-04-01
    max: 2021-04-02T12:07:09.8206
```

As for other keywords, You can do it globally in the YAML or put it in a category.
Min and max bounds are inclusive. Both are required. Please note that `DateTime("2022-04-01") == DateTime("2022-04-01T00:00:00.000")`, it matters when you set a maximum date.
Supported date formats are ISO Date, ISO DateTime from the Julia Dates package.
Note that some FITS files use four digits for "milliseconds". In this case the fourth digit is thrown. If you have a file with the `DATE-OBS= 2021-11-25T12:07:09.8206`, and you specify the max range `2021-11-25T12:07:09.8206`, your file will correctly be included.
