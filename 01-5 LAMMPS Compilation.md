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

> [!NOTE]
> - ```make no-all``` 是關掉所有的 package， ```make yes-gpu``` 是打開 GPU 的 package。
> - 用 ```make package-status``` 確認所有 package 是開還是關。

在 src/ 確認 lmp_gpu 存在。

```
ls -l lmp_gpu
```

供參：https://sites.google.com/site/rangsiman1993/comp-chem/program-install/install-lammps-pk-gpu?pli=1#h.p_lazO7LUaLY-6
