## nginx 是什么？

> “Nginx是一款轻量级的 HTTP 服务器，采用事件驱动的异步非阻塞处理方式框架，这让其具有极好的 IO 性能，时常用于服务端的反向代理和负载均衡。”

作为一个没怎么接触过服务端的前端开发，之前对 Nginx 的了解只限于官方的这段介绍。而且“HTTP服务器”、“事件驱动”、“异步非阻塞”，这些 key words 使 Nginx 给我留下的大概印象就是一个跟 Node.js 差不多的，可以用来搭 web 服务器的东西。至于反向代理，负载均衡什么的，这跟我前端有关系吗？所以当时虽然觉得 Nginx 好像很厉害的样子，但也就没有主动去了解与学习。

而我现在随着技术知识面的扩展，逐渐认识到其实 Nginx 和 Node.js 并不是非彼既此的，两者都有自己擅长的领域 Node.js 更擅长于上层具体业务逻辑的处理。而 Nginx 更擅长于底层服务器端资源的处理（静态资源处理转发、反向代理，负载均衡等），两者可以实现完美结合，帮助我们更好的开发。

> 虽然一些前端经常接触到的需求，不用 Nginx，单纯地使用 Node.js 也可以满足和实现。但实际上完全可以把这些工作交给 Nginx，而不是痛苦地在npm中找包或者造轮子。

当然，我目前对于 Nginx 的打算也只是先学会其用法，然后可以 control 基本的服务器配置和运维工作就 ok。

对于 Nginx，我们可以深入探索的有很多，但对前端开发者而言，能够熟悉掌握和编写 Nginx 的核心配置文件 `nginx.conf`，其实已经能解决 80% 的问题了。

## Nginx 的优点：

Nginx ("engine x") 是一个高性能的 HTTP 和反向代理服务器,特点是**占有内存少**，**并发能力强**。(官方测试 Nginx 能够支持5万并发链接，实际生产环境中可以支撑2-4万并发连接数。)

拓展：https://lnmp.org/nginx.html

## 反向代理与负载均衡

首先什么是代理？ 

> 互联网应用基本都基于 CS 基本结构。而代理其实就是在 client 端和真正的 server 端之间增加一层提供特定服务的服务器，即代理服务器。

**正向代理：**

Nginx 不仅可以做反向代理，实现负载均衡。还能用作正向代理来进行上网等功能。反向代理没概念的话，正向代理肯定熟悉，最典型的就是各种翻墙工具，其实 VPN 就是一个正向代理工具。它会把我们访问墙外或其他没权限的服务器(server)的请求，代理到一个可以访问到该网站的代理服务器(proxy)，然后通过这个代理服务器(proxy)拿到真正要访问的服务器(server)上的资源，最后再将请求内容转发给我们客户端(client)。

具体的流程如下图:

......

概括地说：就是客户端和代理服务器可以直接互相访问，属于一个LAN（局域网）；用户对这个代理行为是有感知的，即用户需要自己操作或者配置自己的请求发送到代理服务器；代理服务器通过**代理用户端的请求**来向域外服务器请求响应内容。

**反向代理：**

反向代理则正好相反，一般代理是指代理客户端，而这里代理的对象是服务器，这就是“反向”这个词的意思。即这个反向代理服务器就是服务器集群向客户端提供的一个统一的入口，客户端的请求都要先经过这个 proxy 服务器。然后由这个反向代理服务器去控制和选择内部目标服务器，拿到数据后再返回给客户端。

....

客户端对于反向代理这个过程是无感知的，因为客户端不需要配置及任何额外的操作就可以发送请求进行访问，代理服务器和真正的目标服务器可以直接互相访问，属于一个LAN（服务器内网）此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器的地址，隐藏了真实服务器 IP 地址。代理服务器通过**代理内部服务器**接受域外客户端的请求，并将请求发送到对应的内部服务器上。

**为什么要Nginx反向代理：**

 现在基本所有的大型网站都采用了反向代理，使用反向代理最主要有两个原因：

1. **安全性：**正向代理的客户端能够在隐藏自身信息的同时访问任意网站，这个给网络安全代理了极大的威胁。因此，我们必须把服务器保护起来，使用反向代理后，客户端的请求无法直接访问到真正的内容服务器，只能先经过反向代理服务器。此时我们就可以利用 Nginx 在这层将危险或者没有权限的请求给过滤掉，从而保证内部服务器的安全。
2. **功能性：**反向代理的主要用途是为多个服务器提供负债均衡、缓存等功能。负载均衡就是一个网站的内容被部署在若干服务器上，可以把这些机子看成一个集群， Nginx 可以将接收到的客户端请求“均匀地”分配到这个集群中所有的服务器上（Nginx 内部模块提供了多种负载均衡算法），从而实现服务器压力的平均分配，也叫**负载均衡**。此外，Nginx还带有健康检查功能（服务器心跳检查），会定期轮询向集群里的所有服务器发送健康检查请求，来检查集群中是否有服务器处于 Abnormal state，一旦发现某台服务器异常，那么在以后代理进来的客户端请求都不会被发送到该服务器上（直到后面的健康检查发现该服务器恢复正常），从而保证客户端访问的稳定性。

## 什么是动静分离

为了加快网站的解析速度，可以把动态页面和静态页面由不同的服务器来解析，加快解析速度。降低原来单个服务器的压力。

## 配置反向代理

进入`/etc/nginx`目录，查看`nginx.conf`配置文件，在`http`块中找到这样两句：

```
# include /etc/nginx/conf.d/*.conf;
# include /etc/nginx/sites-enabled/*; // centos 下是没有这个的
```

看看你的这两句有没有注释掉，如果注释了就把`#`号去掉，没有注释的话就跳过这一步。

4.进入`/etc/nginx/conf.d`目录，创建我们自己的配置文件，去名规则最好是域名加端口，这样以后方便找，比如我的：`rockjins-com-8081.conf`，配置文件写入以下内容：

```
upstream rockjins {
    server 127.0.0.1:8081; # 这里的端口号写你node.js运行的端口号，也就是要代理的端口号，我的项目跑在8081端口上
    keepalive 64;
}

server {
    listen 80; #这里的端口号是你要监听的端口号
    server_name 39.108.55.xxx www.rockjins.com rockjins.com; # 这里是你的服务器名称，也就是别人访问你服务的ip地址或域名，可以写多个，用空格隔开

    location / {
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_set_header X-Nginx-Proxy true;
        proxy_set_header Connection "";
        proxy_pass http://rockjins; # 这里要和最上面upstream后的应用名一致，可以自定义
    }
}复制代码
```

5.保存文件后，输入`sudo nginx -t`测试我们的配置文件是否有错误，一般错误都是漏个分号，少个字母之类的，错误提示很精确，没错的话会输出下面两句:

```
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful复制代码
```

6.现在我们需要重启`Nginx`，我们的配置文件才会生效，输入`sudo service nginx reload`;

**配置一个单节点的反向代理**

```javascript
# simple reverse-proxy
server { 
    listen       80;
    server_name  big.server.com;
    access_log   logs/big.server.access.log  main;

    # pass requests for dynamic content to rails/turbogears/zope, et al
    location / {
      proxy_pass      http://127.0.0.1:8080;
    }
  }

```

这里定义的规则是以big.server.com域名来请求Nginx的80端口，会将请求代理到127.0.0.1:8080上。







Todo..

The content of the notes is mainly excerpted from the network,Infringement contact deletion.