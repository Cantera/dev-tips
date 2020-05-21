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
