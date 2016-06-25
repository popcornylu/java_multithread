# Flow Control

Thread之間除了共用資料以外，流程同步也是常用到的一個技巧。如果把人當做一個thread，彼此之間協同合作，是不是常常某A做完了一個動作之後，某B甚至某C才可以繼續做後續的動作呢? 在multi-thread中也常常會有這樣的需求。

## Wait and notify

在java中有關流程控制有一個最基本的primitive，那就是`Object#wait()`跟`Object#notify()`。`wait()`是要等待別thread觸發某個event，`notify()`是觸發event通知別個thread。所以接下來任何有關流程控制的幾乎最底層都是透過wait跟notify來實現。

我打算用一個簡化版的[Producer Consumer Pattern](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem)來解釋flow control。通常Producer會產生message，而consuemer會收message，中間會是一個queue，並且produce時queue是滿的，或是consume時queue是空的，那就會被blocked，直到狀態改變為止。但是為了解釋方便起見，這邊只單純的讓producer thread會丟一個message給cosumer thread而已。下面是一個範例程式:

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

我們分別定義了`produce()`跟`consume()`兩個method。在consume中，我們會呼叫`lock.wait()`，注意wait這個method一定要被呼叫的那個object要是被`sychronized`的，不然直接會有compile error。相對的，在produce中我們會呼叫`lock.notify()`，同樣的也是要包在synchronized中。

在這個例子裡面，我們也可以用`sychronized(this)`, `this.wait()`, `this.notify()`取代。只要確定wait跟notify是對同一個物件呼叫就可以了。而這邊會用一個lock也算是一種比較好的隔離效果，以避免外部取得FlowContorl的instance的人，有機會影響內部結果。

最後看到`main` method，我們起了兩個thread，先是consumer thread，他會等著consume一個message；再來睡了一秒鐘後，我們產生了producer thread，他會produce一個`helloworld` message。最後跑出的結果就會像這樣

```
wait for message
produce message: helloworld
consume message: helloworld
```

有關notify method，除了`Object#notify()`以外，還有一個`Object#notifyAll()`。前者一次只叫醒一個thread，後者會叫醒所有`wait()`此物件的threads。


# Thread Join

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

##
