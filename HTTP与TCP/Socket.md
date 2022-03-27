Socket跟TCP/IP并没有必然的联系。Socket编程接口在设计的时候，就希望也能适应其他的网络协议。所以，Socket的出现只是可以更方便的使用TCP/IP协议栈而已，其对TCP/IP进行了抽象，形成了几个最基本的函数接口。比如create，listen，accept，connect，read和write等等。

**服务端：**

```js
const net = require('net');
const server = net.createServer();
server.on('connection', (client) => {
  client.write('Hi!\n'); // 服务端向客户端输出信息，使用 write() 方法
  client.write('Bye!\n');
  //client.end(); // 服务端结束该次会话
});
server.listen(9000);
```

服务监听9000端口  
下面使用命令行发送http请求和telnet

```js
$ curl http://127.0.0.1:9000
Bye!

$telnet 127.0.0.1 9000
Trying 192.168.1.21...
Connected to 192.168.1.21.
Escape character is '^]'.
Hi!
Bye!
Connection closed by foreign host.
```

## Socket长连接
所谓长连接，指在一个TCP连接上可以连续发送多个数据包，在TCP连接保持期间，如果没有数据包发送，需要双方发检测包以维持此连接(心跳包)，一般需要自己做在线维持。 短连接是指通信双方有数据交互时，就建立一个TCP连接，数据发送完成后，则断开此TCP连接。比如Http的，只是连接、请求、关闭，过程时间较短,服务器若是一段时间内没有收到请求即可关闭连接。其实长连接是相对于通常的短连接而说的，也就是长时间保持客户端与服务端的连接状态。  
通常的短连接操作步骤是：  
连接→数据传输→关闭连接；

而长连接通常就是：  
连接→数据传输→保持连接(心跳)→数据传输→保持连接(心跳)→……→关闭连接；

什么时候用长连接，短连接？  
长连接多用于操作频繁，点对点的通讯，而且连接数不能太多情况，。每个TCP连接都需要三步握手，这需要时间，如果每个操作都是先连接，再操作的话那么处理 速度会降低很多，所以每个操作完后都不断开，次处理时直接发送数据包就OK了，不用建立TCP连接。例如：数据库的连接用长连接， 如果用短连接频繁的通信会造成Socket错误，而且频繁的Socket创建也是对资源的浪费。


### **心跳包**
**实现：**  
服务端：
```js
const net = require('net');

let clientList = [];
const heartbeat = 'HEARTBEAT'; // 定义心跳包内容确保和平时发送的数据不会冲突

const server = net.createServer();
server.on('connection', (client) => {
  console.log('客户端建立连接:', client.remoteAddress + ':' + client.remotePort);
  clientList.push(client);
  client.on('data', (chunk) => {
    let content = chunk.toString();
    if (content === heartbeat) {
      console.log('收到客户端发过来的一个心跳包');
    } else {
      console.log('收到客户端发过来的数据:', content);
      client.write('服务端的数据:' + content);
    }
  });
  client.on('end', () => {
    console.log('收到客户端end');
    clientList.splice(clientList.indexOf(client), 1);
  });
  client.on('error', () => {
    clientList.splice(clientList.indexOf(client), 1);
  })
});
server.listen(9000);
setInterval(broadcast, 10000); // 定时发送心跳包
function broadcast() {
  console.log('broadcast heartbeat', clientList.length);
  let cleanup = []
  for (let i=0;i<clientList.length;i+=1) {
    if (clientList[i].writable) { // 先检查 sockets 是否可写
      clientList[i].write(heartbeat);
    } else {
      console.log('一个无效的客户端');
      cleanup.push(clientList[i]); // 如果不可写，收集起来销毁。销毁之前要 Socket.destroy() 用 API 的方法销毁。
      clientList[i].destroy();
    }
  }
  //Remove dead Nodes out of write loop to avoid trashing loop index
  for (let i=0; i<cleanup.length; i+=1) {
    console.log('删除无效的客户端:', cleanup[i].name);
    clientList.splice(clientList.indexOf(cleanup[i]), 1);
  }
}
```

客户端代码：

```js
const net = require('net');

const heartbeat = 'HEARTBEAT'; 
const client = new net.Socket();
client.connect(9000, '127.0.0.1', () => {});
client.on('data', (chunk) => {
  let content = chunk.toString();
  if (content === heartbeat) {
    console.log('收到心跳包：', content);
  } else {
    console.log('收到数据：', content);
  }
});

// 定时发送数据
setInterval(() => {
  console.log('发送数据', new Date().toUTCString());
  client.write(new Date().toUTCString());
}, 5000);

// 定时发送心跳包
setInterval(function () {
  client.write(heartbeat);
}, 10000);
```

## 定义自己的协议
如果想要使传输的数据有意义，则必须使用到应用层协议比如Http、Mqtt、Dubbo等。基于TCP协议上自定义自己的应用层的协议需要解决的几个问题：

1.  心跳包格式的定义及处理
2.  报文头的定义，就是你发送数据的时候需要先发送报文头，报文里面能解析出你将要发送的数据长度
3.  你发送数据包的格式，是json的还是其他序列化的方式


