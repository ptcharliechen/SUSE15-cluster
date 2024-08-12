## 函式庫 (Library)
編譯時會把許多函式庫寫進參數檔 (VASP是 *makefile.include* ，CONQUEST是 *system.make* )，使編譯時可以鏈接，常見的函式庫有 *libgfortran.so* 、 *libblas.so* 、 *liblapack.so* 等， *libxxxx.so* 是固定命名方式。
```ldd /path/to/execution/file``` 可以看到該執行檔鏈接的函式庫，下圖以VASP641為例。

![圖片1](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/b4d70903-ea37-4884-ae79-96600ff80830)

*libxxxx.so*是固定命名方式，因此去頭去尾，```-l```後面接xxxx作為標記方式，例如：```-lblas```意旨要鏈接 *libblas.so* 函式庫。

函式庫大多放在 */lib* 、 */lib64* 、 */usr/lib* 、 */usr/lib64* ，這些都是預設路徑，因此參數檔毋須詳加註明。如果是特別的函式庫路徑，會以```-L/path/of/library```表示，右為 CONQUEST 中scalapack的函式庫路徑：```-L/work1/***/Conquest/source/src/scalapack -lscalapack```，意即：到 */work1/\*\*\*/Conquest/source/src/scalapack* 找 *libscalapack.so* 函式庫。只寫```-llapack```就是到預設路徑找 *liblapack.so* 函式庫。

- 小練習
1. 使用 Intel compiler 時，會去抓 mkl 資料庫的路徑，存在 *MKLROOT* 變數中，因此```-L$(MKLROOT)/lib/intel64 -lmkl_scalapack_lp64 -lmkl_blacs_intelmpi_lp64```是什麼意思？順著路徑看看這些函式庫是否存在？
2. 參數檔常會看到```-lfftw3```，它在週期性計算中扮演相當重要的角色，網路上找得到下載檔，不過需要編譯，編譯過程中除了會報錯外，鏈接時也可能出問題，因此應先在計算節點尋找是否已經配備該函式庫。用```find```指令到研究室任意計算節點上 (伺服器沒有這些函式庫，別找錯)找找看是否存在該函式庫？

進行```source```時，會把變數PATH後面添加執行時會抓取的檔案的路徑， library 的路徑則是放在變數*LD_LIBRARY_PATH* ( *LIBRARY_PATH* 是在編譯時使用； *LD_LIBRARY_PATH* 是在執行時使用)。slurm submission script在 source Intel compiler時會添加 mkl 函式庫的路徑， VASP641 的 dftd4 函式庫路徑記載於 *vasp_set.sh* ，供執行時抓取。

![圖片2](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/f0778f5c-1827-4b46-bac1-8e3b5227a60d)

## pkg-config
pkg-config可用來檢索系統中函式庫的訊息。

![圖片3](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/dcd27fed-bca0-4989-94a4-6c771a034e7b)

通常會放在lib裡面，使用 ```export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/path/to/pkgconfig```
告知要搜尋該路徑的pkg-config
以上圖為例，輸入要讀取的pkg-config檔
```pkg-config --cflags dftd4```
```pkg-config --libs dftd4```

![圖片6](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/d081a825-d46d-4fb9-8d52-8df933454c92)

```--cflags```是給予標頭檔 (.c file)的鏈接資訊；```--libs```是給予函式庫的鏈接資訊。
將上述資訊貼到參數檔中 (以 dftd4 為例)

![圖片5](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/b2f89bd1-b326-479e-84ce-ed415cfc24d7)

## configure
*configure* 檔會生成 *Makefile*  ，內含使用的編譯器，因此使用前要先引入編譯器。
輸入```./configure```後會產生 *Makefile* ，再執行```make```就可以編譯軟體或函式庫。

*configure* 預設以超級使用者 (superuser)身分編譯，編譯出來的函式庫會放在 */usr/local* 等超級使用者才能編輯的地方，因此使用臺三編譯時，要修改生成檔案的位置： ```./configure --prefix=/path/to/lib``` 即可修改生成的 *Makefile* 的參數。

CC: C語言編譯器；FC: Fortran編譯器；CXX: C++ 編譯器

透過```CC=[C compiler]```更換使用的C語言編譯器，更換其他編譯器同理，例如：```./configure FC=ifort```。
舊版的 intel compiler 是使用 icc 和 ifort， 2024 intel compiler 的已經更換成 icx 和 ifx， *configure* 必要時要更換為```FC=ifort```。

> [!NOTE]
> icc 在2024 compiler已經無效，必須用 icx。

## patch
它是用來內嵌程式碼的工具，不過版本不同時可能會不起作用，就要手動加入程式碼，故了解 *patch* 裡面的意義，能知道如何修改 *patch* 或是在哪裡手工插入程式。下面是一個 *patch* 的範例：

