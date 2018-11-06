**参考文档：**

- [https://www.cnblogs.com/forward-wang/p/5970806.html](https://www.cnblogs.com/forward-wang/p/5970806.html)

- [http://www.cnblogs.com/shizhengwen/p/6911043.html](http://www.cnblogs.com/shizhengwen/p/6911043.html)

**环境：**

- python： 3.7

- celery： 4.2.1


## 什么是生产者与消费者模式

在实际的软件开发过程中，经常会碰到如下场景：某个模块负责产生数据，这些数据由另一个模块来负责处理（此处的模块是广义的，可以是类、函数、线程、进程等）。产生数据的模块，就形象地称为生产者；而处理数据的模块，就称为消费者。

生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过消息队列（缓冲区）来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给消息队列，消费者不找生产者要数据，而是直接从消息队列里取，消息队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。这个消息队列就是用来给生产者和消费者解耦的。

如图所示：

![image_1crk5j4k71nth1hg015vnko8jdd9.png-9.9kB](http://static.zybuluo.com/chuxiaoyi/zxy45sv0m24dmt85rkhdyubt/image_1crk5j4k71nth1hg015vnko8jdd9.png)

通过这样的模式，可以解决并发的问题，而celery就有效的利用这样一个模式的框架。

## 什么是celery

Celery 是一个基于python开发的分布式异步消息任务队列，通过它可以轻松的实现任务的异步处理。

任务队列的输入是称为任务的工作单元，专用工作进程不断监视任务队列以执行新工作。

celery通过消息进行通信，通常使用broker（消息队列）将客户端与worker解藕。客户端向broker中添加消息，然后broker将消息传递给worker。

celery可以有多个worker和broker，这实现了高可用和水平扩展。

如图所示：

![image_1crk6ab8k6b0mmsvvh1u4cve7m.png-15.1kB](http://static.zybuluo.com/chuxiaoyi/mbrw4uzxh97cv8tllr5awxt6/image_1crk6ab8k6b0mmsvvh1u4cve7m.png)

**celery的架构主要由三部分组成：**

- 消息队列（消息中间件）：celery本身无法接收或发送消息，这个时候就需要一个中间人帮celery来完成。这个中间人就是消息中间件，也就是上面说的broker。broker包括：[RabbitMQ](http://www.rabbitmq.com/documentation.html),[Redis](http://www.redis.net.cn/tutorial/3501.html),[MongoDB](https://www.mongodb.com/)等

- worker：执行任务的基本单元。一个celery可以有多个worker。worker运行在分布式的各个系统中。

- backend：用来存储任务的执行结果。celery支持很多种存储结构的方法：Redis，MongoDB，Django ORM，AMQP等

如图所示：

![image_1crk8hjg01a0ibd9mra5a1tel13.png-30.6kB](http://static.zybuluo.com/chuxiaoyi/togusggtlnyqudkyc1a9304l/image_1crk8hjg01a0ibd9mra5a1tel13.png)


## 快速入门

celery的架构分为三部分：broker，worker，backend

那么首先，需要选择一个适合我们的broker。这里会介绍rabbitMQ和redis作为broker的用法

- rabbitMQ

	- 安装

		```
		brew install rabbitmq
		```
	
		安装后的地址在：`/usr/local/Cellar/`

		将`/usr/local/Cellar/rabbitmq/3.7.8/sbin`添加到`.bash_profile`中，便于使用

		```
		export PATH=$PATH:/usr/local/Cellar/rabbitmq/3.7.8/sbin
		```

	- 启动与停止

		```
		sudo rabbitmq-server
		```
		![image_1crkbmffqatp134q14hj1qkn11c7m.png-31.6kB](http://static.zybuluo.com/chuxiaoyi/lmj4ijmlbm4tjmher44ili1n/image_1crkbmffqatp134q14hj1qkn11c7m.png)


		注意，rabbimq停止的时候不要用kill命令，要用下面这个：

		```
		sudo rabbitmqctl stop
		```