下面我们就一起来定义自己的协议，并编写服务的和客户端进行调用：  
定义报文头格式： `length:000000000xxxx`; xxxx代表数据的长度，总长度20,举例子不严谨。  
数据表的格式: Json  
服务端:

```js
const net = require('net');
const server = net.createServer();
let clientList = [];
const heartBeat = 'HeartBeat'; // 定义心跳包内容确保和平时发送的数据不会冲突
const getHeader = (num) => {
  return 'length:' + (Array(13).join(0) + num).slice(-13);
}
server.on('connection', (client) => {
  client.name = client.remoteAddress + ':' + client.remotePort
  // client.write('Hi ' + client.name + '!\n');
  console.log('客户端建立连接', client.name);

  clientList.push(client)
  let chunks = [];
  let length = 0;
  client.on('data', (chunk) => {
    let content = chunk.toString();
    console.log("content:", content, content.length);
    if (content === heartBeat) {
      console.log('收到客户端发过来的一个心跳包');
    } else {
      if (content.indexOf('length:') === 0){
        length = parseInt(content.substring(7,20));
        console.log('length', length);
        chunks =[chunk.slice(20, chunk.length)];
      } else {
        chunks.push(chunk);
      }
      let heap = Buffer.concat(chunks);
      console.log('heap.length', heap.length)
      if (heap.length >= length) {
        try {
          console.log('收到数据', JSON.parse(heap.toString()));
          let data = '服务端的数据数据:' + heap.toString();;
          let dataBuff =  Buffer.from(JSON.stringify(data));
          let header = getHeader(dataBuff.length)
          client.write(header);
          client.write(dataBuff);
        } catch (err) {
          console.log('数据解析失败');
        }
      }
    }
  })

  client.on('end', () => {
    console.log('收到客户端end');
    clientList.splice(clientList.indexOf(client), 1);
  });
  client.on('error', () => {
    clientList.splice(clientList.indexOf(client), 1);
  })
});
server.listen(9000);
setInterval(broadcast, 10000); // 定时检查客户端 并发送心跳包
function broadcast() {
  console.log('broadcast heartbeat', clientList.length);
  let cleanup = []
  for(var i=0;i<clientList.length;i+=1) {
    if(clientList[i].writable) { // 先检查 sockets 是否可写
      // clientList[i].write(heartBeat); // 发送心跳数据
    } else {
      console.log('一个无效的客户端')
      cleanup.push(clientList[i]) // 如果不可写，收集起来销毁。销毁之前要 Socket.destroy() 用 API 的方法销毁。
      clientList[i].destroy();
    }
  }
  // 删除无效的客户端
  for(i=0; i<cleanup.length; i+=1) {
    console.log('删除无效的客户端:', cleanup[i].name);
    clientList.splice(clientList.indexOf(cleanup[i]), 1)
  }
}
```

客户端
```js
const net = require('net');
const client = new net.Socket();
const heartBeat = 'HeartBeat'; // 定义心跳包内容确保和平时发送的数据不会冲突
const getHeader = (num) => {
  return 'length:' + (Array(13).join(0) + num).slice(-13);
}
client.connect(9000, '127.0.0.1', function () {});
let chunks = [];
let length = 0;
client.on('data', (chunk) => {
  let content = chunk.toString();
  console.log("content:", content, content.length);
  if (content === heartBeat) {
    console.log('收到服务端发过来的一个心跳包');
  } else {
    if (content.indexOf('length:') === 0){
      length = parseInt(content.substring(7,20));
      console.log('length', length);
      chunks =[chunk.slice(20, chunk.length)];
    } else {
      chunks.push(chunk);
    }
    let heap = Buffer.concat(chunks);
    console.log('heap.length', heap.length)
    if (heap.length >= length) {
      try {
        console.log('收到数据', JSON.parse(heap.toString()));
      } catch (err) {
        console.log('数据解析失败');
      }
    }
  }
});
// 定时发送数据
setInterval(function () {
  let data = new Date().toUTCString();
  let dataBuff =  Buffer.from(JSON.stringify(data));
  let header =getHeader(dataBuff.length);
  client.write(header);
  client.write(dataBuff);
}, 5000);
// 定时发送心跳包
setInterval(function () {
  client.write(heartBeat);
}, 10000);
```

## Socket连接池
什么是Socket连接池,池的概念可以联想到是一种资源的集合，所以Socket连接池，就是维护着一定数量Socket长连接的集合。它能自动检测Socket长连接的有效性，剔除无效的连接，补充连接池的长连接的数量。从代码层次上其实是人为实现这种功能的类，一般一个连接池包含下面几个属性：
1.  空闲可使用的长连接队列
2.  正在运行的通信的长连接队列
3.  等待去获取一个空闲长连接的请求的队列
4.  无效长连接的剔除功能
5.  长连接资源池的数量配置
6.  长连接资源的新建功能


## 参考文章：
https://segmentfault.com/a/1190000014044351