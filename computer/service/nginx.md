# Nginx



## ~~ 快速入门 ~~



### 什么是Nginx?

Ngnix是一个**高性能的**Http**反向代理**服务器, 也是一个IMAP/POP3/SMTP服务器. (毛子发明的)

> 什么是正向代理? 什么又是反向代理?
>
> * 正向代理: 我们国内想要访问谷歌, 需要用到代理服务器, 流程如下:
>   * 用户浏览器 => 代理服务器 => 访问谷歌服务器 => 处理数据
>   * 谷歌服务器返回结果 => 代理服务器 => 用户浏览器
> * 反向代理: 我们访问Nginx服务器, 它只是帮我们找到需要的服务器:
>   * 用户浏览器 => Nginx代理服务器 => 帮我们找到一个Tomcat服务器
>   * 被找到的Tomcat服务器 => 处理数据 => 用户浏览器

Nginx最大的好处就是可以整合其他模块实现开发更复杂的功能, 最大的坏处就是本身有大量的开发模块, 需要用户进行配置.



### 环境依赖

Nginx的安装需要依赖很多环境, 有许多环境都是可以通过`yum`命令搞定的:

```shell
yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
```

而有一些环境, 例如我们即将要安装的PCRE, 则是需要我们源码包安装的:

```shell
 cd /usr/local/src
 wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
 tar zxvf pcre-8.35.tar.gz
 cd pcre-8.35
 ./configure
 make && make install
```



### 源码包准备

对于Nginx, 我们需要如下的几个包:

