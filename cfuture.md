#Completable Future

Java8在語言層級推出了lambda之後，也伴隨推出了支援lambda的函式庫。其中我把[Stream API](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html), [Optional API](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html), 跟[CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)稱為Java8三神器。這三個都有Functional Language裡[Monad](https://en.wikipedia.org/wiki/Monad_(functional_programming%29)的精神，而CompletableFuture也就是Monadic Future。這邊我們不討論太多Functional Language，讓我們來直接看看CompletableFuture怎麼使用。

