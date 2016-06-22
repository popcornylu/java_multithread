# Synchronization

在執行了Thread之後，下個要問的議題就是要怎麼在Thread之間去做Synchronization(同步)。這不外乎有以下幾個議題

1. Resource sharing: 如果同時存取變數要怎麼解決? 是否可以同時只有我這個Thread可以修改某一個Resource?
2. Flow control: Thread之間可否等另外一個Thread的觸發某個event的時候，我才繼續往下做?
3. Data Passing: 我可否把我一個Thread的結果傳給另外一個Thread去做接下來的任務?


