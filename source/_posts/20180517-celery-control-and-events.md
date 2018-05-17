---
title: Celery 沒告訴你的管理機制：control 和 events
date: 2018-05-17 17:47:09
tags:
- "Utility:Celery"
---

請問多少人看過或使用這個介面，若你有需要對 Celery 的管理了解，可以看看這裡。

但看完後，我建議不要太依賴或相信 Celery 的官方文件，做好 log 比起官方文件幫助更多。

<!-- more -->

若你的 Python 開發環境有使用 Celery，建議如下：

```py
import logging

logger = logging.getLogger('celery')

def func(a, b, c):
    msg_tmpl = '{func_name} invoked {celery_task_name} of celery task with {args} arguments'
    msg = msg_tmpl.format(func_name=func.__name__, celery_task_name=celery_task.__name__, args=[a,b,c])
    logging.info(msg)


@task_prerun.connect
def task_prerun_test(sender=None, body=None, **kwargs):
    msg_tmpl = '{sender} run with {body}'
    msg = msg_tmpl.format(sender=func.__name__, body=celery_task.__name__)
    logging.info(msg)
```