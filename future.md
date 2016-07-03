#Future

[Future](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html)代表了是一個非同步呼叫的回傳結果，而這個結果會在未來某一個時間點可以取得。這樣講有點抽象，那舉個例子好了，送洗衣服就是一個非同步的呼叫，因為衣服是交給別人洗而非自己洗，而洗好的衣服是一個**未來**會發生的結果，這個結果被Future這個class包裝起來。

我們再更具體一點的把它變成實際的程式碼，`Future`是一有一個型態參數的class，使用上大概會像這樣

```java
Future<Clothes> future = laundryService.serviceAsync(myClothes);
```

洗衣店提供了一個非同步的服務，所以回傳一個`Future`代表的是非同步的結果(Asynchronous Result)。因為是非同步，所以這個method不會卡住。拿到這個future之後，我們可以選擇做別的事情，或是馬上blocking取得。

```java
// block until result is available
Clothes clothes = future.get();
```

因為是非同步，所以我們可以一次執行很多個非同步的task，讓他們可以平行的去處理，最後再一次等所有的非同步結果。這在生活中也挺常發生，例如我去夜市買雞排豆花跟珍奶，我也不會呆呆的在每一個攤位前面等他一個一個做好，我可能會跟老闆說等等過來拿，讓這幾個攤位可以**平行**的處理我的tasks。

Future是一個interface，所以需要具體的實作。但通常不需要自己實作，還記得前面[ThreadPool](basicpool.md)的章節就有提到`Future`了嗎? 事實上[ExecutorService#submit](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ExecutorService.html#submit-java.util.concurrent.Callable-)就提供了一個非同步執行的實作，並且回傳一個`Future`，一般來講我們只要使用這個method來實作我們的非同步執行就可以了。

所以上面的程式碼的具體實作可能長這樣

```java
public class LaundaryService {
    private ExecutorService executorService = /*...*/;

    public Future<Clothes> serviceAsync(Clothes dirtyClothes) {
        return executorService.submit(()-> service(dirtyClothes));
    }

    public Clothes service(Clothes dirtyClothes) {
        return dirtyClothes.wash();
    }
}
```

如此一來，這個LaundaryService當中有一個threadpool幫忙洗衣服，當呼叫`serviceAsync`，則會產生一個task在threadpool中執行，並且先回傳一個Future代表這個非同步的結果，就很像是一個取貨單一樣。caller在之後再透過future取得最後完成的結果。