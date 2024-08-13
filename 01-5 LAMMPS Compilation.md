```
git clone https://github.com/lammps/lammps
cd lammps/src
make yes-most
make mpi -j32
```

> [!NOTE]
> `make yes-most` 只是將一些套件準備好「預備編譯」，而非真的編譯。
> `make yes-most` 在 server 執行，`make mpi -j32` 在計算節點執行。

在 src/ 確認 lmp_mpi 存在。

```
ls -l lmp_mpi
```

供參：https://blog.csdn.net/xukang95/article/details/89377180

## GPU

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
make gpu -j8
```

> [!NOTE]
> - ```make no-all``` 是關掉所有的 package， ```make yes-gpu``` 是打開 GPU 的 package。
> - 用 ```make package-status``` 確認所有 package 是開還是關。

在 src/ 確認 lmp_gpu 存在。

```
ls -l lmp_gpu
```

供參：https://sites.google.com/site/rangsiman1993/comp-chem/program-install/install-lammps-pk-gpu?pli=1#h.p_lazO7LUaLY-6
