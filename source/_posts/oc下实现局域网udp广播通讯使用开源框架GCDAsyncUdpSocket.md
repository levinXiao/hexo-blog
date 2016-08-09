---
title: oc下实现局域网udp广播通讯使用开源框架GCDAsyncUdpSocket
date: 2016-08-06 21:11:10
tags: [oc,socket,GCDAsyncUdpSocket,tcp]
categories: iOS
---
#UDP简介

UDP是OSI参考模型中一种无连接的传输层协议，它主要用于不要求分组顺序到达的传输中，分组传输顺序的检查与排序由应用层完成，提供面向事务的简单不可靠信息传送服务。

UDP 协议基本上是IP协议与上层协议的接口。UDP协议适用端口分别运行在同一台设备上的多个应用程序。udp协议通信在发送数据包的时候是不用绑定端口的，系统会自动分配端口，只有在发送消息的时候才需要绑定端口。

网络上已经有编写好的开源类库GCDAsyncSocket 和GCDAsyncUdpSocket 这是GCD版的 比AsyncSocket 和AsyncUdpSocket估计要好用点 用法也很简单，跟http很类似 只要指定服务器的ip和端口 然后再实现各种回调就行。

socket默认情况下就是采用TCP协议，创建之后通信双方的socket会一直保持连接，除非手动close或因为网络原因close，所以，此种状况对服务器而言是有一定资源消耗的，这种模式只适应与对服务器小规模的访问，特别是对于实时性很高的应用，如视频直播、呼叫系统等，而http一般都是短连接的，一次请求完之后客户端便会于服务端端开连接http是凌驾于socket之上的高级协议，而socket是比较底层的通讯方式，只是建立了一个连接通道，具体上面传输什么样的数据，按照什么格式传输，需要你自己定义，所以这就需要重新编写定义服务端与客户端的所应遵循的规定，而http已经被前人们定义使用过了。

#####[GCDAsyncUdpSocket ARC版下载地址(github)][udpsocket下载地址]

[udpsocket下载地址]: https://github.com/roustem/AsyncSocket"

#代码

初始化GCDAsyncUdpSocket类

共有四个初始化方法。

```

- (id)init;

- (id)initWithSocketQueue:(dispatch_queue_t)sq;

- (id)initWithDelegate:(id)aDelegate delegateQueue:(dispatch_queue_t)dq;

- (id)initWithDelegate:(id)aDelegate delegateQueue:(dispatch_queue_t)dq socketQueue:(dispatch_queue_t)sq;

```

在使用GCDAsyncUdpSocket传输数据前必须设置代理和代理队列，否则就会报错。

#####socketQueue是可选的，如果不设置系统会置为NULL，这样GCDAsyncUdpSocket会自动创建一个自己的socketqueue

#####代理队列 delegateQueue和socketQueue可以一样。

这里已经绑定了delegate 不用在类声明中重复实现这个代理了。

#消息发送

若客户端只向服务端发送消息而不用接收到其他的udp消息就可以不用绑定端口

##消息发送的方法

该方法只能用于已经连接了的Socket中

```

- (void)sendData:(NSData *)data withTimeout:(NSTimeInterval)timeout tag:(long)tag;

该方法不能用于已经连接了socket中，只能用于没有长连接的socket中

- (void)sendData:(NSData *)data toHost:(NSString *)host port:(uint16_t)port

withTimeout:(NSTimeInterval)timeout tag:(long)tag;

该方法不能用于已经连接了socket中，只能用于没有长连接的socket中

- (void)sendData:(NSData *)data toAddress:(NSData *)remoteAddr withTimeout:(NSTimeInterval)timeout tag:(long)tag;

```

##例子：

```

- (IBAction)sendToServer:(id)sender {

     NSDictionary *dic = [NSDictionary dictionaryWithObjectsAndKeys:@"my1",            
               @"my1",
              @"my2", @"扣篮大赛",
              @"my3", @"贷记卡",
              @"my4", @"我到了", nil];

    NSString *str = [[SBJson4Writer alloc]stringWithObject:dic];

    NSLog(@"%@",str);

    NSString *host = @"224.0.0.1";

    int port = 3339;

    NSData *data = [msg dataUsingEncoding:NSUTF8StringEncoding];

//host是在服务端设置的host，port也是服务端绑定的port，上文说过如果客户端不需要接收消息，就不用绑定端口
  
    [udpSocket sendData:data toHost:host port:port withTimeout:-1 tag:tag];

    NSLog(@"SENT (%i): %@", (int)tag, msg);

    tag++;

}

```

