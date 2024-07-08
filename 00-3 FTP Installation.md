## 架構

```
sda
├─sdb1  128MB  /boot/efi
├─sdb2   32GB  /
├─swap   32GB  [SWAP]
└─ftp   other  /var/ftp
```

## NTFS-3g

硬碟的格式通常是 NTFS-3g，SLES 並沒有配備。

下載點：https://tuxera.com/opensource/ntfs-3g_ntfsprogs-2022.10.3.tgz

```
tar -zxvf ntfs-3g_ntfsprogs-2022.10.3.tgz
cd ntfs-3g_ntfsprogs-2022.10.3/
./configure
make && make install
```

就可以用 NTFS-3g 掛載硬碟 ```mount -t ntfs-3g /dev/sd** /mnt```。

## 取得舊機器的帳號密碼

由於難以考證前人的密碼，嘗試破解也不易，整個搬過來為佳。不過，由於新舊作業系統的 **/etc/group**、**/etc/passwd** 和 **/etc/shadow** 的內容不一樣，完全覆蓋並不妥，以下圖為例， video 的 GID 就不一樣；因此，把需要的部分貼過來就好。

下圖即是 **/etc/passwd** 部分內容，只複製 video 的成員，其他維持原樣。

![擷取](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/8625c060-35f1-470f-9935-618c20d9fee5)

**/etc/passwd** 其他部分、 **/etc/group** 和 **/etc/shadow** 比照辦理。

## 限制直接從外部登入超級使用者

為防止一般使用者取得超級使用者權限，舊的 FTP 拒絕無登入超級使用者權限的帳號以 ssh (port 22)登入，必須使用 FTP 進入 (port 21)。
然而，FTP 沒有資料加密，形同裸奔，因此新的 FTP 改用 SFTP (port 22)登入，這暗示一般人也可以使用終端機進來，
萬一某帳號被駭，可以透過下指令和輸入正確密碼取得超級使用者權限，為阻卻此風險，要限制無權限使用者使用 ```sudo``` 和 ```su```。

進入 **/etc/ssh/sshd_config** ，將 *PermitRootLogin* 改成 no，這樣就不能直接從外部登入 root，而必須以有權限的使用者登入。

## 禁止低權限使用者使用 sudo 和 su

```
groupadd admin
usermod -aG admin [user]
```

基於安全， [user] 是誰見網管筆記。

> [!NOTE]
> 建立一個群組 admin，該群組內的人可以擁有 ```sudo``` 和 ```su``` 的權限。

```visudo```

*Cmnd_Alias SU = /usr/bin/su*

*%ftpusers ALL=(ALL) ALL, !SU*

*%admin ALL=(ALL) SU*

> [!NOTE]
> 成員隸屬於群組 ftpusers，禁用 ```sudo```；成員隸屬於群組 admin，可用 ```sudo```。

上面的方法只能擋住 ```sudo```，卻擋不住 ```su -```。

```
chown root:admin /usr/bin/su
chmod 750 /usr/bin/su
chmod u+s /usr/bin/su
```

> [!NOTE]
> 關閉群組外成員的執行權限，並允許原本非文件所有者擁有所有者的權限。
> 
> 網路所提供的方法執行 ```su -``` 時，即使沒有超級使用者權限的人也可以以正確的密碼登入，故直接封殺群組外的人使用 */usr/bin/su*。
