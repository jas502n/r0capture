## 改动的代码


https://github.com/BigFaceCat2017/frida_ssl_logger/blob/master/script.js
https://raw.githubusercontent.com/r0ysue/r0capture/main/script.js

```

if (Java.available) {
  Java.perform(function () {
    Java.use("java.net.SocketOutputStream").socketWrite0.overload('java.io.FileDescriptor', '[B', 'int', 'int').implementation = function (fd, bytearry, offset, byteCount) {
      var result = this.socketWrite0(fd, bytearry, offset, byteCount);
      var message = {};
      message["function"] = "HTTP_send";
      message["ssl_session_id"] = "";
      message["src_addr"] = ntohl(ipToNumber((this.socket.value.getLocalAddress().toString().split(":")[0]).split("/").pop()));
      message["src_port"] = parseInt(this.socket.value.getLocalPort().toString());
      message["dst_addr"] = ntohl(ipToNumber((this.socket.value.getRemoteSocketAddress().toString().split(":")[0]).split("/").pop()));
      message["dst_port"] = parseInt(this.socket.value.getRemoteSocketAddress().toString().split(":").pop());
      var ptr = Memory.alloc(byteCount);
      for (var i = 0; i < byteCount; ++i)
        Memory.writeS8(ptr.add(i), bytearry[offset + i]);
      send(message, Memory.readByteArray(ptr, byteCount))
      return result;
    }
    Java.use("java.net.SocketInputStream").socketRead0.overload('java.io.FileDescriptor', '[B', 'int', 'int', 'int').implementation = function (fd, bytearry, offset, byteCount, timeout) {
      var result = this.socketRead0(fd, bytearry, offset, byteCount, timeout);
      var message = {};
      message["function"] = "HTTP_recv";
      message["ssl_session_id"] = "";
      message["src_addr"] = ntohl(ipToNumber((this.socket.value.getRemoteSocketAddress().toString().split(":")[0]).split("/").pop()));
      message["src_port"] = parseInt(this.socket.value.getRemoteSocketAddress().toString().split(":").pop());
      message["dst_addr"] = ntohl(ipToNumber((this.socket.value.getLocalAddress().toString().split(":")[0]).split("/").pop()));
      message["dst_port"] = parseInt(this.socket.value.getLocalPort());
      if (result > 0) {
        var ptr = Memory.alloc(result);
        for (var i = 0; i < result; ++i)
          Memory.writeS8(ptr.add(i), bytearry[offset + i]);
        send(message, Memory.readByteArray(ptr, result))
      }
      return result;
    }
  })
}

```

# r0capture

安卓应用层抓包通杀脚本

## 简介

- 仅限安卓平台，测试安卓7、8、9、10 可用 ；
- 无视所有证书校验或绑定，不用考虑任何证书的事情；
- 通杀TCP/IP四层模型中的应用层中的全部协议；
- 通杀协议包括：Http,WebSocket,Ftp,Xmpp,Imap,Smtp,Protobuf等等、以及它们的SSL版本；
- 通杀所有应用层框架，包括HttpUrlConnection、Okhttp1/3/4、Retrofit/Volley等等；
- 如果有抓不到的情况欢迎提issue，或者直接加vx：r0ysue，进行反馈~

## 用法

- Spawn 模式：

`$ python3 r0capture.py -U -f com.qiyi.video`

- Attach 模式，抓包内容保存成pcap文件供后续分析：

`$ python3 r0capture.py -U com.qiyi.video -p iqiyi.pcap`

建议使用`Attach`模式，从感兴趣的地方开始抓包，并且保存成`pcap`文件，供后续使用Wireshark进行分析。

![](Sample.PNG)



PS：

> 这个项目基于[frida_ssl_logger](https://github.com/BigFaceCat2017/frida_ssl_logger)，之所以换个名字，只是侧重点不同。

> 原项目的侧重点在于抓ssl和跨平台，本项目的侧重点是抓到所有的包。

## 以下是原项目的简介：

[https://github.com/BigFaceCat2017/frida_ssl_logger](https://github.com/BigFaceCat2017/frida_ssl_logger)

### frida_ssl_logger
ssl_logger based on frida
for from https://github.com/google/ssl_logger

### 修改内容
1. 优化了frida的JS脚本，修复了在新版frida上的语法错误；
2. 调整JS脚本，使其适配iOS和macOS，同时也兼容了Android；
3. 增加了更多的选项，使其能在多种情况下使用；

### Usage
  ```shell
    python3 ./ssl_logger.py  -U -f com.bfc.mm
    python3 ./ssl_logger.py -v  -p test.pcap  6666
  ````
