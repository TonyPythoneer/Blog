---
title: 請勿使用 ElasticSearch 的 uid 來做排序
date: 2018-05-03 14:35:15
tags:
- "Database:ElasticSearch"
---

uid 在 ElasticSearch 是一個 `meta data` 的資訊欄位，而且是 `字串` 屬性，不是由 mapping 可定義的資料結構。

當開發人員把資料從 MySQL/PostgreSQL/MongoDB 倒入至 ElasticSearch 時，可以把資料的 primary key 註記為 document 的 uid。

但是作為排序的對象的話，可能會慘不忍睹。

參考討論： https://discuss.elastic.co/t/how-does-sorting-on--id-work/21854/2
