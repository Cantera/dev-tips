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

## Compiling Cantera from Source - _Developer Quickstart_

### Installation Requirements

* You'll need a C++ compiler installed on your system. If you don't already have one, download one of our [recommended compilers](https://cantera.org/compiling/dependencies.html#compilers).

    * _Tip:_ You can determine whether you have a C++ compiler set up on your computer by typing `g++ --version` in the terminal.

* You'll also need a virtual environment manager to host your Cantera build environment and its required packages. This guide uses the conda package management system, which you should install using the [official Miniconda installer](https://docs.conda.io/en/latest/miniconda.html#miniconda) if you don't already have it.

    * _Tip:_ You can determine whether you have conda installed on your computer by typing `conda --version` in the terminal.

    * _Stuck?_ See our more detailed [Conda Requirements](https://cantera.org/compiling/installation-reqs.html#conda-requirements).

* Next, you'll need to initialize a development environment for Cantera. As you may have noticed, as soon as it was installed conda created and activated a virtual environment called `base`. Project-specific packages are not typically installed into `base`, to avoid version conflicts with requirements across different projects. Instead, create a new environment called `cantera-dev` with Cantera's default package dependencies preinstalled by typing the following command in the terminal:

    ```
    conda create --name cantera-dev python=3 scons cython boost numpy ruamel_yaml
    ```

    * For more advanced builds of Cantera, you may need to install [additional packages](https://cantera.org/compiling/dependencies.html#optional-programs) to your environment. 

* You can switch into the `cantera-dev` virtual environment by typing the following command in terminal:
    
    ```
    conda activate cantera-dev
    ```

    * _Tip:_ You can deactivate this environment using the terminal command `conda deactivate` at any time. Switch back to your base environment when you're done working on Cantera!

### Download the Source Code

* Create a personal copy of the official [Cantera repository on Github](https://github.com/Cantera/cantera) by pressing the "Fork" button at the top right of the page. You can make changes to or experiment with the code in your fork without affecting Cantera's version.

* Copy your fork from GitHub to your computer by running this code in the terminal:

    ```
    git clone --recursive https://github.com/{your_username}/cantera.git
    ```

    Be sure to replace `{your_username}` with the username of the GitHub account that you used to fork Cantera. The `--recursive` option is used here to automatically update and initialize Cantera's required [submodules](https://cantera.org/compiling/dependencies.html#other-required-software).

    * _Tip:_ The cloned repository will be stored in a new directory created within your current working directory. Improve organization by first creating a new "Cantera" directory below your home directory, and calling the command from there.

    * _Need a different version?_ See our more detailed [Downloading the Cantera Source Code](https://cantera.org/compiling/source-code.html#sec-source-code).

### Compile and Test

* The Cantera source code can be compiled automatically using the SCons build system that was preinstalled in your `cantera-dev` virtual environment. In the terminal, switch into your newly cloned copy of the `cantera` repository and then issue the build command as follows:

    ```
    cd cantera
    scons build boost_inc_dir=${/absolute/path/to/env}/include
    ```

    Be sure to replace `{/absolute/path/to/env}` with the absolute path to your `cantera-dev` environment. A `boost_inc_dir` location needs to be specified here because Boost is needed by your system's C++ compiler, which won't be able to find the library on its own.

    * _Tip:_ You can determine the absolute paths of your virtual environments by typing `conda info --envs` in the terminal.

    * _Tip:_ You can compile in parallel by adding the `-j #` command line flag to `scons build`, which builds `#` objects in parallel. Don't set it higher than the number of CPUs that you have in your machine, or it might slow the build down rather than speeding it up!

    * For more advanced builds of Cantera, you may specify [additional build options](https://cantera.org/compiling/configure-build-dev.html#determine-configuration-options).

* The build process may take a while. On successful compilation, you should see a message that looks like:

    ```
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

    ```
    scons install prefix={/absolute/path/to/directory}
    ```

    Be sure to replace `{/absolute/path/to/directory}` with the absolute path to your desired install directory. By default, the install location will be your local `cantera` directory.

    * This command installs a selection of input files, resources, and executable samples into the specified directory. Take a look!

    * _Tip:_ If you created a new "Cantera" directory in your home directory, you can use it as your install location by specifying its absolute path as the prefix.

    * _Tip:_ If you're planning on installing Cantera in a particular location, you can instead specify the `prefix` option during building to avoid recompilations during install:

        ```
        scons build boost_inc_dir=${/absolute/path/to/env}/include prefix={/absolute/path/to/directory}
        scons install
        ```

### _Need more information?_
Please refer to the more detailed compilation instructions [here](https://cantera.org/install/compiling-install.html).