```
--- solvation.F_org	2019-02-21 15:56:46.000000000 +0100
+++ solvation.F	2020-09-30 17:19:41.821342000 +0200
@@ -2203,7 +2203,7 @@
 !test
    CALL MY_D_PROD(Ecorr3, SP_CHTOT(1,1), Vcorr, SP_GRIDC)
 !   CALLMPI( M_sum_d(SP_GRIDC%COMM,Ecorr3,1))
-   CALLMPI( M_sum_s(SP_GRIDC%COMM,1,Ecorr3,0,0,0))
+   CALLMPI( M_sum_1(SP_GRIDC%COMM,Ecorr3))
 
 !-------------------------------------------------------------
 !Ecorr4 = Vdiel*n, 
```
***+++/---***
- 補丁頭是分別由---/+++開頭的兩行，用來表示要打補丁的文件。---開頭表示舊文件，+++開頭表示新文件。補丁會保留舊程式，在新程式裡打補丁，上例的舊程式是 *solvation.F_org* ，新程式是 *solvation.F* 。補丁頭是分別由---/+++開頭的兩行，用來表示要打補丁的文件。---開頭表示舊文件，+++開頭表示新文件。

***@@***
- 它通常由一部分不用修改的東西開始和結束，只是用來標示要修改的位置。上例即包含
```
 !test
    CALL MY_D_PROD(Ecorr3, SP_CHTOT(1,1), Vcorr, SP_GRIDC)
 !   CALLMPI( M_sum_d(SP_GRIDC%COMM,Ecorr3,1))
```
以及
```
 
 !-------------------------------------------------------------
 !Ecorr4 = Vdiel*n, 
```
- 他們通常以@@開始，結束於另一個塊的開始或者一個新的補丁頭。'-'後的數字代表該塊在舊文件的行號,修改前該塊總行數；'+'後的數字代表該塊在新文件的行號,修改後該塊總行數。上例要拿掉的```   CALLMPI( M_sum_s(SP_GRIDC%COMM,1,Ecorr3,0,0,0))```，添加```   CALLMPI( M_sum_1(SP_GRIDC%COMM,Ecorr3))```。
- patch 的上下行與待補丁的程式不一致就不會補上去，行數不一致也不會寫上去，最常發生在不同版本或已經補過丁導致行數不一致，所以補完後***要檢查是不是都有寫上去***，沒有的話手動貼上去。

