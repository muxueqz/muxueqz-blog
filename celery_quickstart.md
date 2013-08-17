Title: Celery快速上手
Date: 2013-08-17 13:20
Tags: python,redis,celery,server
Category: Python
Slug: celery_quickstart
Author: muxueqz
Summary: Celery快速上手

## 开始 Celery 的第一步

### 选择你的 Broker

在你正式开始使用 Celery 之前，你需要选择、安装并运行一个 broker。

Broker 是一种负责接收、发送任务消息（task messages）的服务

你可以从以下几种 broker 中选择一个：

 * RabbitMQ : [[http://www.rabbitmq.com/]]

功能完整、安全、饱经实践检验的 broker。如果对你而言，不丢失消息非常重要，RabbitMQ 将是你最好的选择。

请查看 [[/技术分享/Celery/broker-installation]] 以便获得更多关于 RabbitMQ 安装和配置相关的信息。

 * Redis : [[http://redis.io/]]

也是一个功能完整的 broker，但电源故障或异常将导致数据丢失。

请查看 [[/技术分享/Celery/otherqueues-redis]] 以便配置相关的信息。

 * 数据库

不推荐使用数据库作为消息队列，但在非常小的应用中可以使用。Celery 可以使用 SQLAlchemy 和 Django ORMS。请查看 [[/技术分享/Celery/otherqueues-sqlalchemy]] 或 [[/技术分享/Celery/otherqueues-django]]。

 * 更多其他选择。

除了上面列出的以外，还有其他可以选择的传输实现，例如 CouchDB, Beanstalk, MongoDB, and SQS。请查阅 Kombu 的文档以便获得更多信息。

### 安装Celery
    :::bash
    #安装celery
    $ pip install celery
    #安装时区的模块，不然会有时间慢8小时的问题
    $ pip install pytz

== 创建一个简单“任务”（Task） ==

在这个教程里，我们将创建一个简单的“任务”（Task） —— 把两个数加起来。通常，我们在 Python 的模块中定义“任务”。

按照惯例，我们将调用模块 `file`:`tasks.py`，看起来会像这个样子：

`file`:`tasks.py`
    :::python
    from celery.task import task

    @task
    def add(x, y):
        return x + y

此时， `@task` 装饰器实际上创建了一个继承自 :class:`~celery.task.base.Task` 的“类”（class）。除非需要修改“任务类”的缺省行为，否则我们推荐只通过装饰器定义“任务”（这是我们推崇的最佳实践）。

seealso:
    关于创建任务和任务类的完整文档可以在 `../userguide/tasks` 中找到。

== 配置 ==

Celery 使用一个配置模块来进行配置。这个模块缺省北命名为 :file:`celeryconfig.py`。

为了能被 import，这个配置模块要么存在于当前目录，要么包含在 Python 路径中。

同时，你可以通过使用环境变量 `CELERY_CONFIG_MODULE` 来随意修改这个配置文件的名字。

现在来让我们创建配置文件 `celeryconfig.py`.

 1. 配置如何连接 broker（例子中我们使用 RabbitMQ）:
        BROKER_URL = "amqp:''guest:guest@localhost:5672''"
 2. 定义用于存储元数据（metadata）和返回值（return values）的后端:
        CELERY_RESULT_BACKEND = "amqp"

   AMQP 后端缺省是非持久化的，你只能取一次结果（一条消息）。

   可以阅读 :ref:`conf-result-backend` 了解可以使用的后端清单和相关参数。

 3. 最后，我们列出 worker 需要 import 的模块，包括你的任务。

   我们只有一个刚开始添加的任务模块 :file:`tasks.py`::

        CELERY_IMPORTS = ("tasks", )

这就行了。

你还有更多的选项可以使用，例如：你期望使用多少个进程来并行处理（:setting:`CELERY_CONCURRENCY` 设置），或者使用持久化的结果保存后端。可以阅读 :ref:`configuration` 查看更多的选项。

note:

    你可以也使用

        $ celery -A tasks worker --loglevel=info

== 运行 worker 服务器 ==

为了方便测试，我们将在前台运行 worker 服务器，这样我们就能在终端上看到 celery 上发生的事情:

{{{
#!sh
    $ celeryd --loglevel=INFO
}}}

在生产环境中，也许你希望将 worker 在后台以守护进程的方式运行。如果你希望这么做，你可以利用平台或者类似于 `supervisord`_ (查阅 :ref:`daemonizing` 以获得更多信息） 的工具来实现。

可以通过下列命令行获得完整的命令参数清单:

{{{
#!sh
    $  celeryd --help
}}}

supervisord: [[http://supervisord.org]]

== 执行任务（task） ==

我们通过调用 class 类的 `~celery.task.base.Task.delay` 方法执行任务。

`~celery.task.base.Task.apply_async` 方法一个非常方便的方法，通过这个方法我们可以充分控制控制任务执行的参数（参见 :ref:`guide-executing`）。

{{{
#!python
>>> from tasks import add
>>> add.delay(4, 4)
<AsyncResult: 889143a6-39a2-4e52-837b-d80d33efb22d>
}}}

此时，任务已经被发送到了消息 broker。直到有 worker 服务器取走并执行了这个任务，否则 Broker 将一直保存这个消息。

现在我们可以使用任务返回类 `~celery.result.AsyncResult` 来查看 worker 的日志，看看到底发生了什么。如果你配置了一个结果存储类 `~celery.result.AsyncResult` 来保存任务状态，任务执行完毕可获得返回值；任务执行失败则可获得异常/回调等信息。

== Shell里执行任务 ==
{{{
#!sh
#add 为任务名
$ celery call add
}}}

== 保留结果 ==

如果你想跟踪任务状态，Celery 就需要把这些状态信息发送并存储到某处。你可以使用多种后端来达到这个目的: SQLAlchemy/Django ORM, Memcached, Redis,AMQP, MongoDB, Tokyo Tyrant 和 Redis —— 甚至你可以定义你自己的。

在这里，我们将使用 `amqp` 作为保存结果的后端来存储状态消息。可以通过 {{{CELERY_RESULT_BACKEND}}} 来修改后端选项，你可以用于保存结果后端的选项有:

{{{
#!python
CELERY_RESULT_BACKEND = "amqp"
#: 我们希望结果最多保存 5 分钟。 
#: 注意，这个特性需要 2.1.1 以上版本 RabbitMQ 才能支持。
#: 如果你使用的是早期版本的 RabbitMQ，请注释下面这行。
CELERY_TASK_RESULT_EXPIRES = 300
}}}

请通过阅读 Celery/任务结果后端 获得更多相关配置的信息。

现在，我们配置好了保存结果的后端，让我们重新来执行任务。我们再次使用类 `~celery.result.AsyncResult`:

{{{
#!python
>>> result = add.delay(4, 4)
}}}

这里是一些你可以如何处理结果的方法:

{{{
#!python
>>> result.ready() # 任务是否已经执行，如果已经执行完毕，返回值为 True。
False
>>> result.result # 任务还没完成，尚无返回值。
None
>>> result.get()   # 等待，直到任务执行完毕并返回值。
8
>>> result.result # 直接返回结果，不产生任何错误。
8
>>> result.successful() # 任务是否成功执行，如果成功则返回值 True。
True
}}}

如果任务执行过程中发生了异常，`result.successful()` 的返回值将会变成 `False`，同时 `result.result` 将包含一个由 task 产生的异常实例。