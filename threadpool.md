# Thread Pool

前面講的都是開一個新的thread來完成單一個工作，但是開thread是有成本的。如果每一個小的task就開一thread，並且有很多的小task需要處理，那產生thread的overhead是很可觀的。因此，比較好的方法是產生一堆threads，稱之為thread pool，讓這些開好的threads來處理這一堆小工作。

其實thread pool的概念在我們的生活中處處可見。開thread處理task就很像外包一個工作，可能我們會在網路上找一個人過來幫忙處理，等到事情處理完了合約就結束了。但是當事情多了，那就可能直接聘雇一群人當作員工來處理事情，這群正職員工就是thread pool，在此員工就是thread，而工作就是task。

在銀行中也會看到thread pool的例子，辦理金融業務的可能會有三四個櫃檯，而當我們有事情要處理會在取票機中提取一個號碼。當職員處理完一個人的業務時，就會叫下一個人過來處理。在這邊職員就是thread，而我們的號碼牌就是一個task，而這個取票機就是queue。

對!我講到一個重點了，thread pool的三大元素就是thread, task, 跟queue。而其實thread pool就是producer consumer pattern的一種形式。consumer就是一堆threads，當queue中一有工作進來，一個空閒的thread就會取出來做處理。

我把前面message passing的程式碼拿過來改一下，馬上就做出一個thread pool。

```java
public class ThreadPool implements Runnable{
    private final LinkedBlockingQueue<Runnable> queue;
    private final List<Thread> threads;
    private boolean shutdown;

    public ThreadPool(int numberOfThreads) {
        queue = new LinkedBlockingQueue<>();
        threads = new ArrayList<>();

        for (int i=0; i<numberOfThreads; i++) {
            Thread thread = new Thread(this);
            thread.start();
            threads.add(thread);
        }
    }

    public void execute(Runnable task) throws InterruptedException {
        queue.put(task);
    }

    private Runnable consume() throws InterruptedException {
        return queue.take();
    }

    public void run()  {
        try {
            while (!shutdown) {
                Runnable task = this.consume();
                task.run();
            }
        } catch(InterruptedException e) {
        }
        System.out.println(Thread.currentThread().getName() + " shutdown");
    }

    public void shutdown() {
        shutdown = true;

        threads.forEach((thread) -> {
            thread.interrupt();
        });
    }


    public static void main(String[] args) throws InterruptedException {
        ThreadPool threadPool = new ThreadPool(5);
        Random random = new Random();

        for (int i=0; i<10; i++) {
            int fi = i;
            threadPool.execute(() -> {
                try {
                    Thread.sleep(random.nextInt(1000));
                    System.out.printf("task %d complete\n", fi);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }

        Thread.sleep(3000);
        threadPool.shutdown();
    }
}
```

這段程式碼跟producer consumer的code非常的接近，都有一個queue，都有produce跟consume methods。不一樣的是queue裡面放的是一個Runnable，代表的是一個可以執行的task。

另外在constructor中我們有一個參數是`numberOfThreads`，也就是這個thread pool中產生threads的個數，而每個thread的執行內容是一個infinite loop，做的事情就不斷的從queue中拿一個task出來，去執行裡面的內容。

在main method中我產生了一個thread pool，裡面有五個職員(thread)跟一個取票機(queue)。而我抽了十張號碼牌(task)，在當中隨機了睡了一段時間並且印出訊息，最後執行的結果大概會像下面這樣。

```
Thread-1: task 1 complete
Thread-0: task 0 complete
Thread-1: task 5 complete
Thread-1: task 7 complete
Thread-4: task 4 complete
Thread-4: task 9 complete
Thread-3: task 3 complete
Thread-1: task 8 complete
Thread-2: task 2 complete
Thread-0: task 6 complete
Thread-0 shutdown
Thread-3 shutdown
Thread-2 shutdown
Thread-1 shutdown
Thread-4 shutdown
```

其實自己做thread pool就是那麼簡單，沒有什麼高深的學問。當然你不需要自己做thread pool，在java中已經有很多現有的thread pool可以直接拿來使用，不必自己造輪子。接下來介紹一下有哪些thread pool可以拿來使用。

