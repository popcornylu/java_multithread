#Completable Future

Java8在語言層級推出了lambda之後，也伴隨推出了支援lambda的函式庫。其中我把[Stream API](https://docs.oracle.com/javase/8/docs/api/java/util/stream/Stream.html), [Optional API](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html), 跟[CompletableFuture](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html)稱為Java8三神器。這三個都有Functional Language裡[Monad](https://en.wikipedia.org/wiki/Monad_(functional_programming%29)的精神，而CompletableFuture也就是Monadic Future。這邊我還是先不要討論太多Functional Language，讓我們來直接看看CompletableFuture怎麼使用。

CompletableFuture是一個Future的實作，我打算以這四個特性來討論

1. Completable
2. Listenable
3. Composible
4. Joinable

## Completable

所謂的Completable就是這個future可以被complete。其實這要先討論Future跟Promise這兩個概念。

1. Future: 是一個未來會完成的一個結果，算是這個結果的容器
2. Promise: 是可以被改變可以被完成的值，通常是非同步執行的結果。

基本上就是一體兩面啦。對於asynchronous invocation，對於caller看到就是future，對於callee就是看到promise。因此CompletableFuture的completable，就是這個future可以被complete。

所以基本上CompletableFuture會被上面這樣使用

1. 在非同步呼叫時，會先產生一個CompletableFuture，並且回傳給caller
2. 這個CompletableFuture會連同async task一起傳到worker thread中。
3. 當執行玩這個async task，則會呼叫的會先產生一個CompletableFuture的`complete()`
4. 此時這個CompletableFuture的`get()`就可以取得結果的值。

其實這跟我們在[Flow Control](flow_control.md)的章節看到的`wait()`/`notify()`極為相似，比較不一樣的就是這不只是流程同步，還帶有回傳值。

