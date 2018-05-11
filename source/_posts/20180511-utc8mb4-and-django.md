---
title: 20180511-utc8mb4-and-django
date: 2018-05-11 20:17:46
tags:
---

重點節錄， `utf8mb4` 的 mb4 即 most bytes 4。
若你的 MySQL 字串欄位要使用 emoji 時，建議使用 `utf8mb4` 編碼。

若你使用 Django 開發時，在字串欄位驗證，只會做到長度驗證，而不是 bytes 驗證，在制定長度時，要非常小心！

```python
a = u'😀'
print len(a) # 1 -> 1 of length
print repr(a) # u'\U0001f600' -> 8 byte
```

ref: http://seanlook.com/2016/10/23/mysql-utf8mb4/
