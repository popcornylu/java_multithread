# Introduction
Thread(線程，或稱執行緒)是一個美妙的東西，它允許你在同一個address space(定址空間)撰寫concurrency(併發)以及parallelism(並行)的程式。通常跟它會拿來比較的是process(進程)，跟thread不一樣的是，process之間會是在不同的定址空間，因此可以做到比較高度的隔離，但是缺點就是比較難在process之間溝通。因此，thread我們會稱為較lightweight的process。

在Java當中一個JVM就是一個process，在JVM中可以產生非常多的threads。Java thread早在java的最早版本就存在，在那個multi-threading還沒有很成熟的年代，java thread的簡單好用讓大家開始發現寫multi-thread不是什麼難事。

這本書主要目的除了簡單介紹thread以外，主要還是介紹如何寫multi thread的程式。這包含thread之間的synchronization，如何使用thread pool，到進階的asynchronous程式的撰寫。內容是以Java8為基礎，除了是用了很多lambda以外，也介紹了java8才有的CompletableFuture。但如果還未寫過Java8的讀者，相信基本觀念還是會有幫助。

