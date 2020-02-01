## `RabbitMQ`

`RabbitMQ`可以实现完全独立的程序之间的通讯。在多进程中我们知道了`queue`可以用于进程之间的通讯，但是那只限于父进程与子进程，同一个父进程的子进程之间的通信，不能实现完全独立的进程之间的通讯（比如主机A的a进程想和主机B的b进程通讯）。而`RabbitMQ`作为一个通讯的中间人可以帮我们解决上述问题。

1.安装`erlang` 语言，`RabbitMQ`为`erlang`语言开发

2.安装`RabbitMQ`

### 3.`RabbitMQ`的一些小栗子

相关模块`pika`(其实还有几个，我先入个门就暂时先用这个模块...)

#### `example` 1

```python
# producer
import pika

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
ch = conn.channel()
ch.queue_declare(queue='hello_1')
ch.basic_publish(
    exchange='',
    routing_key='hello_1',
    body='hello rabbitMQ'
)
print('send hello rabbitMQ to consumer!')
conn.close()
```

```python
# consumer
import pika

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
ch = conn.channel()
ch.queue_declare(queue='hello_1')

def callback(ch,method,properties,body):
    print("receive from producer %r" % body)

ch.basic_consume(
    queue='hello_1',
    on_message_callback=callback,
    auto_ack=True
)

print('waiting for messages ...')
ch.start_consuming()
```

#### `example` 2

**消息的持久化**

**目的：** 如果服务端宕机，消息不会消失。

```python
# producer
import pika

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
ch = conn.channel()
ch.queue_declare(queue='hello_2',durable=True)  # make the queue durable
ch.basic_publish(
    exchange='',
    routing_key='hello_2',
    body='hello rabbitMQ',
    properties=pika.BasicProperties(           # make the messages persistent
        delivery_mode=2
    )
)
print('send hello rabbitMQ to consumer!')
conn.close()

# consumer
import pika

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
ch = conn.channel()
ch.queue_declare(queue='hello_2',durable=True)     # make the queue durable

def callback(ch,method,properties,body):
    print("receive from producer %r" % body)

ch.basic_consume(
    queue='hello_2',
    on_message_callback=callback,
    auto_ack=True
)

print('waiting for messages ...')
ch.start_consuming()
```

#### `example` 3

以生产者消费者模型为例，当有多个消费者时，生产者会公平的将任务发给消费者，也就是说每一个消费者得到一个任务的机会是均等的，但这存在一个问题，消费者之间的任务处理能力不同。处理能力强的消费者将处于一个非常轻松的状态，而处理能力弱的消费者，则一直处于高负荷状态。这显然不是我们想要的，所以需要针对消费者的任务处理能力给出合理的权重，以此来发放任务。`python`实现这一点非常简单，只需在消费者一端添加一句代码即可

`channel.basic_qos(prefetch_count=1)`

这句话的意思是告诉`RabbitMQ`，我的任务还没又处理完，不要给我发任务

```python
# consumer
import pika
import time

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
ch = conn.channel()
ch.queue_declare(queue='hello_3',durable=True)     # make the queue durable

def callback(ch,method,properties,body):
    print("receive from producer %r" % body)
    time.sleep(9)     # simulate task
    print('done ...')
    ch.basic_ack(delivery_tag = method.delivery_tag)   

ch.basic_qos(prefetch_count=1)    ##################

ch.basic_consume(
    queue='hello_3',
    on_message_callback=callback
    # auto_ack=True
)

print('waiting for messages ...')
ch.start_consuming()
```

#### `example`  4

用`RabbitMQ`实现一个程序的广播...有点像收音机（实时的）

`exchange type:fanout` 最普通的广播

```python
# producer 
import pika

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = conn.channel()

#  declare a exchange
channel.exchange_declare(
    exchange='test',
    exchange_type='fanout'      
)

channel.basic_publish(
    exchange='test',
    routing_key='',
    body='can you hear me ?'
)

print('message send out ...')
conn.close()
```

```python
# consumer 
import pika

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = conn.channel()
channel.exchange_declare(
    exchange='test',
    exchange_type='fanout'
)
# random queue
# random name use '' pass to the positional argument:'queue'
temp = channel.queue_declare('',exclusive=True)
random_queue = temp.method.queue
# print(random_queue)

channel.queue_bind(
    exchange='test',
    queue=random_queue
)
print('waiting for test ...')
def callback(ch,method,properties,body):
    print(ch)
    print(method)
    print(properties)
    print('get msg:',body)

channel.basic_consume(
    queue=random_queue,
    on_message_callback=callback,
    auto_ack=False
)
channel.start_consuming()
```

`exchange type:dirct`  选择性的接收一些消息

```python
# producer
import pika
import sys

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = conn.channel()

key_word =  sys.argv[1] if len(sys.argv) > 1 else 'info'
print(sys.argv[0])  ########  test
msg = ' '.join(sys.argv[2:]) or 'hello world !'

#  declare a exchange
channel.exchange_declare(
    exchange='test2',
    exchange_type='direct'
)

channel.basic_publish(
    exchange='test2',
    routing_key=key_word,
    body=msg
)

print('message send out ...')
conn.close()

##################################################################################

# consumer
import pika
import sys
conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = conn.channel()
channel.exchange_declare(
    exchange='test2',
    exchange_type='direct'
)

# random queue
# random name use '' pass to the positional argument:'queue'
temp = channel.queue_declare('',exclusive=True)
random_queue = temp.method.queue
# print(random_queue)
key_words = sys.argv[1:]
if not key_words:
    print('#' * 30)
    print('input your key word ...')
    sys.exit(1)

for key_word in key_words:
    channel.queue_bind(
        exchange='test2',
        queue=random_queue,
        routing_key=key_word
    )

print('waiting for test ...')

def callback(ch,method,properties,body):
    print(ch)
    print(method)
    print(properties)
    print('get msg:',body)

channel.basic_consume(
    queue=random_queue,
    on_message_callback=callback,
    auto_ack=False
)

channel.start_consuming()
```

`exchange type:topic`  更有选择性的接收一些消息 ... (支持关键字的模糊匹配感觉很好用)

​		`ps:` # 匹配所用关键字

```python
# producer
import pika
import sys

conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = conn.channel()

key_word =  sys.argv[1] if len(sys.argv) > 1 else '.info'
print(sys.argv[0])  ########  test
msg = ' '.join(sys.argv[2:]) or 'hello world !'

#  declare a exchange
channel.exchange_declare(
    exchange='test3',
    exchange_type='topic'     ############################
)

channel.basic_publish(
    exchange='test3',
    routing_key=key_word,
    body=msg
)

print('message send out ...')
conn.close()

##################################################################################

# consumer
import pika
import sys
conn = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = conn.channel()
channel.exchange_declare(
    exchange='test3',
    exchange_type='topic'         #########################
)

# random queue
# random name use '' pass to the positional argument:'queue'
temp = channel.queue_declare('',exclusive=True)
random_queue = temp.method.queue
# print(random_queue)
key_words = sys.argv[1:]
if not key_words:
    print('#' * 30)
    print('input your key word ...')
    sys.exit(1)

for key_word in key_words:
    channel.queue_bind(
        exchange='test3',
        queue=random_queue,
        routing_key=key_word
    )

print('waiting for test ...')

def callback(ch,method,properties,body):
    print(ch)
    print(method)
    print(properties)
    print('get msg:',body)

channel.basic_consume(
    queue=random_queue,
    on_message_callback=callback,
    auto_ack=False
)

channel.start_consuming()
```

