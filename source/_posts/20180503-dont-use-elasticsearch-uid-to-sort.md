---
title: 請不要使用 elasticsearch 的 uid 來做排序
date: 2018-05-03 14:35:15
tags:
---

參考討論： https://discuss.elastic.co/t/how-does-sorting-on--id-work/21854/2

uid 不是在 elasticsearch 做 mapping 時，可以定義 document 的資料結構，因為 uid 屬於 document 的 meta data，不可改變。

而 uid 是屬於「字串」欄位，在作 sort 時，是比較第一個字串來做排序，而非最後一個字串。例如說，當作降序時，會得到以下 uid 排序。

```
999
998
997
...(以此類推)
1050
1049
1048
...(以此類推)
10409
10408
10407
...(以此類推)
```

請務必不要當作「整數」欄位，進行排序。