---
title: 當 Django 轉角遇到 emoji！
date: 2018-05-11 20:17:46
tags:
- "Backend:Django"
- "Database:MySQL"
---

重點事項：

1. 若你是使用 MySQL，請愛用 `utf8mb4` 編碼，將你的所有字串相關欄位轉成該編碼
2. 在 Django 的 `settings.py` 請設置以下屬性，表示 client 讀取 DB 資料時，採用的編碼解讀：
```python
'default': {
    'ENGINE': '...',
    'NAME': '...',
    'USER': '...',
    'PASSWORD': '...',
    'HOST': '...',
    'PORT': '3306',
    'OPTIONS': {
        'charset': 'utf8mb4',  # Here!
    },
},
```
3. 若 Django 有 admin 後台給營運人員操作，請務必注意 `django_admin_log` 這個東西的存在。這個 table 是用戶於 admin 後台的任何操作都會紀錄的儲存對象。建議 `django_admin_log` 結構如下：
```txt
+------------------+-----------------+----------------------+--------------------+--------------------+
| TABLE_NAME       | COLUMN_NAME     | COLUMN_TYPE          | CHARACTER_SET_NAME | COLLATION_NAME     |
+------------------+-----------------+----------------------+--------------------+--------------------+
| django_admin_log | id              | int(11)              | NULL               | NULL               |
| django_admin_log | action_time     | datetime             | NULL               | NULL               |
| django_admin_log | user_id         | int(11)              | NULL               | NULL               |
| django_admin_log | content_type_id | int(11)              | NULL               | NULL               |
| django_admin_log | object_id       | longtext             | utf8               | utf8_general_ci    |
| django_admin_log | object_repr     | varchar(200)         | utf8mb4            | utf8mb4_unicode_ci |
| django_admin_log | action_flag     | smallint(5) unsigned | NULL               | NULL               |
| django_admin_log | change_message  | longtext             | utf8               | utf8_general_ci    |
+------------------+-----------------+----------------------+--------------------+--------------------+
```
4.


推荐參考資源: http://seanlook.com/2016/10/23/mysql-utf8mb4/
