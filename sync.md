# Synchronization

在執行了Thread之後，下個要問的議題就是要怎麼在Thread之間去做Synchronization(同步)。我們分以下幾個議題來討論:

1. Resource Sharing: 如果多個Thread同時存取變數該如何解決? 是否可以同時只有我這個Thread允許修改某一個Resource?
2. Flow Control: 一個Thread可否等到另外一個Thread完成或是執行到某個狀況的時候才繼續往下做?
3. Message Passing: 我可否把我一個Thread的output當作另外一個Thread的input?


