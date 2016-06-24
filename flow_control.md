# Flow Control

Thread之間出除了共用資料以外，流程同步也是常用到的一個技巧。如果把人當做一個thread，彼此之間協同合作，是不是常常某A做完了一個動作之後，某B甚至某C才可以繼續做後續的動作呢? 在multi-thread中也常常會有這樣的需求。

# Wait and notify

在java中有關流程控制有一個最基本的primitive，那就是`Object#wait()`跟`Object#notify()`。`wait()`是要等待別thread觸發某個event，`notify()`是觸發event通知別個thread。所以接下來任何有關流程控制的幾乎最底層都是透過wait跟notify來實現。

下面是一個簡單的範例。

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

這是一個很典型的[Producer Consumer Pattern](https://en.wikipedia.org/wiki/Producer%E2%80%93consumer_problem)