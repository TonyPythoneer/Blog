---
title: 客製化 order 順序：ORM vs MySQL builtin function
date: 2018-03-02 18:30:07
tags:
- "Backend: Django"
- "Database: MySQL"
---

當有一個 request 的是 `/api/v3/banners/?zones=tw-app-feature-1,tw-app-feature-5,tw-app-feature-3,tw-app-feature-4,tw-app-feature-2` 必須符合 query string 的順序時，以下有 ORM 和 MySQL built-in function 兩種處理方式，提供比較：

`order by 填入 case then`：

```python
preserved = Case(*[When(zone__slug=zone, then=index) for index, zone in enumerate(zone_list)])
queryset = queryset.filter(zone__slug__in=zone_list).order_by(preserved)
```

```sql
SELECT
    `ads_banner`.`id`,
    `ads_banner`.`name`,
    `ads_banner`.`zone_id`,
    `ads_banner`.`uri`,
    `ads_banner`.`open_in_new_window`,
    `ads_banner`.`image`,
    `ads_banner`.`created_at`,
    `ads_banner`.`last_modified`
FROM
    `ads_banner`
        INNER JOIN
    `ads_zone` ON (`ads_banner`.`zone_id` = `ads_zone`.`id`)
WHERE
    (`ads_zone`.`site_id` = 1
        AND `ads_zone`.`slug` IN ('tw-app-feature-1' , 'tw-app-feature-5',
        'tw-app-feature-3',
        'tw-app-feature-4',
        'tw-app-feature-2'))
ORDER BY CASE
    WHEN `ads_zone`.`slug` = 'tw-app-feature-1' THEN 0
    WHEN `ads_zone`.`slug` = 'tw-app-feature-5' THEN 1
    WHEN `ads_zone`.`slug` = 'tw-app-feature-3' THEN 2
    WHEN `ads_zone`.`slug` = 'tw-app-feature-4' THEN 3
    WHEN `ads_zone`.`slug` = 'tw-app-feature-2' THEN 4
    ELSE NULL
END ASC
```

MySQL built-in function 的 `FIELD`：

```python
field_func_args = ['{}.{}'.format(Zone._meta.db_table, 'slug'), ] + ['\'{}\''.format(zone) for zone in zone_list]
field_func = 'FIELD({})'.format(', '.join(field_func_args))  # https://dev.mysql.com/doc/refman/5.7/en/string-functions.html#function_field
queryset = queryset \
    .filter(zone__slug__in=zone_list) \
    .extra(select={'order': field_func}, order_by=['order'])
```

```sql
SELECT
    (FIELD(`ads_zone`.`slug`,
            'tw-app-feature-1',
            'tw-app-feature-5',
            'tw-app-feature-3',
            'tw-app-feature-4',
            'tw-app-feature-2')) AS `manual`,
    `ads_banner`.`id`,
    `ads_banner`.`name`,
    `ads_banner`.`zone_id`,
    `ads_banner`.`uri`,
    `ads_banner`.`open_in_new_window`,
    `ads_banner`.`image`,
    `ads_banner`.`created_at`,
    `ads_banner`.`last_modified`
FROM
    `ads_banner`
        INNER JOIN
    `ads_zone` ON (`ads_banner`.`zone_id` = `ads_zone`.`id`)
WHERE
    (`ads_zone`.`site_id` = 1
        AND `ads_zone`.`slug` IN ('tw-app-feature-1',
            'tw-app-feature-5',
            'tw-app-feature-3',
            'tw-app-feature-4',
            'tw-app-feature-2')
        )
ORDER BY `manual` ASC
```

兩者相比的話：
  * 調用 `FIELD` 時， `queryset` 在輸出 SQL 語句會可讀性好很多；但缺點是 Python 要處理成
 SQL 語句的 string 時會比較費工夫，閱讀性比較差。
  * `order by 填入 case then` 在 Python 和 SQL 語句輸出會比較直覺、簡單許多。

如果當你的要 select 的資料不多的時候，用 `order by 填入 case then` 會清爽簡潔許多。
