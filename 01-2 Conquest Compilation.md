## LibXC

Conquest 的強大在於大規模平行運算，以線性組合簡化原子間關係，稱 MSSF (Multi-site support functions)，而內建的 LibXC 無法在 MSSF 使用，需要自編 LibXC。

網址：[https://libxc.gitlab.io/download/previous/](https://libxc.gitlab.io/download/previous/)

從網路上抓檔案 wget [URL]

![圖片1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/aff21b3b-85d3-460b-9007-4017386e43b0)

```tar -jxvf libxc-x.x.x.tar.bz2``` 解壓縮。

```
autoreconf -i
./configure --prefix=/expected/libxc/path FC=ifort
make -j32 && make install
```
> [!NOTE]
> autoreconf 為了產生 configure 檔。

> [!CAUTION]
> 使用 intel compiler 2023 和 2024 時， ```./configure --prefix=/expected/libxc/path``` 的 Fortran 編譯器是 ifx，結果會報錯，因此要改成 ifort。

## Conquest

網址：[https://conquest.readthedocs.io/en/latest/installing.html](https://conquest.readthedocs.io/en/latest/installing.html)

下載點：[https://github.com/OrderN/CONQUEST-release/archive/master.zip](https://github.com/OrderN/CONQUEST-release/archive/master.zip)

如果沒有要改 system.make，參此編譯：

```
unzip CONQUEST-release-master.zip
cd ./src
make -j32
```

> [!NOTE]
> *-j32* 是用三十二核心平行編譯，如果沒有加此標籤即單核心編譯。 VASP 的程式間相依性高，所以多核心編譯容易報錯而中斷， Conquest 相依性低，所以沒有中斷問題。

## CONQUEST v.1.2

修改 src 裡 system.make

### mpif90

本節不用 Intel MKL，Intel 編譯器仍可使用。

```
# Set compilers
FC=mpif90
F77=mpif77

# Linking flags
LINKFLAGS= -L/usr/local/lib
ARFLAGS=

# Compilation flags
# NB for gcc10 you need to add -fallow-argument-mismatch
COMPFLAGS= -O3 $(XC_COMPFLAGS)
COMPFLAGS_F77= $(COMPFLAGS)

# Set BLAS and LAPACK libraries
# MacOS X
# BLAS= -lvecLibFort
# Intel MKL use the Intel tool
# Generic
 BLAS= -llapack -lblas

# Full library call; remove scalapack if using dummy diag module
LIBS= $(FFT_LIB) $(XC_LIB) -L -L/path/to/scalapack -lscalapack $(BLAS)

# LibXC compatibility (LibXC below) or Conquest XC library

# Conquest XC library
#XC_LIBRARY = CQ
#XC_LIB =
#XC_COMPFLAGS =

# LibXC compatibility
# Choose LibXC version: v4 (deprecated) or v5/6 (v5 and v6 have the same interface)
# XC_LIBRARY = LibXC_v4
XC_LIBRARY = LibXC_v5
XC_LIB = -L/path/to/libxc/lib -lxcf90 -lxcf03 -lxc
XC_COMPFLAGS = -I/path/to/libxc/include

# Set FFT library
FFT_LIB=-lfftw3
FFT_OBJ=fft_fftw3.o

# Matrix multiplication kernel type
MULT_KERN = default
# Use dummy DiagModule or not
DIAG_DUMMY =
```

> [!NOTE]
> - *XC_LIBRARY = CQ* 是內建的 LibXC，須註解，同時打開 *XC_LIBRARY = LibXC_v5* 以及 *XC_LIB* 和 *XC_COMPFLAGS* 的註解，並輸入所在的 library 和 include 路徑。
> - LibXC v.5 和 v.6 的介面相同，所以 v.5 以上的版本 *XC_LIBRARY = LibXC_v5* 都適用。
> - 請確認 */usr/lib64* 是否有 *libblas.so*、*liblapack.so* 和 *libfftw3.so*，詳見 [gcc版本](https://github.com/ptcharliechen/SUSE15-cluster/blob/main/01-1%20VASP%20Compilation.md#gcc-%E7%89%88%E6%9C%AC)。
> - libscalapack.so 須自編，詳見 [gcc版本](https://github.com/ptcharliechen/SUSE15-cluster/blob/main/01-1%20VASP%20Compilation.md#gcc-%E7%89%88%E6%9C%AC)。
> - 若是使用 gcc10 或更新的 gcc 編譯器，將
>   
> *COMPFLAGS= -O3 $(XC_COMPFLAGS)*
>
> *COMPFLAGS_F77= $(COMPFLAGS)*
> 
> 改成
> 
> *COMPFLAGS= -O3 $(XC_COMPFLAGS) -fallow-argument-mismatch*
> 
> *COMPFLAGS_F77= $(COMPFLAGS) -fallow-argument-mismatch*


### Intel MKL

```
# Set compilers
FC=mpiifort
F77=mpiifort

# Linking flags
LINKFLAGS=-qmkl=parallel -lmkl_scalapack_lp64 -lmkl_blacs_intelmpi_lp64 -static-intel
ARFLAGS=

# Compilation flags
# NB for gcc10 you need to add -fallow-argument-mismatch
COMPFLAGS=  -g -traceback -O3 -xCORE-AVX512 -fp-model strict -qopt-report $(XC_COMPFLAGS)
COMPFLAGS_F77= $(COMPFLAGS)

# Set BLAS and LAPACK libraries
# MacOS X
# BLAS= -lvecLibFort
# Intel MKL use the Intel tool
# Generic
# BLAS=

# Full library call; remove scalapack if using dummy diag module
LIBS= $(FFT_LIB) $(XC_LIB)

# LibXC compatibility (LibXC below) or Conquest XC library

# Conquest XC library
#XC_LIBRARY = CQ
#XC_LIB =
#XC_COMPFLAGS =

# LibXC compatibility
# Choose LibXC version: v4 or v5
# XC_LIBRARY = LibXC_v4
 XC_LIBRARY = LibXC_v5
 XC_LIB = -L/path/to/libxc/lib -lxcf90 -lxcf03 -lxc
 XC_COMPFLAGS = -I/path/to/libxc/include

# Set FFT library
FFT_LIB= -lfftw3
FFT_OBJ=fft_fftw3.o

# Matrix multiplication kernel type
MULT_KERN = default
# Use dummy DiagModule or not
#DIAG_DUMMY =
```

### AOCL

```
# Set compilers
FC=mpif90
F77=mpif77

# Linking flags
LINKFLAGS= -L/usr/local/lib
ARFLAGS=

# Compilation flags
# NB for gcc10 you need to add -fallow-argument-mismatch
COMPFLAGS= -O3 $(XC_COMPFLAGS)
COMPFLAGS_F77= $(COMPFLAGS)

# Set BLAS and LAPACK libraries
# MacOS X
# BLAS= -lvecLibFort
# Intel MKL use the Intel tool
# Generic
BLAS= -lflame -lblis

# Full library call; remove scalapack if using dummy diag module
LIBS= $(FFT_LIB) $(XC_LIB) -L/path/to/lib_LP64 -lscalapack $(BLAS)

# LibXC compatibility (LibXC below) or Conquest XC library

# Conquest XC library
#XC_LIBRARY = CQ
#XC_LIB =
#XC_COMPFLAGS =

# LibXC compatibility
# Choose LibXC version: v4 (deprecated) or v5/6 (v5 and v6 have the same interface)
# XC_LIBRARY = LibXC_v4
 XC_LIBRARY = LibXC_v5
 XC_LIB = -L/path/to/libxc/lib -lxcf90 -lxcf03 -lxc
 XC_COMPFLAGS = -I/path/to/libxc/include

# Set FFT library
FFT_LIB=-L/path/to/lib_LP64 -lfftw3
FFT_OBJ=fft_fftw3.o

# Matrix multiplication kernel type
MULT_KERN = default
# Use dummy DiagModule or not
DIAG_DUMMY =
```

> [!NOTE]
> - 將BLAS參數的 *-llapack* *-lblas* 改成 *-lflame* *–lblis*。
> - LibXC 用 AOCC/AOCL 編譯為佳，生成 makeFile 檔的步驟只要修改右列 ```./configure --prefix==[expected libxc path] FC=flang CC=clang```。
> - LIBS的 參數前面加上 AOCL 的路徑 (-L/path/to/AOCL 寫到 lib_LP64 ，這樣 *libfftw3.so* 、*libblis.so* 、 *libflame.so* 、 *libscalapack.so* 都會指到該路徑上)。

## CONQUEST v.1.3

在此只示範 Intel MKL 版。以下是要修改的地方：

*FC=mpiifort*

*F77=mpiifort*

*LINKFLAGS=-qmkl=parallel -lmkl_scalapack_lp64 -lmkl_blacs_intelmpi_lp64 -static-intel*

*COMPFLAGS= -g -traceback -O3 -xCORE-AVX512 -fp-model strict -qopt-report $(XC_COMPFLAGS)*


修改 src/system 裡 system.make ，以下是 OMP 版可執行檔。

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
> - v.1.3 才有 OpenMP。
> - MPI 版把 *OMPFLAGS* 的參數刪掉。
> - 其他見 [編譯時注意事項](https://github.com/ptcharliechen/SUSE15-cluster/blob/main/01-0%20Before%20Compilation.md#%E7%B7%A8%E8%AD%AF%E6%99%82%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A0%85) 和 [執行時注意事項](https://github.com/ptcharliechen/SUSE15-cluster/blob/main/01-0%20Before%20Compilation.md#%E5%9F%B7%E8%A1%8C%E6%99%82%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A0%85)。

## 編譯 pseudopotential

```
cp ./src/system.make ./tools/BasisGeneration
cd ./tools/BasisGeneration
make
```
> [!WARNING]
> 如果有 PMI 相關錯誤的話，輸入 ```unset I_MPI_PMI_LIBRARY```

進入 pseudo-and-pao 其中的 PBE 的 test_default_PAOs.sh

加入 *cp "$ele"CQ.ion "$ele".ion* ，有需要的話可添補， *echo ""$ele" is completed."* 追蹤進度。

![圖片2](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/1a574e59-0497-4886-9849-4baf130930b0)

![圖片3](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/debf2974-ce4a-47a6-ad9c-c00b9cb76fb0)

然後輸入 ./test_default_PAOs.sh 以生成 .ion 檔。

> [!WARNING]
> LDA 和 PBEsol 會報錯。

### pseudopotential 精度修改

Conquest 的基組 (basis sets)有右列精度：
minimal (single zeta, SZ)、
small (single zeta and polarisation, SZP)、
medium (double zeta, single polarisation, DZP)、
large (triple zeta, triple polarisation, TZTP)。

預設是 medium。
在 PBE 的資料夾下輸入
```sed –i 's/medium/[precision]/g' */Conquest_ion_input```
即可將 medium 改成指定精度，剩下與上一節相同。

下圖是將precision從 medium 改成 large 的例子。

![圖片5](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/cb49c388-09f9-4504-a07a-7c6188420c99)

### csub

現行的 csub 可以在第二個參數選 pseudopotential 的精度：
```csub [job name] [precision]```。

有 *low* 、 *medium* 、 *high* 三種 precision，抓取方式寫在 **cpreprocess.py** 裡。 **cpreprocess.py** 內部的判斷邏輯寫在 此。
