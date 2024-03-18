# 1.介绍





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

### nodelay 参数

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

第一个请求立即被处理，随后的请求直到数量达到 burst 值的限制都将立即获得处理，而不必等待。也就是说，如果同时到达 5 个请求，在开启了 nodelay 的情况下，这 5 个请求几乎会同时被处理。之后服务器将恢复到每秒处理一个请求的速率。

这样客户端就不用等待了 可以立马获取结果

