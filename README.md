[![Release](https://img.shields.io/github/release/manodeep/Corrfunc.svg)](https://github.com/manodeep/Corrfunc/releases/latest)
[![MIT licensed](https://img.shields.io/badge/license-MIT-blue.svg)](https://raw.githubusercontent.com/manodeep/Corrfunc/master/LICENSE)
[![DOI](https://zenodo.org/badge/19184/manodeep/Corrfunc.svg)](https://zenodo.org/badge/latestdoi/19184/manodeep/Corrfunc)
[![Travis Build](https://travis-ci.org/manodeep/Corrfunc.svg?branch=master)](https://travis-ci.org/manodeep/Corrfunc)
[![Issues](https://img.shields.io/github/issues/manodeep/Corrfunc.svg)](https://github.com/manodeep/Corrfunc/issues)
[![Coverity](https://img.shields.io/coverity/scan/6982.svg)](https://scan.coverity.com/projects/manodeep-corrfunc)

# Description

This repo contains a set of codes to measure the following OpenMP parallelized clustering 
measures in a cosmological box (co-moving XYZ) or on a mock (RA, DEC, CZ). Also, 
contains the associated paper to be published in Astronomy & Computing Journal (at some point). 

# Installation

## Pre-requisites
1. OpenMP capable compiler like ``icc``, ``gcc`` or ``clang >= 3.7``. If not available, please disable ``USE_OMP`` option option in ``theory.options`` and ``mocks.options``. You might need to ask your sys-admin for system-wide installs of the compiler; if you prefer to install your own then ``conda install gcc`` (MAC/linux) or ``(sudo) port install gcc5`` (on MAC) should work. *Note ``gcc`` on macports defaults to ``gcc48`` and the portfile is currently broken on ``El Capitan``*.
2. ``gsl``. Use either ``conda install -c https://conda.anaconda.org/asmeurer gsl`` (MAC/linux) or ``(sudo) port install gsl`` (MAC) to install ``gsl`` if necessary. 
3. ``python >= 2.6`` or ``python>=3.4`` for compiling the C extensions. 
4. ``numpy>=1.7`` for compiling the C extensions. 

*If python and/or numpy are not available, then the C extensions will not be compiled*. 

## Preferred Method

```
$ git clone https://github.com/manodeep/Corrfunc/
$ make 
$ make install
$ python setup.py install (--user)
$ make tests 
```
Assuming you have `gcc` in your ``PATH``, `make` and `make install` should compile and install the C libraries + python extensions within the source directory. If you would like to install the python C extensions in your environment, then ``python setup.py install (--user)`` should be sufficient. 

## Alternative
The python package is directly installable via ``pip install Corrfunc``. 

## Installation notes

If compilation went smoothly, please run ``make tests`` to ensure the code is working correctly. Depending on the hardware and compilation options, the tests might take more than a few minutes. *Note that the tests are exhaustive and not traditional unit tests*. 

While I have tried to ensure that the package compiles and runs out of the box, cross-platform compatibility turns out to be incredibly hard. If you run into any issues during compilation and you have all of the pre-requisistes, please see the [FAQ](FAQ) or [email me](mailto:manodeep@gmail.com). Also, feel free to create a new issue with the `Installation` label. 

## Clustering Measures on a Cosmological box

All codes that work on cosmological boxes with co-moving positions are located in the 
``xi_theory`` directory. The various clustering measures are:

1. ``xi_of_r`` -- Measures auto/cross-correlations between two boxes. The boxes do not need to be cubes.

2. ``xi`` -- Measures 3-d auto-correlation in a cubic cosmological box. Assumes PERIODIC boundary conditions.

3. ``wp`` -- Measures auto 2-d point projected correlation function in a cubic cosmological box. Assumes PERIODIC boundary conditions. 

4. ``xi_rp_pi`` -- Measures the auto/cross correlation function between two boxes. The boxes do not need to be cubes. 

5. ``vpf`` -- Measures the void probability function + counts-in-cells. 

## Clustering measures on a Mock

All codes that work on mock catalogs (RA, DEC, CZ) are located in the ``xi_mocks`` directory. The
various clustering measures are:

1. ``DDrppi`` -- The standard auto/cross correlation between two data sets. The outputs, DD, DR and RR
can be combined using ``wprp`` to produce the Landy-Szalay estimator for $w_p(r_p)$. 

2. ``wtheta`` -- Computes angular correlation function between two data sets. The outputs from 
``DDtheta_mocks`` need to be combined with ``wtheta`` to get the full $\omega(\theta)$

3. ``vpf`` -- Computes the void probability function on mocks. 

# Science options

1. ``PERIODIC`` (ignored in case of wp/xi) -- switches periodic boundary
conditions on/off. Enabled by default. 

2. ``OUTPUT_RPAVG`` -- switches on output of ``<rp>`` in each ``rp`` bin. Can be
a massive performance hit (~ 2.2x in case of wp). Disabled by default.
Needs code option ``DOUBLE_PREC`` to be enabled as well. For the mocks, 
``OUTPUT_RPAVG`` causes only a mild increase in runtime and is enabled by 
default.

3. ``OUTPUT_THETAAVG`` -- switches on output of <theta> in each theta bin. 
Can be extremely slow (~5x) depending on compiler, and CPU capabilities. 
Disabled by default. 


## Mocks

1. ``LINK_IN_DEC`` -- creates binning in declination for mocks. Please check that for 
your desired binning in $r_p$/$\theta$, this binning does not produce incorrect 
results (due to numerical precision). 

2. ``LINK_IN_RA`` -- creates binning in RA once binning in DEC has been enabled. Same 
numerical issues as ``LINK_IN_DEC``

3. ``FAST_DIVIDE`` --  Divisions are slow but required $DD(r_p,\pi)$. This Makefile
option (in mocks.options) replaces the divisions to a reciprocal followed by a 
Newton-Raphson. The code will run ~20% faster at the expense of some numerical precision. 
Please check that the loss of precision is not important for your use-case. Also, note 
that the mocks tests for $DD(r_p, \pi)$ *will fail* if you enable ``FAST_DIVIDE``. 

# Running the codes

The documentation is lacking currently but I am actively working on it. 

## Using the command-line interface
Navigate to the correct directory. Make sure that the options, set in either ``theory.options`` or ``mocks.options`` in the root directory are what you want. If not, edit those two files (and possibly ``common.mk``), and recompile. Then, you can use the command-line executables in each individual subdirectory corresponding to the clustering measure you are interested in. For example, if you want to compute the full 3-D correlation function, ``\xi(r)``, then navigate to ``xi_theory/xi`` and run the executable ``xi``. If you run executables without any arguments, the message will you tell you all the required arguments. 

## Calling from C 
Look under the ``xi_theory/examples/run_correlations.c`` and ``xi_mocks/examples/run_correlations_mocks.c`` to see examples of calling the C API directly. If you run the executables, ``run_correlations`` and ``run_correlations_mocks``, the output will also show how to call the command-line interface for the various clustering measures. 

## Calling from Python 
If all went well, the codes can be directly called from ``python``. Please see ``Corrfunc/call_correlation_functions.py`` and ``Corrfunc/call_correlation_functions_mocks.py`` for examples on how to use the Python interface. 

# Common Code options for both Mocks and Cosmological Boxes

1. ``DOUBLE_PREC`` -- does the calculations in double precision. Disabled
by default. 

2. ``USE_AVX`` -- uses the AVX instruction set found in Intel/AMD CPUs >= 2011
(Intel: Sandy Bridge or later; AMD: Bulldozer or later). Enabled by
default - code will run much slower if the CPU does not support AVX instructions.
On Linux, check for "avx" in /proc/cpuinfo under flags. If you do not have
AVX, but have a SSE4 system instead, email me - I will send you a copy of
the code with SSE4 intrinsics. Or, take the relevant SSE code from the public repo at 
[pairwise](https://manodeep.github.io/pairwise).

3. ``USE_OMP`` -- uses OpenMP parallelization. Scaling is great for DD (perfect scaling
up to 12 threads in my tests) and okay (runtime becomes constant ~6-8 threads in
my tests) for ``DDrppi`` and ``wp``. 


*Optimization for your architecture*

1. The values of ``bin_refine_factor`` and/or ``zbin_refine_factor`` in the countpairs_*.c
files control the cache-misses, and consequently, the runtime. In my trial-and-error
methods, I have seen any values larger than 3 are always slower. But some different
combination of 1/2 for ``(z)bin_refine_factor`` might be faster on your platform. 

2. If you have AVX2/AVX-512/KNC, you will need to rewrite the entire AVX section.

# Author

Corrfunc is written/maintained by Manodeep Sinha. Please contact the [author](mailto:manodeep@gmail.com) in
case of any issues.

# Citing

If you use the code, please cite using the Zenodo DOI. The BibTex entry for the code is  

```
@misc{Corrfunc,
  author       = {Manodeep Sinha},
  title        = {{Corrfunc: Development release to create zenodo DOI 
                   Corrfunc: Development release to create zenodo DOI}},
  month        = nov,
  year         = 2015,
  doi          = {10.5281/zenodo.33655},
  url          = {http://dx.doi.org/10.5281/zenodo.33655}
}
```

# LICENSE

Corrfunc is released under the MIT license. Basically, do what you want
with the code including using it in commercial application.

# Project URL

* website (https://manodeep.github.io/Corrfunc/) 
* version control (https://github.com/manodeep/Corrfunc)
