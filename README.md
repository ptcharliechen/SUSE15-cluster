# 編譯軟體前應該知道的事

## 函式庫 (Library)
編譯時會把許多函式庫寫進參數檔 (VASP是 *makefile.include* ，Conquest是 *system.make* )，使編譯時可以鏈接，常見的函式庫有 *libgfortran.so* 、 *libblas.so* 、 *liblapack.so* 等， *libxxxx.so* 是固定命名方式。
```ldd /path/to/execution/file``` 可以看到該執行檔鏈接的函式庫，下圖以VASP641為例。

![圖片1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/b4d70903-ea37-4884-ae79-96600ff80830)

*libxxxx.so*是固定命名方式，因此去頭去尾，```-l```後面接xxxx作為標記方式，例如：```-lblas```意旨要鏈接 *libblas.so* 函式庫。

函式庫大多放在 */lib* 、 */lib64* 、 */usr/lib* 、 */usr/lib64* ，這些都是預設路徑，因此參數檔毋須詳加註明。如果是特別的函式庫路徑，會以```-L/path/of/library```表示，右為Conquest中scalapack的函式庫路徑：```-L/work1/***/Conquest/source/src/scalapack -lscalapack```，意即：到 */work1/\*\*\*/Conquest/source/src/scalapack* 找 *libscalapack.so* 函式庫。只寫```-llapack```就是到預設路徑找 *liblapack.so* 函式庫。

- 小練習
1. 使用 Intel compiler 時，會去抓 mkl 資料庫的路徑，存在 *MKLROOT* 變數中，因此```-L$(MKLROOT)/lib/intel64 -lmkl_scalapack_lp64 -lmkl_blacs_intelmpi_lp64```是什麼意思？順著路徑看看這些函式庫是否存在？
2. 參數檔常會看到```-lfftw3```，它在週期性計算中扮演相當重要的角色，網路上找得到下載檔，不過需要編譯，編譯過程中除了會報錯外，鏈接時也可能出問題，因此應先在計算節點尋找是否已經配備該函式庫。用```find```指令到研究室任意計算節點上 (伺服器沒有這些函式庫，別找錯)找找看是否存在該函式庫？

進行```source```時，會把變數PATH後面添加執行時會抓取的檔案的路徑， library 的路徑則是放在變數*LD_LIBRARY_PATH* ( *LIBRARY_PATH* 是在編譯時使用； *LD_LIBRARY_PATH* 是在執行時使用)。slurm submission script在```source``` Intel compiler時會添加 mkl 函式庫的路徑， VASP641 的 dftd4 函式庫路徑記載於```vasp_set.sh```，供執行時抓取。

![圖片2](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/f0778f5c-1827-4b46-bac1-8e3b5227a60d)

## pkg-config
pkg-config可用來檢索系統中函式庫的訊息。

![圖片3](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/dcd27fed-bca0-4989-94a4-6c771a034e7b)

通常會放在lib裡面，使用 ```export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/path/to/pkgconfig```
告知要搜尋該路徑的pkg-config
以上圖為例，輸入要讀取的pkg-config檔
```pkg-config --cflags dftd4```
```pkg-config --libs dftd4```

![圖片6](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/d081a825-d46d-4fb9-8d52-8df933454c92)

```--cflags```是給予標頭檔 (```.c file```)的鏈接資訊；```--libs```是給予函式庫的鏈接資訊。
將上述資訊貼到參數檔中 (以 dftd4 為例)

![圖片5](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/b2f89bd1-b326-479e-84ce-ed415cfc24d7)

## configure
```configure```檔會生成 *Makefile*  ，內含使用的編譯器，因此使用前要先引入 (```source```)編譯器。
輸入```./configure```後會產生 *Makefile* ，再執行```make```就可以編譯軟體或函式庫。

*configure* 預設以超級使用者 (superuser)身分編譯，編譯出來的函式庫會放在 */usr/local* 等超級使用者才能編輯的地方，因此使用臺三編譯時，要修改生成檔案的位置： ```./configure --prefix=/path/to/lib``` 即可修改生成的 *Makefile* 的參數。

```CC```: C語言編譯器；```FC```: Fortran編譯器；```CXX```: C++編譯器

透過```CC=[C compiler]```更換使用的C語言編譯器，更換其他編譯器同理，例如：```./configure FC=ifort```。
由於2024 intel compiler的```icc```和```ifort```已經更換成```icx```和```ifx```， *configure* 必要時要更換為```FC=ifort``` (```icc```已經在2024 compiler已經無效，必須用```icx```)。

