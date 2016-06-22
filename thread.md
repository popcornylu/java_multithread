# Introduction
Thread(線程，或稱執行緒)是一個美妙的東西，它允許你在同一個Address Space(定址空間)撰寫Concurrency(併發)以及Parallelism(並行)的程式。通常跟它會拿來比較的是Process(進程)，它會是不同的定址空間，因此可以做到比較高度的隔離，但是缺點就是比較難在Process之間溝通。因此，Thread我們會稱為比較lightweight的Process。

在Java當中一個JVM就是一個Process，在Process中可以產生非常多的Threads。
Java Thread是Java非常美麗的一個起點，早在Java的最早版本就有Thread的存在，在那個Multi-Threading還沒有很成熟的年代，Java Thread的簡單好用讓大家開始發現寫Multi-Thread不是什麼難事。

# How to use

廢話不多說，直接來看Java Thread怎麼使用

```java
new Thread(() -> {
    System.out.println("hello thread");
}).start();
```

都已經Java8的時代了，我直接用Java8的語法來介紹Java Thread。在上面的程式中產生了一個新的Thread，Thread的Constructor是一個實作`java.lang.Runnable`的物件。當呼叫`start()`此method時，則會啟動這個Thread，並且執行`Runnable#run()`。在Java8中有好用的Lambda，我們直接用Lambda來實作那個Runnalbe。當然你也可以用傳統的產生一個新的Class來實作Runnable的方法。因為這個文件不是要教你怎麼寫Java，我就先假設這邊你已經了解寫Java的最基本常識，如果你到這邊還不懂我在寫什麼，那建議可以先拿一本Java入門書來了解Java最基本的撰寫知識。


# 使用時機

通常有以下幾種情況會用Thread

1. IO相關的task，或稱IO bound task。如果同時需要讀很多個檔案，或是同時要處理很多的Sokcet Connection，用Thread的方法去做Blocking Read/Write可以讓程式不會因為等待IO而導致什麼事情都不能做。
2. 執行很耗運算的task，或稱CPU bound task。當這種task多，我們會想要使用多個CPU Core的能力。單執行緒的程式用到只能用到Single Core的好處，也就是程式再怎麼耗CPU，最多就用到一個CPU。當使用Multi-Thraed的時候，就可以把CPU吃飽吃滿不浪費。
3. 非同步執行。其實不管是IO-bound task或是CPU-bound task，我們應該都還是要跟主程式做溝通，所以通常的概念都是開一個Thread去做IO bound或是CPU bound task，等到他們做完了，我再把結果拿到我的主程式做後續的處理。當然新開Thread不是唯一的方法，我們稍後的章節會入續提到很多非同步執行的策略跟方法。
4. 排程。常見的排程方法有三種。第一種是Delay，例如一秒鐘後執行一個job。第二種是週期性的，例如每一秒鐘執行一個job。第三種是指定時間執行一個job。當然這些應用會包裝在`Timer`或是`ScheduledThreadPoolExecutor`中，但是底層都還是用Thread去完成的。
5. Daemon，或是稱之為Service。有些時候我們需要某個Thread固定去等某些event發生的時候才會去做特別的事情。例如寫Server，我們會希望有一個Thread專門去listen某個port的連線訊息。如果我們寫一個Queue的Consumer，我們也會去有個Thread去聽message來的event。


我們在寫Java程式中，其實我們每天都在跟Thread共舞，即便我們沒有直接產生這些Thread，但是可能我們所用的Framework都已經建構在Multi-thread的環境之上，因此我們更需要瞭解Multi-Thread的課題。下個章節我們來討論Thread之間的Synchroinzation的問題。



