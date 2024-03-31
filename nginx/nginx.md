# 1.介绍

## 1.1 nginx优势



* **高并发高性能**
* **可拓展性好**
  * nginx源码内核非常小，功能不多，但是他的每个功能都是通过模块实现的，所以可以通过添加和删除模块来实现不同的功能，且非常方便二次开发
* **高可靠性**
* **热部署**
  * 类似于前端的热更新，而且是丝滑部署，当更新配置的时候，会产生一个.old后缀的文件，比如a客户端正在访问，这时候更新了配置，这时候会将老的配置生成.old后缀的文件 那么a客户端会继续访问.old后缀配置相关的内容，现在b客户端来访问，就会访问新的配置文件
* **开源许可证**



## 1.2 nginx的架构

### 1.2.1 轻量

* 源码只包含核心模块
* 其他非核心功能都是通过模块实现的，可自由选择

### 1.2.2 架构

nginx采用的是多进程(单线程)和多路IO复用模型



### 1.2.3 工作流程

* Nginx在启动后，会有一个master进程和多个相互独立的worker进程
* master进程接收到请求后，会把这个请求分配给worker进程，所以每个worker进程都可能会处理这个请求(有可能第一个worker处理，也可能第二个worker处理)
* master进程能监控worker进程的运行状态，当worker进程退出后(异常情况下)，会启动新的worker进程