## patch
它是用來內嵌程式碼的工具，不過版本不同時可能會不起作用，就要手動加入程式碼，故了解 *patch* 裡面的意義是必須的。下面是一個 *patch* 的範例：

```
--- solvation.F_org	2019-02-21 15:56:46.000000000 +0100
+++ solvation.F	2020-09-30 17:19:41.821342000 +0200
@@ -2203,7 +2203,7 @@
 !test
    CALL MY_D_PROD(Ecorr3, SP_CHTOT(1,1), Vcorr, SP_GRIDC)
 !   CALLMPI( M_sum_d(SP_GRIDC%COMM,Ecorr3,1))
-   CALLMPI( M_sum_s(SP_GRIDC%COMM,1,Ecorr3,0,0,0))
+   CALLMPI( M_sum_1(SP_GRIDC%COMM,Ecorr3))
 
 !-------------------------------------------------------------
 !Ecorr4 = Vdiel*n, 
```
***+++/---***
- 補丁頭是分別由---/+++開頭的兩行，用來表示要打補丁的文件。---開頭表示舊文件，+++開頭表示新文件。補丁會保留舊程式，在新程式裡打補丁，上例的舊程式是solvation.F_org，新程式是solvation.F。補丁頭是分別由---/+++開頭的兩行，用來表示要打補丁的文件。---開頭表示舊文件，+++開頭表示新文件。

***@@***
- 它通常由一部分不用修改的東西開始和結束，只是用來標示要修改的位置。上例即包含
```
 !test
    CALL MY_D_PROD(Ecorr3, SP_CHTOT(1,1), Vcorr, SP_GRIDC)
 !   CALLMPI( M_sum_d(SP_GRIDC%COMM,Ecorr3,1))
```
以及
```
 
 !-------------------------------------------------------------
 !Ecorr4 = Vdiel*n, 
```
- 他們通常以@@開始，結束於另一個塊的開始或者一個新的補丁頭。'-'後的數字代表該塊在舊文件的行號,修改前該塊總行數；'+'後的數字代表該塊在新文件的行號,修改後該塊總行數。上例要拿掉的```   CALLMPI( M_sum_s(SP_GRIDC%COMM,1,Ecorr3,0,0,0))```，添加```   CALLMPI( M_sum_1(SP_GRIDC%COMM,Ecorr3))```。
- patch 的上下行與待補丁的程式不一致就不會補上去，行數不一致也不會寫上去，最常發生在不同版本或已經補過丁導致行數不一致，所以補完後***要檢查是不是都有寫上去***，沒有的話手動貼上去。

