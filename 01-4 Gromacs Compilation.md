官方編譯建議：https://manual.gromacs.org/documentation/current/install-guide/index.html

須使用 cmake 3.18.4 或更新的版本。臺三和214的作業系統的cmake都比該版舊，所以建議抓取較新作業系統的repository或者用module load。 (更新 repository 的指令：zypper up [package name])

官方建議使用 gcc 編譯，版本介於 9 到 11 之間。

mkdir build 
cd build
cmake .. -DGMX_BUILD_OWN_FFTW=ON -DREGRESSIONTEST_DOWNLOAD=ON -DCMAKE_INSTALL_PREFIX=/path/of/gromacs -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DGMX_MPI=ON -DGMX_GPU=OFF
make && make check
make install

GMX_BUILD_OWN_FFTW和REGRESSIONTEST_DOWNLOAD的編譯節點必須要有網路，否則會中止編譯，因為它們會去網路下載FFTw和Regressiontest。Xeon16無法算 2023.3 版。
