# Message Passing

Message passing簡單來講就是一個thread想把資料丟給另外一個thread去做處理。其實在前面flow control的章節在講producer/consumer pattern時，其實就已經點出了一個概念。但是可以看得出我們的實作過於簡略，producer跟consumer更用了同一個message變數，當producer/consumer多了，勢必會造成第一章所說的race condition。那有沒有辦法用更high-level的方式來傳遞資訊呢?

工廠生產的時候，有一個名詞是生產線(pipeline)，每一站都把前一個站的產出變成下一站的來源，而最後一站的產出就是這個工廠的產品。其實這個概念可以想像每一站就是一個thread，而這些站跟站之間的半成品，我們就可以稱它為message。


# Blocking Queue

```java
public class ProducerConsumer {

    LinkedBlockingQueue queue = new LinkedBlockingQueue<>(5);

    public void produce(String message) throws InterruptedException {
        System.out.println("produce message: " + message);
        queue.put(message);

    }

    public void consume() throws InterruptedException {
        System.out.println("consume message: " + queue.take());

    }


    public static void main(String[] args) throws InterruptedException {
        final ProducerConsumer producerConsumer = new ProducerConsumer();

        // Create consumer thread
        new Thread(()->{
            Random random = new Random();
            try {
                while (true) {
                    producerConsumer.consume();
//                    Thread.sleep(random.nextInt(1000));
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();

        // Create producer thread
        new Thread(()->{
            Random random = new Random();
            try {
                int counter = 0;
                while (true) {
                    producerConsumer.produce("message" + counter++);
//                    Thread.sleep(random.nextInt(1000));
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```




