# dev-tips

Tips and Tricks for developing with Cantera

## How to work with this repo

Please edit this README file with the tips and tricks that you come up with. As we collect information, we'll try to organize it in a useful way, but we didn't want to impose some organization beforehand. Please try to put your tips into header sections.

Please make your changes in a branch and make a pull request to add it to `master`. You will not be able to fork this repository to your own account (since this is a private repository), so you will be making branches on this repository. Otherwise, the workflow is basically the same as with a forked repository.

## Run Python Using Modified Code

If you want to test your changes with Python, you can use the `PYTHONPATH` environment variable to add Cantera to the set of directories that Python searches when you `import`. You can use this to run a script or run the interactive interpreter (regular Python or IPython).

In the GIF below, you can see `scons build` running and then the magic incantation:

```console
$ PYTHONPATH=build/python python
```

This opens up `python` add adds the `build/python` directory to Python's search list. From the root of Cantera's source code, the `build/python` directory contains the Python module once you run `scons build`. Then, you can see the `__init__.py` file being modified, run `scons build` again, and the changes to `__init__.py` are reflected when you reload the module.

![PYTHONPATH tip](images/pythonpath-tip.gif)

By using the same trick with `PYTHONPATH`, you can load IPython, or specify a script name that Python should run.

## Run a Single Python Test Case

You can run a subset of the Python tests using the `unittest` module. Note that this doesn't work for the Google Tests (any C++ tests). There are two tricks to this one. First, you have to add `PYTHONPATH=build/python`. Then, you need to identify the test(s) you want to run. All of the Python tests are in the folder `interfaces/cython/cantera/tests`. You can run a single test file:

```console
$ PYTHOPATH=build/python python -m unittest -v cantera.test.test_convert
```

The `PYTHONPATH` is the same as the last tip. Then you run `python -m unittest`, where the `-m` argument gives Python a module to run, in this case `unittest`. Then you have `-v` which is the option for more verbose output, which you can omit if you want. Then you tell `unittest` where to get the tests. We want tests from `cantera` in the `test` module, and in this case, it runs the `test_convert.py` file.

If you look in any of the test files, you'll see that they include different classes to group the tests together. You can run a single class grouping of tests by putting that after the file name:

```console
$ PYTHONPATH=build/python python -m unittest -v cantera.test.test_thermo.TestThermoPhase
```

That would run all the tests in the `TestThermoPhase` class in the `test_thermo.py` file. Last, you can run a single test function with:

```console
$ PYTHONPATH=build/python python -m unittest -v cantera.test.test_purefluid.TestPureFluid.test_critical_properties
```

Just put the test function name after the class name.

## Compiling Cantera from Source - _Linux/macOS Developer Quickstart_

### Installation Requirements