參考網頁：[補丁(patch)的製作與應用](http://linux-wiki.cn/wiki/zh-tw/%E8%A1%A5%E4%B8%81(patch)%E7%9A%84%E5%88%B6%E4%BD%9C%E4%B8%8E%E5%BA%94%E7%94%A8)

## 平行運算

### 進程 (Process)

進程是指被執行且載入記憶體的程式。是作業系統分配資源的最小單位，每個進程都可取得一個獨立 ID (PID)。

> [!NOTE]
> 程式是沒有執行是寫好尚未執行的 code，進程則是執行中的程式。

### 執行緒 (Thread，大陸翻線程)

將進程比喻為產品，而執行續就是產線的工人，一個進程裡至少會有一個執行緒，亦可以多個執行緒操作一個進程，就像聘大量工人應付大量訂單。執行緒是作業系統運算 (排程)的最小單位，實際執行任務的並不是進程，而是進程中的執行緒。

### Multi-processing
建立許多工廠 (CPU)，每個工廠中會分配一名員工 (thread)執行工作，優勢在同一時間內完成較多事。

### Multi-threading
將許多員工聚集到同一個工廠內，優勢是有機會讓相同的工作在比較短的時間內完成。

> [!NOTE]
> 一個 CPU 可能有不只一個執行緒。

> [!NOTE]
> 不同執行緒是可以共享資源，但兩個執行緒若同時存取或改變全域變數，可能會發生同步 (Synchronization) 問題。若執行緒之間互搶資源，則可能產生死結 (Deadlock)，因此使用多線程時必須特別注意。
> 
> Python 的作法是全域直譯器鎖 GIL (Global Interpretor Lock)，一個進程進行運算時只容許一個執行緒運行，其他都被「鎖住」，不過進入 IO (input/output)時會釋放權限給其他執行緒，因此適合用在高 IO 型 (IO-bound)進程，高運算型 (CPU-bound)進程用 multi-processing。C 語言沒有研究，不予說明。

整理自 [程序(進程)、執行緒(線程)、協程，傻傻分得清楚！](https://oldmo860617.medium.com/%E9%80%B2%E7%A8%8B-%E7%B7%9A%E7%A8%8B-%E5%8D%94%E7%A8%8B-%E5%82%BB%E5%82%BB%E5%88%86%E5%BE%97%E6%B8%85%E6%A5%9A-a09b95bd68dd)

### MPI (Message Passing Interface)
MPI 適用於分散式記憶體的計算環境，如超級電腦或計算集群，通過訊息傳遞 (Message Passing)讓多個獨立的計算節點協同工作。與 OpenMP 不同之處在：沒有共用記憶體區域，所以每個進程各自使用一個記憶體區域，因此有利於跨節點計算。

### OpenMP (Open Multi-Processing，簡稱OMP)
OpenMP 用於多核心共享記憶體的計算環境，使其能夠在多核處理器上並行運行。因為用同一個記憶體區域，所以不能跨節點。

> [!Note]
> OMP 屬 Multi-threading，會叫 multi-processing 是強調多核在同一臺機器上運行共享記憶體模式，是早期名詞尚未被清楚定義就出現的術語。

整理自 ChatGPT 的回答。

### OpenMP + MPI Hybrid

兼具 OMP 共享記憶體的和利於跨節點計算的 MPI 的特色。VASP 的 *NPAR* 即是分割多個 partition，每個 partition 會有被分配多個執行緒，每個進程使用一個 partition 計算。VASP 641 的 arch 裡面有 "omp" 做為後綴也是 hybrid 的實作。

### 編譯時注意事項

想要使用 OMP 編譯時，使用 Intel 編譯器編譯時，會在編譯器後面加上 *-qopenmp* ，而使用 OpenMPI 編譯時，則加上 *-fopenmp*。VASP 只要選擇 omp 後綴的 architecture 就可以用了，但 Conquest 要自己加。用 MPI 編譯不用加任何標籤。

> [!NOTE]
> OpenMPI 是 GNU 的 MPI 編譯器，而 OpenMP 是共享記憶體平行化技術。

### 執行時注意事項

> [!CAUTION]
> OMP 和 MPI 的執行檔不互通，用反會報錯。

MPI 在 slurm script 中的寫法：

```
mpiexec.hydra -np [total process number] -ppn [process number per node] vasp_std >& ${JOB}.log
```

*-ppn* 和節點數相乘即 *-np*。

以下為 OpenMP 在 slurm script 中的寫法：

```
mpiexec.hydra -np [total process number] -ppn [process number per node] -genv I_MPI_PIN_DOMAIN=omp -genv I_MPI_PIN=yes -genv OMP_NUM_THREADS=[thread number per prcoess] -genv OMP_PLACES=cores -genv OMP_PROC_BIND=close -genv OMP_STACKSIZE=512m vasp_omp >& ${JOB}.log
```

[thread number per prcoess] $\times$ [process number per node] 等於一個節點的核心數。

[thread number per prcoess] 和 [process number per node] 我通常取開根號最接近的因數相乘， [process number per node] 會略大；例如臺三單節點 56 核心，[process number per node] = 8 ， [thread number per prcoess] = 7。

> [!NOTE]
> VASP 的 OUTCAR 會顯示 cpu time 和 real time ，如下：
> 
> *LOOP:  cpu time     18.6441: real time      2.7045*
>
> 若 real time $\approx$ cpu time 暗指沒有平行化，VASP MPI 計算屬之，因為每個進程各自為政；real time < cpu time 是有平行化，VASP OMP 屬之，上例就是使用 OMP 計算；real time > cpu time 通常是 IO-bound ， CPU 運算時間比總花費時間小很多。

### 補充： Parallel 和 Concurrency

![image](https://github.com/ptcharliechen/SUSE15-cluster/assets/128341777/ceddb069-dcf0-4dca-a6c3-2c91efbeec7a)

圖源：[https://techdifferences.com/difference-between-concurrency-and-parallelism.html](https://techdifferences.com/difference-between-concurrency-and-parallelism.html)

Parallel 是利用多個 CPU 達到同時並行處理任務的需求，程式間關聯低適合用此實作。Concurrent 是許多任務爭搶一個 CPU 的資源，因此一個時間點只會有一個任務正在執行，只是因為切換非常快，使用者通常不會感覺到任務實際上一直在切換，進程間需要彼此的變數時適合用此方法。

> [!NOTE]
> 國高的 rdf.py 即是以 Parallel 實作。

整理自 [程序(進程)、執行緒(線程)、協程，傻傻分得清楚！](https://oldmo860617.medium.com/%E9%80%B2%E7%A8%8B-%E7%B7%9A%E7%A8%8B-%E5%8D%94%E7%A8%8B-%E5%82%BB%E5%82%BB%E5%88%86%E5%BE%97%E6%B8%85%E6%A5%9A-a09b95bd68dd)

### 補充： Asynchronous 與 Synchronous

Asynchronous: 一個櫃檯 (進程)的把工作丟給辦事員後領個號碼牌去做別的櫃檯丟其他工作，叫號 (回傳)時再收取結果並丟下一份工作。常見的程式語言是 JavaScript 和 Node.js 等常用於編寫網站的語言，主因是丟個 request 很快，但收到伺服器回復要一段時間，手上有多個工作時會等很久；缺點是有時候變數還尚未獲得回傳就需要用到該變數，就用非預期的值進行運算產生不如意的值。

Synchronous: 一個櫃檯賦予一個工作並獲得會傳後才去下一個櫃檯辦事。常見的程式語言是不涉及 request 和 response 的程式語言。

整理自 [非同步(Asynchronous)與同步(Synchronous)的差異](https://medium.com/@hyWang/%E9%9D%9E%E5%90%8C%E6%AD%A5-asynchronous-%E8%88%87%E5%90%8C%E6%AD%A5-synchronous-%E7%9A%84%E5%B7%AE%E7%95%B0-c7f99b9a298a)
