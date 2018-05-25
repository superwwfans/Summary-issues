Celery + RabbitMQ + python3.6



celery配置

```python
# celery的消息代理使用rabbitmq
BROKER_URL = "amqp://guest:guest@localhost:5672/%2F"
# clery任务执行结果放入redis中
CELERY_RESULT_BACKEND = "redis://localhost:6379/1"
# 任务序列化和反序列化使用msgpack方案, 如果使用它会产生异常
CELERY_TASK_SERIALIZER = "json"
# 任务执行结果使用json格式
CELERY_RESULT_SERIALIZER  = "json"
# 任务的过期时间
CELERY_TASK_EXPIRES = 60 * 60 * 24
# 指定接受的的内容类型
CELRY_ACCEPT_CONTENT = ["json"]
```

启动后的信息

```
(demo_env3) huang@huang-VirtualBox:~/demo/clery学习$ celery -A celery_test worker -Q web_tasks -l info
 
 -------------- celery@huang-VirtualBox v4.1.0 (latentcall)
---- **** ----- 
--- * ***  * -- Linux-4.10.0-28-generic-i686-with-debian-stretch-sid 2018-05-19 20:54:57
-- * - **** --- 
- ** ---------- [config]
- ** ---------- .> app:         proj:0xb62a85ac
- ** ---------- .> transport:   amqp://guest:**@localhost:5672//
- ** ---------- .> results:     redis://localhost:6379/1
- *** --- * --- .> concurrency: 1 (prefork)
-- ******* ---- .> task events: OFF (enable -E to monitor tasks in this worker)
--- ***** ----- 
 -------------- [queues]
                .> web_tasks        exchange=web_tasks(direct) key=web_tasks
```









发生的异常

```python
[2018-05-19 20:24:40,320: INFO/MainProcess] Received task: celery_test.task.add[f111e7e4-740d-46ca-a53f-f0f2105e9bf3]  
[2018-05-19 20:24:40,326: ERROR/MainProcess] Pool callback raised exception: ContentDisallowed('Refusing to deserialize untrusted content of type msgpack (application/x-msgpack)',)
Traceback (most recent call last):
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/kombu/utils/objects.py", line 42, in __get__
    return obj.__dict__[self.__name__]
KeyError: 'chord'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/kombu/utils/objects.py", line 42, in __get__
    return obj.__dict__[self.__name__]
KeyError: '_payload'

During handling of the above exception, another exception occurred:

Traceback (most recent call last):
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/billiard/pool.py", line 1747, in safe_apply_callback
    fun(*args, **kwargs)
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/celery/worker/request.py", line 366, in on_failure
    self.id, exc, request=self, store_result=self.store_errors,
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/celery/backends/base.py", line 165, in mark_as_failure
    if request.chord:
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/kombu/utils/objects.py", line 44, in __get__
    value = obj.__dict__[self.__name__] = self.__get(obj)
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/celery/worker/request.py", line 485, in chord
    _, _, embed = self._payload
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/kombu/utils/objects.py", line 44, in __get__
    value = obj.__dict__[self.__name__] = self.__get(obj)
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/celery/worker/request.py", line 477, in _payload
    return self.body if self._decoded else self.message.payload
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/kombu/message.py", line 207, in payload
    return self._decoded_cache if self._decoded_cache else self.decode()
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/kombu/message.py", line 192, in decode
    self._decoded_cache = self._decode()
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/kombu/message.py", line 197, in _decode
    self.content_encoding, accept=self.accept)
  File "/home/huang/.pyenv/versions/3.6.5/envs/demo_env3/lib/python3.6/site-packages/kombu/serialization.py", line 253, in loads
    raise self._for_untrusted_content(content_type, 'untrusted')
kombu.exceptions.ContentDisallowed: Refusing to deserialize untrusted content of type msgpack (application/x-msgpack)
```

