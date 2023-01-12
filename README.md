![](https://github.com/fastlib/fCWT-dev/blob/live/img/githubart.png)

The fast Continuous Wavelet Transform (fCWT)
====================================

The fast Continuous Wavelet Transform (fCWT) is a highly optimized C++ library for very fast calculation of the CWT.

Features
========

- Calculating CWT 34-120x faster than all competitors*
- Very high time-frequency resolution (i.e., it does not rely on wavelet estimation)
- Real-time CWT for signals having sample frequencies of up to 200kHz
- Easy Python integration via pip
- Easy MATLAB integration via MEX-files

*Based on C++ performance. fCWT is the fastest CWT library in C++ and Matlab. In Python CCWT is faster for shorter signals and fCWT for longer signals. Please see the benchmark section for more details. Raise an issue if you found a new/faster implementation. I will try to add it to benchmark! 

![](https://github.com/fastlib/fCWT-dev/blob/live/img/audio.png)

Quickstart 
============

fCWT's implementation can be used to accelerate your C++, Matlab, and Python projects! Build the C++ library to achieve the highest efficiency or use the Matlab and Python packages to maximize integration possibilities. 

Install the Python package using pip:
```
pip install fcwt
```

Install the Matlab wrapper using the following MEX command:
```
```

Build fCWT from source:
```
$ git clone https://github.com/fastlib/fCWT.git
$ cd fCWT
$ mkdir -p build
$ cd build
$ cmake ../
$ make 
$ sudo make install
```
See the Installation section for more details about building fCWT from source for both UNIX and Windows systems.


Benchmark
========

Columns are formatted as X-Y, where X is signal length in samples and Y the number of frequencies.

| Implementation        | 10k-300 | 10k-3000 | 100k-300 | 100k-3000 | Speedup factor |
|-----------------------|---------|----------|----------|-----------|----------------|
| fCWT (C++)            | 0.005s  | 0.04s    | 0.03s    | 0.32s     | -              |
| fCWT (Python)         | 0.027s  | 0.23s    | 0.075s   | 0.57s     | -              | 
| fCWT (Matlab)         | 0.072s  | 0.44s    | 0.17s    | 1.55s     | -              |
|                       |         |          |          |           |                |
| CCWT (Python)         | 0.019s  | 0.11s    | 0.15s    | 3.40s     | 10.63x         |
| PyWavelets (Python)   | 0.10s   | 1.17s    | 1.06s    | 12.69s    | 34.29x         |
| Matlab                | 0.75s   | 0.86s    | 1.06s    | 13.26s    | 35.85x         |
| SsqueezePy (Python)   | 0.04s   | 0.43s    | 1.16s    | 17.76s    | 48.00x         |
| SciPy (Python)        | 0.19s   | 1.82s    | 2.11s    | 18.70s    | 50.54x         |
| Rwave (C)             | 0.18s   | 1.84s    | 2.28s    | 23.22s    | 62.75x         |
| Mathematica           | -       | -        | -        | 27.83s    | 75.20x         |
| Wavelib (C++)         | 0.25s   | 2.55s    | 4.85s    | 45.04s    | 121.72x        |


Python Example
==============

```Python
import fcwt
import numpy as np
import matplotlib.pyplot as plt

#Initialize
fs = 1000
n = fs*100 #100 seconds
ts = np.arange(n)

#Generate linear chirp
signal = np.sin(2*np.pi*((1+(20*ts)/n)*(ts/fs)))

f0 = 1 #lowest frequency
f1 = 101 #highest frequency
fn = 200 #number of frequencies

#Calculate CWT without plotting...
freqs, out = fcwt.cwt(signal, fs, f0, f1, fn)

#... or calculate and plot CWT
fcwt.plot(signal, fs, f0=f0, f1=f1, fn=fn)
```

Output:
![](https://github.com/fastlib/fCWT-dev/blob/live/img/pythontest.png)

C++ Example
=======

```cpp
#include <iostream>
#include <vector>
#include <math.h>
#include <fcwt.h>

int main(int argc, char * argv[]) {
    
    int n = 100000; //signal length
    const int fs = 1000; //sampling frequency
    float twopi = 2.0*3.1415;
    
    //3000 frequencies spread logartihmically between 1 and 32 Hz
    const float f0 = 1;
    const float f1 = 32;
    const int fn = 3000;

    //Define number of threads for multithreaded use
    const int nthreads = 8;

    //input: n real numbers
    std::vector<float> sig(n);
    
    //output: n x scales
    std::vector<complex<float>> tfm(n*fn);
    
    //initialize with 1 Hz cosine wave
    for(auto& el : sig) {
        el = cos(twopi*((float)(&el - &sig[0])/(float)fs));
    }
    
    //Create a wavelet object
    Wavelet *wavelet;
    
    //Initialize a Morlet wavelet having sigma=2.0;
    Morlet morl(2.0f);
    wavelet = &morl;

    //Create the continuous wavelet transform object
    //constructor(wavelet, nthreads, optplan)
    //
    //Arguments
    //wavelet   - pointer to wavelet object
    //nthreads  - number of threads to use
    //optplan   - use FFTW optimization plans if true
    //normalization - take extra time to normalize time-frequency matrix
    FCWT fcwt(wavelet, nthreads, true, false);

    //Generate frequencies
    //constructor(wavelet, dist, fs, f0, f1, fn)
    //
    //Arguments
    //wavelet   - pointer to wavelet object
    //dist      - FCWT_LOGSCALES | FCWT_LINFREQS for logarithmic or linear distribution frequency range
    //fs        - sample frequency
    //f0        - beginning of frequency range
    //f1        - end of frequency range
    //fn        - number of wavelets to generate across frequency range
    Scales scs(wavelet, FCWT_LINFREQS, fs, f0, f1, fn);

    //Perform a CWT
    //cwt(input, length, output, scales)
    //
    //Arguments:
    //input     - floating pointer to input array
    //length    - integer signal length
    //output    - floating pointer to output array
    //scales    - pointer to scales object
    fcwt.cwt(&sig[0], n, &tfm[0], &scs);
        
    return 0;
}
```


Installation
==========================

Dependencies
------------

- [Cmake] >=3.10
- C++17 compiler ([GCC] 10+, [Clang] or [Microsoft Visual C++][Visual_Studio] 15.7+);
- [FFTW] >=3.3  (is included in the zip-file)
- [OpenMP] >=5

fCWT has been tested on Mac OSX Mojave 10.14.5, Big Sur 11.6 (both on Intel and Apple Silicon), Windows 10, and Ubutnu 20.04. The benchmark has been performed on a MacBook Pro 2019 having a 2,3 GHz Intel Core i9, 16 GB 2400 MHz DDR4.

Build time settings
-------------------

Settings that may be specified at build time by using [CMake] variables are:
  1. the flag to build a shared library instead of static (default is on);
  2. the flag to build Matlab MEX-files (default is off);
  3. installation directories.

Details:

|CMake variable|Possible values|Default on Unix|Default on Windows|
|:-------------|:--------------|:--------------|:-----------------|
|**The flag to build the shared library**||||
|BUILD_SHARED_LIBS|On \| Off|On|On|
|**The flag to build Matlab MEX-files**||||
|BUILD_MATLAB|On \| Off|Off|Off|
|**Installation directories**||||
|FCWT_MATLAB_DIR|*a path relative to `build`*|"../MATLAB"|"../MATLAB"|
|CMAKE_INSTALL_PREFIX|*an absolute path*|"/usr/local"|"%ProgramFiles (x86)%\fCWT"|
|FCWT_CMAKE_INSTALL_DIR|*a path relative to CMAKE_INSTALL_PREFIX*|"share/fcwt/cmake"|"cmake"|
|FCWT_LIB_INSTALL_DIR|*a path relative to CMAKE_INSTALL_PREFIX*|"lib"|"lib"|
|FCWT_INCLUDE_INSTALL_DIR|*a path relative to CMAKE_INSTALL_PREFIX*|"include"|"include"|

Installation on Unix
---------------------

    $ git clone https://github.com/fastlib/fCWT.git
    $ cd fCWT
    $ mkdir -p build
    $ cd build
    $ cmake ../
    $ make 
    $ sudo make install

Installation on Microsoft Windows
---------------------------------

Run the Developer Command Prompt for Visual Studio and type:

    > git clone https://github.com/fastlib/fCWT.git
    > cd fCWT
    > mkdir -p build
    > cd build
    > cmake -G "Visual Studio 17 2022" ..
    > cmake --build .

A Visual Studio .SLN file has now been created in the build-folder. This project includes several build targets. To build a shared/static library of fCWT, build the 'fCWT' target. To run the example code, set the 'fCWT_example' target as the start-up project and run the code as 'release'.

To install fCWT, run the Elevated Command Prompt (i.e. the command prompt with administrator privileges) in the build-folder and type:

    > cmake -P cmake_install.cmake

To make the installed DLL available for *any* application that depends on it, the symbolic link to the
fcwt.dll should be created:

  - in %SYSTEMROOT%\System32 for the 64-bit DLL on 64-bit host (or for 32-bit DLL on 32-bit host);
  - in %SYSTEMROOT%\SysWOW64 for the 32-bit DLL on 64-bit host.


Usage
=====

Example code
------------

One can run several commands to test fCWT:

`$ ./fCWT_example -h`
Get information about fCWT usage

`$ ./fCWT_example -fcwtoptimize [Length=100000] [Number of threads=8] [Optimization=estimate|measure|patient|exhaustive]`
Calculate all optimization schemes for all powers of two up to the maximal power of two that is calculated based on the [Length] argument, using [Number of threads] threads and using the optimization method specified. If no optimization method is defined it will use the `measure` method. Estimate is the fastest but lowest quality optimization, exhaustive is the slowest but highest quality optimization. Warning: Exhaustive will take some time!

`$ ./fCWT_example -fcwt [Length=100000] [Number of threads=8]`
Calculate the CWT using fCWT for three signals with length=[Length]. The signals are generated by a dynamic sine wave function, random function and non-smooth function generator, respectively. Depending on optimization usage and system hardware, a run-time in the order of seconds is expected for lengths up to 100.000. Furthermore, one will see that signal content is independent of performance. See the source code for more information.

`$ ./fCWT_example -wavelib [Length=100000] [Number of threads=8]`
Calculate the CWT using Wavelib`s C implementation [1] for a signal with length=[Length]. Depending on system hardware, a run-time in the order of several minutes is expected for lengths up to 100.000.

`$ ./fCWT_example -rwave [Length=100000] [Number of threads=8]`
Calculate the CWT using Rwave's C implementation [2] for a signal with length=[Length]. Depending on system hardware, a run-time in the order of several minutes is expected for lengths up to 100.000.

Benchmark reproduction
----------------------

If one wants to reproduce the benchmark results as stated in the article, one has to use a signal length of 100000, 8 threads and calculate optimization plans with the exhaustive method: `$ ./fCWT_example -fcwtoptimize 100000 8 exhaustive` After that:

1. Run fCWT with: `$ ./fCWT_example -fcwt 100000 8`,
2. Run RWave with: `$ ./fCWT_example -rwave 100000 8`,
3. Run Wavelib with: `$ ./fCWT_example -wavelib 100000 8`,
4. Run additional Python, Matlab and Mathematica scripts found in `/src/benchmark`

By default, the source code performs 10 runs for demonstration purposes. To match the number of runs in the paper, adjust the `runs` variable in `main.cpp:128`. It is recommended to close any background processes. 

MATLAB
---------

In the MATLAB-folder, we provided an example live-script titled `example.mlx`. The example includes basic MATLAB-implementation on how to generate fCWT optimization plans and calculate time-frequency matrices using fCWT. To use fCWT with MATLAB, make sure you generate the MEX-files using the build option. For obvious reasons, generation of MEX-files is only possible if MATLAB is installed on your system. 

Note: Expect a decrease in performance when using fCWT via MATLAB. The official benchmark tests MATLAB's CWT implementation via the MATLAB interface and fCWT via the command line. We advice to use fCWT's MATLAB interface solely for plotting purposes. 


License
=======

fCWT is distributed under Apache 2.0 license. For conditions of distribution and use, see file `LICENSE.TXT`


References
==========

[1]     Rafat. Wavelib. https://github.com/rafat/wavelib. Accessed: 2020-04-22.\
[2]     R. Carmona, W.-L. Hwang, and B. Torresani.Practical Time-Frequency Analysis:  Gabor and wavelettransforms, with an implementation in S. Academic Press, 1998. [p1, 7, 10]

[CMake]: https://cmake.org/
[GCC]: https://gcc.gnu.org/
[Clang]: https://clang.llvm.org/
[OpenMP]: https://www.openmp.org/
[FFTW]: https://www.fftw.org/
[Visual_Studio]: https://www.visualstudio.com/
