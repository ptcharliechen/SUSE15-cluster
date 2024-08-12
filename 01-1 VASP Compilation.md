# 編譯新 VASP

本篇僅講 VASP 641 ，之前的版本見 [github](https://github.com/HongScarlet/homework/blob/master/SUSE15%20cluster/15.%20SLES%2015%20Cluster%20New.md#vasp)

## VASP
網址：[https://www.vasp.at/sign_in/](https://www.vasp.at/sign_in/)

![圖片7](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/81589b8a-c439-4c29-8a4c-c41305dd58f1)

![圖片8](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/361a2545-26eb-404d-9ce6-51f1da5a57b1)

帳號、密碼見筆記。

![圖片9](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/400531e7-c126-4de0-af25-5db919a9ee53)

```
tar -zxvf vasp.6.4.1.tgz
gunzip vdw_kernel.bindat.gz
```
解壓縮。

## NEB (vtst)
網址：[http://theory.cm.utexas.edu/vtsttools/download.html](http://theory.cm.utexas.edu/vtsttools/download.html
)

![圖片10](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/7334086f-234e-402d-b8c5-694a5bc536c1)

```tar -zxvf vtstcode-197.tgz```
解壓縮。

> [!WARNING]
> 修改方式可能有所更改，請見：
> [http://theory.cm.utexas.edu/vtsttools/installation.html](http://theory.cm.utexas.edu/vtsttools/installation.html)

*main.F*

```
IF (LCHAIN) CALL chain_init( T_INFO, IO)
```

改成

```
CALL chain_init( T_INFO, IO)
```

另一部分

```
 CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
      LATT_CUR%A,LATT_CUR%B,IO%IU6)
```
改成
```
 CALL CHAIN_FORCE(T_INFO%NIONS,DYN%POSION,TOTEN,TIFOR, &
 	TSIF,LATT_CUR%A,LATT_CUR%B,IO%IU6)
```

*.objects* (是個隱藏檔)

添加
```
 bfgs.o dynmat.o instanton.o lbfgs.o sd.o cg.o dimer.o \
 bbm.o fire.o lanczos.o neb.o qm.o \
 pyamff_fortran/*.o ml_pyamff.o \
 opt.o \
```
在 **chain.o** 前 (每一列前面都要有 Tab)。

![圖片14](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/0c207b71-5d86-4a66-bd28-4ada45e67ebb)

變成

![圖片15](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/b99bb816-9e4f-4d75-ab8e-4606f6180e6a)

在 *src* 內的 *makefile* 添加

```
LIB= lib parser pyamff_fortran
```

![圖片16](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/4ae12fa8-95b3-4866-99e5-2d6b52b55167)


## VASPsol
網址：[https://github.com/henniggroup/VASPsol/tree/master](https://github.com/henniggroup/VASPsol/tree/master)

![圖片11](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/f313bf8d-a6a2-4fb0-b613-bd9c12013424)

```unzip VASPsol-master.zip```
解壓縮。

將其中的 *solvation.F* 貼到 *src* 資料夾裡，覆蓋原本檔案。

進入 src/ 的資料夾裡，輸入：

```
patch -p0 < …/VASPsol-master/src/patches/pbz_patch_610
```

> [!NOTE]
> - patch 會去修改 src 裡的部分檔案，因此是在VASP的 src 裡運作，並引入 (“<“ 是標準輸入)patch的檔案。
> - 在 SUSE15-cluster 裡有相關的 patch 可供下載。

patch 若有缺漏，請一個一個貼。

在 *make.include* 檔內的 **CPP_OPTIONS** 加上 **-Dsol_compat**

如下：

![圖片13](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/39b1b3f7-9ae8-4cf1-999e-1352b8188b38)

由於該軟體的 patch 已多年沒有更新，截至今日 (2024/5/24)官方的 patch 已經刪除，或許近日會補上。 icc 仍可以用官方的 patch 進行編譯，放在 *VASPsol-master.zip* ，不過在 gcc 會報錯，筆者找了非官方的 patch ，放在這個 repository 的 Code 裡。

## unit cell 基矢調整

VASP 的晶胞優化 (ISIF = 3)允許在 9 个自由度上自由弛豫，想固定其中幾個自由度，需要將 constr_cell_relax.F 的程式碼以下文覆蓋。

```
!-----------------------------------------------------------------------
!
! At present, VASP does not allow to relax the cell shape selectively
! i.e. for instance only cell relaxation in x direction.
! To be more precise, this behaviour can not be achieved via the INCAR
! or POSCAR file.
! However, it is possible to set selected components of the stress tensor
! to zero.
! The most convenient position to do this is the routines 
! CONSTR_CELL_RELAX  (constraint cell relaxation).
! FCELL contains the forces on the basis vectors.
! These forces are used to modify the basis vectors according
! to the following equations:
!
!      A_OLD(1:3,1:3)=A(1:3,1:3) ! F90 style 
!      DO J=1,3
!      DO I=1,3
!      DO K=1,3
!        A(I,J)=A(I,J) + FCELL(I,K)*A_OLD(K,J)*STEP_SIZE
!      ENDDO
!      ENDDO
!      ENDDO
! where A holds the basis vectors (in cartesian coordinates).
!
!-----------------------------------------------------------------------

      SUBROUTINE CONSTR_CELL_RELAX(FCELL)
      USE prec
      REAL(q) FCELL(3,3)

!     just one simple example
!     relaxation in x directions only
!      SAVE=FCELL(1,1)
!      FCELL=0   ! F90 style: set the whole array to zero
!      FCELL(1,1)=SAVE
!     relaxation in z direction only
!      SAVE=FCELL(3,3)
!      FCELL=0   ! F90 style: set the whole array to zero
!      FCELL(3,3)=SAVE

      LOGICAL FILFLG
      INTEGER ICELL(3,3)
      INQUIRE(FILE='OPTCELL',EXIST=FILFLG)
      IF (FILFLG) THEN
         OPEN(67,FILE='OPTCELL',FORM='FORMATTED',STATUS='OLD')
         DO J=1,3
            READ(67,"(3I1)") (ICELL(I,J),I=1,3)
         ENDDO
         CLOSE(67)
         DO J=1,3
         DO I=1,3
            IF (ICELL(I,J)==0) FCELL(I,J)=0.0
         ENDDO
         ENDDO
      ENDIF
    
      RETURN
      END SUBROUTINE
```

> [!NOTE]
> 摘要下篇引文：
> 設定 ISIF = 3，並新建一個檔案 OPTCELL。
> 
> 僅調整 $\vec{a}$ x 方向、 $\vec{b}$ x 和 y 方向，OPTCELL 內文如下：
>
> ```
> 100
> 110
> 000
> ```
> 1 是弛豫，0 是固定。

參考：https://blog.shishiruqi.com//2019/05/05/constr/

## CP-VASP

相關 Manual 在此：https://github.com/ptcharliechen/SUSE15-cluster/blob/main/CP-VASP%20Manual.pdf

> [!NOTE]
> 要先把 VASPsol 裝好後才能裝 CP-VASP。

可去 SUSE15-cluster 取得 cp-vasp6.patch

進入 src/ 的資料夾裡，輸入：

```
patch -p0 < …/cp-vasp6.patch
```

![image](https://github.com/user-attachments/assets/13360ea1-b5ec-41ee-ae81-46940d830860)

上圖是跑出來的結果，pot.F、reader.F 和 solvation.F 都跑出 FAILED，代表沒有 patch 成功或是有其他問題，進去這三個檔案確認，把缺的程式碼一個個貼在對應的地方。

## DFT-D4

網址：[https://github.com/dftd4/dftd4](https://github.com/dftd4/dftd4)

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/7dec3274-8014-4f37-b897-b97f78af34ed)

安裝影片：[https://www.bilibili.com/video/BV1nP4y1g7N2/](https://www.bilibili.com/video/BV1nP4y1g7N2/)

> [!CAUTION]
> 筆者撰寫時只支援 intel C compiler 和 gcc ，所以 AOCC ( AMD 的編譯器，在 AMD 的機器運算效率常比使用 intel 編譯器高) 和 CUDA ( GPU ) 都不支援。

國網中心的主機就有 anaconda ，可以使用 module load anaconda version ；研究室的機器直接用 ```conda init``` ，就會將環境寫入 *.bashrc* 裡。

### 安裝 Anaconda

```
wget https://repo.anaconda.com/archive/Anaconda3-5.3.0-Linux-x86_64.sh    # 如果沒有的話，先下載 anaconda
sh Anaconda3-5.3.0-Linux-x86_64.sh                                        # 註釋見下
source ~/.bashrc                                                          # 安裝後輸入，即可在當前頁面使用 conda。
conda create -n dftd4 python=3.7                                          # 為維持基底環境的乾淨，故建立一個虛擬環境，名為 dftd4
conda activate dftd4                                                      # 活化該環境。
conda install meson -c conda-forge                                        # 安裝 meson 及相依套件。
meson –v
ninja --version                                                           # 確認 ninja 和 meson 安裝完成。
```
> [!NOTE]
> ```sh Anaconda3-5.3.0-Linux-x86_64.sh``` 該步驟詳細過程見 [https://ithelp.ithome.com.tw/articles/10237621](https://ithelp.ithome.com.tw/articles/10237621)

### 編譯 D4

module intel compiler & Intel MPI

> [!CAUTION]
> 需要 cmake 3.14 或更新的版本。因此在國高要 module load cmake ，研究室機器已經更新，亦可以 zypper 更新。

```cd [path of dftd4]``` 位置如下

![圖片1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/c484fb4d-75cb-428a-b6b7-5083e6dbd961)

```
FC=ifort CC=icc meson setup _build -Dfortran_link_args=-qopenmp
meson test -C _build --print-errorlogs
meson configure _build --prefix=[expected dftd4 path] -Dapi_v2=true
meson install -C _build
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:[expected dftd4 path]/lib64/pkgconfig/
```

> [!NOTE]
> intel 2024 的 C compiler 為 icx ，Fortran compiler 為 ifx

```pkg-config --cflags dftd4     # 跑出來的結果寫在 make.include 最後面，添加在參數 INCS 後方。```

```pkg-config --libs dftd4       # 跑出來的結果寫在 make.include 最後面，添加在參數 LLIBS 後方。```

在 *make.include* 加上 **CPP_OPTIONS += -DDFTD4**

![圖片3](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/f60101c0-62cc-4b4e-9a88-92825b4ec025)

將
```export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:[expected dftd4 path]/dftd4/lib64```
加到 VASP 環境檔裡 (下圖的環境檔即 *vasp_set.sh* )

![圖片4](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/361ae1f2-14cd-4765-b6a0-96e18b3ed3de)

![圖片5](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/44ed4876-d8f9-4d2a-8e85-90cba45bcca7)

## gcc 版本

大部分與 intel compiler 相同，本節僅敘述不同之處。

擁有超級使用者權限的話，直接用
```
zypper in -y lapack-devel    # BLAS 和 LAPACK 一起裝
zypper in -y fftw3-devel
zypper in -y openblas-devel
```

Scalapack 用 ```zypper se scalapack``` 找，一般選用 openmpi 。

> [!Note]
> BLAS：基礎線性代數操作的數值庫（如向量或矩陣乘法）
> 
> LAPACK：解多元線性方程式、線性系統方程組的最小平方解、計算特徵向量、用於計算矩陣QR分解、以及奇異值分解等問題
> 
> OpenBLAS 可理解為 BLAS 和 LAPACK 的合併版，有超級使用者權限的話，可以自行安裝，國高可請他們安裝。如果想要自編的話，只有 OpenBLAS ， LAPACK 需要用到 BLAS ，而 BLAS 的源碼過舊，現代編譯器不支援，除非找舊版的編譯器，否則無法自編。
> 
> ScaLAPACK：以並行計算求解 LAPACK 面對的問題
> 
> FFTW：快速求解快速傅立葉變換 (Fast Fourier Transformation, FFT) —— 以矩陣求解傅立葉變換，以速度犧牲精度 —— 用於處理週期性結構
> 
> 沒有超級使用者的權限，須從網路上取得 Openblas 、 Scalapack 和 fftw 的套件。

從網路上抓檔案用 wget [URL]

取得 URL 的方式：

### Openblas

網址：[https://www.openblas.net/](https://www.openblas.net/)

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/1a81109f-ae0c-48d8-a77c-736b79f7a0a7)

```
cd [path of openblas]
make –j8
make PREFIX=[expected openblas path] install
```

### FFTW

網址：[https://www.fftw.org/download.html](https://www.fftw.org/download.html)

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/aef15e62-6c34-48ee-b105-1b4cf3915bfd)

```
./configure --prefix=[expected fftw path]
make && make install
```

### ScaLAPACK

網址：[https://www.netlib.org/scalapack/](https://www.netlib.org/scalapack/)

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/917f0228-bf73-4aac-864c-ff1defc4ea8c)

```
cd [path of scalapack]
mv SLmake.inc.example SLmake.inc
make
```

在 *makefile.include* 修改，下圖左邊是修改版，右邊的是原始版本

![擷取 (2)](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/8a69acd4-97c2-437d-9633-fd9ec6112a24)

### Solvation Script

由於該軟體的 patch 已多年沒有更新，截至今日 (2023/10/27)官方的 patch 已經刪除，或許近日會補上。icc 仍可以用官方的patch進行編譯，放在研究室機器 *VASPsol-master.zip* ，不過在 gcc 會報錯，筆者找了非官方的 patch ，放在 *VASPsol-master_gcc.tar.gz* 裡。

### DFT-D4

記得用 gcc 編譯

```FC=ifort CC=icc meson setup _build -Dfortran_link_args=-qopenmp```

換成

```FC=gfortran CC=gcc meson setup _build -Dfortran_link_args=-fopenmp -Dlapack=openblas```

## OpenMP 版本

選擇 *arch* 資料夾裡有 omp 後綴的檔案，改法與前面相同。

> [!NOTE]
> - OpenMP 以共享主記憶體並行執行緒 (thread)，MPI 每個執行緒間記憶體不共享。
> 
> - 執行 DFT-D4 計算時， OpenMP 的效率明顯高於 MPI。
> 
> - Intel Compiler 的 OpenMP 標籤是 -qopenmp，其他的是 -fopenmp。

## aocc 版本

gcc 在 AMD 機器計算較慢，建議使用 AMD 發布的編譯器： AOCC ，以及搭配它 MPI 編譯器： OpenMPI 。

與 intel compiler 不同， AOCC 要另外去找 AOCL ( AOCC 的 MKL )，臺三用 module avail 就可以找到合適的。

用 find 取得 *libblis.so* 、 *libflame.so* 、 *libscalapack.so* 和 *libfftw3.so* 的路徑，分別寫入 **AMDBLIS_ROOT** 、 **AMDLIBFLAME_ROOT** 、 **AMDSCALAPACK_ROOT** 、 **AMDFFTW_ROOT** (修改這些參數時不能有 **lib_LP64** )，下一行的 **lib** 要改成 **lib_LP64** ， fftw 的 **include** 要改成 **include_LP64**，下圖左邊是修改版，右邊的是原始版本。

![擷取](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/61752ae8-99c5-452a-bfab-5f009a58b494)

## GPU 版本

選用 nvhpc_acc 版本，有 acc 代表有使用到 CUDA ，也就是 GPU 計算引擎；沒有 acc 只是用 NVIDIA 的編譯器 (Nvidia HPC SDK)編譯，即使計算節點有 GPU 也不會使用。

確認所使用 CUDA 版本，輸入 ```nvidia-smi``` 即可得知，下圖的版本為 12.4。

![擷取](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/3502819b-861f-4f69-b616-dc4f9e10352b)

然後確認 Nvidia HPC SDK 的版本，會在 */opt/nvdia/hpc_sdk/Linux_x86_64* 中，下圖版本為 24.5。

![擷取3"](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/19f42d56-6dee-4799-aa54-cfe53091bc58)

將 *makefile.include* 改成當前版本，下圖左邊是修改版，右邊的是原始版本

![擷取1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/5552c7d8-3f49-4854-b732-8ac7ee903acf")

修改 fftw3 的路徑，有超級使用者權限可用 ```zypper in -y fftw3-devel``` 取得，否則按 gcc 小節的說明，從源碼自行編譯。

![擷取2](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/c3121227-786f-430d-916f-8861d556ad48)

在 VASP 611 需要 source intel compiler 和 Nvidia HPC SDK (Nvidia 的 compiler)兩種編譯器，而 VASP 641 只要後者就好：

```source /opt/nvidia/hpc.sh```
