# How to use

廢話不多說，直接來看Java Thread怎麼使用

```java
new Thread(() -> {
    System.out.println("hello thread");
}).start();
```

都已經Java8的時代了，我直接用Java8的語法來介紹Java Thread。在上面的程式中產生了一個新的Thread，Thread的Constructor是一個實作`java.lang.Runnable`的物件。當呼叫`start()`此method時，則會啟動這個Thread，並且執行`Runnable#run()`。在Java8中有好用的Lambda，我們直接用Lambda來實作那個Runnalbe。當然你也可以用傳統的產生一個新的Class來實作`Runnable`。因為這個文件不是要教你怎麼寫Java，我就先假設你已經了解寫Java的基本常識，如果你到這邊還不懂我在寫什麼，那建議可以先拿一本Java入門書來了解Java最基本的編程知識。

# 使用時機

通常有以下幾種情況會用Thread

1. IO相關的task，或稱IO bound task。如果同時需要讀很多個檔案，或是同時要處理很多的Sokcet Connection，用Thread的方法去做Blocking Read/Write可以讓程式不會因為等待IO而導致什麼事情都不能做。
2. 執行很耗運算的task，或稱CPU bound task。當這種task多，我們會想要使用多個CPU Core的能力。單執行緒的程式只能用到Single Core的好處，也就是程式再怎麼耗CPU，最多就用到一個CPU。當使用Multi-Thraed的時候，就可以把CPU吃飽吃滿不浪費。
3. 非同步執行。其實不管是IO-bound task或是CPU-bound task，我們應該都還是要跟主程式做溝通，所以通常的概念都是開一個Thread去做IO bound或是CPU bound task，等到他們做完了，我再把結果拿到我的主程式做後續的處理。當然新開Thread不是唯一的方法，我們稍後的章節會陸續提到很多非同步執行的策略跟方法。
4. 排程。常見的排程方法有三種。第一種是Delay，例如一秒鐘後執行一個task。第二種是週期性的，例如每一秒鐘執行一個task。第三種是指定某個時間執行一個task。當然這些應用會包裝在`Timer`或是`ScheduledThreadPoolExecutor`中，但是底層都還是用Thread去完成的。
5. Daemon，或是稱之為Service。有些時候我們需要某個Thread專門去等某些event發生的時候才會去做對應的事情。例如寫Server程式，我們會希望有一個Thread專門去聽某個port的連線訊息。如果我們寫一個Queue的Consumer，我們也會開個Thread去聽message收到的event。

我們在寫Java程式中，其實我們每天都在跟Thread共舞，即便我們沒有直接產生這些Thread，但是我們所用的Framework可能都已經建構在Multi-thread的環境之上，因此我們更需要瞭解Multi-Thread的課題。下個章節我們來討論Thread之間的Synchronization的問題。