* `nginx-1.11.3.tar.gz  ` Nginx源码包, [Nginx全版本下载地址](http://nginx.org/download/)
* `ngx_cache_purge-2.3.tar.gz` Nginx缓存清除模块, [Nginx缓存清除模块全版本下载地址](http://labs.frickle.com/files/)
* `echo-nginx-module-0.59.tar.gz` Nginx信息输出模块, [Nginx信息输出模块下载地址](https://github.com/openresty/echo-nginx-module/archive/v0.59.tar.gz)
* `gnosek-nginx-upstream-fair-a18b409.tar.gz` Nginx负载均衡模块, [Ningx负载均衡模块下载地址](https://codeload.github.com/gnosek/nginx-upstream-fair/legacy.tar.gz/master)

我们可以通过`wget`命令快速获取(如果wget没有用, 可以通过`yum -y install wget`来安装):

```shell
wget http://nginx.org/download/nginx-1.11.3.tar.gz; \
wget http://labs.frickle.com/files/ngx_cache_purge-2.3.tar.gz; \
wget https://github.com/openresty/echo-nginx-module/archive/v0.59.tar.gz; \
wget https://codeload.github.com/gnosek/nginx-upstream-fair/legacy.tar.gz/master
# 上面这些链接很多没用..... 去网盘找吧
```

然后我们对其进行依次解压缩:

```shell
tar -zxvf ???.tar.gz -C /usr/local/src
```

> 以后我们整合其他技术还会使用到其他的模块, 现在先使用缓存清除, 信息输出, 负载均衡三个模块.



### 编译前配置

和我们安装很多软件一样, 在编译前我们需要执行`./configure`命令来配置我们的编译选项, 我们的Nginx编译安装, 需要指定安装的文件夹, 所以我们提前建立好这些文件夹.

```shell
mkdir -p /usr/local/nginx/ \
{logs,conf,fastcgi_temp,sbin,client_body_temp,proxy_temp,uwsgi_temp,scgi_temp}
```

然后就可以进行配置了:

```shell
./configure --prefix=/usr/local/iginx/ \
--sbin-path=/usr/local/nginx/sbin/ \
--with-http_ssl_module \
--conf-path=/usr/local/nginx/conf/nginx.conf \
--pid-path=/usr/local/nginx/logs/nginx.pid \
--error-log-path=/usr/local/nginx/logs/error.log \
--http-log-path=/usr/local/nginx/logs/access.log \
--http-fastcgi-temp-path=/usr/local/nginx/fastcgi_temp \
--http-client-body-temp-path=/usr/local/nginx/client_body_temp \
--http-proxy-temp-path=/usr/local/nginx/proxy_temp \
--http-uwsgi-temp-path=/usr/local/nginx/uwsgi_temp \
--http-scgi-temp-path=/usr/local/nginx/scgi_temp \
--add-module=/usr/local/src/echo-nginx-module-0.59 \
--add-module=/usr/local/src/ngx_cache_purge-2.3 \
--add-module=/usr/local/src/gnosek-nginx-upstream-fair-a18b409
```

至于详细有什么可以配置的, 可以通过`./configure --help`命令来查看配置项:

* `--prefix` 指定我们Nginx安装的主目录, 一般都会配置
* `--xxx-path` 指定Nginx的一些目录所在的配置, 一般不配置, 可以自动生成
* `--with-xxx` 开启一些默认关闭的模块
* `--without-xxx` 关闭一些默认开启的模块
* `--add-module` 添加一些我们自己下载好的模块



### 编译 & 安装

```shell
make && make install
```



### 启动

在我们编译前配置的`--prefix`指定的目录中就是我们已经安装好的Nginx了, 我们可以执行其中的`sbin/nginx`命令来启动我们的Nginx:

```shell
[root@nginx nginx]# tree sbin conf
sbin
└── nginx
conf
├── fastcgi.conf
├── fastcgi.conf.default
├── nginx.conf
├── nginx.conf.default
......
[root@nginx nginx]# sbin/nginx
```

然后我们就可以通过`ps -ef | grep nginx`命令, 以及`netstat -anp |grep nginx`命令来查看Nginx的状态了.

> 通过浏览器访问`http://192.168.??.??:80`, 其中IP为Nginx所在机器的IP, 端口为80也可以证明Nginx已经运行.



## ~~ Nginx 代理 Tomcat ~~



### 单个Tocmat整合

Nginx的核心配置文件就是`--prefix/conf/nginx.conf`, 我们可以轻松和Tomcat进行整合:

```shell
server {
	listen       80;
	server_name  localhost;
	location / {
		proxy_pass http://192.168.124:8080;		# 被代理服务器的URL
		proxy_redirect off;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
	}
}
```

我们如何才能知道我们的配置文件是否有语法错误呢? 可以通过`nginx -t`来实现:

```shell
[root@nginx conf]# ../sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

我们看到了**is successful**字样, 说明没有语法错误, 可以放心地动态加载这个配置文件了.

```shell
ngnix -s reload
```

代理完成以后, 我们访问: `NginxIP:NginxPort`, Nginx就会帮我们代理到`TomcatIP:TomcatPort`下了, 而ContextPath和QueryString都会被等量传递过去.

> 我们可能会发现我们的Nginx是无法访问



### Tomcat集群整合

我们无非就是定义一组`集群`, 然后为其取一个名字, 然后让`proxy_pass`指向这个集群即可.

```shell
upstream firene {
	server 192.168.124.1:8080;
	server 192.168.124.80:8080;
}

server {
	listen       80;
	server_name  localhost;
	location / {
		proxy_pass http://firene;
		proxy_redirect off;
		proxy_set_header Host $host;
		proxy_set_header X-Real-IP $remote_addr;
		proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
   }
}
```



### 权重配置

我们可以指定哪一个Tomcat服务器应该拥有更多的访问量, 也就是权重啦~

```
upstream appserver {
	server 192.168.124.81:8080 weight=1;
	server 192.168.124.82:8080 weight=5;
	server 192.168.124.83:8080 weight=9;
}
```



### 宕机 & 备份主机

* `down` 该主机不参与服务了.
* `backup` 表示该主机平常不参与服务, 如果所有的主机性能都不够用了, 才回去使用这个主机.

```
upstream appserver {
	server 192.168.124.81:8080 down;
	server 192.168.124.82:8080 backup;
}
```



## Location配置详解

### Location

`location [ ,=,~,~*,^~] /uri/ { … }`

* `_` 以指定的字符串开头
* `=` 完全等于指定的字符串
* `~` 以指定的正则表达式结尾(区分大小写)
* `~*` 以指定的正则表达式结尾(不区分大小写)
* `^~` 以指定的字符串开头, 如果模式匹配, 停止搜索
* `!~`  `!~*` 取反

---

> 在看接下来的内容之前, 我们先规定几个规则:
>
> ```shell
> # Nginx配置
> location {location} {
>     alias {redirect}; 
> }
> # 浏览器访问
> ${browser}
> # 最终访问路径
> ${result}
> ```


### alias

别名配置, 用于访问<u>系统文件</u>.

```shell
公式: ${result} = ${redirect} + ( ${browser} - ${location} )
```

### root

根路径配置, 用于访问<u>系统文件</u>.

```shell
公式: ${result} = ${redirect} + ${browser}
```

### proxy_pass

反向代理配置, 用于<u>代理请求</u>, 适用于服务器负载均衡的场景, 转发请求到`proxy_pass`配置的URL，是否会附加`location`配置路径与`proxy_pass`配置的路径后是否有`/`有关，有`/`则不附加.








