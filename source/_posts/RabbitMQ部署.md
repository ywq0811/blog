---
title: RabbitMQ部署和使用文档
categories:
  - Docker
tags:
  - rabbitMQ
toc: true # 是否启用内容索引
---

## RabbitMQ部署和使用文档

#### 1.拉取镜像

```go
docker pull rabbitmq
```

#### 2.启动服务

```go
docker run -d --restart always --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:latest
```

#### 3.简单的go实现发送和接收消息代码

`Sned.go`

```go
func main() {
	// 1. 尝试连接RabbitMQ，建立连接
	// 该连接抽象了套接字连接，并为我们处理协议版本协商和认证等。
	conn, err := amqp.Dial("amqp://guest:guest@ip:port/")
	if err != nil {
		fmt.Printf("connect to RabbitMQ failed, err:%v\n", err)
		return
	}
	defer conn.Close()

	// 2. 创建一个通道，大多数API都是用过该通道操作的。
	ch, err := conn.Channel()
	if err != nil {
		fmt.Printf("open a channel failed, err:%v\n", err)
		return
	}
	defer ch.Close()

	// 3. 声明消息要发送到的队列
	q, err := ch.QueueDeclare(
		"task_queue", // name
		false,         // 持久的
		false,        // delete when unused
		false,        // 独有的
		false,        // no-wait
		nil,          // arguments
	)
	if err != nil {
		fmt.Printf("declare a queue failed, err:%v\n", err)
		return
	}

	// 4. 将消息发布到声明的队列
	body := "Hello World!" //发送的消息
	err = ch.Publish(
		"",     // exchange
		q.Name, // routing key
		false,  // 立即
		false,  // 强制
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(body),
		})
	if err != nil {
		fmt.Printf("publish a message failed, err:%v\n", err)
		return
	}
	log.Printf(" [x] Sent %s", body)
}
```

`receive.go`

```go
func main() {
	//1.建立连接	
	conn, err := amqp.Dial("amqp://guest:guest@ip:port/")
	if err != nil {
		fmt.Printf("connect to RabbitMQ failed, err:%v\n", err)
		return
	}
	defer conn.Close()
	
    //2.获取channel
	ch, err := conn.Channel()
	if err != nil {
		fmt.Printf("open a channel failed, err:%v\n", err)
		return
	}
	defer ch.Close()

	//3.声明队列
	q, err := ch.QueueDeclare(
		"task_queue", // name
		false,         // 声明为持久队列
		false,        // delete when unused
		false,        // exclusive
		false,        // no-wait
		nil,          // arguments
	)
	if err != nil {
		fmt.Printf("ch.Qos() failed, err:%v\n", err)
		return
	}

	//4.获取接收消息的delivery通道
	msgs, err := ch.Consume(
		q.Name, // queue
		"",     // consumer
		true,  // auto-ack, 如果是false,关闭自动消息确认
		false,  // exclusive
		false,  // no-local
		false,  // no-wait
		nil,    // args
	)
	if err != nil {
		fmt.Printf("ch.Consume failed, err:%v\n", err)
		return
	}

	forever := make(chan bool)
	go func() {
		for d := range msgs {
			log.Printf("Received a message: %s", d.Body)
		}
	}()

	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
}    
```

