與 CPU 相同的部分參考網管筆記或 [github](https://github.com/HongScarlet/homework/blob/master/SUSE15%20cluster/15.%20SLES%2015%20Cluster%20New.md)，15SP5 或更新版的解說見[此](https://github.com/ptcharliechen/SUSE15-cluster/blob/main/00-1%20New%20Version%20OS%20Installation.md)，只講 GPU 的部分。

> [!CAUTION]
> - CUDA 版本與 linux 核心版本須互相搭配，因此必須注意 CUDA 和 SLES 釋出的日期是否匹配 (例如 CUDA 11.7 與 15SP4 搭配；CUDA 12.4 與 15SP5 搭配)，否則 CUDA driver 無法自動安裝。
> - 由於 HPC SDK 和 CUDA 版本必須互相搭配，所以先確定所使用的 HPC SDK 版本所搭配的 CUDA 的版本。

> [!CAUTION]
> 安裝 kernel-devel、gcc-fortran、gcc-c++ 等與 Linux 核心相關的套件須與所使用的 SLES 版本一致 (更準確來說是 Linux driver 版本須與 kernel-devel 套件版本一致)，也就是說：裝完上述的套件後再用 /work1/pkg/pkg.sh 更換 module 的版本。

Nvidia HPC SDK 網址：[https://developer.nvidia.com/nvidia-hpc-sdk-releases](https://developer.nvidia.com/nvidia-hpc-sdk-releases)

CUDA 網址：[https://developer.nvidia.com/cuda-toolkit-archive](https://developer.nvidia.com/cuda-toolkit-archive)

# Nvidia HPC SDK

下圖為 SDK 24.5 的安裝檔，紅框表示使用的 CUDA 為 12.4。

![擷取](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/b3cfce54-857a-4d0b-b4c1-31379a20be83)

按指示安裝，下圖紅框處選 "1"。

![擷取5](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/a72ea781-3a4f-4024-af86-1c6b05b92010)


註：
1. 在臺灣時間的白天抓檔案會比較快 (約十幾分鐘)，晚上以後 (也就是美國白天)會相當慢 (慢則一小時以上)。
2. 由於計算節點沒有連網際網路，所以要取得壓縮檔 ( tar file ) 解壓縮而非選擇 zypper。

# CUDA

> [!CAUTION]
> CUDA 安裝之後發現弄錯要去掉重裝可能會失敗，甚至重灌比較快。

按下圖選擇按鈕，安裝方式選擇 rpm 。

![擷取1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/15a0ca28-67c7-4a6b-92a5-fb83ce38e1e6)

用 wget 和 rpm 後，使用 ```zypper se cuda``` 會出現下圖，選擇 "cuda" 下面的選項安裝 (以下圖為例，即 "cuda-12-4" )。

![擷取3](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/46bc3ec9-fc77-4961-ab77-8b39a6cd7958)

下圖紅框的部分，可以選 "a" 或 "t"。

![擷取4](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/b0dc5e31-4aa9-490c-a4aa-cb860aa10631)

以下為 SDK 24.5 的 source 檔：
```
NVARCH=`uname -s`_`uname -m`; export NVARCH
NVCOMPILERS=/opt/nvidia/hpc_sdk; export NVCOMPILERS
HPCSDK=24.5; export HPCSDK
CUDA=12.4; export CUDA
MANPATH=$MANPATH:$NVCOMPILERS/$NVARCH/$HPCSDK/compilers/man; export MANPATH
PATH=$NVCOMPILERS/$NVARCH/$HPCSDK/compilers/bin:/opt/nvidia/hpc_sdk/Linux_x86_64/$HPCSDK/comm_libs/mpi/bin:$PATH; export PATH

export CUDA_ROOT=$NVCOMPILERS/$NVARCH/$HPCSDK/compilers
export CUDA_PATH=$NVCOMPILERS/$NVARCH/$HPCSDK/cuda/$CUDA/targets/x86_64-linux
export CUDA_MATH=$NVCOMPILERS/$NVARCH/$HPCSDK/math_libs/$CUDA/targets/x86_64-linux

export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$NVCOMPILERS/$NVARCH/$HPCSDK/cuda/$CUDA/targets/x86_64-linux/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$NVCOMPILERS/$NVARCH/$HPCSDK/math_libs/$CUDA/targets/x86_64-linux/lib
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:$NVCOMPILERS/$NVARCH/$HPCSDK/compilers/extras/qd/lib
```
**HPCSDK** 和 **CUDA** 的參數隨版本更動。

# GID 校正

15SP5 (計算節點) **video** (GPU 的 group) 的 GID 是 483，而 15SP2 (伺服器)是 485，不在同一個 group 而無法使用 GPU 計算。

```
vim /etc/modprobe.d/50-nvidia-default.conf
```

**NVreg_DeviceFileGID** 改成 485，儲存退出重啟。

