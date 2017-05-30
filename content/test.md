Title: Celery体验
Date: 2017-5-31 10:20
Category: python

#### Celery体验

Celery是Python开发的分布式任务调度模块,可以用它来实现异步发送邮件通知的服务。

##### 安装Celery
```
pip3 install Celery
```

##### 编写任务task.py
```
from celery import Celery
from sendNotifyByEmail import send_mail

celery = Celery('tasks', broker='redis://localhost:6379/0')


@celery.task
def sendmail(sub='系统出错了', content='bug'):
    send_mail(sub, content)
```

##### 启动Celery
```
celery -A tasks worker --loglevel=info
```

##### 测试发送服务
```
>>> from tasks import sendmail
>>> sendmail.delay(content='111')
<AsyncResult: 1fc52805-a972-4e94-99c1-21836ebff7c9>
```

##### 任务处理结果
```
[tasks]
  . tasks.sendmail

[2016-04-09 16:52:54,620: INFO/MainProcess] Connected to redis://localhost:6379/0
[2016-04-09 16:52:54,634: INFO/MainProcess] mingle: searching for neighbors
[2016-04-09 16:52:55,644: INFO/MainProcess] mingle: all alone
[2016-04-09 16:52:55,656: WARNING/MainProcess] celery@zhangzedeMacBook-Pro.local ready.
[2016-04-09 16:53:01,374: INFO/MainProcess] Received task: tasks.sendmail[1fc52805-a972-4e94-99c1-21836ebff7c9]
[2016-04-09 16:53:03,862: INFO/MainProcess] Task tasks.sendmail[1fc52805-a972-4e94-99c1-21836ebff7c9] succeeded in 2.485852560999774s: None
```

