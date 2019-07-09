## iOS Object-C UDP 简易交互

懒人理解：socket通信

                    1）服务端：我就管收，收到算收不到拉倒，我不管你是谁。
                    
                    2）客户端：我就管发，成功与否我不管，我管你你谁。
                    
简易代码：

**1）服务端：**

```
- (void)udpServer {
  int serverSockerId = -1;
  ssize_t len = -1;
  socklen_t addrlen;
  char buff[1024];
  char k[1024];
  struct sockaddr_in ser_addr;
  
  // 创建socket
  serverSockerId = socket(AF_INET, SOCK_DGRAM, 0);
  if(serverSockerId < 0) {
    // 建立服务套接字 失败
  }
  addrlen = sizeof(struct sockaddr_in);
  bzero(&ser_addr, addrlen);
  ser_addr.sin_family = AF_INET;
  ser_addr.sin_addr.s_addr = htonl(INADDR_ANY);
  // 端口号
  ser_addr.sin_port = htons(123456);
  // 绑定端口号
  if(bind(serverSockerId, (struct sockaddr *)&ser_addr, addrlen) < 0) {
    // 服务器连接套接字失败
  }
  
  do {
    bzero(buff, sizeof(buff));
    
    // 接收客户端的消息
    len = recvfrom(serverSockerId, buff, sizeof(buff), 0, (struct sockaddr *)&ser_addr, &addrlen);
    // 显示客户端的网络地址
    NSLog(@"receive from %s\n", inet_ntoa(ser_addr.sin_addr));
    // 显示客户端发来的字符串
    NSLog(@"recevce:%s", buff);
    
    // 将接收到的客户端发来的消息，发回客户端
    // 将字串返回给client端
    sprintf(k, "%s", "Good Morning!");
    sendto(serverSockerId, k, len, 0, (struct sockaddr *)&ser_addr, addrlen);
  } while (strcmp(buff, "exit") != 0);
  
  NSLog(@"end loop");
  // 关闭socket
  close(serverSockerId);
}
```

**2）客户端：**

```
- (void)udpClient {
  int clientSocketId;
  ssize_t len;
  socklen_t addrlen;
  struct sockaddr_in client_sockaddr;
  char buffer[256] = "Hello, server, how are you?";
  
  // 创建Socket
  clientSocketId = socket(AF_INET, SOCK_DGRAM, 0);
  if(clientSocketId < 0) {
    // 创建失败
    return;
  }
  
  addrlen = sizeof(struct sockaddr_in);
  bzero(&client_sockaddr, addrlen);
  client_sockaddr.sin_family = AF_INET;
  client_sockaddr.sin_addr.s_addr = inet_addr("169.254.8.184");
  client_sockaddr.sin_port = htons(150431);
  // 循环发100个 
  int count = 100;
  do {
    bzero(buffer, sizeof(buffer));
    char xy[1024] = "Hello";
    xy[5] = count;
    sprintf(buffer, "%s%d", "Msg",count);
    
    // 发送消息到服务端
    // 注意:UDP是面向无连接的，因此不用调用connect()
    // 将字符串传送给server端
    len = sendto(clientSocketId, buffer, sizeof(buffer), 0, (struct sockaddr *)&client_sockaddr, addrlen);
    if (len > 0) {
      // 发送成功
    } else {
      // 发送失败
    }
    
    // 接收来自服务端返回的消息
    bzero(buffer, sizeof(buffer));
    count--;
    sleep(0.1);
  } while (count >= 0);
  // 关闭socket
  // 由于是面向无连接的，消息发出处就可以了，不用管它收不收得到，发完就可以关闭了
  close(clientSocketId);
}
```