參考網頁：[補丁(patch)的製作與應用](http://linux-wiki.cn/wiki/zh-tw/%E8%A1%A5%E4%B8%81(patch)%E7%9A%84%E5%88%B6%E4%BD%9C%E4%B8%8E%E5%BA%94%E7%94%A8)
# 編譯新 VASP

## VASP
網址：[https://www.vasp.at/sign_in/](https://www.vasp.at/sign_in/)

![圖片7](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/81589b8a-c439-4c29-8a4c-c41305dd58f1)

![圖片8](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/361a2545-26eb-404d-9ce6-51f1da5a57b1)

帳號、密碼見筆記。

![圖片9](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/400531e7-c126-4de0-af25-5db919a9ee53)

```tar -zxvf vasp.6.4.1.tgz``` 
```gunzip vdw_kernel.bindat.gz```
解壓縮。


## NEB (vtst)
網址：[http://theory.cm.utexas.edu/vtsttools/download.html](http://theory.cm.utexas.edu/vtsttools/download.html
)

![圖片10](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/7334086f-234e-402d-b8c5-694a5bc536c1)

```tar -zxvf vtstcode-197.tgz``` 解壓縮。


修改方式可能有所更改，請見：
[http://theory.cm.utexas.edu/vtsttools/installation.html](http://theory.cm.utexas.edu/vtsttools/installation.html)

```main.F```

```IF (LCHAIN) CALL chain_init( T_INFO, IO)```

改成

```CALL chain_init( T_INFO, IO)```

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

```.objects``` (是個隱藏檔)

添加
```
 bfgs.o dynmat.o instanton.o lbfgs.o sd.o cg.o dimer.o \
 bbm.o fire.o lanczos.o neb.o qm.o \
 pyamff_fortran/*.o ml_pyamff.o \
 opt.o \
```
在 chain.o 前 (每一列前面都要有 Tab)。

![圖片14](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/0c207b71-5d86-4a66-bd28-4ada45e67ebb)

變成

![圖片15](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/b99bb816-9e4f-4d75-ab8e-4606f6180e6a)

src 內的 makefile

```LIB= lib parser pyamff_fortran```

![圖片16](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/4ae12fa8-95b3-4866-99e5-2d6b52b55167)


## VASPsol
網址：[https://github.com/henniggroup/VASPsol/tree/master](https://github.com/henniggroup/VASPsol/tree/master)

![圖片11](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/f313bf8d-a6a2-4fb0-b613-bd9c12013424)

```unzip VASPsol-master.zip``` 解壓縮。


將其中的 *solvation.F* 貼到 *src* 資料夾裡，覆蓋原本檔案。
下指令```patch -p0 < …/VASPsol-master/src/patches/pbz_patch_610```
- patch 會去修改 src 裡的部分檔案，因此是在VASP的 src 裡運作，並引入 (“<“ 是標準輸入)patch的檔案。
- 一般使用者進入的IP沒有 patch，故在超級使用者進入的IP下指令。
在 make.include 檔內的 CPP_OPTIONS 加上```-Dsol_compat```

如下：

![圖片13](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/39b1b3f7-9ae8-4cf1-999e-1352b8188b38)

由於該軟體的 patch 已多年沒有更新，截至今日 (2024/5/24)官方的patch已經刪除，或許近日會補上。icc 仍可以用官方的patch進行編譯，放在 VASPsol-master.zip，不過在 gcc 會報錯，筆者找了非官方的 patch ，放在這個repository的Code裡。

## DFT-D4

網址：[https://github.com/dftd4/dftd4](https://github.com/dftd4/dftd4)

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/7dec3274-8014-4f37-b897-b97f78af34ed)

安裝影片：[https://www.bilibili.com/video/BV1nP4y1g7N2/](https://www.bilibili.com/video/BV1nP4y1g7N2/)

筆者撰寫時只支援 intel C compiler 和 gcc ，所以 AOCC ( AMD 的編譯器，在 AMD 的機器運算效率常比使用 intel 編譯器高) 和 CUDA ( GPU ) 都不支援。

國網中心的主機就有 anaconda ，可以使用 module load anaconda version ；研究室的機器直接用 ```conda init``` ，就會將環境寫入 .bashrc 裡。

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

```sh Anaconda3-5.3.0-Linux-x86_64.sh``` 該步驟詳細過程見 [https://ithelp.ithome.com.tw/articles/10237621](https://ithelp.ithome.com.tw/articles/10237621)

### 編譯 D4

module intel compiler & Intel MPI

cd [path of dftd4] 位置如下

![圖片1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/c484fb4d-75cb-428a-b6b7-5083e6dbd961)

```
FC=ifort CC=icc meson setup _build -Dfortran_link_args=-qopenmp    # 註： intel 2024 的 C compiler 為 icx ，Fortran compiler 為 ifx
meson test -C _build --print-errorlogs
meson configure _build --prefix=~/pkg/vasp.6.4.1/dftd4 -Dapi_v2=true
meson install -C _build
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:~/pkg/vasp.6.4.1/dftd4/lib64/pkgconfig/
```

```pkg-config --cflags dftd4```

跑出來的結果寫在 make.include 最後面，添加在參數 INCS 後方。

```pkg-config --libs dftd4```

跑出來的結果寫在 make.include 最後面，添加在參數 LLIBS 後方。

在 make.include 加上 ```CPP_OPTIONS += -DDFTD4```

![圖片3](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/f60101c0-62cc-4b4e-9a88-92825b4ec025)

將
```export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/.../pkg/vasp.6.4.1/dftd4/lib64```
加到 VASP 環境檔裡 (下圖的環境檔即 vasp_set.sh )

![圖片4](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/361ae1f2-14cd-4765-b6a0-96e18b3ed3de)

![圖片5](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/44ed4876-d8f9-4d2a-8e85-90cba45bcca7)

## gcc 版本

大部分與intel compiler相同，本節僅敘述不同之處。

擁有超級使用者權限的話，直接用
```
zypper in -y lapack-devel    # BLAS 和 LAPACK 一起裝
zypper in -y fftw3-devel
zypper in -y openblas-devel
```

Scalapack 用 ```zypper se scalapack``` 找，一般選用 openmpi 。

補充：

BLAS：基礎線性代數操作的數值庫（如向量或矩陣乘法）

LAPACK：解多元線性方程式、線性系統方程組的最小平方解、計算特徵向量、用於計算矩陣QR分解、以及奇異值分解等問題

OpenBLAS 可理解為 BLAS 和 LAPACK 的合併版，有超級使用者權限的話，可以自行安裝，國高可請他們安裝。如果想要自編的話，只有 OpenBLAS ， LAPACK 需要用到 BLAS ，而 BLAS 的源碼過舊，現代編譯器不支援，除非找舊版的編譯器，否則無法自編。

ScaLAPACK：以並行計算求解LAPACK面對的問題

FFTW：快速求解快速傅立葉變換 (Fast Fourier Transformation, FFT) —— 以矩陣求解傅立葉變換，以速度犧牲精度 —— 用於處理週期性結構

沒有超級使用者的權限，須從網路上取得 Openblas 、 Scalapack 和 fftw 的套件。

從網路上抓檔案用 wget [URL]

取得 URL 的方式：

### Openblas

網址：[https://www.openblas.net/](https://www.openblas.net/)

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/1a81109f-ae0c-48d8-a77c-736b79f7a0a7)

cd [path of openblas]

```make –j8
make PREFIX=…… install
```

### FFTW

網址：[https://www.fftw.org/download.html](https://www.fftw.org/download.html)

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/aef15e62-6c34-48ee-b105-1b4cf3915bfd)

```./configure --prefix=……
make && make install
```

### ScaLAPACK

網址：[https://www.netlib.org/scalapack/](https://www.netlib.org/scalapack/)

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/917f0228-bf73-4aac-864c-ff1defc4ea8c)

cd [path of scalapack]

```
mv SLmake.inc.example SLmake.inc
make
```

### Solvation Script

由於該軟體的 patch 已多年沒有更新，截至今日 (2023/10/27)官方的patch已經刪除，或許近日會補上。icc 仍可以用官方的patch進行編譯，放在 VASPsol-master.zip，不過在 gcc 會報錯，筆者找了非官方的 patch ，放在 VASPsol-master_gcc.tar.gz 裡。

### DFT-D4

記得用 gcc 編譯

由於要求 cmake 3.14 或更新的版本，因此在國高要 module load cmake ，研究室機器已經更新。

```FC=ifort CC=icc meson setup _build -Dfortran_link_args=-qopenmp```

換成

```FC=gfortran CC=gcc meson setup _build -Dfortran_link_args=-fopenmp -Dlapack=openblas```

## aocc 版本

gcc 在 AMD 機器計算較慢，建議使用 AMD 發布的編譯器： AOCC ，以及搭配它MPI編譯器： OpenMPI 。

與 intel compiler 不同， AOCC 要另外去找 AOCL ( AOCC 的 MKL )，臺三用 module avail 就可以找到合適的。

用 find 取得 libblis.so 、 libflame.so 、 libscalapack.so 和 libfftw3.so 的路徑，分別寫入 AMDBLIS_ROOT 、 AMDLIBFLAME_ROOT 、 AMDSCALAPACK_ROOT 、 AMDFFTW_ROOT (修改這些參數時不能有 lib_LP64 )，下一行的 lib 要改成 lib_LP64 ， fftw 的 include 要改成 include_LP64 。

註： lib_LP64 和 include_LP64 分別為資料夾名稱，代表上述資料夾的位置。

## GPU 版本

選用 nvhpc_acc 版本，有 acc 代表有使用到 CUDA ，也就是 GPU 計算引擎；沒有 acc 只是用 NVIDIA 的編譯器 (Nvidia HPC SDK)編譯，即使計算節點有 GPU 也不會使用。

確認所使用 CUDA 版本，輸入 ```nvidia-smi``` 即可得知，下圖的版本為 12.4。

<img width="551" alt="擷取" src="https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/3502819b-861f-4f69-b616-dc4f9e10352b">

然後確認 Nvidia HPC SDK 的版本，會在 ```/opt/nvdia/hpc_sdk/Linux_x86_64``` 中，下圖版本為 24.5。

<img width="319" alt="擷取3" src="https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/19f42d56-6dee-4799-aa54-cfe53091bc58">

將 makefile.include 改成當前版本，下圖右邊的是原始版本，左邊是修改版

<img width="770" alt="擷取1" src="https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/5552c7d8-3f49-4854-b732-8ac7ee903acf">

修改 fftw3 的路徑，有超級使用者權限可用 ```zypper in -y fftw3-devel``` 取得，否則按前文從原碼自行編譯。



# Conquest

