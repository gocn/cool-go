# kafka客户端 Shopify/sarama 库

## 1. sarama 是什么？

[sarama](https://github.com/Shopify/sarama) 的出现意味着，golang 有完整的apache kafka 客户端文档的全部用例了

## 2. 为什么那么受欢迎？

- 他完整，兼容性好
- 不需要CGO的支持
- 支持的golang版本从1.15到目前最新的1.16   
- 支持的kafka版本横跨2.6到2.8，并且之前发布的老版本依然支持

## 3. 怎么使用

下载扩展库 `go get github.com/Shopify/sarama` 即可。

1. 安装完首先在test中写一个队列任务发布者：

   kafka的集群，自己用docker在wsl下搭建就行了，网上例子非常多，这里就不详细解答了

   请看我代码逐行解释：

```go
package tester

import (
	"fmt"
	"github.com/Shopify/sarama"
	"log"
	"os"
	"testing"
	"time"
)

//集群地址
var address = []string{"192.168.83.89:19092","192.168.83.89:29092","192.168.83.89:39092"}

//创建kafka任务发布者
func TestKafkaProducter(t *testing.T) {
    //配置发布者
	config := sarama.NewConfig()
    //确认返回，记得一定要写，因为本次例子我用的是同步发布者
	config.Producer.Return.Successes = true
    //设置超时时间 这个超时时间一旦过期，新的订阅者在这个超时时间后才创建的，就不能订阅到消息了
	config.Producer.Timeout = 5 * time.Second
    //连接发布者，并创建发布者实例
	p, err := sarama.NewSyncProducer(address, config)
	if err != nil {
		log.Printf("sarama.NewSyncProducer err, message=%s \n", err)
		return
	}
    //程序退出时释放资源
	defer p.Close()
    //设置一个逻辑上的分区名，叫安彦飞
	topic := "anyanfei"
    //这个是发布的内容
	srcValue := "sync: this is a message. index=%d"
    //发布者循环发送0-9的消息内容
	for i:=0; i<10; i++ {
		value := fmt.Sprintf(srcValue, i)
        //创建发布者消息体
		msg := &sarama.ProducerMessage{
			Topic:topic,
			Value:sarama.ByteEncoder(value),
		}
        //发送消息并返回消息所在的物理分区和偏移量
		partition, offset, err := p.SendMessage(msg)
		if err != nil {
			log.Printf("send message(%s) err=%s \n", value, err)
		}else {
			fmt.Fprintf(os.Stdout, value + "发送成功，partition=%d, offset=%d \n", partition, offset)
		}
		time.Sleep(500*time.Millisecond)
	}
}
```

2. 我们开始来创建订阅者，俗称消费者：

   同样是在tester包下进行编写，测试用嘛（<u>**一定要注意，实际项目中不能用test，test默认超时时间是10分钟，如果有测试用例一直占用切超过10分钟会发生panic**</u>）

   ```go
   package tester
   
   import (
   	"context"
   	"fmt"
   	"github.com/Shopify/sarama"
   	"log"
   	"os"
   	"os/signal"
   	"strings"
   	"sync"
   	"syscall"
   	"testing"
   	"time"
   )
   
   ...发布者的代码块...
   
   func TestKafkaConsumer(t *testing.T) {
   	newKafkaConsumer()
   }
   
   //开始创建kafka订阅者
   func newKafkaConsumer(){
       /**
       	group:
   		 设置订阅者群 如果多个订阅者group一样，则随机挑一个进行消费，当然也可以设置轮训，在设置里面修改；
   		 若多个订阅者的group不同，则一旦发布者发布消息，所有订阅者都会订阅到同样的消息；
   		topics:
   		 逻辑分区必须与发布者相同，还是用安彦飞，不然找不到内容咯
   		 当然订阅者是可以订阅多个逻辑分区的，只不过因为演示方便我写了一个，你可以用英文逗号分割在这里写多个
       */
   	var (
   		group    = "Consumer1" 
   		topics   = "anyanfei"
   	)
   
   	log.Println("Starting a new Sarama consumer")
   	//配置订阅者
   	config := sarama.NewConfig()
       //配置偏移量
   	config.Consumer.Offsets.Initial = sarama.OffsetNewest
   	//开始创建订阅者
   	consumer := Consumer{
   		ready: make(chan bool),
   	}
   	//创建一个上下文对象，实际项目中也一定不要设置超时（当然，按你项目需求，我是没见过有项目需求要多少时间后取消订阅的）
   	ctx, cancel := context.WithCancel(context.Background())
       //创建订阅者群，集群地址发布者代码里已定义
   	client, err := sarama.NewConsumerGroup(address, group, config)
   	if err != nil {
   		log.Panicf("Error creating consumer group client: %v", err)
   	}
   	
       //创建同步组
   	var wg sync.WaitGroup
   	wg.Add(1)
   	go func() {
   		defer wg.Done()
   		for {
   			/**
                   官方说：`订阅者`应该在无限循环内调用
                   当`发布者`发生变化时
                   需要重新创建`订阅者`会话以获得新的声明
                   
                   所以这里把订阅者放在了循环体内
   			*/
   			if err := client.Consume(ctx, strings.Split(topics, ","), &consumer); err != nil {
   				log.Panicf("Error from consumer: %v", err)
   			}
   			// 检查上下文是否被取消，收到取消信号应当立刻在本协程中取消循环
   			if ctx.Err() != nil {
   				return
   			}
               //获取订阅者准备就绪信号
   			consumer.ready = make(chan bool)
   		}
   	}()
   	
   	<-consumer.ready // 获取到了订阅者准备就绪信号后打印下面的话
   	log.Println("Sarama consumer up and running!...")
   	
       //golang优雅退出的信号通道创建
   	sigterm := make(chan os.Signal, 1)
       //golang优雅退出的信号获取
   	signal.Notify(sigterm, syscall.SIGINT, syscall.SIGTERM)
   	//创建选择器，如果不是上下文取消或者用户ctrl+c这种系统级退出，则就不向下执行了
       select {
   	case <-ctx.Done():
   		log.Println("terminating: context cancelled")
   	case <-sigterm:
   		log.Println("terminating: via signal")
   	}
       //取消上下文
   	cancel()
   	wg.Wait()
       //关闭客户端
   	if err = client.Close(); err != nil {
   		log.Panicf("Error closing client: %v", err)
   	}
   }
   
   //重写订阅者，并重写订阅者的所有方法
   type Consumer struct {
   	ready chan bool
   }
   
   // Setup方法在新会话开始时运行的，然后才使用声明
   func (consumer *Consumer) Setup(sarama.ConsumerGroupSession) error {
   	// Mark the consumer as ready
   	close(consumer.ready)
   	return nil
   }
   
   // 一旦所有的订阅者协程都退出，Cleaup方法将在会话结束时运行
   func (consumer *Consumer) Cleanup(sarama.ConsumerGroupSession) error {
   	return nil
   }
   
   // 订阅者在会话中消费消息，并标记当前消息已经被消费。
   func (consumer *Consumer) ConsumeClaim(session sarama.ConsumerGroupSession, claim sarama.ConsumerGroupClaim) error {
   	for message := range claim.Messages() {
   		log.Printf("Message claimed: value = %s, timestamp = %v, topic = %s", string(message.Value), message.Timestamp, message.Topic)
   		session.MarkMessage(message, "")
   	}
   
   	return nil
   }
   ```

3. 下面开始执行：

   打开两个订阅者挂起：

   订阅者1：

   ```
   PS E:\go_code\src\go-web-demo\tester> go test -v -run TestKafkaConsumer
   === RUN   TestKafkaConsumer
   2021/06/20 16:41:51 Starting a new Sarama consumer
   2021/06/20 16:41:52 Sarama consumer up and running!...
   2021/06/20 16:41:59 Message claimed: value = sync: this is a message. index=0, timestamp = 2021-06-20 16:41:58.975 +0800 CST, topic = anyanfei
   2021/06/20 16:41:59 Message claimed: value = sync: this is a message. index=1, timestamp = 2021-06-20 16:41:59.56 +0800 CST, topic = anyanfei
   2021/06/20 16:42:00 Message claimed: value = sync: this is a message. index=2, timestamp = 2021-06-20 16:42:00.152 +0800 CST, topic = anyanfei
   2021/06/20 16:42:00 Message claimed: value = sync: this is a message. index=3, timestamp = 2021-06-20 16:42:00.698 +0800 CST, topic = anyanfei
   2021/06/20 16:42:01 Message claimed: value = sync: this is a message. index=4, timestamp = 2021-06-20 16:42:01.251 +0800 CST, topic = anyanfei
   2021/06/20 16:42:01 Message claimed: value = sync: this is a message. index=5, timestamp = 2021-06-20 16:42:01.793 +0800 CST, topic = anyanfei
   2021/06/20 16:42:02 Message claimed: value = sync: this is a message. index=6, timestamp = 2021-06-20 16:42:02.379 +0800 CST, topic = anyanfei
   2021/06/20 16:42:03 Message claimed: value = sync: this is a message. index=7, timestamp = 2021-06-20 16:42:02.929 +0800 CST, topic = anyanfei
   2021/06/20 16:42:03 Message claimed: value = sync: this is a message. index=8, timestamp = 2021-06-20 16:42:03.491 +0800 CST, topic = anyanfei
   2021/06/20 16:42:04 Message claimed: value = sync: this is a message. index=9, timestamp = 2021-06-20 16:42:04.03 +0800 CST, topic = anyanfei
   ```

   订阅者2：

   ```
   PS E:\go_code\src\go-web-demo\tester> go test -v -run TestKafkaConsumer
   === RUN   TestKafkaConsumer
   2021/06/20 16:41:23 Starting a new Sarama consumer
   2021/06/20 16:41:24 Sarama consumer up and running!...
   2021/06/20 16:41:59 Message claimed: value = sync: this is a message. index=0, timestamp = 2021-06-20 16:41:58.975 +0800 CST, topic = anyanfei
   2021/06/20 16:41:59 Message claimed: value = sync: this is a message. index=1, timestamp = 2021-06-20 16:41:59.56 +0800 CST, topic = anyanfei
   2021/06/20 16:42:00 Message claimed: value = sync: this is a message. index=2, timestamp = 2021-06-20 16:42:00.152 +0800 CST, topic = anyanfei
   2021/06/20 16:42:00 Message claimed: value = sync: this is a message. index=3, timestamp = 2021-06-20 16:42:00.698 +0800 CST, topic = anyanfei
   2021/06/20 16:42:01 Message claimed: value = sync: this is a message. index=4, timestamp = 2021-06-20 16:42:01.251 +0800 CST, topic = anyanfei
   2021/06/20 16:42:01 Message claimed: value = sync: this is a message. index=5, timestamp = 2021-06-20 16:42:01.793 +0800 CST, topic = anyanfei
   2021/06/20 16:42:02 Message claimed: value = sync: this is a message. index=6, timestamp = 2021-06-20 16:42:02.379 +0800 CST, topic = anyanfei
   2021/06/20 16:42:03 Message claimed: value = sync: this is a message. index=7, timestamp = 2021-06-20 16:42:02.929 +0800 CST, topic = anyanfei
   2021/06/20 16:42:03 Message claimed: value = sync: this is a message. index=8, timestamp = 2021-06-20 16:42:03.491 +0800 CST, topic = anyanfei
   2021/06/20 16:42:04 Message claimed: value = sync: this is a message. index=9, timestamp = 2021-06-20 16:42:04.03 +0800 CST, topic = anyanfei
   ```

   一开始是没有消息的，会阻塞在Sarama consumer up and running!...  

   直到我开启了发布者，才会产生上述订阅者的内容：

   ```
   PS E:\go_code\src\go-web-demo\tester> go test -v -run TestKafkaProducter
   sync: this is a message. index=0发送成功，partition=0, offset=12 
   sync: this is a message. index=1发送成功，partition=4, offset=13 
   sync: this is a message. index=2发送成功，partition=7, offset=12 
   sync: this is a message. index=3发送成功，partition=4, offset=14 
   sync: this is a message. index=4发送成功，partition=7, offset=13 
   sync: this is a message. index=5发送成功，partition=5, offset=8 
   sync: this is a message. index=6发送成功，partition=4, offset=15 
   sync: this is a message. index=7发送成功，partition=1, offset=9 
   sync: this is a message. index=8发送成功，partition=1, offset=10 
   sync: this is a message. index=9发送成功，partition=4, offset=16
   ```

   



## 总结

[Shopify/sarama](https://github.com/Shopify/sarama) 这个库用来做kafka的golang客户端库，我给你说，这真不戳~~~

以上所有内容均采用最新官方案例做示例

## 参考资料

- [https://github.com/Shopify/sarama](https://github.com/Shopify/sarama)