[工作流程图](https://excalidraw.com/#json=tvponPHAvQTD7eXarIzyw,PSCPS4j4Fb8198InfMruOA)



* worker进程数，一般会设置成机器的cpu核数，因为更多的worker数，只会导致进程相互竞争cpu，从而带来不必要的上下文切换
* 使用多进程模式，不仅能提高并发率，而且进程之间相互独立，一个worker进程挂了不会影响到其他worker进程



**举例**

比如现在酒店有领班的，领班的会分配不同服务员接待的区域，领班一般是不会去接待的，他只负责统筹服务员，比如现在有一个服务员要请假了，那么他就会赶紧安排新的服务员去负责这片区域。

那么master就相当于领班，当有worker进程挂了，会启动新的worker进程



# 2.安装

## Linux下安装

**CentOS 7**

```bash
yum install nginx -y  # 安装

nginx -v  # 查看版本号

nginx -V # 查看编译时的参数
```



**nginx -V 参数表示的意思**

```
--prefix  # 把nginx安装到哪个目录下
--sbin-path # 可执行文件的路径
--modules-path # 安装的额外模块路径
--conf-path # 配置文件的路径
--error-log-path # 错误日志路径
--http-log-path= # 普通日志路径
--pid-path # 进程ID路径
--lock-path # 加锁文件路径

-------下面是demo-----------
--prefix=/usr/share/nginx
--sbin-path=/usr/sbin/nginx 
--modules-path=/usr/lib64/nginx/modules
--conf-path=/etc/nginx/nginx.conf
--error-log-path=/var/log/nginx/error.log
--http-log-path=/var/log/nginx/access.log 
--http-client-body-temp-path=/var/lib/nginx/tmp/client_body
--http-proxy-temp-path=/var/lib/nginx/tmp/proxy
--http-fastcgi-temp-path=/var/lib/nginx/tmp/fastcgi 
--http-uwsgi-temp-path=/var/lib/nginx/tmp/uwsgi
--http-scgi-temp-path=/var/lib/nginx/tmp/scgi 
--pid-path=/run/nginx.pid
--lock-path=/run/lock/subsys/nginx 
--user=nginx
--group=nginx
--with-compat
--with-debug
--with-file-aio 
--with-google_perftools_module
--with-http_addition_module
--with-http_auth_request_module
--with-http_dav_module
--with-http_degradation_module
--with-http_flv_module 
--with-http_gunzip_module 
--with-http_gzip_static_module 
--with-http_image_filter_module=dynamic
--with-http_mp4_module
--with-http_perl_module=dynamic
--with-http_random_index_module 
--with-http_realip_module 
--with-http_secure_link_module
--with-http_slice_module 
--with-http_ssl_module 
--with-http_stub_status_module
--with-http_sub_module 
--with-http_v2_module
--with-http_xslt_module=dynamic
--with-mail=dynamic
--with-mail_ssl_module
--with-pcre
--with-pcre-jit 
--with-stream=dynamic
--with-stream_ssl_module
--with-stream_ssl_preread_module
--with-threads
--with-cc-opt='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong
--param=ssp-buffer-size=4 -grecord-gcc-switches -specs=/usr/lib/rpm/redhat/redhat-hardened-cc1 -m64 -mtune=generic' --with-ld-opt='-Wl,-z,relro -specs=/usr/lib/rpm/redhat/redhat-hardened-ld -Wl,-E'
```

# 3.目录

## 3.1 安装目录

**查看nginx安装的配置目录和文件**

* 查看安装nginx的时候会安装哪些文件，和当前文件的路径
* 下面的xxx文件都是这个命令显示的文件

```bash
rpm -ql nginx
```

## 3.2 配置文件核心结构

```nginx
```



## 3.3 主配置文件

### 3.3.1 核心配置文件

目录地址：**/etc/nginx/nginx.conf**



```nginx
# nginx的key 和value 是靠空格分隔 空格的数量是任意的

user             nginx;  # 启用nginx的用户
worker_processes auto; # worker进程数量 一般要和cpu核数相等

error_log /var/log/nginx/error.log warn; # 错误日志的路径 warn 表示日志的格式 有的配置文件可能没有
pid       /run/nginx.pid; # 进程ID写入的文件


events {
    worker_connections 1024; # 工作进程的最大连接数 如果超过这个数的连接就会被丢掉
}

http {
  	# 定义日志的格式
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main; # 访问日志写入的文件 mian表示使用这中格式写入

    sendfile            on; # 零拷贝模式开关
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65; # 保持连接的超时时间
    types_hash_max_size 4096;
  	# gzip                on; # 是否启用gzip压缩

    include             /etc/nginx/mime.types; # 包含的mime文件  比如客户端访问index.js那么服务器就会返回一个Content—type字段值就是 根据访问的文件(index.js)生成的类型 这个类型一定是mime文件的某个类型，因为这样nginx才认识
    default_type        application/octet-stream;  # 如果现在返回的一个类型不在mime文件中 那么就会使用这个默认的类型返回给浏览器

   
    include /etc/nginx/conf.d/*.conf;  # 还要包含这个文件下的所有配置文件

}

```



### 3.3.2 默认http服务配置文件

目录地址：**/etc/nginx/nginx.conf.default** 

```nginx
# http只有一个。里面可以有多个server
http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main; # 访问日志写入的文件 mian表示使用这中格式写入

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;
  
   # 配置服务的 也是最核心的配置文件
    server {
        listen       80; # 监听的端口 nginx启动后默认监听80端口
        server_name  localhost; # 服务名字或者是域名或者是IP

        #charset koi8-r; # 字符集 俄语的字符集 因为作者是俄罗斯的 所以默认写了一个这个 可以自己设置

        #access_log  logs/host.access.log  main;
				
    		# location是重点之中的重点 
    		# 表示如果访问/ 那么就会进入这个配置 然后去html文件夹下找到index.html 如果index.html没有就找 index.htm
        location / {
            root   html; # 文件根路径 
            index  index.html index.htm; # 默认访问的文件
        }

        #error_page  404              /404.html; 

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html; # 如果状态码是500 502 503 504 重定向50x.html
        location = /50x.html {
            root   html;
        }
    }    
}
```

# 4.server模块

**nginx有一个主配置文件`nginx.conf`里面有一个，里面只有一个`http`, `http	`中可以有多个`server`**



## 4.1.root和alias的区别

* 使用root 那么在访问的时候就会**拼接location对应的地址**

```nginx
# 当访问/random的时候 nginx会去 /opt/app/random文件夹下寻找文件
server {
  location /random {
    root /opt/app;
    index index.html;
  }
}
```





* 使用alias 访问的时候**不会**拼接location对应的地址

```nginx
# 当访问/random的时候 nginx会去 /opt/app文件夹下寻找文件
server {
  location /random {
    root /opt/app;
    index index.html;
  }
}
```



# 5.核心模块

**在使用下面的模块时候 使用`nginx -V`检查一下是否有下面的模块 避免不生效**

##  5.1 监控nginx客户端的状态

**模块名**

```
--with-http_stub_status_module
```

**语法**

```
# Syntax 表示语法
# Default 表示默认值
# Context 表示这个模块可以写在哪里 这里表示可以写在server的location中

Syntax: stub_status on/of;
Default: -
Context: server->location
```



**举例**

```nginx
# 可以在nginx.conf文件的server的location配置

http {
	server {
		# 配置一个location
		location /status {
			stub_status on;
		}
	}
}

# 配了一个location 然后重启一下nginx 在页面输入xxx.xxx.xxx/status
# 在页面可以看到下面的内容

Active connections: 2 # 当前nginx正在处理活动的连接数
# server表示名称 比如server accepts表示 服务器处理的连接数  server handled  server requests
# accepts  总共处理的连接数
# handled  成功创建的握手数
# requests 总共处理的请求数
server accepts handled requests
 					25 			25 			23 
# Reading 读取到客户端Header信息 表示正在读取客户端Header的数量是0
# Writing 正在返回给客户端Header信息
# Waiting 等待  如果开启了keep-alive 那么他的数量是 Active connections - (Reading+Writing)
Reading: 0 Writing: 1 Waiting: 1 


# 这些信息的作用？
# 可以查看nginx当前的负载 
```

## 5.2.随机模块

**模块名**

```
--with-http_random_index_module
```

**作用**

```
在给定的目录下随机显示一个页面来显示
```

**语法**

```
Syntax: random_index  on/off;
Default: off
Contxt: location
```

**例子**

```nginx
# 当访问xxx.xxx.xxx:8888/random的时候会随机显示/opt/app文件夹下的一个文件来显示
server {
    listen 8888;
    server_name localhost;
    location  /random {
         alias /opt/app;
         random_index on;
      }
}
```

## 5.3 内容替换模块

 **模块名**

```
--with-http_sub_module
```

**语法**

```
Syntax: sub_filter string replacement;
Default: --
Context: http、service、location
```



**例子**

```nginx
# 当访问/random时 会把name 替换成jack 会把<h1>name</h1> 替换成 <h1>jack</h1> 返回给浏览器
# 比如现在index.html中有 name abc <h1>name</h1>这些内容。
# 那么在响应的时候会把匹配到的替换掉 返回的就是 jack abc <h1>jack</h1>

server {
    listen 8888;
    server_name localhost;
    location  /random {
         alias /opt/app;
         index index.html;
         sub_filter 'name' 'jack';
         sub_filter <h1>name</h1> <h1>jack</h1>;
      }
}
```

## 5.4 请求限制

**模块名**

```
# 连接频率限制
--with-limit_conn_module
# 请求频率限制
--with-limit_req_module
```

### 5.4.1 ab模块

**作用**

```
这是Apache的ab命令，可以模拟多线程并发请求，测试服务器负载压力，也可以测试nginx、lighthttp、IIs等其他Web服务器的压力，因为我们这个模块是限制请求的，所以需要用到ab模块来测试
```

**安装**

```
yum -y install httpd-tools
```

**使用**

```
# -n 总共的请求数
# -c 并发的请求数
ab -n 40 -c 20 http://127.0.0.1/
```

### 5.4.2.共享连接内存

**要想使用请求限制 需要先分配一个共享内存**



nginx有一个master进程和n(一般根据cpu核数)个worker进程，一个客户端(IP)给服务器发请求，会随机分配给一个worker



我们可以这么做 ，在内存中开辟一块空间，这个空间会对所有的worker进行内存共享，这里面存放着所有的限制连接的数据，比如我们限制每秒钟每个IP只能连接一次，现在有一个IP给服务器发请求，这时候会分配到一个worker，这个worker就会往这个共享内存中写入一条数据 比如 ip地址为xxx的 在14:30:22的时候发起了一个请求，建立了一次连接等信息，如此同时这个ip又发送了一个请求到这个服务器，被别的worker来处理了，这时候这个worker会先去共享空间查询一下是否存在限制，这时候正好有一条这个ip发的请求时间是在1秒内，那么这时候就不符合我们设置的1s内1一个IP只能发送1次请求，所以就会断开这次连接



**语法**

```
# key：我们可以以请求的IP为key  zone：空间的名词 size：为空间申请的大小
# 说白了key就是worker过来找的时候能够比对的标识符 用请求的IP为key

Syntax: limit_conn_zone key zone=name:size
Default: --
Context: http(定义在server以外)
```

**举例**

```nginx
http {
  # $binary_remote_addr 是nginx的变量 表示请求的客户端的IP地址
  # 这段表示 key是客户端IP 空间的名称是conn_zone(自定义的)大小是10m
  limit_conn_zone $binary_remote_addr zone=conn_zone:10m;
  server: {
    xxx
  }
}
```

### 5.4.3 连接限制

**语法**

```
# zone表示共享空间的名称 number：限制数量(每个客户端每秒同时连接的数量)

Syntax: limit_conn zone number
Default: --
Context: http、server、location
```

**附加属性**

```nginx
 # 断开连接的返回的状态码 默认503
 limit_conn_status 500; 
 # 断开连接日志的等级 info|notice|warn|error
 limit_conn_log_level warn;
 # 每秒服务器只能给客户端发送50字节
 limit_rate 50;
```



**举例**

```nginx
http {
  # 创建共享空间
  limit_conn_zone $binary_remote_addr zone=conn_zone:10m;
  
  server {
    	
      location /zone {
        alias /opt/app;
        index blue.html;
      	# 共享空间是conn_zone 每秒钟同一个IP只能发起一个连接
      	# 这里需要注意的是 一个连接可以发无数个请求
      	# 虽然我们这里限制了1个连接。我们用ab测试的时候假如并发设置10 总的请求设置成10。可能10个请求都成功了 也肯只有一个请求成功 原因就是可能这10个请求都走的这个连接
        limit_conn conn_zone 1;
        # 断开连接的返回的状态码 默认503
        limit_conn_status 500; 
        # 断开连接日志的等级
        limit_conn_log_level warn;
        # 每秒服务器只能给客户端发送50字节
        limit_rate 50;
    }
  }
}
```

### 5.4.4 请求限制

**注意:** limit_req生效是在limit_conn之前



**定义共享内存 语法**

```
# 因为请求限制的原理是依靠共享内存的 解释参考上面的共享连接内存
# 所以要配置一下共享内存
# key 一般使用客户端ip  zone 自定义名称 size 大小 rate 速率
Syntax: limit_req_zone key zone=name:size rate=rate
Default:--
Context: http(定义在server以外)
```



**请求限制 语法**

```
# zone 共享内存名称
# burst 下面具体讲解  nodelay 下面具体讲解
Syntax: limit_req zone=name [burst=number] [nodelay]
Default: --
Context: http,server,location
```



**案例**

```nginx
# 定义了共享空间 key是IP地址。名称是req_zone 内存10m。每秒只能响应一条数据
limit_req_zone $binary_remote_addr zone=req_zone:10m rate=1r/s;

# 使用 ab -n 10 -c http://127.0.0.1/8888/req
# 总请求数是10条 并发是1条
# 我测试的是不到1s就发送了这个10条请求 所以成功的请求只有1条 失败了9条
# 因为我们设置了每秒只能响应一条数据

server {    
    listen 8888;
    server_name localhost;

    location /req {
        alias /opt/app;
        index blue.html;
        limit_req zone=req_zone; 
    }
}
```

### 5.4.5 burst参数

burst 是干嘛的呢 看下面详细的讲解

limit_req 使用的是漏斗算法 什么是漏斗算法算法呢，下面举一个例子

现在有一个水龙头 🚰，我们来接水，假设我们限制每秒只能接一滴水，这时候水龙头每秒就滴一滴水，也符合我们的限制，但是这时候水龙头每秒滴 10 滴水了，那么明显不符合我们的需求了，每秒多余的 9 滴水我们只能抛弃了

现在我们换一种方法 我们在水龙头下面放一个漏斗(桶、盆都行) 这个漏斗每秒固定的只能流出一滴水 他的容量是 5 滴 下面我们分解一下 这 10 滴水怎么处理的

**注意**

在描述事件或者过程的时间线时，以“第 0 秒”开始是一种常见的方式，用来表示从某个特定动作发起的初始时刻。这并不是说真正存在一个零长度的第 0 秒，而是为了方便计算和解释，在时间序列的起点指定为第 0 秒。

- 第 0 秒的时候 来了 10 滴 然后每秒只能处理 1 滴，当前没有正在处理的，所以会处理 1 滴, 剩余 9 滴 因为漏斗的容量是 5 所以这 9 滴只有 5 滴进入漏斗
- 第 1 秒 漏斗流出一滴 还剩 4 滴
- 第 2 秒 漏斗流出一滴 还剩 3 滴
- 第 3 秒 漏斗流出一滴 还剩 2 滴
- 第 4 秒 漏斗流出一滴 还剩 1 滴
- 第 5 秒 漏斗流出一滴 还剩 0 滴
- 所以用时 5 秒 一共处理了 6 滴

水龙头其实就代表了客户端(浏览器)，因为我们无法控制浏览器每秒的请求数量，可能当前浏览器发起了 10 条请求，每秒请求一次，也可能每秒请求 5 次那么两秒就请求完毕了，或者说 1 秒就请求 10 次

虽然我们无法控制客户端 但是我们可以控制漏斗的恒定流速 那么 burst 其实就是充当漏斗 我们可以设定这个漏斗可以缓存多少数据

**案例**

```nginx
# 定义了共享空间 key是IP地址。名称是req_zone 内存10m。每秒只能响应一条数据
limit_req_zone $binary_remote_addr zone=req_zone:10m rate=1r/s;

server {
  location /req {
        alias /opt/app;
        index blue.html;
        limit_req zone=req_zone burst=5;
    }
}

```

### 5.4.6 nodelay 参数

上面 burst 案例我们说到 浏览器一次发送了 10 条请求 ，然后成功响应了 6 条 耗费 5s 中 但是让用户等待 5s 是不好的行为

如果设置了 nodelay，则即使请求暂时超出了每秒允许的频率，只要总数没有超过 burst 大小，这些请求也会立即得到处理，而不会被延迟。开启 nodelay 后，服务器会立刻处理 burst 内所有的请求，然后恢复正常的速率控制。

**案例**

```nginx
# 定义了共享空间 key是IP地址。名称是req_zone 内存10m。每秒只能响应一条数据
limit_req_zone $binary_remote_addr zone=req_zone:10m rate=1r/s;

server {
  location /req {
        alias /opt/app;
        index blue.html;
        limit_req zone=req_zone burst=5 nodelay;
    }
}

```

结合案例 我们设置了 nodelay 然后浏览器并发了 10 个请求

第一个请求立即被处理，随后的请求直到数量达 到 burst 值的限制都将立即获得处理，而不必等待。也就是说，如果同时到达 5 个请求，在开启了 nodelay 的情况下，这 5 个请求几乎会同时被处理。之后服务器将恢复到每秒处理一个请求的速率。

这样客户端就不用等待了 可以立马获取结果



##  5.5 访问控制

### 5.5.1 基于Ip地址的访问控制

**模块名**

```
http_access_moudle
```

**作用**

```
可以设置允许访问的ip。也可以设置不允许访问的ip。有两种写法
```

**语法(allow)**

```
# 值可以是 IP地址 或者 all
# 这个表示允许哪些可以访问
Syntax: allow address|all
Default: --
Context: http,server.location.limit_except
```

**语法(deny)**

```
# 值可以是 IP地址 或者 all 或者 CIDR
# 这个表示不允许哪个访问 也就是除了设置的地址其余的都能访问
Syntax: deny address|CIDR|all
Default: --
Context: http,server.location.limit_except
```

**案例**

```nginx
http {
  server {
    location /admin {
      alias /opt/app;
      # 表示只要是这个ip访问/admin 都不允许访问
      deny 119.120.221.100;
      # 可以设置多个ip
      deny 119.120.221.101;
      # 可以和allow同时使用
      allow 119.120.221.101;
    }
    
    location /admin1 {
      alias /opt/app;
      # 表示只允许这个ip访问/admin1 其余ip地址访问/admin1都被拒绝
      allow 119.120.221.100;
      # 可以设置多个ip
      allow 119.120.221.101;
    }
  }
}
```

# 6 静态资源web服务器

## 6.1 静态资源和动态资源

* **静态资源**：一般客户端发送请求到web服务器，web服务器从内存再取到相应的文件，返回给客户端，客户端解析并渲染出来

```nginx
# 比如我想访问某个图片 那么直接会从文件夹去拿，这就是静态资源
http {
  server {
     location ~ .*\.(jpg|jpeg|png|gif) {
            root /usr/share/nginx/html;
        }
  }
}
```

* **动态资源**：一般客户端请求的动态资源，现将请求将于web容器，web容器连接数据库，数据库处理数据之后，将内容交给web服务器，web服务器返回给客户端解析渲染处理

```nginx
# 现在所有的api开头的请求都会被代理到http://localhost:3000 这就是动态资源
http {
  server {
    location ~ ^/api {
            rewrite ^/api/(.*) /$1 break;
            proxy_pass  http://localhost:3000;
        }
  }
}
```

## 6.2 CDN

* CDN全称是Content Delivery Network，即内容分发网络
* CDN系统能够实时的根据网络流量和各节点的连接、负载状况以及到用户的距离和响应时间等综合信息将用户的请求重新导向离用户最近的服务器节点上。其目的是使用户可以就近去的所需内容，解决网络拥挤的状况，提高用户访问房展的响应速度。

**案例**

```nginx
```



## 6.3 tcp_nopush和tcp_nodelay

**tcp_nopush**

* 在开启sendfile的情况下，会合并多个数据包，提高网络包的传输效率
* 假如我们现在在下载电影，每秒都会响应数据 这时候开启这个就会合并多个响应 比如每3秒响应一次 

```
Syntax: tcp_nopush on/off
Default: off
Context: http、server、location
```



**tcp_nodelay**

* 在keepalive连接下，提高网络包的实时传输性
* 假如现在每秒响应一个数据，那么nginx会立刻响应，一般用于需要实时传输数据的情况，比如游戏数据 

```
Syntax: tcp_nodelay on/off
Default: on
Context: http、server、location
```



## 6.4 gzip

**gzip**

* 控制gzip压缩率的
* 压缩比率越高，文件被压缩的体积越小，对应的加载资源的速度就越慢
* 1-9级别

```
Syntax: gzip on/off
Default: off
Context: http、server、location
```



**gzip_comp_level**

* 控制gzip压缩率的
* 压缩比率越高，文件被压缩的体积越小，对应的加载资源的速度就越慢
* 1-9级别

```
Syntax: gzip_comp_level level
Default: 1
Context: http、server、location
```



**gzip_http_version**

* 用于识别http协议的版本，早期的浏览器不支持gzip压缩，用户会看到乱码，所以为了支持前期版本加了此选项。默认在http/1.0的协议下不开启gzip压缩。

```
Syntax: gzip_http_version 1.0/1.1
Default: gzip_http_version 1.1
Context: http、server、location
```



**http_gzip-static_module**

* 如果我们每次都在nginx中进行gzip压缩，那么假如现在并发1000个请求那么就要执行1000次gzip，性能肯定是不高的
* 如果启用了这个模块，就会先在文件夹中找到同名的`gz`文件是否存在
* 比如现在有接口来找a.txt,那么nginx会先将a.txt压缩(不会动原文件)然后才返回，现在我们直接将a.txt压缩成`a.txt.gz`，当请求过来 会先找`a.txt.gz`文件，然后直接返回

```
Syntax: gzip_static on/off
Default: off
Context: http、server、location
```



**案例**

```nginx
http {
  server {
      # 图片就不开启gzip压缩了 因为本身图片就是被压缩过的
        location ~ .*\.(jpg|jpeg|png|gif) {
            gzip off;
            root /usr/share/nginx/html;
        }

        location ~ .*\.(html|js|css) {
            gzip on;
      			# 当返回内容大于此值时才会使用gzip进行压缩,以K为单位,当值为0时，所有页面都进行压缩。
            gzip_min_length 1k;
            gzip_http_version 1.1;
            gzip_comp_level   9;
            #设置需要压缩的MIME类型,如果不在设置类型范围内的请求不进行压缩
            gzip_types text/css application/javascript;
            root /usr/share/nginx/html;
        }

        location ~ ^download {
           gzip_static on;
           tcp_nopush on;
           root /usr/share/nginx/html;
        }
  }
}
```

# 7 浏览器缓存

nginx天生就支持浏览器缓存策略，也就是响应的时候会携带etag和last-modified字段，所以不用做任何配置



[浏览器缓存](https://excalidraw.com/#json=Pp460J3hFs7VN19GFMzqz,IB7-UgLaR8RzbbEsl0IfaA)



唯一需要配置的就是过期时间



## 7.1 expires

注意如果设置了expires就相当于使用强制缓存了



**作用**

```
添加Cache-Control、Expires头
```

**语法**

```
Syntax: expires time
Default: off
Context: http、server、location
```



**案例**

* 理论上设置了expires，当时间没过期，那么当我再次请求这个图片的时候就不会想服务器发送请求，而是直接使用缓存
* 但是因为浏览器的差异，如果使用的是chrome那么即使设置了expires，没有过期，还是会发送请求，IE不会发送可以试一下
* 如果使用的是img标签src等于这种图片，那么chrome就不会向服务器发送请求，而是直接使用缓存，这个请求的status是(200 OK (from memory cache))

```nginx
http {
  server {
      location ~ .*\.(jpg|jpeg|png|gif) {
      		# 过期时间24小时
          expires 24h;
          root /usr/share/nginx/html;
      }
  }
}
```

# 8 跨域

解决跨域其实原理就是添加一个响应头



**语法**

```
Syntax: add_header name value
Default: --
Context: http、server、location
```

**案例**

```nginx
http {
  server {
     location ~ ^/api {
      			# 添加一个响应头 允许所有源
            add_header Access-Control-Allow-Origin *;
            # 添加一个响应头 允许的方法是什么
            add_header Access-Control-Allow-Methods GET,POST;
            rewrite ^/api/(.*) /$1 break;
            proxy_pass  http://localhost:3000;
            
        }
  }
}
```

# 9 防盗链

* 防止网站资源被盗用
* 保证信息安全
* 防止流量过量
* 区别哪些请求是非正常的请求

**什么是**

```
比如现在a网站有一个图片，在没有设置防盗链的情况下 任何网站拿到这个图片的url都可以显示，但是这有一个问题，比如现在b网站显示的全部是a网站的图片的，那么b网站就不用在服务器里存储图片了，直接使用a网站的链接即可，这肯定是不允许的  所以我们可以设置防盗链 设置允许哪些网站可以访问我们服务器的资源，那些不被允许的网站想要链接我们网站的资源就会报错304(没权限)

原理
当浏览器发起请求时会在requestHeader中携带Referer字段  那么nginx就是通过这个字段的值和我们设置允许访问的值做比对，进而判断是否有权限

注意 当直接在浏览器输入资源地址请求头中是没有Referer的 所以配置的时候要是不允许浏览器地址栏访问 就不要在规则中设置none
```

**语法**

```
# server_names、IP是必填的 none、block可以不存在
Syntax: valid_referers none、block、server_names、IP
Default: --
Context: server、location
```

**案例**

```nginx
http {
  server {
     location ~ .*\.(jpg|jpeg|png|gif) {
      			# 配置允许访问的地址。当none(请求头没有referer或者值是空) 或者 bolcked 或者192.168.1.112地址
            valid_referers none blocked 192.168.1.112;
      			# 如果不是允许的地址 返回403
            # $invalid_referer系统自带的变量 可以捕获到valid_referers之外的内容
            if ($invalid_referer) {
                return 403;
            }

            root /usr/share/nginx/html;
        }
  }
}
```

# 10 代理服务

**代理服务分成正向代理和反向代理**



**语法**

```
Syntax: proxy_pass URL
Default: --
Context: server、location
```



## 10.1 正向代理

* 正向代理的对象是客户端，服务端看不到真正的客户端

**案例**

```nginx
# 假如www.jetli.com.cn我们现在的网络访问不了
# 然后我们就可以让nginx进行代理 进而访问该网站
# 这里要注意 要修改本地的hosts文件要让www.jetli.com.cn域名指向我们的服务器
# 不然我们的服务器拦截 不了

http {
    server {
   # chrome的域名解析地址 一定要写不然输入域名解析不了
   resolver 8.8.8.8;

    listen  7777;
    # 这里一定要配置想要访问的域名 不然进不来nginx
    server_name localhost www.jetli.com.cn;

    location / {
      # $http_host 请求的主机名
      # $request_uri  请求路径
      # 假如我们在地址栏请求 www.jetli.com.cn/abc
      # 那么$http_host就是 www.jetli.com.cn
      # $request_uri就是/abc
       proxy_pass http://$http_host$request_uri;
    }
  }
  
  # 我们还可以使用正向代理 代理外网
  # 比如我们有一个香港的服务器
  # 然后做如下配置
  # 然后当访问7777端口就会代理到yutube了
  server {
    listen  7777;
    server_name localhost;

    location / {
       proxy_pass http://www.yutube.com;
    }
  }

}


```



## 10.2 反向代理

* 反省代理的对象是服务端，客户端看不到真正的服务端

**案例**

```nginx
http {
  server {
    location ~ ^/api {
            rewrite ^/api/(.*) /$1 break;
            # 设置正向代理 
      			# 客户端本来想访问http://localhost:3000 
      			# 但是实际上访问的是nginx 然后nginx去访问3000端口 然后结果给到客户端
            proxy_pass  http://localhost:3000;
            
        }
  }
}
```

