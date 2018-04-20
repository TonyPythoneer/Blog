---
title: SNS 高併發訂閱 topic 測試案例
date: 2018-04-20 14:13:48
tags:
- "Program Language:Python"
---

當時想做一個「精準廣播」的功能，把符合資格的裝置名單，建立成一個 AWS 的 SNS topic 作為發送對象。
目前使用了 python 的 mutiplethread/mutipleporcess 和 celery，仍然不是很滿意。
結論是，如果使用 instance 的架構，一定無法應付大量客製化訂閱所造成的延遲，我覺得需要有個能瞬間 scale out 的架構來消耗大量任務，例如： serverless 來完成這件事情。

<!-- more -->

## 測試案例

1. 單機逐一發送
* 範例：
```py
for endpoint_arn in endpoint_arns:
    subscripbe(topic_arn, endpoint_arn)
```
* task 發送數目： 35715
* 訂閱耗時：139m44.731s
* 訂閱速度： 4.26 devices/sec
* 評論： 作為基準參考，了解本機到 SNS server 的 network time 耗時多久。


2. 單機使用 `threadpool`，process 開 10 個
* 範例：
```py
from multiprocessing.dummy import Pool
def func(endpoint_arn):
    subscripbe(topic_arn, endpoint_arn)

pool = Pool(10)
try:
    pool.map(func, notifications)
except Error as err:
    pass
pool.close()
pool.join()
```
* task 發送數目： 11968
* 訂閱耗時：139m44.731s
* 訂閱速度： 12.7 devices/sec
* 評論： 很快。但執行負擔會集中當下的 process。


3. 單機使用 `processpool`，process 開 10 個
* 範例：
```py
from multiprocessing import Pool
def func(endpoint_arn):
  subscripbe(topic_arn, endpoint_arn)

pool = Pool(10)
try:
  pool.map(func, notifications)
except Error as err:
  pass
pool.close()
pool.join()
```
* task 發送數目： 173
* 訂閱耗時： 1m1.183s
* 訂閱速度： 2.8 devices/sec
* 結論： 載入 context 給 process 和釋放 process 開銷太大，比單機逐一發送請求還差。


4. 單機使用 4 個 worker 的 celery，使用 group
* 範例：
```py
@task(ignore_result=True)
def subscribe_topic(topic_arn, endpoint_arn):
    client = AWSSNSClient()
    try:
        client.subscribe(topic_arn, endpoint_arn)
    except BotoClientError as err:
        print err

group_job = group([subscribe_topic.s(topic_arn, endpoint_arn) for endpoint_arn in endpoint_arns])
res = = group_job()
```
* task 發送數目：14150
* task publish celery 時間： 5s
* task publish celery 速度： 2830 task/s
* 訂閱耗時： 20m35s
* 訂閱速度： 11.457 devcies/sec
* 評論： 雖然 celery group 可以快速派且大量 subtask 利用空閒的 worker 執行任務。但是請注意有 task publish 的 latery 存在！


5. 單機使用 4 個 worker 的 celery，呼叫 chunks 設為 10，以 group 方式執行
* 範例：
```py
@task(ignore_result=True)
def subscribe_topic(topic_arn, endpoint_arn):
    client = AWSSNSClient()
    try:
        client.subscribe(topic_arn, endpoint_arn)
    except BotoClientError as err:
        print err

def func():
    chunk = subscribe_topic.chunks(((topic_arn, arn) for arn in some_endpoint_arns), 10)
    grp = chunk.group()
    res = grp()
    return res
```
* task 發送數目：1000
* task publish 時間： 18ms
* task publish 速度： 55.55 task/s
* 訂閱耗時： 1m11s
* 訂閱速度： 14.08 devices/sec
* 評論：
  相比平行發佈，celery 為了組裝合適大小的工作量作為 subtask，因此 task publish 這段期間延遲非常嚴重
  但訂閱速度提升原因是，每個 worker 在第一次初始化時，會耗損於建立 session 這段時間；後續的 subscribe 會重複利用這個 session 不斷發送。


6. 單機使用 4 個 worker 的 celery，呼叫 chunks 設為 128，以 group 方式執行
* 範例：
```py
@task(ignore_result=True)
def subscribe_topic(topic_arn, endpoint_arn):
    client = AWSSNSClient()
    try:
        client.subscribe(topic_arn, endpoint_arn)
    except BotoClientError as err:
        print err

def func():
    chunk = subscribe_topic.chunks(((topic_arn, arn) for arn in some_endpoint_arns), 128)
    grp = chunk.group()
    res = grp()
    return res
```
* task 發送數目：1000
* task publish 時間： 18ms
* task publish 速度： 55.55 task/s
* 訂閱耗時： 1m11s
* 訂閱速度： 14.08 devices/sec
* 評論： 跟上一個情境相比，沒有很大差異。但請注意 1 個 worker 需要消耗 128 個任務，需避免長期佔據 worker 的問題。


7. 單機使用 4 個 worker 的 celery，呼叫 chunks 設為 256，以 group 方式執行
* 範例：
```py
@task(ignore_result=True)
def subscribe_topic(topic_arn, endpoint_arn):
    client = AWSSNSClient()
    try:
        client.subscribe(topic_arn, endpoint_arn)
    except BotoClientError as err:
        print err

def func():
    chunk = subscribe_topic.chunks(((topic_arn, arn) for arn in some_endpoint_arns), 256)
    grp = chunk.group()
    res = grp()
    return res
```
* task 發送數目：1000
* task publish 時間： 14ms
* 訂閱耗時： 1m11s
* 訂閱速度： 15.15 devices/sec
* 評論： 跟上一個情境相比，沒有很大差異。但請注意 1 個 worker 需要消耗 256 個任務，需避免長期佔據 worker 的問題。


## 結論

如果使用 instance 的架構，一定無法應付大量客製化訂閱所造成的延遲，我覺得需要有個能瞬間 scale out 的架構來消耗大量任務，例如： serverless 來完成這件事情。


## 參考資源

參考文章：
* [celery 官方文件 - Canvas: Designing Workflows](http://docs.celeryproject.org/en/3.1/userguide/tasks.html)
* [Celery 踩坑记](http://skyrover.me/2017/08/13/Celery%20%E8%B8%A9%E5%9D%91%E8%AE%B0/)

實驗所用的硬體規格：
* Intel(R) Core(TM) i5-6500 CPU @ 3.20GHz
* MemTotal： 16 GB
