#Future

[Future](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/Future.html)代表了是一個非同步呼叫的回傳結果，而這個結果會在未來某一個時間點可以取得。這樣講有點抽象，那舉個例子好了，送洗衣服就是一個非同步的呼叫，因為衣服是交給別人洗而非自己洗，而洗好的衣服是一個**未來**會發生的結果，這個結果就是被Future這個class包裝起來。我們再更具體一點的把它變成實際的程式碼，`Future`是一有一個型態參數的class，使用上大概會像這樣

```java
Future<Clothes> result = laundryService.serviceAsync(myClothes);
```

洗衣店提供了一個非同步的服務，所以回傳一個`Future`代表的是非同步的結果(Asynchronous Result)。因為是非同步，所以這個method不會卡住。拿到這個future之後，我們可以選擇做別的事情，或是馬上blocking取得。

```java
// block until result is available
Clothes clothes = result.get();
```

因為是非同步，所以我們可以一次執行很多個非同步的task，讓他們可以平行的去處理。最後再一次等所有的非同步結果，這個生活中也真的挺常發生的，例如我去夜市買雞排豆花跟珍奶，我也不會呆呆的在每一個攤位前面等他一個一個做好，我可能會跟老闆說等等過來拿，讓這幾個攤位可以平行的處理我的tasks。

Future是一個interface，所以需要具體的實作。但是其實不需要自己實作，還記得前面[ThreadPool](basicpool.md)的章節