# Flow Control

Thread之間除了共用資料以外，流程同步也是常用到的一個技巧。如果把人當做一個thread，彼此之間協同合作就是flow control。是不是常常會有某A做完了一個動作之後，某B甚至某C才可以繼續做後續的動作呢? 在multi-thread中也常常會有這樣的需求。

## Wait and notify

在java中有關流程控制有一個最基本的primitive，那就是`Object#wait()`跟`Object#notify()`。當一個thread對一個物件呼叫`wait()`之後會完全卡住，要一直到另外一個thread觸法`notify()`才可以繼續往下執行。幾乎所有有關流程控制的邏輯最底層都是透過wait跟notify來實現。

我打算用一個簡化版的[Producer Consumer Pattern](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem)來解釋flow control。通常producer負責產生message，而consuemer負責接收並處理message。中間會有一個queue，當produce時queue是滿的，或是consume時queue是空的，那caller thread就會被blocked，直到狀態解除為止。但是為了方便解釋起見，這邊拿掉了queue，而只單純的讓producerd丟一個message給consumer而已。下面是一個範例程式:

```java
public class FlowControl {

    // Lock for synchronization
    private Object lock = new Object();
    // Message to pass
    private String message = null;

    public void produce(String message) {
        System.out.println("produce message: " + message);
        synchronized (lock) {
            this.message = message;
            lock.notify();
        }
    }

    public void consume() throws InterruptedException {
        System.out.println("wait for message");
        synchronized (lock) {
            lock.wait();
            System.out.println("consume message: " + message);
        }
    }


    public static void main(String[] args) throws InterruptedException {
        final FlowControl flowControl = new FlowControl();

        // Create consumer thread
        new Thread(()->{
            try {
                flowControl.consume();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // Sleep for 1 sec
        Thread.sleep(1000);

        // Create producer thread
        new Thread(()->{
            flowControl.produce("helloworld");
        }).start();
    }
}
```

我們分別定義了`produce()`跟`consume()`兩個methods。在consume中，我們會呼叫`lock.wait()`，注意wait這個method一定要讓被呼叫的那個object被包在**sychronized block**之中，不然直接會有compile error。相對的，在produce中我們會呼叫`lock.notify()`，同樣的也是要包在synchronized中。

在這個例子裡面，我們也可以用`this`來取代`lock`物件。也就是用`sychronized(this)`, `this.wait()`, `this.notify()`取代。但這邊會用一個獨立的lock有個優點就是有比較好的隔離效果，以避免外部取得`FlowContorl`此class的instance的人，有機會影響內部結果。

最後看到`main` method，我們起了兩個threads，先是consumer thread，他會等著consume一個message；再來睡了一秒鐘後，我們產生了producer thread，他會produce一個`helloworld` message。最後跑出的結果就會像這樣

```
wait for message
produce message: helloworld
consume message: helloworld
```

另外有一個`Object#notifyAll()` method其實跟`Object#notify()`類似前者一次叫醒所有waiting threads，後者只會叫醒一個waiting thread。


## Thread Join

另外一種更簡單的做法就是thread join。**Join**這個動詞在流程同步的時候常常會使用，他代表的就是等待別人的完成；而相對的詞是**Fork**，代表的是把工作發包給別的thread開始做。

下面的程式碼則是一個簡單的範例，我們產生一個worker thread來執行我們的task，在我們的main thread中去**join**這個worker

```java
// Create workder thread
Thread worker = new Thread(() -> {
    try {
        System.out.println("worker start");
        Thread.sleep(1000);
        System.out.println("worker complete");
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
});

worker.start();

System.out.println("master wait");
worker.join();
System.out.println("master complete");
```

此程式碼的output會是這樣，可以看到master要等worker完成後，才會繼續往下執行。

```
master wait
worker start
worker complete
master complete
```

## Other Utilities

其他還有一些工具可以幫你做Flow Control

- [CyclicBarrier](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/CyclicBarrier.html): Barrier算是一個幾同步的時候很常用的術語，代表的是所有的thread都到某著stage的時候，再繼續往下走。這概念就很像出遊的時候都會在一個地點集合，全部都到齊了再一起出發的概念。
- [CountDownLatch](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/CountDownLatch.html): 其實就如其名，是一個倒數的概念。每呼叫一次`countdown()`則倒數的counter就會減一，當一個thread去呼叫`await()`時，會一直卡在那邊，直到counter歸零為止，才會繼續往下走。
- [Future](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/Future.html) and [Completable Future](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/CompletableFuture.html): 這個是Flow Control更高層次的封裝，因為內容比較多，我們會在後面的章節討論。
