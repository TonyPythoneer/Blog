---
title: Celery 沒告訴你 signal 的事
date: 2018-03-02 18:30:07
tags:
- "Program Language:Python"
---




```python
#@after_task_publish.connect(sender='playlist.tasks.test')
@after_task_publish.connect
def after_task_publish_test(sender=None, body=None, **kwargs):
    print '=====after_task_publish'
    print body
    print sender
    print time.sleep(1)
    print time.time()

#@before_task_publish.connect(sender='playlist.tasks.test')
@before_task_publish.connect
def before_task_publish_test(sender=None, body=None, **kwargs):
    print '=====before_task_publish'
    print body
    print sender
    print time.sleep(1)
    print time.time()

#@task_prerun.connect(sender='playlist.tasks.test')
@task_prerun.connect
def task_prerun_test(sender=None, body=None, **kwargs):
    print '=====task_prerun'
    print body
    print sender
    print time.sleep(1)
    print time.time()

#@task_postrun.connect(sender='playlist.tasks.test')
@task_postrun.connect
def task_postrun_test(sender=None, body=None, **kwargs):
    print '=====task_postrun'
    print body
    print sender
    print time.sleep(1)
    print time.time()
```

```txt
=====before_task_publish
{'expires': None, 'utc': True, 'args': ('aaa', 'bbb'), 'chord': None, 'callbacks': None, 'errbacks': None, 'taskset': None, 'id': 'd4624317-0869-40b9-bcda-52c305eb2ed2', 'retries': 0, 'task': 'playlist.tasks.test', 'timelimit': (None, None), 'eta': None, 'kwargs': {}}
playlist.tasks.test
None
1511856143.6
=====after_task_publish
{'expires': None, 'utc': True, 'args': ('aaa', 'bbb'), 'chord': None, 'callbacks': None, 'errbacks': None, 'taskset': None, 'id': 'd4624317-0869-40b9-bcda-52c305eb2ed2', 'retries': 0, 'task': 'playlist.tasks.test', 'timelimit': (None, None), 'eta': None, 'kwargs': {}}
playlist.tasks.test
None
1511856144.61

celery.redirected: WARNING [2017-11-28 16:01:55] -------------------------------------------------------------------------------
celery.worker.strategy: INFO [2017-11-28 16:02:23] Received task: playlist.tasks.test[d4624317-0869-40b9-bcda-52c305eb2ed2]
celery.redirected: WARNING [2017-11-28 16:02:23] =====task_prerun
celery.redirected: WARNING [2017-11-28 16:02:23] None
celery.redirected: WARNING [2017-11-28 16:02:23] <@task: playlist.tasks.test of streetvoice:0x7f6137f6ce90 (v2 compatible)>
celery.redirected: WARNING [2017-11-28 16:02:24] None
celery.redirected: WARNING [2017-11-28 16:02:24] 1511856144.61
celery.redirected: WARNING [2017-11-28 16:02:24] test:
celery.redirected: WARNING [2017-11-28 16:02:24] aaa
celery.redirected: WARNING [2017-11-28 16:02:24] bbb
celery.redirected: WARNING [2017-11-28 16:02:24] =====task_postrun
celery.redirected: WARNING [2017-11-28 16:02:24] None
celery.redirected: WARNING [2017-11-28 16:02:24] <@task: playlist.tasks.test of streetvoice:0x7f6137f6ce90 (v2 compatible)>
celery.redirected: WARNING [2017-11-28 16:02:25] None
celery.redirected: WARNING [2017-11-28 16:02:25] 1511856145.61
```