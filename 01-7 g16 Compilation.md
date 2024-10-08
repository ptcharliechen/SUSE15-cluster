PGI compiler 隸屬於 Nvidia HPC SDK，如何下載請見：
https://github.com/ptcharliechen/SUSE15-cluster/blob/main/00-2%20GPU%20Node%20Installation.md#nvidia-hpc-sdk

儘量找和作業系統差不多時間推出的編譯器版本，glibc 版本才能吻合。

```
export g16root=/path/to/g16/root   # 不要進 g16
source /path/to/hpc/sdk/compiler
cd g16/
source bsd/g16.profile
bsd/bldg16
```

約等四十分鐘編譯完成。

```
chmod -R 750 $g16root/g16
chown -R root:[cluster group] $g16root/g16
```

由於 Gaussian 要求必須在同群組內才能操作 (權限不能設 777)，所以要把權限設為 750，並把 Gaussian 加入 cluster 成員所在的群組裡。

```
cd g16/
mkdir scratch
chmod 770 scratch
export GAUSS_SCRDIR=$g16root/g16/scratch
```

*scratch* 是計算時的臨時資料夾，必須隸屬於 g16。

> [!NOTE]
> 寫 slurm 檔一定要有
> ```
> export g16root=/path/to/g16/root
> export GAUSS_SCRDIR=$g16root/g16/scratch
> source $g16root/g16/bsd/g16.profile
> ```
> 否則抓不到 g16。

g09 編譯方式同理。

供參：[http://bbs.keinsci.com/thread-10814-1-1.html](http://bbs.keinsci.com/thread-10814-1-1.html)、[http://bbs.keinsci.com/thread-14301-1-1.html](http://bbs.keinsci.com/thread-14301-1-1.html)、[https://gaussian.com/running/](https://gaussian.com/running/)
