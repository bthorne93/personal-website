---
title: What's in `pytest`
date: 2019-02-08
permalink: /posts/2019/02/08
tags:
  - pytest
  - python
  - unittest
image:
  caption: >-
    Image credit: [Photo by Ferenc Almasi on
    Unsplash](https://unsplash.com/photos/EWLHA4T-mso
  focal_point: ''
  preview_only: false
---

This is just a list of useful `pytest` facilities that I need to keep track of.

## How to do fixtures properly

*Fixtures* allow you to access the same resource from multiple tests, or even modules, providing a "baseline of upon which tests can be executed" (from the `pytest` documentation). A fixture constitutes a function, the name of which can be passed to a test function, which uses it in its testing. Fixtures can be parameterized to provide many input cases to a given test.

As an example, perhaps we have a set of functions that operate on `HEALPix` maps, and would like to test them consistently on the same set of maps, without having to explicitly build the map in each test function:

```python
import pytest
import healpy as hp;def hpix_map():
    np.random.seed(1234)
    nside = 64
    npix = hp.nside2npix(nside)
    ma = np.random.randn(npix)
    return ma, nside

def test_healpy(hpix_map):
    ma, nside = hpix_map
    assert hp.get_nside(ma) == nside
```

Running this test gets:

```bash
$ pytest tests/test_healpy.py
================================================================================== test session starts ==================================================================================
platform linux -- Python 3.7.6, pytest-5.3.5, py-1.8.1, pluggy-0.13.1
rootdir: /home/bthorne/gdrive/work/communication/website/content/post/pytest
plugins: cov-2.8.1, arraydiff-0.3, remotedata-0.3.2, openfiles-0.4.0, hypothesis-5.4.1, doctestplus-0.5.0, astropy-header-0.1.2
collected 1 item                                                                                                                                                                        

tests/test_healpy.py .                                                                                                                                                            [100%]

=================================================================================== 1 passed in 0.39s ===================================================================================
```

The way discovery happens with this code is:

1. `pytest` discovers `test_healpy` due to the `test_` prefix.
2. `test_healpy` requires a function argument `hpix_map`, which `pytest` then looks for.
3. `pytest` finds the `hpix_map` fixture, due to its fixture decorator, and creates an instance of it.
4. `test_healpy(<hpix_map instance>)` is run.

### Sharing fixtures

Fixtures can of course be shared between function in a test file, but they can also be shared between test files. To do this, include the fixture in the `conftest.py` file, and it will be automatically discovered by all tests, without the need to import in each test file.

This isolation of fixtures in `conftest.py` can be a simple way of providing an interface for tests to any data required during testing.

### Parameterizing fixtures

We can provide a set of values over which features should be iterated each time they are used in the test code. This requires the use of the `request` argument, and the `params` argument to the fixture decorator. Straying from these argument names results in `pytest` looking for other fixtures to use as arguments, rather than the parameterized variable. 

```python 
@pytest.fixture(params=[64, 128, 256])
def hpix_map(request):
    npix = hp.nside2npix(request.param)
    ma = np.random.randn(npix)
    return ma, request.param

def test_healpy(hpix_map):
    ma, nside = hpix_map
    assert hp.get_nside(ma) == nside
```

## Parameterizing test functions

Fixtures can be parameterized, and so can functions. This works similarly to fixtures, but with a different decorator, and no constraints on the name of the function argument:

```python
@pytest.mark.parametrize("x": [0, 1])
def test_healpy(x):
    pass
```