#Conclusions

很快的就講到尾聲(其實我寫了好幾個晚上 XD)，讓我們來回顧一下講了什麼

在Java Thread章節中，我們講了thread最基本的概念，並且知道怎麼開thread，怎麼定義thread要做的事情。

有了thread之後，那開始要解決synchronization的問題。其實這個章節講的是java最底層的同步機制，說真的除了`synchronized` keyword，大部分幾乎不會直接用到。但是透過resource sharing, flow control, message passing三種概念，來幫助勾勒出同步最常見的幾種情境。

接下來講到thread pool，我們在thread pool中提到了single queue/multi consumer這種傳統的thread pool，我把它類比成**取票機**。接下來還介紹了queue-per-thread並且透過work stealing方式的`ForkJoinPool`，這種pool對於divide and conquer分而治之的方式特別適合，我把它類比成**團隊做大專案**的概念。

最後講到`Future`跟`CompletableFuture`。Future是async invocation中caller看到對未來回傳值的一個容器，而CompleableFuture是Future的實作，但也扮演了Promise的角色，他代表的是Async invocation的callee在做完時**承諾**要把結果放進去。

而CompletableFuture除了Completable的特性外，其實還有Listenable, Composable, Combinable的特性。並且用monad的map/flatmap來transform這些sync methods變成async methods。

不知道您有沒有發現，每一章節都把前一章節的概念延伸。

1. `synchorinzed` keyword
2. `wait()`/`notify()`包在synchronized中，並且實作producer/consumer
3. 用blockqueue來做producer/consumer
4. 用producer/consumer的概念來做thread pool
5. 透過thread pool的submit來回傳非同步的結果，也就是Future
6. CompletableFuture可以在thread pool中執行

基本上這份文件應該已經涵蓋了所有在java中你所需要的基本multithread知識。而且也看到了整個jdk有關concurrency的演進

- JDK最初版的thread
- JDK1.5的`java.util.concurrent`
- JDK1.7的`ForkJoinPool`
- JDK1.8的`CompletableFuture`

希望這個教學各位會滿意。有任何疑問歡迎留言討論。至於這份文件嚴禁商業用途，而且不允予許轉載。如覺得這份文件很值得分享，歡迎分享[Java多執行緒的基本常識](https://www.gitbook.com/book/popcornylu/java_multithread/details)。

