#Basic Pool

在Java中，thread pool都會實作一個介面[Executor](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executor.html)，事實上更明確的說是實作[ExecutorService](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html)這個介面。前者只定義了一個簡單的`execute` method，就跟我前面一個章節的`execute`定義一模一樣，就是在thread pool中執行一個task。

```java
public interface Executor {
    void execute(Runnable command);
}
```

後者繼承了Exectuor介面，定義了更多的method，

```java
public interface ExecutorService extends Executor {
    void shutdown();

    List<Runnable> shutdownNow();

    boolean isShutdown();

    boolean isTerminated();

    boolean awaitTermination(long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> Future<T> submit(Callable<T> task);

    <T> Future<T> submit(Runnable task, T result);

    Future<?> submit(Runnable task);

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks)
        throws InterruptedException;

    <T> List<Future<T>> invokeAll(Collection<? extends Callable<T>> tasks,
                                  long timeout, TimeUnit unit)
        throws InterruptedException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks)
        throws InterruptedException, ExecutionException;

    <T> T invokeAny(Collection<? extends Callable<T>> tasks,
                    long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}

```


Methods | Description
--------|-----------------
submit | 可以呼叫沒有回傳值的task(Runnable)跟有回傳值的task(Callable)，並且會回傳一個[Future](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html)。這個Future的概念容我稍後介紹。
invokeAll| 一次執行多個task，並且取得所有task的future objects 
invokeAny | 一次執行多個task，並且取得第一個完成的task的future object
shutdown<br>shutdownNow<br> | 讓整個thread pool的threads都停止，簡單講就是打烊了。
awaitTermination | 等待所有shutdown後的task都執行完成。可以說是打烊並且所有善後都處理完了。

另外還有一種較特殊的thread pool稱為[ScheduledExecutorService](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ScheduledExecutorService.html)。他除了可以做原本submit task到thread pool以外，還可以讓這個task不會立刻執行。他的定義如下:

```java
public interface ScheduledExecutorService extends ExecutorService {

    public ScheduledFuture<?> schedule(Runnable command,
                                       long delay, TimeUnit unit);

    public <V> ScheduledFuture<V> schedule(Callable<V> callable,
                                           long delay, TimeUnit unit);

    
    public ScheduledFuture<?> scheduleAtFixedRate(Runnable command,
                                                  long initialDelay,
                                                  long period,
                                                  TimeUnit unit);

    
    public ScheduledFuture<?> scheduleWithFixedDelay(Runnable command,
                                                     long initialDelay,
                                                     long delay,
                                                     TimeUnit unit);

}
```

Methods | Description
--------|-----------------
schedule | 讓task可以在一段時間之後才執行。
scheduleAtFixedRate | 除了可以讓task在一段時間之後才執行之外，還可以固定週期執行

在看完thread pool的抽象定義之後，我們來看看有哪些現成的實作可以拿來使用。

## Executors

在Java中，大部分的thread pool都是透過[Executors](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Executors.html)來產生，裡面有非常多的factory method來去產生不同類型的thread pool。

Name | Description
-----|---------------
Executors.newSingleThreadPool() <br> Executors.newSingleThreadScheduledExecutor() | 產生一個只有一個thread的thread pool。
Executors.newFixedThreadPool(int nThreads) | 產生固定thread數量的thread pool
Executors.newCachedThreadPool() | 產生沒有數量上限的thread pool。Thread的數量會根據使用狀況動態的增加或是減少。
Executors.newScheduledThreadPool(int corePoolSize) | 產生固定數量可以做scheduling的thread pool。
Executors.newWorkStealingPool() | 產生work stealing的thread pool，這種類型的thread pool下一章會介紹。


基本上大部分的情況之下，我們已經不會自己產生thread了，透過thread pool的方式可以更有效的管理我們的threads。再想想前面的銀行取票機的例子，也許一個銀行需要一般業務的取票機，另外有一個外匯的取票機，還有可能一個專屬於VIP的取票機。根據不同的業務需求，我們用不同的thread pool去管理。可以避免較為重要的工作，反而被一些比較不重要但是做比較久的task卡住。在我們的multi thread的環境之下，我們也會有類似的情境。

接下來我們介紹一個比較特別的pool，稱之為ForkJoinPool。