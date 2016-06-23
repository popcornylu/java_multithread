# Synchronization

在執行了thread之後，下個要問的議題就是要怎麼在thread之間去做synchronization(同步)。我們分以下幾個議題來討論:

1. Resource Sharing: 如果多個threads同時存取變數該如何解決? 是否可以同時只有我這個thread允許修改某一個resource?
2. Flow Control: 一個thread可否等到另外一個thread完成或是執行到某個狀況的時候才繼續往下做?
3. Message Passing: 我可否把我一個thread的output當作另外一個thread的input?


