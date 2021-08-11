# JFR Performance Analysis

This project is used to do a performance analysis of the JDK Flight Recorder feature in OpenJDK and GraalVM. It runs a specified program from [jfr-tests](https://github.com/rh-jmc-team/jfr-tests) a set number of times, as both a GraalVM native image, and as a standard OpenJDK Java program, monitoring the performance, and generating CSVs with all the performance data, which can then be analyzed with the provided spreadsheet template.




## Setup

The `setup` script is used to perform all the necessary setup tasks. The tasks to perform are specified by the options given to the script.

```
Usage: sh setup [OPTIONS]
Perform setup tasks.

Task-specifying options:
  -a perform all setup tasks
  -d install dependencies
  -g get GraalVM
  -o get OpenJDK
  -j get jfr-tests
  -t set toprc

Non-task-specifying options:
  -h print help

Optional environment variables:
  GRAALVM_URL the URL from which to download a prebuilt GraalVM
  OPENJDK_URL the URL from which to download a prebuilt OpenJDK
  JFR_TESTS_URL the URL with which to clone a jfr-tests Git repository
```

### `-a` perform all setup tasks

This option runs all the available setup tasks.

### `-d` install dependencies

This task installs all the necessary dependencies with `dnf`.

### `-g` get GraalVM

This task downloads and extracts a prebuilt GraalVM. By default, it uses the latest [graalvm-ce-dev-builds](https://github.com/graalvm/graalvm-ce-dev-builds/releases) release, but `GRAALVM_URL` can be used to specify a different prebuilt GraalVM.

### `-o` get OpenJDK

This task downloads and extracts a prebuilt OpenJDKp. By default, it uses an AdoptOpenJDK [openjdk11-upstream-binaries](https://github.com/AdoptOpenJDK/openjdk11-upstream-binaries/releases/) release, but `OPENJDK_URL` can be used to specify a different prebuilt OpenJDK.

### `-j` get jfr-tests

This task clones a jfr-tests Git repository. By default it uses [rh-jmc-team/jfr-tests](https://github.com/rh-jmc-team/jfr-tests.git), but `JFR_TESTS_URL` can be used to specify a different repository.

### `-t` set toprc

This sets the user's `toprc` to the `toprc` in this repository. This step is always necessary, because the the script will not be able to parse the output from `top` otherwise.


## Run

The `run` script is used to do a performance analysis. It assumes that `setup` has been run and `build/` contains all the necessary projects, or that the appropriate environment variables are set, if a project is not in `build/`.

```
Usage: sh run [OPTIONS] PROGRAM
Do a performance analysis.

Options:
  -h print help
  -n RUNS set the number of test runs

Optional environment variables:
  GRAALVM_HOME the path to a build of GraalVM
  JAVA_HOME the path to a build of OpenJDK
  JFR_TESTS the path to a jfr-tests Git repository
```

### Test Program

The fully qualified name of a jfr-tests class must be given, and this class will be the test program that gets run. The class must be the non-clean version (e.g. `TestMultipleEvents`, not `TestMultipleEventsClean`), as the script relies on the test program to start the JFR recording.

### Results

Seven CSV files make up the results of the performance analysis. They are stored in `results/`.

#### GraalVM Native Image Size

`GraalVM_SIZE.csv` contains a single number, which is the size of the GraalVM native image, in bytes.

#### CPU Usage

`GraalVM_CPU.csv` and `OpenJDK_CPU.csv` contain CPU usage data for the test program, when run as a GraalVM native image, and as a standard OpenJDK Java program, respectively. The files contain lines equal to the number of test runs done, and each line corresponds to the data from the test run of the same ordinal as the line number (the first line contains data for the first test run, etc.). Each line contains all the `CPU%` data output by `top`, for its test run.

#### Memory Usage

`GraalVM_RES.csv` and `OpenJDK_RES.csv` contain memory usage data for the test program, when run as a GraalVM native image, and as a standard OpenJDK Java program, respectively. The files contain lines equal to the number of test runs done, and each line corresponds to the data from the test run of the same ordinal as the line number (the first line contains data for the first test run, etc.). Each line contains all the `RES (KiB)` data output by `top`, for its test run.

#### Runtime Data

`GraalVM_TIME.csv` and `OpenJDK_TIME.csv` contain runtime data for the test program, when run as a GraalVM native image, and as a standard OpenJDK Java program, respectively. The files contain lines equal to the number of test runs done, and each line corresponds to the data from the test run of the same ordinal as the line number (the first line contains data for the first test run, etc.). Each line contains contains the elapsed, system, and user time of its test run, in that order.





## Beaker

This script is designed to be used in a Beaker environment, where a simple and consistent one-line command that sets up everything from scratch and does a performance analysis, with no human interaction, is desired. This script will handle all the required setup, and then do a performance analysis of the specified test program, with 100 runs each of GraalVM and OpenJDK. 
```
Usage: sh beaker PROGRAM
```




## Spreadsheet

The `template.xslx` spreadsheet template in this repository can be used as an easy way to analyze the results from a performance analysis. It calculates the average, median, minimum, and maximum numbers for all the different CSVs. Note that the median, minimum, and maximum numbers are calculated over all the numbers in a CSV. For example, the minimum of `GRAALVM_RES.csv` is the smallest number that appears in `GRAALVM_RES.csv`, and not the average of the smallest number from each line in `GRAALVM_RES.csv`.

To use this spreadsheet template, open it in a spreadsheet program, and then import all the CSVs into the same spreadsheet document, as a seperate sheet. The formulas in the template reference the CSVs by name, so their names must not be changed, but the order in which they are imported does not matter. The cells containing the formulas may need to be manually forced to recalculate after importing the CSVs (in Google Sheets this can be done by pressing `ENTER` on all of them). 
