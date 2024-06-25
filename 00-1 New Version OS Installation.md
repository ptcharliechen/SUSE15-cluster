> [!WARNING]
> 僅適用 15SP5 以後的版本，較舊的版本的 zypper 比較沒那麼「聰明」，不會抓未安裝的相依套件。
> 
> 本文僅編寫如何快速在計算節點安裝新版本 OS ，涉及較私密部分見手冊。
> 
> 另外， slurm node 最好跟 slurm controller 版本一致，所以會有舊版沒有的 repository 切換。

**粗體字**代表在該檔案寫入的文字；*斜體字*代表檔案名稱。

Server 代表是在伺服器操作；Client 代表在計算節點操作。

> [!NOTE]
> 安裝時設定好網路，即可不用待在吵雜的機房裡操作。

# SSH

```
Server :~ # ssh-copy-id root@Client
Server :~ # vim /etc/hosts.sh
```
**scp /etc/hosts root@Client:/etc/hosts**
```
Server :~ # /etc/hosts.sh
```

# NIS、NFS 和 防火牆

```
Client :~ # zypper in -y yast2-nis-client yast2-nfs-client
```

防火牆、NIS 和 NFS 見手冊。
> [!NOTE]
> ypbind 可以不用 zypper 不安裝，進入 yast 的 NIS Client 會自動安裝。

```
Client :~ # getent passwd
Client :~ # ll /whome
```

# 基礎套件安裝

```
Client :~ # /work1/pkg/pkg.sh
Client :~ # zypper in -y kernel-devel lapack-devel fftw-devel openblas-devel
```
> [!NOTE]
> gcc 在 kernel-devel 灌入。
```
Client :~ # zypper se scalapack
```

找到合適的 ScaLAPACK 版本灌入，通常選用 OpenMPI 版。

# Chrony

```
Client :~ # vim /etc/chrony.config
```
**server 192.168.1.1**

```
Client :~ # systemctl enable chronyd --now
Client :~ # chronyc sources
Client :~ # chronyc burst 4/4
Client :~ # chronyc makestep
```

# munge 和 slurm-node

```
Client :~ # /work1/pkg/pkg_sp2.sh
Client :~ # zypper in -y slurm-node
Client :~ # /work1/pkg/pkg.sh
Server :~ # scp /etc/munge/munge.key root@Client:/etc/munge
Client :~ # systemctl enable munge --now
Server :~ # munge -n | ssh Client umunge
```

> [!NOTE]
> munge 會在安裝 slurm node 時同步安裝。

> [!NOTE]
> munge 只要能 ssh 並回傳成功，代表 munge key 一樣，暗指同時確認
> ```
> Server :~ # md5sum /etc/munge/munge.key
> Client :~ # md5sum /etc/munge/munge.key
> Server :~ # munge -n
> Client :~ # munge -n
> Server :~ # munge -n | unmunge
> Client :~ # munge -n | unmunge
> ```
> 的 key 是否相同。

> [!NOTE]
> */work1/pkg/pkg.sh* 包含 15SP2 和 15SP5 的 repository ，不過 15SP5 的優先度略高於 15SP2 ，因應日益更新的軟體需要較新的套件支援。不過涉及和伺服器溝通的 slurm 最好與其版本一致，所以使用 */work1/pkg/pkg_sp2.sh* 將 repository 改成 15SP2 以安裝 slurm-node ，之後再改回 15SP5。

> [!NOTE]
> */work1/pkg/pkg_sp5.sh* 只有 15SP5 的 repository。

```
Server :~ # vi /etc/slurm/slurm.conf
```
新增計算節點資訊並把計算節點加入 partition 。

e.g.

- CPU: **NodeName=e01 State=idle Sockets=2 CoresPerSocket=8**

- GPU: **NodeName=g01 State=idle Gres=gpu:4 Sockets=1 CoresPerSocket=8**

> [!Note]
> GPU 需要新增 */etc/slurm/gres.conf*
> 
> e.g.
> 
> - **NodeName=g01 Name=gpu File=/dev/nvidia[0-3]**
> 
> 如果是 GPU 要傳 *gres.conf* 給 GPU 計算節點， CPU 計算節點不用：
> 
> ```Server :~ # scp /etc/slurm/gres.conf root@Client:/etc/slurm```

```
Server :~ # vim /etc/slurm/slurmconf.sh
```

**scp /etc/slurm/slurm.conf root@Client:/etc/slurm/.**

```
Server :~ # /etc/slurm/slurmconf.sh
Client :~ # vi /usr/lib/systemd/system/slurmd.service
```
**#PIDFile=/var/run/slurm/slurmd.pid**

> [!NOTE]
> 註釋 PIDFile。

```
Client :~ # vi /etc/tmpfiles.d/slurm.conf
```

**d /var/run/slurm 0770 root root -**

```
Client :~ # mkdir -p /var/log/slurm
Client :~ # chown -R slurm:slurm /var/log/slurm
Client :~ # systemctl enable slurmd --now
Server :~ # systemctl restart slurmctld
```

# GPU

見 [00-2 GPU Node Installation.md](https://github.com/ptcharliechen/SUSE15-cluster/blob/main/00-2%20GPU%20Node%20Installation.md)
