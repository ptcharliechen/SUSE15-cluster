# 編譯新 Conquest

## LibXC

網址：[https://libxc.gitlab.io/download/previous/](https://libxc.gitlab.io/download/previous/)

從網路上抓檔案 wget [URL]

![圖片1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/aff21b3b-85d3-460b-9007-4017386e43b0)

```tar -jxvf libxc-x.x.x.tar.bz2``` 解壓縮。

```
autoreconf -i
./configure --prefix==[expected libxc path] FC=ifort
make && make install
```
> [!NOTE]
> - autoreconf 為了產生 configure 檔。
>
> - 使用 intel compiler 2023 和 2024 時， ```./configure --prefix==[expected libxc path]``` 的 Fortran 編譯器是 ifx，結果會報錯，因此要改成 ifort。
