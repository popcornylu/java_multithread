# Resource Sharing

Resource Sharing是Multi-Thread第一個要遇到的課題，也是最重要的課題。Thread之間本來就是共用同一個address space，所以大家都可以用到所有在JVM中的物件，只要存取得到。那就馬上衍生的問題就是大家一起用的時候該怎麼辦?

在Java中有一個非常好用的keyword是`synchronized`，他有以下幾種用法

```java
synchronized(myReource) {
   // do something to my resource.
}
```

上面這段code可以保證同時只有你目前執行中的Thread可以跑這段code，當然如果你在要執行這段程式碼的時候，已經有其他thread在使用，那就要等他執行行完這個Block你才有機會輪到你去使用。

另外一種做法是包在method的定義前面

```java
public synchronized void myMethod() {
}

public static synchronized void myStaticMethod() {
}
```

這會等同於

```java
public void myMethod() {
    synchronized(this) {
    }
}

public static synchronized void myStaticMethod() {
     synchronized(MyClass.class) {
     }
}
```

# Deadlock

基本上sychronized本質上就是一種lock，當你執行

```
synchronized(myObject){...}
```

的時候就是鎖定myObject物件。當如果有Thread先lock A再想lock B，而另一個Thread是先lock B再lock A，那就是會造成俗稱的DeadLock。我們在撰寫程式的時候，應該要統一先取得A在取得B的順序，也就是先後關係要一致。再來說需要把resource分到A跟B那麼細嗎? 還是統一就去取得A就好了，這個就牽涉到設計邏輯了。


# Race Condition

另外一個既端就是Race Condition，通常發生在沒有對resource做synchonized保護。例如下面這個程式碼就是會有race condition

```java
public class MyClass {
    private int i;
    
    public int getAndIncr() {
        i++
    }
}
```

因為i++在java底層是分三個動作，取得值，值+1，存回變數。如果流程如下

```
Thread 1: get value: 100
Thread 2: get value: 100
Thread 2: incr value: 101
Thread 2: set value: 101
Thread 1: incr value: 101
Thread 1: set value: 101
```

可以看到這就會造成錯誤。因此如果改成以下這種寫法就不會出錯
```java
public class MyClass {
    private int i;
    
    public synchronized int getAndIncr() {
        i++
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
當一個Resource可以在Multi-Thread可以正常運作就可以稱為Thread Safe。此時你可以很放心的在Multi Thread的環境使用這些Resource。但是Thread Safe並不是一個Component設計必要條件，畢竟有捨有得，讓Resource可以是Thread Safe必定有其代價，例如用了很多synchonized，但如果你只是在single thread的環境使用，那豈不是增加了執行的overhead? 

因此，在Java Collection中，大部分的Collection其實都不是Thread Safe，如果讓你的Collection是Thread Safe，可以透過[Collections](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html)中的很多Helper method來提供。例如

```java
syncedCol = Collections.synchronizedCollection(myCol);
syncedList = Collections.synchronizedList(myList);
syncedSet = Collections.synchronizedSet(mySet);
syncedMap = Collections.synchronizedMap(myMap);
```

則可以把原本的Non Thread Safe的容器wrap成Thread Safe的容器。

# Immutable Object

Immutable(不能修改) Object是Resource Sharing的策略。概念是Resource讓大家Share沒關係，也不需要Lock，但是裡面的資料完全不能修改。但如果有想修改怎麼辦? 此時以產生取代修改，也就是會再產生另外一個immutable object。在Java中最經典的例子就是`String`，所有的String的物件是不能修改的，如果我們執行以下的code

```
newStr = str + "2";
```

此時的`newStr`跟`str`會是兩個獨立的object。這種方式最大的好處是即便很多Thread去存取也不用去lock，可以增加平行化的程度。


# Other Utitlities

在Java中還有其他部分也跟Resource Sharing有關，例如

- [Semaphore](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/Semaphore.html): 可以算是countable lock，也就是會有一定數量的resources可以取得。
- [ReadWriteLock](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/locks/ReentrantReadWriteLock.html): Lock分Read跟Write，理論上Read可以Multiple，Write只能Single。Read/Write lock把Read跟Write分開可以讓synchronization的成本降低ㄡ
- [Atomics](https://docs.oracle.com/javase/8/docs/api/index.html?java/util/concurrent/atomic/package-summary.html): 上面已經介紹過，把一些primitive可以保證是atomic(不可分割的) operation。

由於講下去太細了(~~其實是我懶~~)，所以相關內容就不在這邊討論了。