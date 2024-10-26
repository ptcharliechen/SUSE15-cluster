```
git clone https://github.com/lammps/lammps
cd lammps/src
make yes-most
make mpi -j32
```

> [!NOTE]
> - `make yes-most` 只是將一些套件準備好「預備編譯」，而非真的編譯。
> - `make yes-most` 在 server 執行，`make mpi -j32` 在計算節點執行。

在 src/ 確認 lmp_mpi 存在。

```
ls -l lmp_mpi
```

供參：https://blog.csdn.net/xukang95/article/details/89377180

## GPU

進入 GPU 節點，引入 OpenMPI 和 HPC SDK

```
source /usr/lib/hpc/gnu7/mpi/openmpi/3.1.6/bin/mpivars.sh
source /opt/nvidia/hpc.sh
```

```
vim lib/gpu/Makefile.mpi
```

確認 CUDA_HOME 和 CUDA_LIB 的參數

```
CUDA_HOME = /usr/local/cuda
CUDA_LIB = -L$(CUDA_HOME)/lib64  -L$(CUDA_HOME)/lib64/stubs
```

到 https://developer.nvidia.com/cuda-gpus 確認 CUDA_ARCH 的參數。以 v100 為例，Compute Capability 為 7.0，故取消註解 ```CUDA_ARCH = -arch=sm_70```。

供參：https://arnon.dk/matching-sm-architectures-arch-and-gencode-for-various-nvidia-cards/

```
CUDA_ARCH
```

進入 lib/gpu

```
make -f Makefile.mpi -j 8
```

確認 libgpu.a 是否存在 ```ls *.a```

退回到 LAMMPS 的根目錄 ```cd ../../```

```
vi src/MAKE/OPTIONS/Makefile.gpu
```

確認 LIB 的參數

```
LIB = -L/usr/local/cuda -lcudart
```

進入 src/

```
make no-all
make yes-gpu
make yes-most
make no-openmp
make mpi -j8
```

使用的 MPI 指令為
```
mpirun -np 1 lmp_mpi -sf gpu -pk gpu 1 neigh no -i in.lmp
```

如果有兩個 GPU，可以改成
```
mpirun -np 2 lmp_mpi -sf gpu -pk gpu 2 neigh no -i in.lmp
```

> [!WARNING]
> - 在 LAMMPS 中，一個 GPU 只能用一條 process，所以 -np 和 -pk gpu 必須相等，否則會報錯。
> - -pk gpu N 後面必須要有 neigh no，否則也會報錯。
> - 供參：https://docs.lammps.org/Errors_messages.html

> [!NOTE]
> - ```make no-all``` 是關掉所有的 package， ```make yes-gpu``` 是打開 GPU 的 package。
> - 用 ```make package-status``` 確認所有 package 是開還是關。

在 src/ 確認 lmp_gpu 存在。

```
ls -l lmp_gpu
```

供參：https://sites.google.com/site/rangsiman1993/comp-chem/program-install/install-lammps-pk-gpu?pli=1#h.p_lazO7LUaLY-6

https://hackmd.io/@isc21/rJY5v-AZO

## NequIP

### 安裝套件

使用 pip 安裝

```
pip3 install numpy==1.24.1
pip3 install torch==1.13.1+cu117 --extra-index-url https://download.pytorch.org/whl/cu117
pip3 install nequip
```

> [!NOTE]
> - 用 NequIP 建議的 pytorch 版本較為安全。
> - Numpy 近期推出 2.0 版，沒人知道會發生什麼事，用 1.0 版比較安全。

```
source /opt/nvidia/hpc.sh
source /path/to/intel/mkl/path/var.sh
```

> [!NOTE]
> CUDA compiler 用 nvcc，MKL 用 intel MKL 就好。

### cuDNN

如果有 cuDNN 就別管本節，這需要網管權限操作。

到 https://developer.nvidia.com/cudnn-archive 取得 cuDNN，選擇 CUDA 11.x 8.9.x 版，然後用 **Local Installer for Linux x86_64 (Tar)**。

下載後，執行

```
tar -Jxvf cudnn-linux-x86_64-8.9.x.xx_cuda11-archive.tar.xz
```

把 lib/ 放到 GPU 計算節點的 /usr/local/cuda/lib64，把 include/ 放到 /usr/local/cuda/include。

### LAMMPS 外掛 NequIP

```
git clone https://github.com/mir-group/pair_nequip.git
./patch_lammps.sh /path/to/lammps/
source /opt/nvidia/hpc.sh
export LD_LIBRARY_PATH=/usr/local/cuda:$LD_LIBRARY_PATH
cd lammps
mkdir build
cd build
cmake ../cmake -DCMAKE_PREFIX_PATH=`python -c 'import torch;print(torch.utils.cmake_prefix_path)'`
make -j$(nproc)
```

> [!NOTE]
> 抓 CUDA 以及 cuDNN 等 library 的路徑。

### LAMMPS 外掛 MACE

export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH
export PATH=/usr/local/cuda/bin:$PATH
cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=$(pwd) -D CMAKE_CXX_STANDARD=17 -D CMAKE_CXX_STANDARD_REQUIRED=ON -D BUILD_MPI=ON -D BUILD_SHARED_LIBS=ON -D PKG_KOKKOS=ON -D Kokkos_ENABLE_CUDA=ON -DCMAKE_PREFIX_PATH=`python -c 'import torch;print(torch.utils.cmake_prefix_path)'` -D PKG_ML-MACE=ON ../cmake
