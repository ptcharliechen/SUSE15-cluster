# 編譯新 Quantum Espresso

下載網址：[https://www.quantum-espresso.org/download-page/](https://www.quantum-espresso.org/download-page/)

## Intel 編譯器

```
./configure CC=icc CXX=icpc F90=ifort F77=ifort MPIF90=mpiifort --with-scalapack=intel
make all -j16
```

> [!NOTE]
> Intel 2024 compiler 要將 F90 換成 ifx，其他要不要換都可以。

> [!NOTE]
> 使用不同編譯器編譯時，用**源碼** (官網下載的 QE )編，不要拿其他編譯器編好的複製過來清空 (make clean)後編譯，途中會出錯。

## gcc 編譯器

```
./configure CC=gcc CXX=g++ F90=gfortran F77=gfortran MPIF90=mpifort --with-scalapack=openmpi
module 或 source intel mkl (只要mkl就好)
make all
```

使用 gcc 編譯器建議仍使用 intel 的 mkl ，因為 intel 仍有配備 gcc 版本可使用的 library 。

## pslibrary

Pslibrary 會自動生成 QE 所須的 pseudopotential 檔案。

```
git clone https://github.com/dalcorso/pslibrary
```

使用 intel 編譯器編譯。如果 module load 會報錯，直接 source intel 編譯器之 compiler、mpi、mkl 的 env/vars.sh 。

QE_path 當中的 PWDIR 修改成 QE 的根路徑。

```
. ./make_all_ps
```

> [!NOTE]
> make_all_ps 會讀取 QE 當中的 bin/ld1.x ，是故要設定 PWDIR 成 QE 的路徑。make_all_ps 會補 bin/ ，所以寫到根目錄。

![image](https://github.com/user-attachments/assets/0fc25e4e-dfbf-430d-829d-5c2d1e921daf)