##端口绑定和广播开启

方法：

以下方法用于服务端，客户端可以跳过

```

//绑定端口

- (BOOL)bindToPort:(uint16_t)port error:(NSError **)errPtr;

- (BOOL)bindToPort:(uint16_t)port interface:(NSString *)interface error:(NSError **)errPtr;

//绑定到一个地址，可不设置

- (BOOL)bindToAddress:(NSData *)localAddr error:(NSError **)errPtr;

//绑定到一个HOST和端口

- (BOOL)connectToHost:(NSString *)host onPort:(uint16_t)port error:(NSError **)errPtr;

//开启广播，若不实现这个方法，默认是关闭的

- (BOOL)enableBroadcast:(BOOL)flag error:(NSError **)errPtr;

```

例子：

```

- (IBAction)startServer:(id)sender {

    int port = 3339;

    NSError *error = nil;

    if (![udpServer bindToPort:port error:&error]) {

        NSLog(@"Error starting server (bind): %@", error);

        return;

    }

    if (![udpServer enableBroadcast:YES error:&error]) {

        NSLog(@"Error enableBroadcast (bind): %@", error);

        return;

     }

    if (![udpServer joinMulticastGroup:@"224.0.0.1"  error:&error]) {

        NSLog(@"Error joinMulticastGroup (bind): %@", error);

        return;
  
    }

    if (![udpServer beginReceiving:&error]) {

        [udpServer close];

        NSLog(@"Error starting server (recv): %@", error);

        return;

    }

    NSLog(@"udp servers success starting %hd", [udpServer localPort]);

    isRunning =true;

}

```

####特别说明：

若客户端也要实现接收到服务器发送的消息，也必须实现上述代码，只是不需要实现enableBraodcast:error:这个方法，这样在客户端就可以接收到消息了

另外，客户端的端口号不需要与服务端的端口号保持一致。这时候，从另外一个角度来说，客户端变成了一个简易的服务端。

###代理方法 GCDAsyncUdpSocketDelegate

方法：

```

//在绑定address成功后回调

- (void)udpSocket:(GCDAsyncUdpSocket *)sock didConnectToAddress:(NSData *)address;

- (void)udpSocket:(GCDAsyncUdpSocket *)sock didNotConnect:(NSError *)error;

//发送消息后回调，不关心是否成功发送。

- (void)udpSocket:(GCDAsyncUdpSocket *)sock didSendDataWithTag:(long)tag;

- (void)udpSocket:(GCDAsyncUdpSocket *)sock didNotSendDataWithTag:(long)tag dueToError:(NSError *)error;

//在接收到消息后回调这个方法

- (void)udpSocket:(GCDAsyncUdpSocket *)sock didReceiveData:(NSData *)data

fromAddress:(NSData *)address

withFilterContext:(id)filterContext;

//在Socket连接关闭后回调

- (void)udpSocketDidClose:(GCDAsyncUdpSocket *)sock withError:(NSError *)error;

```

##例子：

服务端接收到消息：

```

-(void)udpSocket:(GCDAsyncUdpSocket *)sock didReceiveData:(NSData *)data fromAddress:(NSData *)address withFilterContext:(id)filterContext{

    NSLog(@"%@%d",[[NSString alloc]initWithData:data encoding:NSUTF8StringEncoding],[sock connectedPort]);

    [udpServer sendData:data toAddress:address withTimeout:-1 tag:0];

}

```

这里做了两件事情，第一件事情是将接收到的消息打印出来，第二件事情就是重新发送一个消息给来源接收。这样目的是是发送端知道接收端己收到消息。从而做出相应的处理。

#####[Demo下载(CSDN)][Demo下载地址]

[Demo下载地址]: http://download.csdn.net/detail/xiaoamani/7456259"