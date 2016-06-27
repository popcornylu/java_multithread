# Message Passing

Message passing簡單來講就是一個thread想把資料丟給另外一個thread去做處理。其實在前面flow control的章節在講producer/consumer pattern時，其實就已經點出了一個概念。但是可以看得出我們的實作過於簡略，producer跟consumer共用了同一個message變數，當producer/consumer多了，勢必會造成前面所說的**race condition**。那有沒有辦法用更high-level的方式來傳遞資訊呢?

工廠生產的時候，有一個名詞是生產線(pipeline)，每一站都把前一個站的產出變成下一站的來源，而最後一站的產出就是這個工廠的產品。其實這個概念可以想像每一站就是一個thread，而這些站跟站之間的半成品就可以稱它為message。而站跟站之間，會透過一個輸送帶當作管道傳遞，而這個管道我們稱為pipe或是queue。

## Blocking Queue

在Java中有一個非常好用並且現成的東西幫我們做Consumer/Producer Pattern，那就是[BlockingQueue](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/BlockingQueue.html)，我們可以透過它輕易做到之前所說的queue滿了讓producer block，以及queue空了讓conusmer block的結果。我把前面一個章節有關producer consumer的code改用BlockingQueue去作實現。

```java
public class ProducerConsumer {

    private LinkedBlockingQueue queue = new LinkedBlockingQueue<>(5);

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
                    Thread.sleep(random.nextInt(1000));
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
                    Thread.sleep(random.nextInt(1000));
                }
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```
可以看得到我們在produce中呼叫了`BlockingQueue#put()`，它會放一個message進queue中，如果滿了會block；同樣的在consume的地方呼叫`BlockingQueue#take()`，它會從queue中取一筆資料出來，如果queue是空的則會block。為了刻意讓queue比較容易滿，我把queue的constructor帶`5`進去，這代表的是它的容量。我們可以改變`main`中的sleep time，來模擬**供過於求**跟**供不應求**的狀況。

## Pipe

有使用Unix/Linux的讀者應該很熟悉[pipe](https://en.wikipedia.org/wiki/Pipeline_(software%29)也就是`|`這個shell operation，這是Unix當初在設計的時候非常重要的[philosophy](https://en.wikipedia.org/wiki/Unix_philosophy):

- Write programs that do one thing and do it well.
- Write programs to work together.
- Write programs to handle text streams, because that is a universal interface.

下面就是一個常見的案例

```bash
tail accesslog | awk '{print $1}' | sort | uniq -c  | sort -nr
```

我們把前面process的stdout當作下一個process的stdin。透過這行指令，我們馬上建立一個pipeline，最後把結果印出到terminal之上，有沒有很像我剛剛說的工廠的情境?

在Java中thread之間也可以這麼做(雖然這麼做不常見 ^^)，利用的是[PipedOutputStream](https://docs.oracle.com/javase/8
/docs/api/java/io/PipedOutputStream.html)跟[PipedInputStream](https://docs.oracle.com/javase/8/docs/api/java/io/PipedInputStream.html)這兩個class，兩者可組合成一個Pipe，其中outputstream的產出就會是inputstream的輸入。下面是個範例。

```java
public class Pipe {

    private PipedInputStream in;
    private PipedOutputStream out;

    public Pipe() throws IOException {
        in = new PipedInputStream(4096);
        out = new PipedOutputStream(in);
    }

    public String readAll() throws IOException {
        StringBuilder stringBuilder = new StringBuilder();
        try (BufferedReader reader = new BufferedReader(new InputStreamReader(in))) {
            String line;
            while ((line = reader.readLine()) != null) {
                stringBuilder.append(line);
            }
        }

        return stringBuilder.toString();
    }

    public void writeAll(String string) {
        try (PrintStream printStream = new PrintStream(out)){
            printStream.print(string);
        }
    }

    public static void main(String[] args) throws IOException {
        Pipe pipe = new Pipe();
        
        // produce a message
        new Thread(() -> {
            pipe.writeAll("hello pipe!!");
        }).start();

        // consume the message
        new Thread(() -> {
            try {
                System.out.println(pipe.readAll());
            } catch (IOException e) {
                e.printStackTrace();
            }
        }).start();
    }
}
```

## Other Utility

- [Pipe](https://docs.oracle.com/javase/8/docs/api/java/nio/channels/Pipe.html): nio的版本。其實跟`PipedInputStream`跟`PipedOutputStream`類似。
- [Future](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/Future.html) and [Completable Future](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/CompletableFuture.html): 其實兩個都同時扮演了flow control跟message passing。一樣是留在後面的章節再做討論。

## Recap

到這裡我們討論了resource sharing, flow control, message passing。這可以說是multi-threading中最重要的三個議題。但是mult-thread程式不好撰寫，即使是知道了上面這些課題，還是很容易出錯。因此有必要把更high-level，更一般化的問題包裝成更好用的介面。後面章節我們會把thread pool跟asynchronous invokation這兩個常用的情境做更進一步介紹。
