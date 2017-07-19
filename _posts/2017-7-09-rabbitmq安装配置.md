## Installation

------

See the [downloads and installation](http://www.rabbitmq.com/download.html) page for information on the most recent release and how to install it

安装较新版本的rabbitmq需要更新erlang软件包
```
curl -s https://packagecloud.io/install/repositories/rabbitmq/erlang/script.rpm.sh | sudo bash
yum install erlang-18.3.4.5-1.el6.x86_64
```


## Get Started

------

[AMQP 0-9-1 Overview]为原始的RabbitMQ协议提供了一个简要的概述

RabbitMQ是一个中间件，用来接收，存储和转发二进制格式的数据(messages)。

messages可以用被存储进一个队列，队列受限于主机内存和磁盘限制，即一个大的消息缓冲，producers可以将信息发送到一个队列中，consumers可以尝试从一个队列中获取数据。

**Sending**

![mq-01.png](https://aaron-13.github.io/images/mq-01.png)


建立一个与RabbitMQ server的连接并发送一个简单的消息到队列中,可以根据实际修改对应的RabbitMQ Server的地址。创建一个hello的队列并传送到RabbitMQ Server中。

![mq-02.png](https://aaron-13.github.io/images/mq-02.png)

信息不会直接被传送至队列，总是需要通过一个exchange来传送。可以用一个空字符串来使用默认的exchange身份。这个exchange很特殊，可以允许我们指定消息需要发送到哪个队列。这个队列名需要在`routing_key`参数中被指定。

![mq-03.png](https://aaron-13.github.io/images/mq-03.png)

**Receiving**

```
List Queues

rabbitmqctl list_queues
```

![mq-04.png](https://aaron-13.github.io/images/mq-04.png)


完整的脚本

provider
```python
#!/usr/bin/python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.71.216'))
channel = connection.channel()

channel.queue_declare(queue='hello')

channel.basic_publish(exchange='',routing_kye='hello',body='hello world!')
print(" [X] Send 'hello world!'")

connection.close()

```

receiver

```python
#!/usr/bin/python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.71.216'))

channel = connection.channel()

channel.queue_declare(queue='hello')

def callback(ch,method,properties,body):
	print ("[x] Received %r" % body)

channel.basic_consume(callback,queue='hello',no_ack=True)

print('[*] Waiting for messages. To exit press Ctrl+C')

channel.start_consuming()
```

[其他语言](https://www.rabbitmq.com/tutorials/tutorial-one-python.html)



## Work Queues

------

创建一个工作队列用于分配给多个worker的消费任务。这个概念在web应用中很有用，它可以处理复杂的任务在一个短的http请求窗口中。

使用time.sleep()来假装繁忙

安排任务到工作队列
```python
#!/usr/bin/python

import sys
message = ''.join(sys.argv[1:]) or "hello world!"
channel.basic_publish(exchange='',
	routing_key='task_queue',
	body=message.properties=pika.BasicProperties(
	delivery_mode = 2,# make message persistent
	))
print("[x] Sent %r" % message)

```

查看rabbitmq中的队列信息：
`rabbitmqctl list_queues`

当RabbitMQ退出时，或者发生冲突时，会清除队列和信息。为保障信息不会丢失，我们需要将队列和信息都持久化。

队列持久化:
```
channel.queue_declare(queue='hello',durable=True)
```

信息持久化:
```
channel.basic_public(exchange='',
		routing_key="task_queue",
		body=message,
		properties=pika.BasicProperties(
			delivery_mode=2,#make message persistent
		))
```


new_task.py
```
#!/usr/bin/python

import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters('192.168.71.216'))
channel = connection.channel()

channel.queue_declare(queue="task_queue",durable=True)

message=''.join(sys.argv[1:]) or "hello world"
channel.basic_publish(exchange='',
	routing_key="task_queue",
	body=message,
	properties=pika.BasicProperties(
		delivery_mode = 2, # make message persistent
	))
print("[x] Sent %r" % message)

connection.close()
```


worker.py
```
#!/usr/bin/python

import pika
import time

connection = pika.BlockingConnection(pika.ConnectionParameters(host='192.168.71.216'))
channel = connection.channel()

channel.queue_declare(queue='task_queue',durable=True)
print('[*] waiting for message. To exit press Ctrl+C')

def callback(ch,method,properties,body):
	print('[x] received %r' % body)
	time.sleep(body.count(b'.'))
	print('[x] done')
	ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(callback,queue="task_queue")
channel.start_consuming()
```


## Publish/Subscribe

RabbitMQ消息模型：producer不会直接发送信息到一个队列，实际上，producer甚至不知道message被传递到哪个队列中。
producer只能将信息发送到一个exchange。exchange很简单，一遍从producer端接收信息，另一端将这些信息推送到队列中。
exchange必须精确地知道如何处理接收到的信息，如是将信息添加到一个队列中还是添加到众多的队列中，消息是否应该被丢弃，这些规则都被exchange type定义。

![mq-05.png](https://aaron-13.github.io/images/mq-05.png)

可用的exchange type: direct,topic,headers和fanout.

```
channel.exchange_declare(exchange='logs',type='fanout')
```

fanout这个exchange会广播所有接收到的信息给它所知的所有队列，这正是记录器所需要记录的。

可以通过`rabbitmqctl list_exchanges`命令来查看所有的exchanges。

默认的exchange：空字符串或者匿名的exchange，信息会被路由到routing_key所指定的队列。

```
channel.basic_publish(exchange='log',
	routing_key='',
	body=message)
```

**Temporary queues**

创建一个空的队列，队列名称为随机
```
result = channel.queue_declare()
```

result.method.queue包含了一个随机的队列名，看起来像amq.gen-JzTY20BRgKO-HjmUJj0wLg.

当关闭消费者consumer时，队列应该被删除。使用exclusive标签
```
result = channel.queue_declare(exclusive=True)
```


**Bindings**

![mq-06.png](https://aaron-13.github.io/images/mq-06.png)

exchange将信息发送到queue，它们这种关系叫做binding。

```
channel.queue_bind(exchange="log",queue=result.method.queue)
```

列出所有已经存在的binding`rabbitmqctl list_bindings`

![mq-07.png](https://aaron-13.github.io/images/mq-07.png)


emit_log.py
```
#!/usr/bin/python

import pika 
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(host="192.168.71.216"))
channel = connection.channel()

channel.exchange_declare(exchange="logs",type="fanout")
message=''.join(sys.argv[1:]) or "info:hello world"
channel.basic_publish(exchange="logs",
	routing_key='',
	body=message
	)
print("[x] sent %r" % message)

connection.close()
```


receive_logs.py
```
#!/usr/bin/python

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters(host="192.168.71.216"))
channel = connection.channel()

channel.exchange_declare(exchange="logs",type="fanout")

result = channel.queue_declare(exclusive=True)
queue.name = result.method.queue

channel.queue_bind(exchange="logs",queue=queue_name)

print("[*]waiting for logs")

def callback(ch,method,properties,body):
	print('[*] %r' % body)

channel.basic_consume(callback,queue=queue_name,no_ack=True)

channel.start_consuming()
```


**Routing**

bindings能使用额外一个routing_key参数，为了避免混淆basic_publish参数我们叫做binding key。
```
channel.queue_bind(exchange="logs",queue=queue_name,routing_key="black")
```

**Direct exchange**

使用fanout exchange，并不是很灵活，会进行无脑的广播。
使用direct exchange，信息会发送到binding key精确匹配对应的队列。binding key会精确匹配消息中含有routing key。

![mq-08.png](https://aaron-13.github.io/images/mq-08.png)

Multiple bindings
![mq-09.png](https://aaron-13.github.io/images/mq-09.png)

Emitting logs

使用routing key来记录日志严重级别
```
channel.exchange_declare(exchange="direct_logs",
	type='direct')

channel.basic_publish(exchange="direct_logs",
	routing_key=severity,
	body=message)
```

日志严重级别：info,warning,error



**Subscribing**
```
result = channel.queue_declare(exclusive=True)
queue_name = result.methdo.queue

for severity in severities:
	channel.queue_bind(exchange="direct_logs",
		queue=queue_name,
		routing_key=severity
		)
```



![mq-10.png](https://aaron-13.github.io/images/mq-10.png)

emit_log_direct.py
```
#!/usr/bin/python

import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(host="192.168.71.216"))
channel = connection.channel()

channel.exchange_declare(exchange="direct_logs",type='direct')

severity = sys.argv[1:] if len(sys.argv) > else 'info'
message = ''.join(sys.argv[1:]) or 'hello world'

channel.basic_publish(exchange="direct_logs",
	routing_key=severity,
	body=message)

print('[*] sent %r:%r' % (severity,message))
connection.close()
```


receive_logs_direct.py
```
#!/usr/bin/python

import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(host="192.168.71.216"))
channel = connection.channel()

channel.exchange_declare(exchange="direct_logs",type='direct')
result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue

severities = sys.argv[1:]
if not severities:
	sys.stderr.write("Useage: %s \[info\] \[warning\] [error]\n " % sys.argv[0])
	sys.exit(1)

for severity in severities:
	channel.queue_bind(exchange='direct_logs',
                       queue=queue_name,
                       routing_key=severity)

print(' [*] Waiting for logs. To exit press CTRL+C')

def callback(ch, method, properties, body):
    print(" [x] %r:%r" % (method.routing_key, body))

channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)

channel.start_consuming()
```


**Topic**

message使用topic exchange不能使用任意的routing_key,它必须是一个列表，以逗号分隔。可以使任意单词，通常连接到信息的一些特征，最大255个字节。
topic exchange像direct exchange一样，来绑定匹配的key。但对于binding_keys两种特殊情况：

*(start)可以替代任意一个词
#(hash)可以替代0个或任意多个单词

![mq-11.png](https://aaron-13.github.io/images/mq-11.png)

```
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='topic_logs',
                         type='topic')

routing_key = sys.argv[1] if len(sys.argv) > 2 else 'anonymous.info'
message = ' '.join(sys.argv[2:]) or 'Hello World!'
channel.basic_publish(exchange='topic_logs',
                      routing_key=routing_key,
                      body=message)
print(" [x] Sent %r:%r" % (routing_key, message))
connection.close()
```


receive_logs_topic.py
```
#!/usr/bin/env python
import pika
import sys

connection = pika.BlockingConnection(pika.ConnectionParameters(host='localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='topic_logs',
                         type='topic')

result = channel.queue_declare(exclusive=True)
queue_name = result.method.queue

binding_keys = sys.argv[1:]
if not binding_keys:
    sys.stderr.write("Usage: %s [binding_key]...\n" % sys.argv[0])
    sys.exit(1)

for binding_key in binding_keys:
    channel.queue_bind(exchange='topic_logs',
                       queue=queue_name,
                       routing_key=binding_key)

print(' [*] Waiting for logs. To exit press CTRL+C')

def callback(ch, method, properties, body):
    print(" [x] %r:%r" % (method.routing_key, body))

channel.basic_consume(callback,
                      queue=queue_name,
                      no_ack=True)

channel.start_consuming()
```

|  name  | Default | Descriptions |
| :---:  | :---:   | :---:        |
| RABBITMQ_NODE_IP_ADDRESS | the empty string(bind all network interfaces) | use the tcp_listeners key in rabbitmq.config | 
| RABBITMQ_NODE_PORT |　5672 |  |
| RABBITMQ_DIST_PORT | RABBITMQ_NODE_PORT + 20000 | 端口用来为内部节点和命令行使用，如果配置文件设置了kernel.inet_dist_listen_min or kernel.inet_dist_listen_max 键，此配置将被忽略 |
| RABBITMQ_NODENAME | Unix: rabbit@$HOSTNAME  Windows:rabbit@%COMPUTERNAME% | |
| RABBITMQ_CONF_ENV_FILE | $RABBITMQ_HOME/etc/rabbitmq/rabbitmq-env.conf | |
| RABBITMQ_USE_LONGNAME | | 如果设置为True，RabbitMQ会使用完整的主机名来确认节点。可能不会转换主机短名称和长名称在不重置的情况下 |
| RABBITMQ_SERVICENAME| Windows Service: RabbitMQ | |
| RABBITMQ_CONSOLE_LOG | Windows Service: | |
| RABBITMQ_CTL_ERL_ARGS | Unix: +P 1048576 +t 5000000 +stbt db +zdbb1 32000    Windows:None | 调用RabbitMQ Server的时候，使用erl命令的标准参数 |
| RABBITMQ_SERVER_DDITIONAL_ERL_ARGS | UNIX:None Windows:None | 调用RabbitMQ Server，使用erl命令额外添加的参数 如果+K true需要覆盖上面的参数|
| RABBITMQ_SERVER_START_ARGS | None | 不会覆盖ERL ARGS |


**The RabbitMQ config file**

网络连接配置
![mq-12.png](https://aaron-13.github.io/images/mq-12.png)


安全配置
![mq-13.png](https://aaron-13.github.io/images/mq-13.png)
允许guest用户从其他主机访问此服务，取消掉其注释，并去掉后面逗号

默认的VHOST设置
![mq-14.png](https://aaron-13.github.io/images/mq-14.png)

资源限制
![mq-15.png](https://aaron-13.github.io/images/mq-15.png)
disk默认存储为50MB

集群配置
![mq-16.png](https://aaron-13.github.io/images/mq-16.png)

Statistics Collection && advanced options
![mq-17.png](https://aaron-13.github.io/images/mq-17.png)

...




