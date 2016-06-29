#ForkJoin Pool

在java7的時候，推出了一種很不一樣的thread pool，他實作了[working stealing scheduling](https://en.wikipedia.org/wiki/Work_stealing)。這是什麼概念呢? 試想有一個大的專案，如果一個人無法完成的話，那我們很直覺的會把大的專案，切成很多中型的task，再把他分成幾個小的task，讓整個團隊可以協力完成。這些小的task完成之後，中型的task再把它整合起來，最終完成整合後就完成了大的專案。

咦，這不就是[recursion](https://en.wikipedia.org/wiki/Recursion)或是[divide & conquer](https://en.wikipedia.org/wiki/Divide_and_conquer_algorithms)嗎? 沒錯，很多的演算法都是類似這種divide and conquer的形式。這種算法的特性是，要等到所有的subtask都完成之後，才可以算出中間的結果，再來可以算出最後完整的結果。而ForkJoin Pool就是為了解決這種問題平行化的solution。

## Work Stealing

相較於一般的Thread Pool，大家共用一個queue，ForkJoin Pool是**每個thread有一個自己的queue**，而且是雙向的queue(deque)。當一個task進來，他會把一部分的工作fork(切)出來先放到queue的最後面，另外一部分開始做。當然可能做進去後，發現task還是太大，就會繼續切更小，並再放到queue的最後方。如此一邊切一邊往下執行，直到task夠小可以直接運算為止。

當一個小工作完成之後，他會從最後端的task拿出來(其實這邊比較像stack的行為)，繼續往下做。當然recursion的程式除了divide以外，還有merge results的動作，這邊稱之為join。join會取得每個subtask的結果，最終成為目前task的結果回傳回去。

我們先來比較費式數列recursion版本跟forkjoin版本的差異

*recursion*
```java
public int fib(int n) {
    if (n <= 1)
        return n;
    return fib(n-1) + fib(n-2);
}
```
*forkjoin*
```java
public Integer compute() {        
    if (n <= 1)
        return n;
    Fibonacci f1 = new Fibonacci(n - 1);
    f1.fork();
    Fibonacci f2 = new Fibonacci(n - 2);
    return f2.compute() + f1.join();
}
```

那其他thread呢? 當其他thread身上的queue是空的時候，他會去別的thread的queue，從queue的前頭steal一個task到自己的queue當中。之後的行為就跟前面一模一樣。這有沒有很像一個團隊做專案的時候的行為? 所以以後就可以把thread pool類比為取票機，forkjoin pool就很像做專案一樣。

## How to use

前面講的是概念，現在來講實際怎麼實作。在Java中，可以用一個現有的common forkjoin pool，也就是`ForkJoinPool.commonPool()`。而我們的recursion task要繼承於[RecursiveTask](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/RecursiveTask.html)或是[RecursiveAction](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/RecursiveAction.html)。前者是有回傳值，後者沒有。如果完整的code會長這樣。之後實作`compute()`裡面執行divide and conquer的邏輯即可。下面是上面費式數列的完整代碼。

```java
public class Fibonacci extends RecursiveTask<Integer> {
    final int n;
    Fibonacci(int n) { this.n = n; }

    public Integer compute() {
        if (n <= 1)
            return n;
        Fibonacci f1 = new Fibonacci(n - 1);
        f1.fork();
        Fibonacci f2 = new Fibonacci(n - 2);
        return f2.compute() + f1.join();
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        Fibonacci fibonacci = new Fibonacci(10);
        ForkJoinPool.commonPool().execute(fibonacci);
        System.out.println(fibonacci.get());
    }
}
```

#跟傳統Thread Pool比較

其實這是兩種不同的執行策略。分別是producer consumer跟recursion。前者適合每個task是獨立的，他把不管大事小事都可以平均分攤在每個thread做；後者是透過divide and conquer的演算法，用work stealing的方式去執行。所以主要還是要看你的task是哪一種類型比較多。

而ForkJoinPool有一個很大的好處是減少thread因為blocking造成context switching。不管是fork, compute, join都幾乎不會blocking(只有join少數情況會要等待結果)。這可以讓thread一直保持running的狀態，一直到時間到了被context switch，而不是自己卡住了馬上context switch。

但ForkJoinPool對於不可分割的task，並且處理時間差異很大的情境比較不適合，畢竟每個thread都有一個queue。就很像在大賣場排隊結帳，只要運氣不好排到一個前面卡比較久的task就要等比較久。但是別排又沒有閒到可以把你steal走，那就沒有辦法做到先到先處理的特性了。

#Java8 Parallel Stream
Java8的[Stream API](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html)的parallel stream事實上也是用ForkJoinPool，他透過[Spliterator](https://docs.oracle.com/javase/8/docs/api/java/util/Spliterator.html)來定義怎麼去split(或稱fork)你的input data。若執行結果需要collect，就會join回最後的result。
