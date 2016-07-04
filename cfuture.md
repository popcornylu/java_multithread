#CompletableFuture

Java8在語言層級推出了lambda之後，也伴隨推出了支援lambda的函式庫。其中我把[Stream API](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html), [Optional API](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html), 跟[CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)稱為Java8三神器。這三個都有Functional Language裡[Monad](https://en.wikipedia.org/wiki/Monad_(functional_programming%29)的精神，而CompletableFuture也就是Monadic Future。這邊我還是先不要討論太多Functional Language，讓我們來直接看看CompletableFuture怎麼使用。

先來看一個最簡單的例子
```java
CompletableFuture<Void> future =
CompletableFuture.runAsync(() -> {
    try {
        Thread.sleep(1000);
        System.out.println("hello");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

future.get();
System.out.println("world");
```
以上的代碼會印出
```
hello
world
```

透過lambda的特性，我們同樣可以把非同步呼叫弄的跟開thread一樣簡單。在CompletableFuture定義了幾個static methods，幫助我們快速的非同步執行。

Method | Description
-------|-------------------
[runAsync(Runnable runnable)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#runAsync-java.lang.Runnable-) | 非同步的執行一個沒有回傳值的task，並且在預設的thread pool中執行。預設為 [ForkJoinPool.commonPool()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ForkJoinPool.html#commonPool--)
[runAsync(Runnable runnable, Executor executor)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#runAsync-java.lang.Runnable-java.util.concurrent.Executor-) | 非同步的執行一個沒有回傳值的task，並且在指定的thread pool之中執行。
[supplyAsync(Supplier<U> supplier)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#supplyAsync-java.util.function.Supplier-) | 非同步的執行一個有回傳值的task，並且在預設的thread pool之中執行。
[supplyAsync(Supplier<U> supplier, Executor executor)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#supplyAsync-java.util.function.Supplier-java.util.concurrent.Executor-) | 非同步的執行一個有回傳值的task，並且在指定的thread pool之中執行。

咦? 那所謂的Completable是什麼意思? 那前一章所介紹的Future有什麼不一樣? 事實上CompletableFuture是一個Future的實作，至於Completable，我打算以這四個特性來討論

1. Completable
2. Listenable
3. Composible
4. Combinable

## Completable

所謂的Completable就是這個future可以被complete。其實這要先討論Future跟Promise這兩個概念。

1. Future: 是一個未來會完成的一個結果，算是這個結果的容器。所以前一章節我們整章都在介紹這個。
2. Promise: 是可以被改變可以被完成的值，通常是非同步執行的結果。而CompletableFuture就是一個promise，可以**被完成**。

基本上就是一體兩面啦。對於asynchronous invocation，對於caller看到就是future，對於callee就是看到promise。而CompletableFuture就同時扮演了Future跟Promise兩種角色。

所以基本上CompletableFuture會被上面這樣使用

1. 在非同步呼叫時，會先產生一個CompletableFuture，並且回傳給caller
2. 這個CompletableFuture會連同async task一起傳到worker thread中。
3. 當執行玩這個async task，則會呼叫的會先產生一個CompletableFuture的`complete()`
4. 此時這個CompletableFuture的`get()`就可以取得結果的值。

其實這跟我們在[Flow Control](flow_control.md)的章節看到的`wait()`/`notify()`極為相似，比較不一樣的就是這不只是流程同步，還帶有回傳值。除了complete以外，當執行錯誤的時候，也可以呼叫`completeExceptionally()`。

在completable這個特性裡，我們把屬於caller/consumer用的Future介面，以及callee/provider用的Completable放在一起，我們來檢視一下有哪些跟Completable相關


Method |  Description
-------|----------------------
[complete(T t)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#complete-T-) | 完成非同步執行，並回傳結果
[completeExceptionally(Throwable ex)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#completeExceptionally-java.lang.Throwable-) | 非同步執行不正常的結束

有了以上的概念，我們很快的可以很快地寫出`CompletableFuture.runAsync()`可能的邏輯

```java
public static CompletableFuture<Void> runAsync(Runnable runnable) {
    CompletableFuture<Void> future = new CompletableFuture<>();
    ForkJoinPool.commonPool().execute(() -> {
        try {
            runnable.run();
            future.complete(null);
        } catch (Throwable throwable) {
            future.completeExceptionally(throwable);
        }
    });

    return future;
}
```

在Google的[Guava library](https://github.com/google/guava)中也可以看到completable的蹤影，那就是[SettableFuture](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/SettableFuture.html)。

## Listenable

對於asynchronous invocation的caller來講，`Future`只提供了一個pulling result的方法，更多時候我們想要的是**好了叫我**這種語意。因此*Listenable*的特性，就是我們可以註冊一個callback，讓我可以listen執行完成的event。

在CompletableFuture主要是透過[whenComplete()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#whenComplete-java.util.function.BiConsumer-)跟[handle()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#handle-java.util.function.BiFunction-)這兩個method。

Method |  Description
-------|----------------------
[whenComplete()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#whenComplete-java.util.function.BiConsumer-) | 當完成時，把result或exception帶到callback function中。
[handle()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#handle-java.util.function.BiFunction-) | 當完成時，把result或exception帶到callback function中，並且回傳最後的結果。

我再把最上面的例子改寫成用listener的方式
```java
CompletableFuture.runAsync(() -> {
    try {
        Thread.sleep(1000);
        System.out.println("hello");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
}).whenComplete((result, throwable) -> {
    System.out.println("world");
});
```

這兩個method以及包含後面會提到的method都有三種變形，分別是

- xxxx(function): function會用前個執行的thread去呼叫。
- xxxxAsync(function): function會用非同步的方式呼叫，並用預設的thread pool。
- xxxxAsync(function, executor): function會用非同步的方式呼叫，並用指定的thread pool。

由於基本邏輯相似，之後就不再重述。

同樣在Guava library中也可以看到listenable的蹤影，那就是[ListenableFuture](http://google.github.io/guava/releases/snapshot/api/docs/com/google/common/util/concurrent/ListenableFuture.html)。

## Composible

有了Listenable的特性之後，我們就可以做到當完成時，在做下一件事情。如果接下來又是一個非同步的工作，那就可能會串成非常多層，我們稱之為callback hell。下面是個例子


```java
public static void sleep(long time) {
    try {
        System.out.printf("sleep for %d milli\n", time);
        Thread.sleep(time);
        System.out.printf("wake up\n");
    } catch (InterruptedException e) {
    }
}

public static void main(String[] args) throws InterruptedException {
    CompletableFuture<Void> future = 
    CompletableFuture
    .runAsync(() -> sleep(1000))
    .whenComplete((result, throwable) -> {
        if (throwable != null) {
            return;
        }

        CompletableFuture
        .runAsync(() -> sleep(1000))
        .whenComplete((result2, throwable2) -> {
            if (throwable2 != null) {
                return;
            }

            CompletableFuture
            .runAsync(() -> sleep(1000))
            .whenComplete((result3, throwable3) -> {
                if (throwable2 != null) {
                    return;
                }

                System.out.println("Done");
            });
        });
    });
```

這個程式碼這樣三層可能已經受不了了，如果更多層應該會有噁心的感覺。這還不打緊，如果再加上錯誤處理，那可能更是暈頭轉向。

對於這種一連串的invocation，如果可以把這些async function組起來，變成一個單一future，可能會舒服許多。先來看最後的結果，我們再來討論細節。

```java
CompletableFuture
.runAsync(() -> sleep(1000))
.thenRunAsync(() -> sleep(1000))
.thenRunAsync(() -> sleep(1000))
.whenComplete((r, ex) -> System.out.println("done"));
```

有沒有覺得清爽許多?這就是Composible的魅力。

在CompletableFuture中，它提供了非常多的compose methods來幫助我們組合各種sync methods變成async methods。我來列舉一下

Method | Trasnformer | To Type
-------|-----------|-----------
[thenRun()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#thenRun-java.lang.Runnable-) | ```Runnable``` | ```CompletableFuture<Void>```
[thenAccept()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#thenAccept-java.util.function.Consumer-) | ```Consumer<T>``` | ```CompletableFuture<Void>```
[thenApply()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#thenApply-java.util.function.Function-) | ```Function<T, U>``` |  ```CompletableFuture<U>```
[thenCompose()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#thenCompose-java.util.function.Function-) | ```Function<T, CompletableFuture<U>>``` |  ```CompletableFuture<U>```

型態的部分我有稍微調整一下，讓它比較容易讀。但是我們都可以看到他們都有一個特性，就是把原本某個CompletableFuture的type parameter，經過一個transformer後，轉成另外一個Type的CompletableFuture，這就是Monad中的**map**。而最後一個因為他的回傳值本來就是CompletableFuture，這種轉換我們稱之為**flatmap**。其實同樣的概念在Optional API跟Stream API都找得到，有興趣可以去尋寶一下。

這些method也都有`xxx()`, `xxxAsync(func)`, `xxxAsync(func, executor)`三個版本，就如前面所述。

經過這樣的轉換過程，我們把很多的future合併成單一的future。這些轉換我們沒有看到任何的exception處理，因為在任何一個階段出現exception，對於整個包起來的future就是exception。所以我們就是希望把每一個小的async invocation **compose**成一個大的async invocation。

同樣在guava library中，我們可以看到composible的蹤影，他是放在[Futures](https://google.github.io/guava/releases/19.0/api/docs/com/google/common/util/concurrent/Futures.html)下面的`transformXXX()`相關的methods。

## Combinable

最後，async的流程有些時候不會是單一條路的，有時候更像是[DAG](https://en.wikipedia.org/wiki/Directed_acyclic_graph)(Directed Acyclic Graph)。例如做一個爬蟲程式(Crawler)，我們排一個文章的時候，可能會抓到很多個外部鏈結，這時候就會繼續展開更多非同步的task。等到到了某個停止條件，我們就要等所有爬蟲的task完成，最終等於執行完這個大的async task。

這時候我們會希望把多個future完成時當作一個future的complete，這就是combinable的概念。跟composible的概念不同的是，composible是一個串一個，比較像是串連的感覺；相對的combinable，就比較像是並聯。

來看看CompletableFuture針對這種應用有哪些method，假設原始形態`CompletableFuture<T>`


Method | With | Transformer | Return Type
-------|-----------|-----------
[runAfterBoth()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#runAfterBoth-java.util.concurrent.CompletionStage-java.lang.Runnable-) | `CompletableFuture<?>` | `Runnable` | `CompletableFuture<Void>`
[runAfterEither()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#runAfterEither-java.util.concurrent.CompletionStage-java.lang.Runnable-) | `CompletableFuture<?>` | `Runnable` | `CompletableFuture<Void>`
[thenAcceptBoth()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#thenAcceptBoth-java.util.concurrent.CompletionStage-java.util.function.BiConsumer-) | `CompletableFuture<U>` | `BiConusmer<T,U>` | `CompletableFuture<Void>`
[acceptEither()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#acceptEither-java.util.concurrent.CompletionStage-java.util.function.Consumer-) | `CompletableFuture<T>` | `Conusmer<T>` | `CompletableFuture<Void>`
[applyToEither()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#applyToEither-java.util.concurrent.CompletionStage-java.util.function.Function-) | `CompletableFuture<T>` | `Function<T,U>` | `CompletableFuture<U>`
[thenCombine()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#thenCombine-java.util.concurrent.CompletionStage-java.util.function.BiFunction-) | `CompletableFuture<U>` | `BiFunction<T,U,V>` | `CompletableFuture<V>`

跟Composible那邊的method不一樣的是多了一個*with*，代表的是combine的對象。這些method都有可以把兩個future **combine**成一個future的特色。而**both**跟**either**，代表的是兩個都完成才算完成，還是其中一個完成則算完成。

除了兩兩combine的這些method以外，CompletableFuture還有提供兩個static methods來做combine多個futures。

Method | Description
-------|-------------
[allOf(...)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#allOf-java.util.concurrent.CompletableFuture...-) | 回傳一個future，其中所有的future都完成此future才算完成。 
[anyOf(...)](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#anyOf-java.util.concurrent.CompletableFuture...-) | 回傳一個future，其中任何一個future完成則此future就算完成。


## 總結

CompletableFuture跟lambda的組合，在java8中帶來了非同步的生力軍。Lambda讓之前的annoymous inner class來實作async task會變成簡潔非常多，而Completable future又多了composible跟combinable，讓複雜的非同步流程變得非常的簡潔。

再來就如前面講的，大部分的method都有**async**，以及**async with executor**的版本。所以我們可以很明確指定到底我的task是擺在哪一個thread pool跑。對於UI程式，常常有一個pattern就是先async到worker thread pool去執行，處理完再到UI thread去update UI並且呈現，這個流程在新的CompletableFuture下變得更為簡潔容易。

