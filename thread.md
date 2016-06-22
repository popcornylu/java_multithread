# Introduction

Java Thread是Java非常美麗的一個起點，早在Java的最早的時候就有這個Library，在那個Multi-Threading還沒有很成熟的年代，Java Thread的簡單好用讓大家開始發現寫Multi-Thread不是什麼難事。

# How to use

廢話不多說，直接來看Java Thread怎麼使用

```java
new Thread(() -> {
    System.out.println("hello thread");
}).start();
```

在上面的程式中產生了一個新的Thread，必且執行Lambda內的那段程式碼，跑完之後，Thread的生命週期則結束。
