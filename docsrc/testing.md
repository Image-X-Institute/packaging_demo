# Add tests

Test cases are simple tests that you can automatically run whenever you (or someone else) changes the code. Good code testing is a bit of an art form, but even bad testing is much better than no testing! At a minimum, your tests should ensure that at least your code runs without crashing. The inclusion or absence of tests, as well as the extent of test coverage is one of the things that developers will look to when considering whether to use your package. If the tests are there, it's generally an indication of a reasonably high quality code base.

We will use [pytest](https://docs.pytest.org/en/7.1.x/) to run our tests, but every (proper) language should have some kind of testing framework.

Inside DemoPythonPackage, create a folder called testing. If you are doing this in order your package should look like this:

![](__resources/progress_tests.png)

Inside tests, create a file called test_sine_wave_utilities.py and copy the below code:

```python
from pathlib import Path
import sys
import numpy as np
'''
For testing/ example purposes it's safest to manually append the path variable to ensure our 
package will always be found. This isn't necessary once we actually install the package 
because installation in python essentially means "copying the package to a place where it can be found"
'''
this_file_loc = Path(__file__)
sys.path.insert(0, str(this_file_loc.parent.parent))
# note: insert(0) means that the path is above is the FIRST place that python will look for imports
# sys.path.append means that the path above is the LAST place
from MyPackage import sine_wave_utilities as swu

def test_amplitude():
    x, y = swu.generate_sine_data(Amplitude=10)
    assert y.max() > 9.9  # can't guarantee max=10 because we have finite samples
    assert y.min() < -9.9

def test_range():
    x, y = swu.generate_sine_data(x_max=10, x_min=-8)
    assert x.max() == 10
    assert x.min() == -8

def test_n_samples():
    x, y = swu.generate_sine_data(n_samples=192)
    assert y.shape[0] == 192

def test_phase():
    x_0, y_0 = swu.generate_sine_data(Phase=0)
    x_1, y_1 = swu.generate_sine_data(Phase=1)
    # if phase works, at a minimum we should see that these two data sets are different...
    assert not np.equal(y_0, y_1).all()
```

To execute these tests, we just have to run [pytest](https://docs.pytest.org/en/7.1.x/) from the root directory. Pytest is smart enough to find our tests by itself.

```bash
# from command line:
pytest
```

When you run these tests, **you should get a failure!** (edit: I had to remove this feature when implemeting automatic testing, but you can do the opposite change below to trigger an errror) This is because I did not actually implement the phase variable properly. To fix this, go to MyPackage / demonstrate_sine_plotter.py and make the following change:

```python
# change
y = (Amplitude * np.sin(x*Frequency))
# to
y = (Amplitude * np.sin(x*Frequency)) - Phase
```

now if you execute the tests again, they should all pass/

## assessing test cover

A key aspect of your testing framework is how many lines of your package actually get executed during testing. There is an easy way to check this, using the package [coverage](https://coverage.readthedocs.io/en/6.3.3/)

```bash
# from command line:
pytest --cov-report xml --cov-report term --cov=MyPackage/
```

This reveals that we only have 47% coverage on our sine_wave_utilities module:

```bash
Name                                Stmts   Miss  Cover
-------------------------------------------------------
MyPackage\__init__.py                   1      0   100%
MyPackage\sine_wave_utilities.py       15      8    47%
-------------------------------------------------------
TOTAL                                  37      8    74%

```

Why is this so low!? The answer is that we haven't written any tests at all for the ```data_plotter``` method.

Now at this point, we can do a few things:

1. Write some more tests. This is tricky for methods that generate a function for two reasons; 
   1. generating plots from the command line often blocks further execution of scripts
   2. It is hard to explain to a code if a plot is 'right' or not.

2. Just accept our bad coverage and move on
3. tell coverage we intentionally left that method out of our test framework by putting ```# pragma: no cover``` next to the method definition. (**nb: I already did this on the master branch**)

I am going to take the third option, after which our coverage report looks like this:

```bash
Name                                Stmts   Miss  Cover
-------------------------------------------------------
MyPackage\__init__.py                   1      0   100%
MyPackage\sine_wave_utilities.py        6      0   100%
-------------------------------------------------------
TOTAL                                  28      0   100%

```
