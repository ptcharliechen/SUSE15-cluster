# 編譯新 Conquest

## LibXC

Conquest 的強大在於大規模平行運算，以線性組合簡化原子間關係，稱 MSSF (Multi-site support functions)，而內建的 LibXC 無法在 MSSF 使用，需要自編 LibXC。

網址：[https://libxc.gitlab.io/download/previous/](https://libxc.gitlab.io/download/previous/)

從網路上抓檔案 wget [URL]

![圖片1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/aff21b3b-85d3-460b-9007-4017386e43b0)

```tar -jxvf libxc-x.x.x.tar.bz2``` 解壓縮。

```
autoreconf -i
./configure --prefix==[expected libxc path] FC=ifort
make && make install
```
> [!NOTE]
> - autoreconf 為了產生 configure 檔。

> [!WARNING]
> - 使用 intel compiler 2023 和 2024 時， ```./configure --prefix==[expected libxc path]``` 的 Fortran 編譯器是 ifx，結果會報錯，因此要改成 ifort。

## CONQUEST v.1.3

以下是該版本的可執行檔。

```
# This is an example system-specific makefile. You will need to adjust
# it for the actual system you are running on.

# Set compilers
FC=mpiifort

# OpenMP flags
# Set this to "OMPFLAGS= " if compiling without openmp
# Set this to "OMPFLAGS= -fopenmp" if compiling with openmp
OMPFLAGS=-qopenmp
# Set this to "OMP_DUMMY = DUMMY" if compiling without openmp
# Set this to "OMP_DUMMY = " if compiling with openmp
OMP_DUMMY = 

# Set BLAS and LAPACK libraries
# MacOS X
# BLAS= -lvecLibFort
# Intel MKL use the Intel tool
# Generic
#BLAS= -llapack -lblas
# Full scalapack library call; remove -lscalapack if using dummy diag module.
# If using OpenMPI, use -lscalapack-openmpi instead.
# If using Cray-libsci, use -llibsci_cray_mpi instead.
#SCALAPACK = -lscalapack

# LibXC: choose between LibXC compatibility below or Conquest XC library

# Conquest XC library
#XC_LIBRARY = CQ
#XC_LIB =
#XC_COMPFLAGS =

# LibXC compatibility
# Choose LibXC version: v4 (deprecated) or v5/6 (v5 and v6 have the same interface)
#XC_LIBRARY = LibXC_v4
XC_LIBRARY = LibXC_v5
XC_LIB = -L/path/to/libxc/lib -lxcf90 -lxcf03 -lxc
XC_COMPFLAGS = -I/path/to/libxc/include

# Set FFT library
FFT_LIB=-lfftw3
FFT_OBJ=fft_fftw3.o

LIBS= $(FFT_LIB) $(XC_LIB) $(SCALAPACK) $(BLAS)

# Compilation flags
# NB for gcc10 you need to add -fallow-argument-mismatch
COMPFLAGS= -g -traceback -O3 -xCORE-AVX512 -fp-model strict -qopt-report $(OMPFLAGS) $(XC_COMPFLAGS)

# Linking flags
LINKFLAGS= -qmkl=parallel -lmkl_scalapack_lp64 -lmkl_blacs_intelmpi_lp64 -static-intel -L/usr/local/lib $(OMPFLAGS)

# Matrix multiplication kernel type
MULT_KERN = default
# Use dummy DiagModule or not
DIAG_DUMMY =
```

> [!NOTE]
> - v.1.3 才有OpenMP。
> - OpenMPI 的 OpenMP 的標示是 *-fopenmp* ，而 Intel 版本的是 *-qopenmp*，要設定 *OMPFLAGS=-qopenmp*。
> - *XC_LIBRARY = CQ* 是內建的 LibXC，須註解，同時打開 *XC_LIBRARY = LibXC_v5* 以及 *XC_LIB* 和 *XC_COMPFLAGS* 的註解。
> - LibXC v.5 和 v.6 的介面相同，所以 v.5 以上的版本 *XC_LIBRARY = LibXC_v5*。