* You'll need a C++ compiler installed on your system. If you don't already have one, download one of our [recommended compilers](https://cantera.org/compiling/dependencies.html#compilers).

  * _Tip:_ You can determine whether you have a C++ compiler set up on your computer by typing `g++ --version` in the terminal.

* You'll also need a virtual environment manager to host your Cantera build environment and its required packages. This guide uses the conda package management system, which you should install using the [official Miniconda installer](https://docs.conda.io/en/latest/miniconda.html#miniconda) if you don't already have Miniconda, Anaconda, or Python installed in another way.

  * _Tip:_ You can determine whether you have conda installed on your computer by typing `conda --version` in the terminal.

  * _Stuck?_ See our more detailed [Conda Requirements](https://cantera.org/compiling/installation-reqs.html#conda-requirements).

* Next, you'll need to initialize a development environment for Cantera. As you may have noticed, as soon as it was installed conda created and activated a virtual environment called `base`. Project-specific packages are not typically installed into `base`, to avoid version conflicts with requirements across different projects. Instead, create a new environment called `cantera-dev` with Cantera's default package dependencies preinstalled by typing the following command in the terminal:

  ```console
  conda create --name cantera-dev python=3 scons cython boost numpy ruamel_yaml git
  ```

  * For more advanced builds of Cantera, you may need to install [additional packages](https://cantera.org/compiling/dependencies.html#optional-programs) to your environment.

* You can switch into the `cantera-dev` virtual environment by typing the following command in terminal:

  ```console
  conda activate cantera-dev
  ```

  * _Tip:_ You can deactivate this environment using the terminal command `conda deactivate` at any time. Switch back to your base environment when you're done working on Cantera!

### Download the Source Code

* Create a personal copy of the official [Cantera repository on Github](https://github.com/Cantera/cantera) by pressing the "Fork" button at the top right of the page. You can make changes to or experiment with the code in your fork without affecting Cantera's version.

* Copy your fork from GitHub to your computer by running this code in the terminal:

  ```console
  git clone --recursive https://github.com/{your_username}/cantera.git
  ```

  Be sure to replace `{your_username}` with the username of the GitHub account that you used to fork Cantera. The `--recursive` option is used here to automatically update and initialize Cantera's required [submodules](https://cantera.org/compiling/dependencies.html#other-required-software).

  * _Tip:_ The cloned repository will be stored in a new directory created within your current working directory. Improve organization by first creating a new "Cantera" directory below your home directory, and calling the command from there.

  * _Need a different version?_ See our more detailed [Downloading the Cantera Source Code](https://cantera.org/compiling/source-code.html#sec-source-code).

### Compile and Test

* The Cantera source code can be compiled automatically using the SCons build system that was preinstalled in your `cantera-dev` virtual environment. In the terminal, switch into your newly cloned copy of the `cantera` repository and then issue the build command as follows:

  ```console
  cd cantera
  scons build boost_inc_dir=${CONDA_PREFIX}/include
  ```

  The part with `${CONDA_PREFIX}` is an **environment variable replacement**. When you build Cantera, SCons will replace `${CONDA_PREFIX}` with the value of an environment variable with the name `CONDA_PREFIX`. This environment variable is set by Conda to point at your currently activated environment.

  A `boost_inc_dir` location needs to be specified here because Boost is needed by your system's C++ compiler, which won't be able to find the library on its own since the Conda directories are not a standard location. Standard locations on macOS and Linux systems are `/usr/include` and `/usr/local/include`.

  * _Tip:_ You can compile in parallel by adding the `-j #` command line flag to `scons build`, which builds `#` objects in parallel. Don't set it higher than the number of CPUs that you have in your machine, or it might slow the build down rather than speeding it up!

  * For more advanced builds of Cantera, you may specify [additional build options](https://cantera.org/compiling/configure-build-dev.html#determine-configuration-options).

* The build process may take a while. On successful compilation, you should see a message that looks like:

  ```console
  *******************************************************
  Compilation completed successfully.

  - To run the test suite, type 'scons test'.
  - To install, type '[sudo] scons install'.
  *******************************************************
  ```

  As indicated, you may choose to run `scons test` to make sure that your code is working correctly.

  * _Stuck?_ See our more detailed [Compile Cantera & Test](https://cantera.org/compiling/configure-build-dev.html#compile-cantera-test).

### _(Optional)_ Install

* Install the compiled code into a directory of your choice by typing the following into the terminal:

  ```console
  scons install prefix={/absolute/path/to/directory}
  ```

  Be sure to replace `{/absolute/path/to/directory}` with the absolute path to your desired install directory. By default, the install location will be your local `cantera` directory.

  * This command installs a selection of input files, resources, and executable samples into the specified directory. Take a look!

  * _Tip:_ If you created a new "Cantera" directory in your home directory, you can use it as your install location by specifying its absolute path as the prefix.

  * _Tip:_ If you're planning on installing Cantera in a particular location, you can instead specify the `prefix` option during building to avoid recompilations during install:

    ```console
    scons build boost_inc_dir=${CONDA_PREFIX}/include prefix={/absolute/path/to/directory}
    scons install
    ```

### Arch linux compiling from source issues.

#### import math.h issue

When compiling `Cantera` on Arch Linux,
```
$ lsb_release -a
LSB Version:	1.4
Distributor ID:	Arch
Description:	Arch Linux
Release:	rolling
Codename:	n/a

$ gcc --version
gcc (GCC) 10.1.0
Copyright (C) 2020 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE
```


I ran into the issue (found inside config.log)
```
scons: Configure: Checking for double x; log(x) in C library None...
scons: Configure: ".sconf_temp/conftest_15.c" is up to date.
scons: Configure: The original builder output was:
  |.sconf_temp/conftest_15.c <-
  |  |
  |  |
  |  |#include "math.h"
  |  |
  |  |int
  |  |main() {
  |  |  double x; log(x);
  |  |return 0;
  |  |}
  |  |
  |
scons: Configure: ".sconf_temp/conftest_15.o" is up to date.
scons: Configure: The original builder output was:
  |gcc -o .sconf_temp/conftest_15.o -c -pthread -O3 -Wno-inline -g -DNDEBUG -I/home/anthony-walker/.conda/envs/cantera-dev/include .sconf_temp/conftest_15.c
  |
scons: Configure: Building ".sconf_temp/conftest_15" failed in a previous run and all its sources are up to date.
scons: Configure: The original builder output was:
  |gcc -o .sconf_temp/conftest_15 -pthread .sconf_temp/conftest_15.o
  |
scons: Configure: (cached) no
```


**Steps to reproduce the issue**:
```
$ git clone git@github.com:Cantera/cantera.git --recurse-submodules
$ cd cantera
$ conda create --name cantera-dev python=3 scons cython boost numpy ruamel_yaml --yes
$ conda activate cantera-dev
$ echo "boost_inc_dir = '$CONDA_PREFIX/include'" >> cantera.conf
$ echo "python_package = 'full'" >> cantera.conf
$ scons build -j 12
.
.
.
g++ -o build/python/cantera/_cantera.cpython-38-x86_64-linux-gnu.so -pthread -shared build/temp-py/_cantera.os -Lbuild/lib -lcantera
collect2: error: ld returned 1 exit status
scons: *** [build/lib/libcantera_shared.so.2.5.0] Error 1
scons: building terminated because of errors.
```

**Steps to solve issue**

My first attempt at a solution was to independently install sundials because it was seemingly causing the problem I tried this from both **conda** and the **AUR**. This allowed the code to compile successfully but it would fail during runtime with `undefined reference` errors to `CVODE` variables.

The solution I found to this problem was to use a different version of `gcc`. I installed `gcc-7` from the **AUR** and added `CC` and `CXX` paths to the `cantera.conf`.
```
cantera.conf:
CXX = '/usr/bin/g++-7'
CC = '/usr/bin/gcc-7'
prefix = '/home/anthony-walker/.conda/envs/cantera-dev'
python_package = 'full'
f90_interface = 'n'
boost_inc_dir = '/home/anthony-walker/.conda/envs/cantera-dev/include'
```

```
$ scons clean
$ scons build -j 12
$ scons install prefix=$CONDA_PREFIX
$ source $CONDA_PREFIX/bin/setup_cantera
```
### Debugging the source code

When you modify Cantera's source code, it is highly likely that you break the code and need to debug it. Following are some ways to debug your code -

*  Build the source code with `debug` options. Modify the `cantera.conf` file by adding the `debug` flags with no optimizations as follows
```
cantera.conf:
debug_flags='-g -D_GLIBCXX_ASSERTIONS'
optimize = 'no'
```
The flag `D_GLIBCXX_ASSERTIONS` is useful to see the stack trace. For more details about debug flags available in Cantera, see [debug_options](https://cantera.org/compiling/config-options.html#debug).

*  The error statements inside the source code are pretty self-explanatory. These sentences mention the filename and function name causing the error and point to the exact source of error.

    For example, trying to set the negative temperature will generate following error -

    ```
    $ python.exe test.py
    Traceback (most recent call last):
    File "test.py", line 37, in <module>
    gas.TP = = -500, 101325
    File "interfaces\cython\cantera\thermo.pyx", line 1145, in cantera._cantera.ThermoPhase.TP.__set__
    cantera._cantera.CanteraError:
    ***********************************************************************
    CanteraError thrown by Phase::setTemperature:
    temperature must be positive. T = -500.0
    ***********************************************************************
    ```

    In this case the error is thrown by the function `setTemperature` in `Phase` class (i.e. from the file `phase.cpp`).

*  To debug the code, which I have modified -  I typically put print statements inside the source code to check the values of certain variables.

  E.g. Inside the source code, if I want to see the value of variable `varExample` in the function `funcExample`, then I put the command inside the function as
  ```
  writelog("In funcExample:: value of varExample is {} \n", varExample);
  ```
  Here single curly brackets { } act as a placeholder for the variable to be printed. Everything else that is written inside double quotes will get printed as it is on the screen. For example, if `varExample = 10`, then above command will print
  ```
  In funcExample:: value of varExample is 10

  ```
  _Tip_: One needs to build the source code again before running the main example file. This method is inefficient, but sometimes useful for a quick check.  

*  For the segmentation fault errors, using GDB (or similar tools such as LLDB) on Linux is useful. It can be run as follows:
   ```
   gdb --args /path/to/python/executable
   ```
   It opens the `gdb` prompt. To use GDB with Cantera built from source, the commands are:
   ```
   cd cantera
   PYTHONPATH=build/python gdb --args /path/to/python/executable
   ```
   Then run the example python file in terminal as
   ```
   run example.py
   ```
   The run stops whenever segmentation fault or other error is encountered. Then type ```where```. It shows the entire trace of the current run.

   One can also set the breakpoint as
   ```
   breakpoint set --name setState_TP
   ```
   In this case, the run will stop when the function `ThermoPhase::setState_TP` is encountered.

   _Tip_: The above method works only on Linux.
   
   `gdb` can also be used to debug errors in tests in the Python or `gtest` test suites. For the Python test suite, you can run a specific test in `gdb` with a command like:
   ```
   PYTHONPATH=build/python gdb --args python -m unittest cantera.test.test_kinetics.TestDuplicateReactions.test_forward_multiple
   ```
   
   Running one of the `gtest` tests is slightly more involved. To run a few tests out of the `thermo` test suite, the following commands can be used:
   ```
   scons test-thermo
   cd test/work
   LD_LIBRARY_PATH=../../build/lib CANTERA_DATA=../../build/data:../data PYTHONPATH=../../build/python gdb --args ../../build/test/thermo/thermo --gtest_filter=WaterProps*
   ```
   
   By default, `gdb` does not provide a very readable view of C++ standard library objects such as `std::vector<double>`. To use improved "pretty printers",
   put the following in the `.gdbinit` file in your home directory:
   ```
   python
   import sys
   sys.path.insert(0, '/usr/share/gcc-10/python')
   from libstdcxx.v6.printers import register_libstdcxx_printers
   register_libstdcxx_printers (None)
   end
   ```
   You may need to change the number suffix to the directory that is present on your system. The `gcc-10` directory should be present on Ubuntu 20.04.

## Build documentation 
  
The doc files are under `cantera/doc` directory. After editing the doc files, run
```
pip install -U sphinxcontrib-matlabdomain sphinxcontrib.doxylink sphinxcontrib.katex
```
to install the necessary dependencies. To build the documentation, run
```
scons build doxygen_docs=Y sphinx_docs=Y
```
The generated `.html` files are saved in `cantera/build/docs`. 
## Migrate Cantera from one conda environment to another

Suppose you already have Cantera built in one conda environment, and you want to build it in another conda environment. First activate the new conda environment.
Next, change `Prefix` and `boost_inc_dir` parameters in `cantera/cantera.conf` file to point to the new conda environment.
Make sure SCONS is installed in the new environment. To install SCONS, run
```
conda install scons
```
Clean the previous build by
```
scons clean
```
, then build Cantera again with
```
scons build
```
### _Need more information?_

Please refer to the more detailed compilation instructions [here](https://cantera.org/install/compiling-install.html).
