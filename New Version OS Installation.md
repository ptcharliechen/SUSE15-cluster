> [!WARNING]
> 僅適用 15SP5 以後的版本，較舊的版本的 zypper 比較沒那麼「聰明」，不會抓未安裝的相依套件。
> 本文僅編寫如何快速在計算節點安裝新版本 OS ，涉及較私密部分見手冊。
> 另外， slurm node 最好跟 slurm controller 版本一致，所以會有舊版沒有的 repository 切換。

Server 代表是在伺服器操作；Client 代表在計算節點操作。

> [!NOTE]
> 安裝時設定好網路，即可不用待在吵雜的機房裡操作。

```
Server :~ # ssh-copy-id root@Client
Server :~ # scp /etc/hosts root@Client:/etc
Client :~ # zypper in -y yast2-nis-client yast2-nfs-client
```

防火牆、NIS 和 NFS 見手冊。

> [!NOTE]
> ypbind 可以不用 zypper 不安裝，進入 yast 的 NIS Client 會自動安裝。

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

```
Client :~ # zypper in -y slurm-node
Server :~ # scp /etc/munge/munge.key root@Client:/etc/munge
Server :~ # munge -n | ssh Client umunge
```
> [!NOTE]
> munge 會在安裝 slurm node 時同步安裝

> [!NOTE]
> munge 只要能 ssh 並回傳成功，代表 munge key 是一樣的
