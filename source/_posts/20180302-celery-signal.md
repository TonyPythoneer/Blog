---
title: Celery 沒告訴你 task signal 的事
date: 2018-03-02 18:30:07
tags:
- "Program Language:Python"
- "Utility:Celery"
---

![message-queue-workflow](/images/2018/3/message-queue-workflow.png)

{% centerquote %}
從 Celery 的 task signal 了解其工作生命周期如何運作。

當 sender 到 message queue，再到 receiver 的傳遞過程。
{% endcenterquote %}

<!-- more -->

當時遇到一個問題，因而研究 celery signal 來解決問題。

情境如下：
> 在一個音樂平台上，當用戶上傳一首單曲，會執行新增單曲並紀錄於 database ，接著會異步執行更新歌單封面的動作，從現有的歌曲封面中組成拼貼作為歌單封面。

遇到問題是：
> 當用戶正在頻繁地上傳歌曲時，同時也會產生大量的「拼貼封面製作」異步執行，不斷地更新歌單的封面圖片，造成該 row data 發生 lock，以致於 database 操作失敗。

解決思路：
> 當要對該歌單執行「拼貼封面製作」時，設置 task 延遲執行時間，例如： 30 分鐘，這筆 task 紀錄在 redis 裏面。若接下來 sender 有繼續發送對該歌單的 task，註明該 task 為 `銷毀`，表示 receiver 拿到該 task 時不會做出執行動作。

解決方法如下：

* 當 web server(receiver) 發送 task 時：

這裡我是設置 30 分鐘後延遲執行拼貼。

```py
tasks.create_playlist_mosaic.apply_async((playlist.id,), countdown=60 * 30)
```

* 在 task 發現前，設置一個 before_task_publish 的 signal，在 signal 發送前捕捉它：

由於我使用的 celery 是老舊的版本，請參考 [body 的結構](http://docs.celeryproject.org/en/latest/internals/protocol.html#version-1)。

當在 celery 設置 `countdown` 時，他會建立一個 task。而 body 的 `eta` 是發派任務後，設定 `countdown` 或 `eta` 的 Celery 轉換結果，表示  `即將執行的預定時間`。可以用此時間數據跟當下時間對比，換得 `延遲時間偏移`，作為儲存在 redis 的 expire time。這樣可以完成對單一資源已經執行延遲執行，註銷後續 task 的處理了。

```py
@before_task_publish.connect
def before_task_publish_create_playlist_mosaic(sender=None, body=None, **kwargs):
    if sender == create_playlist_mosaic.name:
        if not body['eta']:
            return

        eta_datetime = parser.parse(body['eta'])
        now_datetime = timezone.now()
        diff_datetime = eta_datetime - now_datetime
        expire_seconds = int(diff_datetime.total_seconds())

        func_kwargs = inspect.getcallargs(create_playlist_mosaic.__class__.run, *body['args'], **body['kwargs'])
        rdb_key = 'celery:{task_name}:{unique_id}'.format(task_name=create_playlist_mosaic.__name__, unique_id=func_kwargs['playlist_id'])

        if redis_db.exists(rdb_key):
            old_task_id = redis_db.get(rdb_key)
            AsyncResult(old_task_id).revoke()
        with redis_db.pipeline() as pipe:
            pipe.set(rdb_key, body['id']) \
                .expire(rdb_key, expire_seconds) \
                .execute()
```

在研究和解決途中，因而進一步接觸 [celery signal](http://docs.celeryproject.org/en/latest/userguide/signals.html) 的章節，想要紀錄一下當時的原生想法和試驗結果。

已知 celery 的 Task Signals 提供類型如下：

* before_task_publish - sender 發佈 task 之前。
* after_task_publish - sender 發佈 task 之後。
* task_prerun - receiver 執行 task 之前。
* task_postrun - receiver 執行 task 之後。
* task_retry - receiver 執行 task 時，發生「重試」。
* task_success - receiver 執行 task 時，發生「成功」。
* task_failure - receiver 執行 task 時，發生「失敗」。
* task_revoked - receiver 執行 task 時，發生「註銷」。
* task_unknown - receiver 無法辨識。當 project 和 celery 的 task 產生落差時，會發生的狀況。
* task_rejected - 這個我不曉得。

我好奇 sender 和 receiver 的執行順序起見，於是我做了以下測試：

這是 sender 的程式碼。

```python
@after_task_publish.connect
def after_task_publish_test(sender=None, body=None, **kwargs):
    print '=====after_task_publish'
    print body
    print sender
    print time.sleep(1)
    print time.time()

@before_task_publish.connect
def before_task_publish_test(sender=None, body=None, **kwargs):
    print '=====before_task_publish'
    print body
    print sender
    print time.sleep(1)
    print time.time()

@task_prerun.connect
def task_prerun_test(sender=None, body=None, **kwargs):
    print '=====task_prerun'
    print body
    print sender
    print time.sleep(1)
    print time.time()

@task_postrun.connect
def task_postrun_test(sender=None, body=None, **kwargs):
    print '=====task_postrun'
    print body
    print sender
    print time.sleep(1)
    print time.time()
```

這是 sender 的 log 訊號。

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
```

這是 receiver 的 log 訊號。

```txt
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

由此可知，當 message queue 為空時，signal 的執行順序為：

before_task_publish -> after_task_publish_test -> task_prerun -> task_postrun

心得：

雖然這個問題只花我 2 天時間解決，但這個 fix 令我感到很唐突。我覺得不需要為了提供花俏的功能，把這個動作寫在 Backend 裏面，實在是很愚蠢的事情！而且，每次 celery task 完成，會將新的拼貼圖片上傳至 S3 上，當用戶有頻繁地操作時，勢必衍生 task 而無用的圖片不斷上傳，卻未意識這個功能會造成無謂的 IT 支出在這上面！

如果是我，當歌單的封面為空時，我會直接取得歌單內第 1 個有封面的歌曲為封面，這樣不就解決了嘛！真的是很無聊。
