# Resource Sharing

Resource sharing是multi-thread第一個要遇到的課題，也是最重要的課題。如第一章所言，thread之間本來就是共用同一個address space，所以大家都可以用到在JVM中的所有物件，只要存取得到。那就馬上衍生的問題就是，若大家一起共用的時候該怎麼管理狀態?

在Java中有一個非常好用的keyword叫做`synchronized`，我們用它來處理物件共用的問題。它有以下幾種用法

```java
synchronized(myReource) {
   // do something to my resource.
}
```

上面這段code可以保證同時只有你目前執行中的thread可以跑這段code，當然如果你在要執行這段程式碼的時候，已經有其他thread在使用，那就要等它執行完才有機會輪到你去使用。

另外一種寫法是包在method的定義前面

```java
public synchronized void myMethod() {
    //code
}

public static synchronized void myStaticMethod() {
    //code
}
```

其實這會等同於

```java
public void myMethod() {
    synchronized(this) {
        //code
    }
}

public static void myStaticMethod() {
     synchronized(MyClass.class) {
         //code
     }
}
```

也就是可以寫成method level的描述子，來保證這個method同時只有一個thread會去存取目前該method所屬的instance或是class。

# Deadlock

基本上sychronized就是一種lock，當你執行`synchronized(myObject){}`的同時，就是鎖定`myObject`物件。當如果有thread先lock **A**再想lock **B**，而另一個thread是先lock **B**再lock **A**，**那就是會造成俗稱的deadlock**。我們在撰寫程式的時候，應該要統一以先取得A再取得B的順序，也就是前後關係要一致，這樣就可以避免掉deadlock。再來可以思考真的要把lock分到**A**跟**B**那麼細嗎? 還是統一就去取得A就好了，但這就牽涉到程式設計的議題了。

# Race Condition

另外一個極端就是**Race condition**，通常發生在沒有對resource做`synchronized`保護。例如下面這個程式碼就是會有race condition

```java
public class MyClass {
    private int i;
    
    public int getAndIncr() {
        return i++
    }
}
```

因為i++在java底層是分三個動作: 分別是取得值，值+1，存回變數。如果流程如下

```
Thread 1: get value: 100
Thread 2: get value: 100
Thread 2: incr value: 101
Thread 2: set value: 101
Thread 1: incr value: 101
Thread 1: set value: 101
```

上面如果兩個thread是這樣執行就可能造成結果錯誤。因此如果改成以下這種寫法就不會出錯
```java
public class MyClass {
    private int i;
    
    public synchronized int getAndIncr() {
        return i++
    }
}
```

甚至可以直接用[AtomicInteger](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/atomic/AtomicInteger.html)來解決這個問題 
```java
public class MyClass {
    private AtomicInteger i = new AtomicInteger();

    public int getAndIncr() {
        return i.getAndIncrement();
    }
}
```

# Thread Safe
當一個class或method可以在multi-thread環境下不會有race condition，我們可以稱此class或method為**thread safe**。此時你可以很放心的在multi-thread的環境操作這些resource而不用再包`synchronized`。但是thread safe並不是一個class或method一定要提供的責任，畢竟讓resource可以是thread safe必定有其代價。例如用了很多`synchronized`，但如果你只是在single thread的環境使用，那豈不是增加了執行的overhead? 另一種思考是，讓使用library的人去決定`synchronized`包裝的granuarity(顆粒度)也許會更適合。

因此，在java collection library中，大部分的collection其實都不是thread safe。如果想讓你的collection是thread safe，可以透過[Collections](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)中的很多helper methods來提供。例如

```java
syncedCol = Collections.synchronizedCollection(myCol);
syncedList = Collections.synchronizedList(myList);
syncedSet = Collections.synchronizedSet(mySet);
syncedMap = Collections.synchronizedMap(myMap);
```

則可以把原本的non-thread-safe的容器包成thread safe的容器。

# Immutable Object

Immutable(不能修改) object是另外一種resource sharing的策略。概念是resource在唯獨的情況下沒有共用的問題，也不需要lock，只要裡面資料不會更動。但如果想要修改怎麼辦? 此時以產生取代修改，也就是會再產生另外一個**immutable object**。在Java中最經典的例子就是`String`，所有的String的物件是不能修改的，如果我們執行以下的code

```
newStr = str + "2";
```

此時的`newStr`跟`str`會是兩個獨立的object。這種方式最大的好處是即便很多thread去存取也不用去lock，可以增加平行化的程度。


# Other Utilities

在java中還有其他部分也跟resource sharing有關，例如

- [Semaphore](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/Semaphore.html): 可以算是countable lock，也就是會有一定數量的resources可以取得。
- [ReadWriteLock](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/locks/ReentrantReadWriteLock.html): Lock分read跟write，理論上read access可以multiple thread，write access只能single thread。Read/Write lock把read跟write分開可以讓synchronization的成本降低
- [Atomics](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/atomic/package-summary.html): 上面已經介紹過，把一些操控primitives的動作變成是atomic(不可分割的) operation。

由於講下去太細了(~~其實是我懶~~)，所以相關內容就不在這邊討論了。