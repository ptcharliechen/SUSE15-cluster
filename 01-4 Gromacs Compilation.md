官方 FTP：https://ftp.gromacs.org/gromacs/

官方編譯建議：https://manual.gromacs.org/documentation/current/install-guide/index.html

須使用 cmake 3.18.4 或更新的版本。臺三和 214 的作業系統的 cmake 都比該版舊，所以建議抓取較新作業系統的 repository 或者用 module load。

> [!NOTE]
> 更新 repository 的指令：```zypper up [package name]```

官方建議使用 gcc 編譯，版本介於 9 到 11 之間，別忘記引入 OpenMPI。

```
mkdir build 
cd build
cmake .. -DGMX_BUILD_OWN_FFTW=OFF -DREGRESSIONTEST_DOWNLOAD=ON -DCMAKE_INSTALL_PREFIX=/path/of/gromacs -DCMAKE_C_COMPILER=gcc -DCMAKE_CXX_COMPILER=g++ -DGMX_MPI=ON -DGMX_GPU=OFF
make && make check
make install
```

REGRESSIONTEST_DOWNLOAD 為 ON 時會去網路下載 Regressiontest，故編譯節點必須有網路，否則會 cmake 會中止處理。

GMX_BUILD_OWN_FFTW 為 OFF 時，須保證作業系統裡有 FFTW。

> [!NOTE]
> - Xeon16 無法算 2023.3 版以及更新的版本。
> - 在研究室機器編譯：先在伺服器 cmake，再到計算節點 make。
