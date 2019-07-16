---
layout: post
title:  "python局域网传输"
date:   2019-07-16 15:31:01 +0800
categories: python
tag: python
typora-root-url: ../../arslixiaojie.github.io
---

两台电脑办公，有时候需要互相传点东西，又不想用分享功能，在网上搜了一下，看到了可以用python来写一个简单的服务器。

```python
import SimpleHTTPServer
import socket
import SocketServer

def get_host_ip():
    ip = ""
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        s.connect(('8.8.8.8', 80))
        ip  = s.getsockname()[0]
    finally:
        s.close()

    return ip


if __name__ == "__main__":
    ip = get_host_ip()
    if ip == "" :
        print("can not get ip")
    else:
        handler = SimpleHTTPServer.SimpleHTTPRequestHandler
        httpd = SocketServer.TCPServer((ip, 8888), handler)
        print "http server is at: http://"+ip+":8888"
        httpd.serve_forever()
```



这样就可以通过ip和端口来访问到这个脚本所在的目录。