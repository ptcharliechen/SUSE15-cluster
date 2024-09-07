# Python Script

## 編譯 Python

作業系統都會附 python，用 ```which python``` (Python 2) 和 ```which python3``` (Python 3)就可以確認直譯器的位置。
目前僅 vfreq.py 仍使用 Python 2，不過新的作業系統沒有，若國高的 module 沒有支援，就到 Python 官網下載源碼自編。

編者以 Python 2.7.18 為例，

```
wget https://www.python.org/ftp/python/2.7.18/Python-2.7.18.tgz
```

取得源碼，其他的版本請見 Python 的 FTP： https://www.python.org/ftp/python

```
tar -zxvf Python-2.7.18.tgz
cd Python-2.7.18
./configure --prefix=/destination/of/python/path
make -j16 && make install
```

```export``` 連結新編 python，再用 ```which``` 確認是否可以使用。

## Python 環境

Python 3 有許多強大的套件，如： NumPy、Pandas、Matplotlib、SciPy，預設的 Python 3 並沒有支援；另外，研究室自編的 script 有用 Cython 加速運算，因此必須引入相關套件才能運轉。
創造新的環境通常不會用 root 權限在原始的 python 直譯器引入所須的套件 (國高也沒提供如此權限)，通常會建立一個虛擬環境 (venv)後引入。

創建虛擬環境，以名稱 public 為例：

```
python3 -m venv public
```

在原地建立一個名字為 public 的資料夾，裡面就是該虛擬環境。

```source public/bin/activate``` 活化環境，再用 ```which``` 確認是否可以使用。

> [!NOTE]
> ```
> source deactivate
> ```
> 關閉該環境。

引入自編 script 所須的套件。

```
pip3 install pandas matplotlib seaborn scipy Cython ase
```

> [!NOTE]
> 如果出現 SSL 相關的 WARNING，如
> ```
> WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL becas not available.")': /simple/pip/
> ```
> 先安裝 ```libopenssl-devel```。


script/ 資料夾裡所有 .py 檔的第一行都有 ```#!/home/xxxx/...``` ， "#!" 被稱為 shebang，用於指定編譯器或直譯器的路徑。不要一個個進去修改直譯器，善用 sed 一次修改。

> [!NOTE]
> 把 .py 為後綴的檔案當中的 hello 改為 world：```sed -i 's/hello/world/g' *.py```
> - -i 是編輯文檔內字串。
> - s (substitution)是取代。
> - g (global)是凡 hello 全改成 world，如果沒有 g 的話，只換第一個 hello。
> - \* 是任意字元出現 0 次或以上。也就是只要 .py 作後綴的文檔就改，包含檔名就叫 .py 的檔案。
> 
> 若要把 /home/user/python/bin/python3 改成 /home/user1/python/bin/python3，不是```sed -i 's//home/user/python/bin/python3//home/user1/python/bin/python3/g' *.py```，
> 無法辨識哪個是路徑的正斜線，哪個是分隔的正斜線，必須外加[跳脫字元](https://mimigd.com/python/140/) (escape character)，讓路徑的正斜線得以被辨識，所以要改成
> ```sed -i 's/\/home\/user\/python\/bin\/python3/\/home\/user1\/python\/bin\/python3/g' *.py```

除了 shebang 外， sys.path.append 裡的文字也要修改。

# 自編 script

script/kit 裡面的 accelerate.pyx 是用來加速部分計算的 script，以 Cython 寫成，以下是將 accelerate.pyx 利用 Cython 編譯器翻譯成高效 C++ 程式碼。

打開 script/kit/setup.py，更換 shebang 的 Python 3 直譯器，執行
```
python3 setup.py build_ext --inplace
```

到新生成的 build 資料夾裡，用 ```ln -s``` 將 accelerate.o 和 accelerate.xxxx.so 連結到 script/kit。

> [!NOTE]
>
> setup.py 所適用的 Cython 的版本為 3.0.9，可能會因 Cython 的版本變動而有所修改。
