# 一. Nginx系列一：概念和配置

## 1. 功能介绍

- **基本的 HTTP 服务器功能**  

- **邮件代理服务器功能**  

- **TCP/UDP 代理服务器功能**  

- **架构和可扩展性**  

## 2.Nginx 特性

NGINX 有什么不同？ NGINX 使用可扩展的事件驱动架构，而不是更传统的过程驱动架构。这需要更低的内存占用，并且当并发连接扩大时，使内存使用更可预测。在传统的 Web 服务器体系结构中，每个客户端连接作为一个单独的进程或线程处理，随着网站的流行度增加，并发连接数量的增加， Web 服务器减慢，延迟了对用户的响应。从技术的角度来看，产生一个单独的进程/线程需要将 CPU 切换到新的任务并创建一个新的运行时上下文，消耗额外的内存和 CPU 时间，从而对性能产生负面影响。NGINX 开发的目标是实现 10 倍以上的性能，优化服务器资源的使用，同时也能够扩展和支持网站的动态增长。 因此， NGINX 成为最知名的模块化，事件驱动，异步，单线程 Web 服务器和 Web 代理之一。  

Nginx 是一个高性能的 Web 和反向代理服务器, 它具有很多非常优越的特性:  

**作为 Web 服务器**  

相比 Apache， Nginx 使用更少的资源，支持更多的并发连接，体现更高的效率，这点使 Nginx尤其受到虚拟主机提供商的欢迎。能够支持高达 50,000 个并发连接数的响应，感谢 Nginx为我们选择了 epoll and kqueue 作为开发模型。

 **作为负载均衡服务器**  

Nginx 既可以在内部直接支持 Rails 和 PHP，也可以支持作为 HTTP 代理服务器 对外进行服务。 Nginx 用 C 编写, 不论是系统资源开销还是 CPU 使用效率都比 Perlbal 要好的多。  

**作为邮件代理服务器**  

Nginx 同时也是一个非常优秀的邮件代理服务器(最早开发这个产品的目的之一也是作为邮件代理服务器)， Last.fm 描述了成功并且美妙的使用经验。Nginx 安装非常的简单，配置文件 非常简洁(还能够支持 perl 语法)， Bugs 非常少的服务器:
Nginx 启动特别容易，并且几乎可以做到 7*24 不间断运行，即使运行数个月也不需要重新启动。你还能够在 不间断服务的情况下进行软件版本的升级 。

## 3. Nginx 架构

​	处理并发连接的传统的基于进程或线程的模型涉及使用单独的进程或线程处理每个连接，并阻止网络或输入/输出操作。 根据应用，在内存和 CPU 消耗方面可能非常低效。 产生一个单独的进程或线程需要准备一个新的运行时环境，包括分配堆和堆栈内存，以及创建新的执行上下文。 额外的 CPU 时间也用于创建这些项目，这可能会导致由于线程在过多的上下文切换上的转机而导致性能下降。 所有这些并发症都表现在较老的 Web 服务器架构(如 Apache)中。 这是提供丰富的一般应用功能和优化的服务器资源使用之间的一个折衷。从一开始 nginx 就是一个专门的工具，可以实现更高性能，更密集和经济地使用服务器资源，同时实现网站的动态发展，所以它采用了不同的模式。 它实际上受到各种操作系统中高级事件机制的不断发展的启发。发展结果变成是一个模块化的，事件驱动的，异步的，单线程的非阻塞架构的 nginx 代码基础。nginx 大量使用复用和事件通知，并专门用于分离进程的特定任务。 连接在有限数量的单线程进程称为工作(worker)的高效运行循环中处理。 在每个工作(worker)中， nginx 可以处理每秒数千个并发连接和请求。  

## 4. 代码结构

​	nginx 工作(worker)码包括核心和功能模块。 nginx 的核心是负责维护严格的运行循环，并在请求处理的每个阶段执行模块代码的适当部分。 模块构成了大部分的演示和应用层功能。 模块读取和写入网络和存储，转换内容，执行出站过滤，应用服务器端包含操作，并在代理启动时将请求传递给上游服务器 。nginx 的模块化架构通常允许开发人员扩展一组 Web 服务器功能，而无需修改 nginx 内核。 nginx 模块略有不同，即核心模块，事件模块，阶段处理程序，协议，可变处理程序，  过滤器，上游和负载平衡器。 nginx 不支持动态加载的模块; 即在构建阶段将模块与核心一起编译。

在处理与接受，处理和管理网络连接和内容检索相关的各种操作时， nginx 在基于 Linux，Solaris 和 BSD 的操作系统中使用事件通知机制和一些磁盘 I/O 性能增强，如： kqueue， epoll，和事件端口。 目标是为操作系统提供尽可能多的提示，以便及时获取入站和出站流量，磁盘操作，读取或写入套接字，超时等异步反馈。 对于每个基于 Unix 的 nginx 运行的操作系统，大量优化了复用和高级 I/O 操作的不同方法的使用。    

![image-20220105223938460](C:\Users\wangzhen\AppData\Roaming\Typora\typora-user-images\image-20220105223938460.png)

## 5. 工作模式

如前所述， nginx 不会为每个连接生成一个进程或线程。 相反，工作(worker)进程接受来自共享“listen” 套接字的新请求，并在每个工作(worker)内执行高效的运行循环，以处理每个工作(worker)中的数千个连接。 没有专门的仲裁或分配与 nginx 工作(worker)的联系; 这个工作(worker)是由操作系统内核机制完成的。 启动后，将创建一组初始侦听套接字。 然后，工作(worker)在处理 HTTP 请求和响应时不断接受，读取和写入套接字。  

运行循环是 nginx 工作(worker)代码中最复杂的部分。 它包括全面的内部调用，并且在很大程度上依赖异步任务处理的想法。 异步操作通过模块化，事件通知，广泛使用回调函数和微调定时器来实现。 总体而言，关键原则是尽可能不阻塞。 nginx 仍然可以阻塞的唯一情况是工作(worker)进程没有足够的磁盘存储。  

由于 nginx 不会连接一个进程或线程，所以在绝大多数情况下，内存使用非常保守，非常有效。 nginx 也节省 CPU 周期，因为进程或线程没有持续的创建 - 销毁模式。 nginx 的作用是检查网络和存储的状态，初始化新连接，将其添加到运行循环中，并异步处理直到完成，此时连接被重新分配并从运行循环中删除。 结合仔细使用系统调用(syscall)和精确实
现支持接口(如 pool 和 slab 内存分配器)， nginx 通常可以在极端工作负载下实现中到低的CPU 使用  。

在一些磁盘使用和 CPU 负载模式，应调整 nginx 工作(worker)的数量。 在这里说一点基础规则：系统管理员应该为其工作负载尝试几个配置。 一般建议可能如下：如果负载模式是 CPU 密集型的，例如，处理大量 TCP/IP，执行 SSL 或压缩，则 nginx 工作(worker)的数量应与 CPU 内核数量相匹配; 如果负载大多是磁盘 I/O 绑定，例如，从存储或重代理服务
不同的内容集合 - 工作(worker)的数量可能是核心数量的一到两倍。有些工程师会根据个人存储单元的数量选择工作(worker)的数量，但这种方法的效率取决于磁盘存储的类型和配置 。

nginx 的开发人员将在即将推出的版本中解决的一个主要问题是如何避免磁盘 I/O 上的大多数阻塞。 目前，如果没有足够的存储性能来提供特定工作(worker)生成的磁盘操作，该工作(worker)可能仍然阻止从磁盘读取/写入。 存在许多机制和配置文件指令来减轻此类磁盘 I/O 阻塞情况。要注意的是，诸如： sendfile 和 AIO 之类的选项的组合通常会为磁盘性能带来很大的余量。 应该根据数据集，可用于 nginx 的内存量和底层存储架构来规划安装一个 nginx 服务器。  

现有工作(worker)模式的另一个问题是与嵌入式脚本的有限支持有关。 一个使用标准的 nginx 分发，只支持嵌入 Perl 脚本。一个简单的解释：关键问题是嵌入式脚本阻止任何操作或意外退出的可能性。 这两种类型的行为将立即导致工作(worker)挂起的情况，同时影响到数千个连接。 更多的工作(worker)计划是使 nginx 的嵌入式脚本更简单，更可靠，适用于更广泛的应用。  

## 6. nginx 进程角色

nginx 在内存中运行多个进程; 有一个主进程和几个工作(worker)进程。 还有一些特殊用途的过程，特别是缓存加载器和缓存管理器。 所有进程都是单线程版本为 1.x 的 nginx。所有进程主要使用共享内存机制进行进程间通信。主进程作为 root 用户运行。 缓存加载器，缓存管理器和工作(worker)则以无权限用户运行。  

主程序负责以下任务：  

- 读取和验证配置  
- 创建，绑定和关闭套接字  
- 启动，终止和维护配置的工作(worker)进程数  
- 重新配置，无需中断服务  
- 控制不间断的二进制升级(如果需要，启动新的二进制并回滚)  
- 重新打开日志文件  
- 编译嵌入式 Perl 脚本  

工作(worker)进程接受，处理和处理来自客户端的连接，提供反向代理和过滤功能，并执行几乎所有其他的 nginx 能力。 关于监视 nginx 实例的行为，系统管理员应该关注工作(worker)进程，因为它们是反映 Web 服务器实际日常操作的过程。

缓存加载器进程负责检查磁盘缓存项目，并使用缓存元数据填充 nginx 的内存数据库。本质上，缓存加载器准备 nginx 实例来处理已经存储在磁盘上的特定分配的目录结构中的文件。 它遍历目录，检查缓存内容元数据，更新共享内存中的相关条目，然后在所有内容清洁并准备使用时退出。
缓存管理器主要负责缓存到期和无效。 在正常的 nginx 操作期间它保持在内存中，并且在失败的情况下由主进程重新启动。    

## 7. Nginx安装

**准备第三方支持库源码：**  

- nginx-1.16.1.tar.gz    //源文件
- openssl-1.1.1g.tar.gz //加密
- pcre-8.44.tar.gz     // 轻量级函数库
- zlib-1.2.11.tar.gz   // 解压缩

**先解压每个包**

```linux
$ tar xzvf nginx-1.16.1.tar.gz 
$ tar xzvf openssl-1.1.1g.tar.gz
$ tar xzvf pcre-8.44.tar.gz
$ tar xzvf zlib-1.2.11.tar.gz 
```

编译安装

```
$ cd nginx-1.16.1
$ ./configure --prefix=/usr/local/nginx --with-http_realip_module --
with-http_addition_module --with-http_gzip_static_module --withhttp_secure_link_module --with-http_stub_status_module --with-stream --
with-pcre=/home/wz/pcre-8.41 --withzlib=/home/wz/zlib-1.2.11 --withopenssl=/home/wz/openssl-1.1.0g

$ make
$ sudo make install
```

在/usr/local/目录下面，产生了 nginx 的目录  

**启动nginx**

```
$ ./sbin/nginx –c ./conf/nginx.conf  

使用 ps 查看nginx的进程
$ ps -aux|grep nginx
```

![image-20220105231414801](C:\Users\wangzhen\AppData\Roaming\Typora\typora-user-images\image-20220105231414801.png)

输入本地ip， 可以看到nginx的页面

![image-20220105225559073](C:\Users\wangzhen\AppData\Roaming\Typora\typora-user-images\image-20220105225559073.png)

## 8. Nginx的快速入门

nginx 有一个主进程和几个工作进程。 主进程的主要目的是读取和评估配置，并维护工作进程。 工作进程对请求进行实际处理。 nginx 采用基于事件的模型和依赖于操作系统的机制来有效地在工作进程之间分配请求。 工作进程的数量可在配置文件中定义，并且可以针对给定的配置进行修改，或者自动调整到可用 CPU 内核的数量在配置文件中确定 nginx 及其模块的工作方式。 默认情况下，配置文件名为 nginx.conf，并放在目录： /usr/local/nginx/conf, /etc/nginx, 或 /usr/local/etc/nginx 中。在前面安装文章配置中，使用的安装配置目录是： /usr/local/nginx/conf 。所以可以在这个目录下找到这个配置文件。 

**启动，停止和重新加载Nginx的配置** 

要启动 nginx，请运行可执行文件。 当 nginx 启动后，可以通过使用-s 参数调用可执行文件
来控制它。 使用以下语法：  

```
$ nginx -s signal
```

信号(signal)的值可能是以下之一：

- stop - 快速关闭服务
-  quit - 正常关闭服务
-  reload - 重新加载配置文件
- reopen - 重新打开日志文件  

**提供静态内容服务(静态网站)**  

一个重要的 Web 服务器任务是提供文件(如图像或静态 HTML 页面)。这里我们来学习如何实现一个示例，根据请求，文件将从不同的本地目录提供： /usr/local/nginx/html/www(包含HTML 文件)和/usr/local/nginx/html/images(包含图像)。这将需要编辑配置文件，并使用两个位置块在 http 块内设置服务器块。首先，创建/usr/local/nginx/html/www 目录，并将一个包含任何文本内容的 exmaple.html 文件放入其中，并创建/usr/local/nginx/html/images 目录并在其中放置一些图像。

```
root@ubuntu:/usr/local/nginx# mkdir -p /data/images
root@ubuntu:/usr/local/nginx#
```

分别在上面创建的两个目录中放入两个文件： /usr/local/nginx/html/www/exmaple.html 和 /
usr/local/nginx/html/1.png, 2.png, 3.png， /usr/local/nginx/html/ww 文件的内容就一行，如下

```html
<h2> Nginx Demo.</h2>   
```

接下来，打开配置文件(/usr/local/nginx/conf/nginx.conf)。 默认的配置文件已经包含了服务器块的几个示例，大部分是注释掉的。 现在注释掉所有这样的块，并启动一个新的服务器块：  

```
http {
	server {
	}
}
```

通常，配置文件可以包括服务器监听的端口和服务器名称区分的几个 server 块。当 nginx 决
定哪个服务器处理请求后，它会根据服务器块内部定义的 location 指令的参数测试请求头中
指定的 URI。将以下 location 块添加到服务器(server)块：

```
http {
	server {
		location / {
			root /usr/local/nginx/html/www;
			}
	}
}
```

该location 块指定与请求中的 URI 相比较的“/”前缀。 对于匹配请求， URI 将被添加到 root指令中指定的路径(即/data/www)，以形成本地文件系统上所请求文件的路径。 如果有几个匹配的 location 块， nginx 将选择具有最长前缀来匹配 location 块。 上面的 location 块提供最短的前缀长度为 1，因此只有当所有其他 location 块不能提供匹配时，才会使用该块。接下来，添加第二个 location 块：  

```
http {
	server {
		location / {
			root /usr/local/nginx/html/www;
		}
		location /images/ {
			root /usr/local/nginx/html;
		}
	}
}
```

它将是以/images/(位置/也匹配这样的请求，但具有较短前缀，也就是“/images/”比“/”长)的
请求来匹配。
server 块的最终配置应如下所示：  

```
server {
	location / {
		root /usr/local/nginx/html/www;
		}
	location /images/ {
		root /usr/local/nginx/html;
	}
}
```

这已经是一个在标准端口 80 上侦听并且可以在本地机器上访问的服务器( http://localhost/ )的工作配置。 响应以/images/开头的 URI 的请求，服务器将从/data/images 目录发送文件。例如，响应 http://192.168.199.133:8080/images/logo.jpg 请求， nginx 将发送服务上的/usr/local/nginx/html/images/logo.jpg 文件。 如果文件不存在， nginx 将发送一个指示 404 错误的响应。 不以/images/开头的 URI 的请求将映射到/usr/local/nginx/html/www 目录。 例如 ， 响 应 http://192.168.199.133:8080/example.html 请 求 时 ， nginx 将 发 送/usr/local/nginx/html/www /example.html 文件。要应用新配置，如果尚未启动 nginx 或者通过执行以下命令将重载信号发送到 nginx 的主进程： 

root@ubuntu:/usr/local/nginx# /usr/local/nginx/sbin/nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is
ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is
successful
root@ubuntu:/usr/local/nginx# /usr/local/nginx/sbin/nginx -s reload
如果错误或异常导致无法正常工作，可以尝试查看目录**/usr/local/nginx/logs** 或/**var/log/nginx中的 access.log 和 error.log** 文件中查找原因。打开浏览器或使用 CURL 访问 Nginx 服务器如下所示:   

![image-20220105231705456](C:\Users\wangzhen\AppData\Roaming\Typora\typora-user-images\image-20220105231705456.png)

完整的 nginx.conf 文件配置内容如下：  

```
events {
	worker_connections 1024;
}
http {
	server {
		listen 8080;
		server_name localhost;
		location / {
			root /usr/local/nginx/html/www;
		}
		location /images/ {
			root /usr/local/nginx/html;
		}
	}
}
```

# 二. Nginx 的数据结构

## 1. nginx_int_t

Nginx 使用ngx_int_t 封装有符号整型，使用nginx_unit_t封装无符号整型。

```c
typedef intptr_t        ngx_int_t;
typedef uintptr_t       ngx_uint_t;
typedef intptr_t        ngx_flag_t;
```

## 2. ngx_str_t

在nginx源码目录的src/core下面的ngx_string.h|c里面，包含了字符串的封装以及字符串相关操作的api。nginx提供了一个带长度的字符串结构ngx_str_t，它的原型如下：

```c
typedef struct {
    size_t      len;
    u_char     *data;
} ngx_str_t;
```

从结构体当中，data指向字符串数据的第一个字符，字符串的结束用长度来表示，而不是由’\0’来表示结束。所以，在写nginx代码时，处理字符串的方法跟我们平时使用有很大的不一样，但要时刻记住，字符串不以’\0’结束，尽量使用nginx提供的字符串操作的api来操作字符串。 那么，nginx这样做有什么好处呢？首先，通过长度来表示字符串长度，减少计算字符串长度的次数。其次，nginx可以重复引用一段字符串内存，data可以指向任意内存，长度表示结束，而不用去copy一份自己的字符串(因为如果要以’\0’结束，而不能更改原字符串，所以势必要copy一段字符串)。我们在ngx_http_request_t结构体的成员中，可以找到很多字符串引用一段内存的例子，比如request_line、uri、args等等，这些字符串的data部分，都是指向在接收数据时创建buffer所指向的内存中，uri，args就没有必要copy一份出来。这样的话，减少了很多不必要的内存分配与拷贝。 正是基于此特性，在nginx中，必须谨慎的去修改一个字符串。在修改字符串时需要认真的去考虑：是否可以修改该字符串；字符串修改后，是否会对其它的引用造成影响。在后面介绍ngx_unescape_uri函数的时候，就会看到这一点。但是，使用nginx的字符串会产生一些问题，glibc提供的很多系统api函数大多是通过’\0’来表示字符串的结束，所以我们在调用系统api时，就不能直接传入str->data了。此时，通常的做法是创建一段str->len + 1大小的内存，然后copy字符串，最后一个字节置为’\0’。比较hack的做法是，将字符串最后一个字符的后一个字符backup一个，然后设置为’\0’，在做完调用后，再由backup改回来，但前提条件是，你得确定这个字符是可以修改的，而且是有内存分配，不会越界，但一般不建议这么做。 接下来，看看nginx提供的操作字符串相关的api。

```c
#define ngx_string(str)     { sizeof(str) - 1, (u_char *) str }
```

ngx_string(str)是一个宏，它通过一个以’\0’结尾的普通字符串str构造一个nginx的字符串，鉴于其中采用sizeof操作符计算字符串长度，因此参数必须是一个常量字符串。

```c
#define ngx_null_string     { 0, NULL }
```

定义变量时，使用ngx_null_string初始化字符串为空字符串，符串的长度为0，data为NULL。

```c
#define ngx_str_set(str, text)      \
    (str)->len = sizeof(text) - 1; (str)->data = (u_char *) text

```

ngx_str_set用于设置字符串str为text，由于使用sizeof计算长度，故text必须为常量字符串。

```c
#define ngx_str_null(str)   (str)->len = 0; (str)->data = NULL
```

ngx_str_null用于设置字符串str为空串，长度为0，data为NULL。

上面这四个函数，使用时一定要小心，ngx_string与ngx_null_string是“{，}”格式的，故只能用于赋值时初始化，如：

```c
ngx_str_t str = ngx_string("hello world");
ngx_str_t str1 = ngx_null_string();
```

如果向下面这样使用，就会有问题，

```c
ngx_str_t str, str1;
str = ngx_string("hello world");    // 编译出错
str1 = ngx_null_string;                // 编译出错
```

这种情况，可以调用ngx_str_set与ngx_str_null这两个函数来做:

```c
ngx_str_t str, str1;
ngx_str_set(&str, "hello world");
ngx_str_null(&str1);
```

按照C99标准，您也可以这么做：

```c
ngx_str_t str, str1;
str  = (ngx_str_t) ngx_string("hello world");
str1 = (ngx_str_t) ngx_null_string;
```

另外要注意的是，ngx_string与ngx_str_set在调用时，传进去的字符串一定是常量字符串，否则会得到意想不到的错误(因为ngx_str_set内部使用了sizeof()，如果传入的是u_char*，那么计算的是这个指针的长度，而不是字符串的长度)。如：

```c
ngx_str_t str;
u_char *a = "hello world";
ngx_str_set(&str, a);    // 问题产生
```

```c
u_char * ngx_cdecl ngx_sprintf(u_char *buf, const char *fmt, ...);
u_char * ngx_cdecl ngx_snprintf(u_char *buf, size_t max, const char *fmt, ...);
u_char * ngx_cdecl ngx_slprintf(u_char *buf, u_char *last, const char *fmt, ...);
```

上面这三个函数用于字符串格式化，ngx_snprintf的第二个参数max指明buf的空间大小，ngx_slprintf则通过last来指明buf空间的大小。推荐使用第二个或第三个函数来格式化字符串，ngx_sprintf函数还是比较危险的，容易产生缓冲区溢出漏洞。在这一系列函数中，nginx在兼容glibc中格式化字符串的形式之外，还添加了一些方便格式化nginx类型的一些转义字符，比如%V用于格式化ngx_str_t结构。在nginx源文件的ngx_string.c中有说明：

```c
/*
 * supported formats:
 *    %[0][width][x][X]O        off_t
 *    %[0][width]T              time_t
 *    %[0][width][u][x|X]z      ssize_t/size_t
 *    %[0][width][u][x|X]d      int/u_int
 *    %[0][width][u][x|X]l      long
 *    %[0][width|m][u][x|X]i    ngx_int_t/ngx_uint_t
 *    %[0][width][u][x|X]D      int32_t/uint32_t
 *    %[0][width][u][x|X]L      int64_t/uint64_t
 *    %[0][width|m][u][x|X]A    ngx_atomic_int_t/ngx_atomic_uint_t
 *    %[0][width][.width]f      double, max valid number fits to %18.15f
 *    %P                        ngx_pid_t
 *    %M                        ngx_msec_t
 *    %r                        rlim_t
 *    %p                        void *
 *    %V                        ngx_str_t *
 *    %v                        ngx_variable_value_t *
 *    %s                        null-terminated string
 *    %*s                       length and string
 *    %Z                        '\0'
 *    %N                        '\n'
 *    %c                        char
 *    %%                        %
 *
 *  reserved:
 *    %t                        ptrdiff_t
 *    %S                        null-terminated wchar string
 *    %C                        wchar
 */

```

这里特别要提醒的是，我们最常用于格式化ngx_str_t结构，其对应的转义符是%V，传给函数的一定要是指针类型，否则程序就会coredump掉。这也是我们最容易犯的错。比如：

```c
ngx_str_t str = ngx_string("hello world");
char buffer[1024];
ngx_snprintf(buffer, 1024, "%V", &str);    // 注意，str取地址
```

## 3. ngx_buf_t

这个ngx_buf_t就是这个ngx_chain_t链表的每个节点的实际数据。该结构实际上是一种抽象的数据结构，它代表某种具体的数据。这个数据可能是指向内存中的某个缓冲区，也可能指向一个文件的某一部分，也可能是一些纯元数据（元数据的作用在于指示这个链表的读取者对读取的数据进行不同的处理）。该数据结构位于src/core/ngx_buf.h|c文件中。我们来看一下它的定义。

```c
struct ngx_buf_s {
    u_char          *pos;
    u_char          *last;
    off_t            file_pos;
    off_t            file_last;

    u_char          *start;         /* start of buffer */
    u_char          *end;           /* end of buffer */
    ngx_buf_tag_t    tag;
    ngx_file_t      *file;
    ngx_buf_t       *shadow;


    /* the buf's content could be changed */
    unsigned         temporary:1;

    /*
     * the buf's content is in a memory cache or in a read only memory
     * and must not be changed
     */
    unsigned         memory:1;

    /* the buf's content is mmap()ed and must not be changed */
    unsigned         mmap:1;

    unsigned         recycled:1;
    unsigned         in_file:1;
    unsigned         flush:1;
    unsigned         sync:1;
    unsigned         last_buf:1;
    unsigned         last_in_chain:1;

    unsigned         last_shadow:1;
    unsigned         temp_file:1;

    /* STUB */ int   num;
};
```

| **pos:**           | 当buf所指向的数据在内存里的时候，pos指向的是这段数据开始的位置。 |
| ------------------ | :----------------------------------------------------------- |
| **last:**          | 当buf所指向的数据在内存里的时候，last指向的是这段数据结束的位置。 |
| **file_pos:**      | 当buf所指向的数据是在文件里的时候，file_pos指向的是这段数据的开始位置在文件中的偏移量。 |
| **file_last:**     | 当buf所指向的数据是在文件里的时候，file_last指向的是这段数据的结束位置在文件中的偏移量。 |
| **start:**         | 当buf所指向的数据在内存里的时候，这一整块内存包含的内容可能被包含在多个buf中(比如在某段数据中间插入了其他的数据，这一块数据就需要被拆分开)。那么这些buf中的start和end都指向这一块内存的开始地址和结束地址。而pos和last指向本buf所实际包含的数据的开始和结尾。 |
| **end:**           | 解释参见start。                                              |
| **tag:**           | 实际上是一个void*类型的指针，使用者可以关联任意的对象上去，只要对使用者有意义。 |
| **file:**          | 当buf所包含的内容在文件中时，file字段指向对应的文件对象。    |
| **shadow:**        | 当这个buf完整copy了另外一个buf的所有字段的时候，那么这两个buf指向的实际上是同一块内存，或者是同一个文件的同一部分，此时这两个buf的shadow字段都是指向对方的。那么对于这样的两个buf，在释放的时候，就需要使用者特别小心，具体是由哪里释放，要提前考虑好，如果造成资源的多次释放，可能会造成程序崩溃！ |
| **temporary:**     | 为1时表示该buf所包含的内容是在一个用户创建的内存块中，并且可以被在filter处理的过程中进行变更，而不会造成问题。 |
| **memory:**        | 为1时表示该buf所包含的内容是在内存中，但是这些内容确不能被进行处理的filter进行变更。 |
| **mmap:**          | 为1时表示该buf所包含的内容是在内存中, 是通过mmap使用内存映射从文件中映射到内存中的，这些内容确不能被进行处理的filter进行变更。 |
| **recycled:**      | 可以回收的。也就是这个buf是可以被释放的。这个字段通常是配合shadow字段一起使用的，对于使用ngx_create_temp_buf 函数创建的buf，并且是另外一个buf的shadow，那么可以使用这个字段来标示这个buf是可以被释放的。 |
| **in_file:**       | 为1时表示该buf所包含的内容是在文件中。                       |
| **flush:**         | 遇到有flush字段被设置为1的的buf的chain，则该chain的数据即便不是最后结束的数据（last_buf被设置，标志所有要输出的内容都完了），也会进行输出，不会受postpone_output配置的限制，但是会受到发送速率等其他条件的限制。 |
| **sync:**          |                                                              |
| **last_buf:**      | 数据被以多个chain传递给了过滤器，此字段为1表明这是最后一个buf。 |
| **last_in_chain:** | 在当前的chain里面，此buf是最后一个。特别要注意的是last_in_chain的buf不一定是last_buf，但是last_buf的buf一定是last_in_chain的。这是因为数据会被以多个chain传递给某个filter模块。 |
| **last_shadow:**   | 在创建一个buf的shadow的时候，通常将新创建的一个buf的last_shadow置为1。 |
| **temp_file:**     | 由于受到内存使用的限制，有时候一些buf的内容需要被写到磁盘上的临时文件中去，那么这时，就设置此标志 。 |

对于此对象的创建，可以直接在某个ngx_pool_t上分配，然后根据需要，给对应的字段赋值。也可以使用定义好的2个宏：

```c
#define ngx_alloc_buf(pool)  ngx_palloc(pool, sizeof(ngx_buf_t))
#define ngx_calloc_buf(pool) ngx_pcalloc(pool, sizeof(ngx_buf_t))
```

对于创建temporary字段为1的buf（就是其内容可以被后续的filter模块进行修改），可以直接使用函数ngx_create_temp_buf进行创建;

```c
ngx_buf_t *ngx_create_temp_buf(ngx_pool_t *pool, size_t size);
```

该函数创建一个ngx_but_t类型的对象，并返回指向这个对象的指针，创建失败返回NULL。对于创建的这个对象，它的start和end指向新分配内存开始和结束的地方。pos和last都指向这块新分配内存的开始处，这样，后续的操作可以在这块新分配的内存上存入数据。

| **pool:** | 分配该buf和buf使用的内存所使用的pool。 |
| --------- | -------------------------------------- |
| **size:** | 该buf使用的内存的大小。                |

为了配合对ngx_buf_t的使用，nginx定义了以下的宏方便操作。

```c
/*返回这个buf里面的内容是否在内存里。*/
#define ngx_buf_in_memory(b)        (b->temporary || b->memory || b->mmap)


/*返回这个buf里面的内容是否仅仅在内存里，并且没有在文件里。*/
#define ngx_buf_in_memory_only(b)   (ngx_buf_in_memory(b) && !b->in_file)

/*返回该buf是否是一个特殊的buf，只含有特殊的标志和没有包含真正的数据。*/
#define   ngx_buf_special(b)    \
    ((b->flush || b->last_buf || b->sync) \
     && !ngx_buf_in_memory(b) && !b->in_file)

/*返回该buf是否是一个只包含sync标志而不包含真正数据的特殊buf。*/
#define ngx_buf_sync_only(b)          \
    (b->sync      	\
     && !ngx_buf_in_memory(b) && !b->in_file && !b->flush && !b->last_buf)

/*返回该buf所含数据的大小，不管这个数据是在文件里还是在内存里。*/
#define     ngx_buf_size(b)     \
    (ngx_buf_in_memory(b) ? (off_t) (b->last - b->pos): \
                            (b->file_last - b->file_pos))
```

## 4. ngx_list_t

ngx_list_t是Nginx封装的链表容器，它在Nginx中使用的很频繁，例如HTTP的头部就是用ngx_list_t存储的。

```c
typedef struct ngx_list_part_s  ngx_list_part_t;

struct ngx_list_part_s {
    void             *elts;
    ngx_uint_t        nelts;
    ngx_list_part_t  *next;
};


typedef struct {
    ngx_list_part_t  *last;
    ngx_list_part_t   part;
    size_t            size;
    ngx_uint_t        nalloc;
    ngx_pool_t       *pool;
} ngx_list_t;
```

 `ngx_list_t` 描述整个链表，而 `ngx_list_part_t` 只描述链表的一个元素，但是它存储的是一个数组，拥有连续的内存，他需要依赖`ngx_list_t` 里面的`size`和`nalloc` 来表示数组的容量，同时依靠`ngx_list_part_t` 成员中的`nelts`来表示数组当前已使用了多少容量。这样设计有什么好处呢？

- 链表中存储的元素是灵活的，它可以是任何一种数据结构
- 链表元素需要占用的内存由ngx_list_t管理，它可通过数组分配好了
- 小块的内存使用链表访问效率是低下的，使用数组通过偏移量来直接访问内存则要高效得多。

**ngx_list_t**

- part: 链表的首个数组元素
- last: 指向链表的最后一个数组元素
- size: 链表每个`ngx_list_part_t` 元素都是一个数组，size表示每个数组元素占用的空间大小
- nalloc: 链表的数组元素一旦分配后是不可更改的。表示的是`ngx_list_part_t` 数组的容量
- pool: 链表中管理内存分配的内存池对象

**ngx_list_part_t**

- elts: 指向数组的起始地址
- nelts: 表示数组中已经使用了多少个元素，nelts必须小于nalloc
- next: 下一个链表元素的`ngx_list_part_t`的地址

事实上，ngx_list_t中的所有数据都是由ngx_pool_t类型的pool内存池分配的，他们通常是连续的内存。

![image-20220110230252088](C:\Users\wangzhen\AppData\Roaming\Typora\typora-user-images\image-20220110230252088.png)

上图是由3个`ngx_list_part_t` 数组元素组成的 `ngx_list_t` 链表可能拥有的一种内存分布结构。pool内存池为其分配了连续的内存，最前端内存存储的是`ngx_list_t` 结构体的成员，紧接着是第一个`ngx_list_part_t`结构占用的内存，然后是`ngx_list_part_t`结构指向的数组，他们一共占用size*nalloc字节，表示数组中拥有nalloc个大小为size的元素，其后面是第2个`ngx_list_part_t`结构以及它所指向的数组。以此类推。

## 5. ngx_table_elt_t

```c
typedef struct {
    ngx_uint_t        hash;
    ngx_str_t         key;
    ngx_str_t         value;
    u_char           *lowcase_key;
} ngx_table_elt_t;
```

`ngx_table_elt_t` 就是一个key/value对， ngx_str_t类型的key、value成员分别存储的是名字、值字符串。hash成员表明`ngx_table_elt_t` 也可以是某个散列表数据结构(ngx_hash_t类型)中的成员。lowcase_key指向的是全小写的key字符串。

# 三. Nginx 高级数据结构

Nginx的高级数据包括`ngx_queue_t, ngx_array_t, ngx_list_t, ngx_rbtree_t, ngx_radix_tree_t, ngx_hash_t`。

## 1. ngx_queue_t

`ngx_queue_t `双向链表是Nginx提供的轻量级链表容器，与Nginx的内存池无关，因此这个链表不会负责分配内存来存放元素，这个数据结构仅仅把已经分配好的内存元素使用双向链表连接起来，所以对每个用户来说，只需要增加两个指针的空间即可，消耗的内存很少。同时，`ngx_queue_t `还提供了一个非常简易的插入排序法。相对于Nginx其他顺序容器，它的优势在于:

-  实现了排序功能
- 非常轻量级，是一个纯粹的双向链表，它不负责链表元素所占内存的分配，与Nginx封装的`ngx_pool_t`内存池完全无关。
- 支持两个链表间的合并

```c
typedef struct ngx_queue_s  ngx_queue_t;

struct ngx_queue_s {
    ngx_queue_t  *prev;
    ngx_queue_t  *next;
};
```

它的实现非常简单，仅有两个成员prev, next。 

**实现访问API:**

```      c
ngx_queue_init(q)                  //初始化链表  q为空
ngx_queue_empty(h)                 //检测链表是否为空
ngx_queue_insert_head(h, x)        //将x插入到h的头部
ngx_queue_insert_tail(h, x)        //将x插入到h的尾部
ngx_queue_head(h)                  //返回链表h中的第一个元素的结构体指针
ngx_queue_last(h)                  //返回链表h中的最后一个元素的结构体指针
ngx_queue_sentinel(h)              //返回链表容器结构体的指针
ngx_queue_remove(x)                //移除x结构体元素
ngx_queue_split(h, q, n)           // 以元素q为界分为h和n两个链表，其中h是前半部分(不包括q)，n是后半部分(包括q)
ngx_queue_add(h, n)                //合并链表，将n添加到h的尾部
ngx_queue_middle(h)                //返回链表的中心元素，假如有N个，返回N/2+1个元素
ngx_queue_sort(h, compfunc)        //使用插入排序对链表进行排序，compfunc需要自己实现
```

对于链表中的每一个元素，其类型可以是任意的struct结构体，但这个结构体中必须要有一个ngx_queue_t 类型的成员，在向链表容器中添加、删除时都是使用结构体中的ngx_queue_t类型成员的指针。当ngx_queue_t作为链表的元素成员使用时，他具有下面4种用法:

```c
ngx_queue_next(q)                  //返回链表q的下一个元素
ngx_queue_prev(q)                  //返回q的上一个元素
ngx_queue_data(q, type, link)      //返回q元素所属结构体中可在任意位置包含ngx_queue_t的地址
ngx_queue_insert_after(q, x)       //和ngx_queue_insert_head相同，因为是双向链表，没有头尾之分
```

**Test SampleCode:**

```c
#include <stdio.h>
#include <string.h>

#include "ngx_config.h"

#include "ngx_core.h"


#include "ngx_list.h"
#include "ngx_palloc.h"
#include "ngx_string.h"
#include "ngx_queue.h"

ngx_queue_t queueContainer;

typedef struct {
    u_char *str;
    ngx_queue_t qEle;
    int num;
}TestNode;

ngx_int_t compTestNode(const ngx_queue_t* a, const ngx_queue_t *b) {
    TestNode *aNode = ngx_queue_data(a, TestNode, qEle);
    TestNode *bNode = ngx_queue_data(b, TestNode, qEle);

    return aNode->num > bNode->num;
}

volatile ngx_cycle_t *ngx_cycle;
 
void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log,
			ngx_err_t err, const char *fmt, ...)
{



}

int main() {
    TestNode node[5];

    ngx_queue_init(&queueContainer);

    for (int i = 0; i < 5; i++) {
        node[i].num = i;
        
    }
    ngx_queue_insert_tail(&queueContainer, &node[1].qEle);
    ngx_queue_insert_tail(&queueContainer, &node[4].qEle);
    ngx_queue_insert_tail(&queueContainer, &node[0].qEle);
    ngx_queue_insert_tail(&queueContainer, &node[2].qEle);
    ngx_queue_insert_tail(&queueContainer, &node[3].qEle);

    ngx_queue_sort(&queueContainer, compTestNode);

    ngx_queue_t *q;
    for (q = ngx_queue_head(&queueContainer); q != ngx_queue_sentinel(&queueContainer); q = ngx_queue_next(q)) {
        TestNode *eleNode = ngx_queue_data(q, TestNode, qEle);
        printf("%d\n", eleNode->num);
    }

    return 0;
}
```

**编译:**

```tex
gcc -o ngx_queue_main ngx_queue_main.c  -I ../../nginx-1.16.1/src/core/ -I ../../nginx-1.16.1/objs/  -I ../../nginx-1.16.1/src/os/unix/ -I ../../pcre-8.44/ -I ../../nginx-1.16.1/src/event/ ../../nginx-1.16.1/objs/src/core/ngx_list.o ../../nginx-1.16.1/objs/src/core/ngx_string.o ../../nginx-1.16.1/objs/src/core/ngx_palloc.o ../../nginx-1.16.1/objs/src/os/unix/ngx_alloc.o../../nginx-1.16.1/objs/src/core/ngx_queue.o
```

## 2. ngx_array_t

`ngx_array_t` 是一个顺序的动态数组，在Nginx大量使用，支持达到数组容量上限时动态改变数组的大小。`ngx_array_t` 和C++ STL中的vector很类似，它内置了Nginx封装的内存池，因此，他分配的内存可以在内存池中申请得到。具备下面几个特点:

- 访问速度快
- 允许元素个数具备不确定性
- 负责元素占用内存的分配，这些内存由内存池统一管理

```c
typedef struct {
    void        *elts;    //指向数组的首地址
    ngx_uint_t   nelts;   //数组中已经使用的元素个数
    size_t       size;   //每个元素占用的内存大小
    ngx_uint_t   nalloc; // 当前数组中能够容纳元素个数的总大小
    ngx_pool_t  *pool;    //内存池对象
} ngx_array_t;
```

提供了如下API:

```c
ngx_array_init(ngx_array_t *array, ngx_pool_t *pool, ngx_uint_t n, size_t size); //初始化1个已经存在的动态数组，并预分配n个大小为size的内存空间
ngx_array_create(ngx_pool_t *p, ngx_uint_t n, size_t size); //创建1个动态数组，并预分配n个大小为size的内存空间
ngx_array_destroy(ngx_array_t *a);                     // 销毁数组 和create配对使用
ngx_array_push(ngx_array_t *a);                        // 向当前a动态数组中添加1个元素，返回的是这个新添加元素的地址
ngx_array_push_n(ngx_array_t *a, ngx_uint_t n);        //向当前a动态数组中添加n个元素，返回的是新添加这批元素中第一个元素的地址
```

**动态数组的扩容方式:**

`ngx_array_push` 和`ngx_array_push_n`都可能引发扩容操作，`ngx_array_push` 方法将会申请 `ngx_array_t`结构体中size字节的大小，而`ngx_array_push_n`方法将会申请n个size字节大小的内存。每次扩容的大小将受制于内存池的以下两种情形:

-  如果当前内存池中剩余的空间的大于或者等于本次需要新增的空间，那么本次扩容将只扩容新增的空间。例如: 对于`ngx_array_push`来说，就是扩充1个元素，而对于`ngx_array_push_n`来说，就是扩充n个元素。
- 如果当前内存池中剩余的空间小于本次需要新增的空间，那么对`ngx_array_push`方法来说，会将原先动态数组的容量扩容一倍，而对于`ngx_array_push_n` 来说，如果参数`n`小于原先动态数组的容量，将会扩容一倍；如果参数`n`大于原先动态数组的容量，这时会分配`2n`大小的空间，扩容超过一倍，此时动态数组会被分配在全新的内存块上，会把原先的元素复制到新的动态数组中。

SampleCode:

```c
#include <stdio.h>
#include <string.h>

#include "ngx_config.h"

#include "ngx_core.h"


#include "ngx_list.h"
#include "ngx_palloc.h"
#include "ngx_string.h"



#define N		10

typedef struct _key {
	int id;
	char name[32];
} Key;

volatile ngx_cycle_t *ngx_cycle;
 
void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log,
			ngx_err_t err, const char *fmt, ...)
{



}


void print_array(ngx_array_t *array) {

	Key *key = array->elts;

	int i = 0;
	for (i = 0;i < array->nelts;i ++) {
		printf("%s .\n", key[i].name);
	}

}

int main() {

	ngx_pool_t *pool = ngx_create_pool(1024, NULL);

	ngx_array_t *array = ngx_array_create(pool, N, sizeof(Key));

	int i = 0;
	Key *key = NULL;
	for (i = 0;i < 25;i ++) {

		key = ngx_array_push(array);
		key->id = i+1;
		sprintf(key->name, "array %d", key->id);
		
	}

	key = ngx_array_push_n(array, 10);
	for (i = 0;i < 10;i ++) {
		key[i].id = 25+i+1;
		sprintf(key[i].name, "array %d", key[i].id);
	}

	print_array(array);

}
```

编译:

```c
gcc -o ngx_array_main ngx_array_main.c -I ../../nginx-1.16.1/src/core/ -I ../../nginx-1.16.1/objs/  -I ../../nginx-1.16.1/src/os/unix/ -I ../../pcre-8.44/ -I ../../nginx-1.16.1/src/event/  ../../nginx-1.16.1/objs/src/core/ngx_string.o ../../nginx-1.16.1/objs/src/core/ngx_palloc.o ../../nginx-1.16.1/objs/src/os/unix/ngx_alloc.o  ../../nginx-1.16.1/objs/src/core/ngx_array.o
```

## 3. ngx_rbtree_t

`ngx_rbtree_t`是使用红黑树实现的一种关联容器，Nginx模块的核心(定时器管理、文件缓存模块等)在需要快速检索、查找的场合下都使用了`ngx_rbtree_t`容器。

```c
typedef struct ngx_rbtree_node_s  ngx_rbtree_node_t;

struct ngx_rbtree_node_s {
    ngx_rbtree_key_t       key;              //uint的关键字
    ngx_rbtree_node_t     *left;             //左子节点
    ngx_rbtree_node_t     *right;           //右子节点
    ngx_rbtree_node_t     *parent;           //父节点
    u_char                 color;           //节点的颜色， 0:黑色 1:红色
    u_char                 data;           //1字节的节点数据 
};


typedef struct ngx_rbtree_s  ngx_rbtree_t;
//为解决不同节点含有相同关键字的元素冲突问题，红黑树设置了ngx_rbtree_insert_pt指针，这样可灵活地添加冲突元素
typedef void (*ngx_rbtree_insert_pt) (ngx_rbtree_node_t *root,
    ngx_rbtree_node_t *node, ngx_rbtree_node_t *sentinel);

struct ngx_rbtree_s {
    ngx_rbtree_node_t     *root; //指向树的根节点
    ngx_rbtree_node_t     *sentinel; //指向NULL哨兵节点
    ngx_rbtree_insert_pt   insert;  //表示红黑树添加元素的函数指针，它决定在添加新节点时的行为究竟是替换还是新增
};
```

**添加数据的三种API:**

```c
//向红黑树添加数据节点，每个数据节点的关键字都是唯一的，不存在同一个关键字有多个节点的问题
ngx_rbtree_insert(ngx_rbtree_t *tree, ngx_rbtree_node_t *node);  
//添加数据节点的关键字可以不是唯一的，但它们是以字符串作为唯一的标识，存放在ngx_str_node_t结构体的str成员中
ngx_rbtree_insert_value(ngx_rbtree_node_t *root, ngx_rbtree_node_t *node, ngx_rbtree_node_t *sentinel);
//添加的数据节点的关键字表示时间或者时间差
ngx_rbtree_insert_timer_value(ngx_rbtree_node_t *root, ngx_rbtree_node_t *node, ngx_rbtree_node_t *sentinel);
```

红黑树的API:

```c
ngx_rbtree_init(tree, s, i)   //初始化红黑树
ngx_rbtree_insert(ngx_rbtree_t *tree, ngx_rbtree_node_t *node);   //向红黑树添加节点
ngx_rbtree_delete(ngx_rbtree_t *tree, ngx_rbtree_node_t *node);  //从红黑树中删除节点
```

在初始化红黑树的时候，需要先分配好保存在红黑树的`ngx_rbtree_t`结构体，以及`ngx_rbtree_node_t`类型的哨兵节点，并选择或者自定义`ngx_rbree_insert_pt`类型的节点添加函数。

Samplecode:

```c
#include <stdio.h>
#include <string.h>

#include "ngx_config.h"

#include "ngx_core.h"


#include "ngx_list.h"
#include "ngx_palloc.h"
#include "ngx_string.h"
#include "ngx_rbtree.h"


volatile ngx_cycle_t *ngx_cycle;
 
void ngx_log_error_core(ngx_uint_t level, ngx_log_t *log,
			ngx_err_t err, const char *fmt, ...)
{



}


int main() {

	ngx_rbtree_t rbtree;
	ngx_rbtree_node_t sentinel;
	
	ngx_rbtree_init(&rbtree, &sentinel, ngx_str_rbtree_insert_value);

	ngx_str_node_t strnode[10];
	ngx_str_set(&strnode[0].str, "he");
	ngx_str_set(&strnode[1].str, "jon");
	ngx_str_set(&strnode[2].str, "Issac");
	ngx_str_set(&strnode[3].str, "tursom");
	ngx_str_set(&strnode[4].str, "will");

	ngx_str_set(&strnode[5].str, "birate");
	ngx_str_set(&strnode[6].str, "ren");
	ngx_str_set(&strnode[7].str, "stephen");
	ngx_str_set(&strnode[8].str, "ultimate");
	ngx_str_set(&strnode[9].str, "he");

	int i = 0;
	for (i = 0;i < 10;i ++) {
		strnode[i].node.key = i;
		ngx_rbtree_insert(&rbtree, &strnode[i].node);
	}

	ngx_str_t str = ngx_string("will");
	
	ngx_str_node_t *node = ngx_str_rbtree_lookup(&rbtree, &str, 0);
	if (node != NULL) {
		printf(" Exit\n");
	}
}
```

编译:

```c
gcc -o ngx_array_main ngx_array_main.c -I ../../nginx-1.16.1/src/core/ -I ../../nginx-1.16.1/objs/  -I ../../nginx-1.16.1/src/os/unix/ -I ../../pcre-8.44/ -I ../../nginx-1.16.1/src/event/  ../../nginx-1.16.1/objs/src/core/ngx_string.o ../../nginx-1.16.1/objs/src/core/ngx_palloc.o ../../nginx-1.16.1/objs/src/os/unix/ngx_alloc.o  ../../nginx-1.16.1/objs/src/os/unix/ngx_rbtree.o 
```

## 4. ngx_hash_t 

`ngx_hash_t `是 nginx 自己的 hash 表的实现。定义和实现位于 `src/core/ngx_hash.h|c` 中。`ngx_hash_t `的实现也与数据结构教科书上所描述的 hash 表的实现是大同小异。对于常用的解决冲突的方法有线性探测，二次探测和开链法等。 `ngx_hash_t` 使用的是最常用的一种，也就是开链法，这也是 STL 中的 hash 表使用的方法  。但是 ngx_hash_t 的实现又有其几个显著的特点:  

- `ngx_hash_t `不像其他的 hash 表的实现，可以插入删除元素，它只能一次初始化，就构建起整个 hash 表以后，既不能再删除，也不能在插入元素了。  
- `ngx_hash_t` 的开链并不是真的开了一个链表，实际上是开了一段连续的存储空间，几乎可以看做是一个数组。这是因为` ngx_hash_t `在初始化的时候，会经历一次预计算的过程，提前把每个桶里面会有多少元素放进去给计算出来，这样就提前知道每个桶的大小了。那么就不需要使用链表，一段连续的存储空间就足够了。这也从一定程度上节省了内存的使用。

从上面的描述，我们可以看出来，这个值越大，越造成内存的浪费。就两步，首先是初始化，然后就可以在里面进行查找了。下面我们详细来看一下。  

```c
//ngx_hash_t 的初始化
ngx_int_t ngx_hash_init(ngx_hash_init_t *hinit, ngx_hash_key_t *names, ngx_uint_t nelts);
```

首先我们来看一下初始化函数。该函数的第一个参数 hinit 是初始化的一些参数的一个集合。 names 是初始化一个 `ngx_hash_t `所需要的所有 key 的一个数组。而 nelts 就是 key 的个数。下面先看一下 `ngx_hash_init_t `类型，该类型提供了初始化一个 hash 表所需要的一些基本信息。  

```c
typedef struct {
    ngx_hash_t       *hash;
    ngx_hash_key_pt   key;

    ngx_uint_t        max_size;
    ngx_uint_t        bucket_size;

    char             *name;
    ngx_pool_t       *pool;
    ngx_pool_t       *temp_pool;
} ngx_hash_init_t;

/*
hash: 该字段如果为 NULL，那么调用完初始化函数后，该字段指向新创建出来的hash 表。如果该字段不为 NULL，那么在初始的时候，所有的数据被插入了这个字段所指的      hash 表中。
key：指向从字符串生成 hash 值的 hash 函数。 nginx 的源代码中提供了默认的实现函数 ngx_hash_key_lc。
max_size：hash 表中的桶的个数。该字段越大，元素存储时冲突的可能性越小，每个桶中存储的元素会更少，则查询起来的速度更快。当然，这个值越大，越造成内存的浪费              也越大， (实际上也浪费不了多少)。
bucket_size：每个桶的最大限制大小，单位是字节。如果在初始化一个 hash 表的时候，发现某个桶里面无法存的下所有属于该桶的元素，则 hash 表初始化失败。
name： 该 hash 表的名字。
pool：该 hash 表分配内存使用的 pool。
temp_pool：该 hash 表使用的临时 pool，在初始化完成以后，该 pool 可以被释放和销毁掉。
*/
```

下面来看一下存储 hash 表 key 的数组的结构 。

```c
typedef struct {
    ngx_str_t         key;
    ngx_uint_t        key_hash;
    void             *value;
} ngx_hash_key_t;
```

key 和 value 的含义显而易见，就不用解释了。 key_hash 是对 key 使用 hash 函数计算出来的值。 对这两个结构分析完成以后，我想大家应该都已经明白这个函数应该是如何使用了吧。该函数成功初始化一个 hash 表以后，返回 `NGX_OK`，否则返回 `NGX_ERROR`。  

```c
//在 hash 里面查找 key 对应的 value。实际上这里的 key 是对真正的 key（也就是 name）计算出的 hash 值。 len 是 name 的长度。
//如果查找成功，则返回指向 value 的指针，否则返回 NULL
void *ngx_hash_find(ngx_hash_t *hash, ngx_uint_t key, u_char *name, size_t len);
```

## 5 ngx_hash_wildcard_t  

nginx 为了处理带有通配符的域名的匹配问题，实现了 ngx_hash_wildcard_t 这样的 hash 表。  他可以支持两种类型的带有通配符的域名。一种是通配符在前的，例如： `“ *.abc.com ”`，也可以省略掉星号，直接写成`.abc.com`。这样的 key，可以匹配 `www.abc.com`， `qqq.www.abc.com `之类的。另外一种是通配符在末尾的，例如： `“mail.xxx.* ”`，请特别注意通配符在末尾的不像位于开始的通配符可以被省略掉。这样的通配符，可以匹配 `mail.xxx.com`、` mail.xxx.com.cn`、`mail.xxx.net `之类的域名。

有一点必须说明，就是一个 `ngx_hash_wildcard_t `类型的 hash 表只能包含通配符在前的 key或者是通配符在后的 key。不能同时包含两种类型的通配符的 key。 `ngx_hash_wildcard_t `类型 变 量 的 构 建 是 通 过 函 数 `ngx_hash_wildcard_init` 完 成 的 ， 而 查 询 是 通 过 函 数`ngx_hash_find_wc_head` 或者 `ngx_hash_find_wc_tail `来做的。 `ngx_hash_find_wc_head `是查询包含通配符在前的 key 的 hash 表的，而 `ngx_hash_find_wc_tail `是查询包含通配符在后的 key的 hash 表的。  

下面详细说明这几个函数的用法:

```c
ngx_int_t ngx_hash_wildcard_init(ngx_hash_init_t *hinit, ngx_hash_key_t *names, ngx_uint_t nelts);
/*
hinit:  构造一个通配符 hash 表的一些参数的一个集合。
names:  构造此 hash 表的所有的通配符 key 的数组。特别要注意的是这里的 key 已经都是被预处理过的。例如： “*.abc.com”或者“.abc.com”被预处理完成以后，变成了“com.abc.”。而“mail.xxx.*”则被预处理为“mail.xxx.”。为什么会被处理这样？这里不得不简单地描述一下通配符 hash 表的实现原理。当构造此类型的 hash 表的时候，实际上是构造了一个 hash 表的一个“链表”，是通过 hash 表中的 key“链接”起来的。比如：对于“*.abc.com”将会构造出 2 个 hash 表，第一个 hash 表中有一个 key 为com 的表项，该表项的 value 包含有指向第二个 hash 表的指针，而第二个 hash 表中有一个表项 abc，该表项的 value 包含有指向*.abc.com 对应的 value 的指针。那么查询的时候，比如查询 www.abc.com 的时候，先查 com，通过查 com 可以找第二级的 hash 表，在第二级 hash 表中，再查找 abc，依次类推，直到在某一级的hash 表中查到的表项对应的 value 对应一个真正的值而非一个指向下一级 hash 表的指针的时候，查询过程结束。 这里有一点需要特别注意的，就是 names 数组中元素的 value 值低两位 bit 必须为 0（有特殊用途）。如果不满足这个条件，这个hash 表查询不出正确结果。
nelts: names 数组元素的个数。
*/

// 该函数执行成功返回 NGX_OK，否则 NGX_ERROR。
```

```c
//该函数查询包含通配符在前的 key 的 hash 表的。
void *ngx_hash_find_wc_head(ngx_hash_wildcard_t *hwc, u_char *name, size_t len);
/*
hwc: hash 表对象的指针。
name: 需要查询的域名，例如: www.abc.com。
len: name 的长度
*/
```

## 6. ngx_hash_combined_t  

组合类型 hash 表，该 hash 表的定义如下 :

```c
typedef struct {
    ngx_hash_t            hash;
    ngx_hash_wildcard_t  *wc_head;
    ngx_hash_wildcard_t  *wc_tail;
} ngx_hash_combined_t;
```

从其定义显见，该类型实际上包含了三个 hash 表，一个普通 hash 表，一个包含前向通配符的 hash 表和一个包含后向通配符的 hash 表。  nginx 提供该类型的作用，在于提供一个方便的容器包含三个类型的 hash 表，当有包含通配符的和不包含通配符的一组 key 构建 hash 表以后，以一种方便的方式来查询，你不需
要再考虑一个 key 到底是应该到哪个类型的 hash 表里去查了。构造这样一组合 hash 表的时候，首先定义一个该类型的变量，再分别构造其包含的三
个子 hash 表即可。对于该类型 hash 表的查询， nginx 提供了一个方便的函数 `ngx_hash_find_combined`。  

```c
void *ngx_hash_find_combined(ngx_hash_combined_t *hash, ngx_uint_t key, u_char *name, size_t len);
/*
hash: 此组合 hash 表对象
key : 根据 name 计算出的 hash 值
name: key的具体内容
len: name的长度
*/
```

该函数在此组合 hash 表中，依次查询其三个子 hash 表，看是否匹配，一旦找到，立即返回查找结果，也就是说如果有多个可能匹配，则只返回第一个匹配的结果。  返回查询的结果，未查到则返回 NULL。  

## 7. ngx_hash_keys_arrays_t  

大家看到在构建一个 `ngx_hash_wildcard_t `的时候，需要对通配符的哪些 key 进行预处理。这个处理起来比较麻烦。而当有一组 key，这些里面既有无通配符的 key，也有包含通配符的 key 的时候。我们就需要构建三个 hash 表，一个包含普通的 key 的 hash 表，一个包含前向通配符的 hash 表，一个包含后向通配符的 hash 表（或者也可以把这三个 hash 表组合成一个 `ngx_hash_combined_t`）。在这种情况下，为了让大家方便的构造这些 hash 表， nginx提供给了此辅助类型。该类型以及相关的操作函数也定义在 `src/core/ngx_hash.h|c` 里。我们先来看一下该类型的定义。  

```c
typedef struct {
    ngx_uint_t        hsize;

    ngx_pool_t       *pool;
    ngx_pool_t       *temp_pool;

    ngx_array_t       keys;
    ngx_array_t      *keys_hash;

    ngx_array_t       dns_wc_head;
    ngx_array_t      *dns_wc_head_hash;

    ngx_array_t       dns_wc_tail;
    ngx_array_t      *dns_wc_tail_hash;
} ngx_hash_keys_arrays_t;

/*
hsize : 将要构建的 hash 表的桶的个数。对于使用这个结构中包含的信息构建的三种类型的 hash 表都会使用此参数。
pool: 构建这些 hash 表使用的 pool。
temp_pool: 在构建这个类型以及最终的三个 hash 表过程中可能用到临时 pool。该 temp_pool 可以在构建完成以后，被销毁掉。这里只是存放临时的一些内存消耗。
keys : 存放所有非通配符 key 的数组。
keys_hash: 这是个二维数组，第一个维度代表的是 bucket 的编号，那么keys_hash[i]中存放的是所有的 key 算出来的 hash 值对 hsize 取模以后的值为 i 的 key。假设有 3 个 key,分别是 key1,key2 和 key3 假设 hash 值算出来以后对 hsize 取模的值都是 i，那么这三个 key 的值就顺序存在 keys_hash[i][0],keys_hash[i][1], keys_hash[i][2]。该值在调用的过程中用来保存和检测是否有冲突的 key 值，也就是是否有重复。
dns_wc_head: 放前向通配符 key 被处理完成以后的值。比如： “*.abc.com” 被处理完成以后，变成 “com.abc.” 被存放在此数组中。
dns_wc_tail: 存放后向通配符 key 被处理完成以后的值。比如： “mail.xxx.*” 被处理完成以后，变成 “mail.xxx.” 被存放在此数组中。
dns_wc_head_hash: 该值在调用的过程中用来保存和检测是否有冲突的前向通配符的 key值，也就是是否有重复。
dns_wc_tail_hash: 该值在调用的过程中用来保存和检测是否有冲突的后向通配符的 key值，也就是是否有重复。
*/
```

在定义一个这个类型的变量，并对字段 pool 和 temp_pool 赋值以后，就可以调用函数`ngx_hash_add_key` 把所有的 key 加入到这个结构中了，该函数会自动实现普通 key，带前向通配符的 key 和带后向通配符的 key 的分类和检查，并将这个些值存放到对应的字段中去，然后就可以通过检查这个结构体中的 keys、 dns_wc_head、 dns_wc_tail 三个数组是否为空，来决定是否构建普通 hash 表，前向通配符 hash 表和后向通配符 hash 表了（在构建这三个类型的 hash 表的时候，可以分别使用 keys、 dns_wc_head、 dns_wc_tail 三个数组）。构建出这三个 hash 表以后，可以组合在一个 `ngx_hash_combined_t` 对象中，使用
`ngx_hash_find_combined` 进行查找。或者是仍然保持三个独立的变量对应这三个 hash 表，自己决定何时以及在哪个 hash 表中进行查询。  

```c
//一般是循环调用这个函数，把一组键值对加入到这个结构体中。返回 NGX_OK 是加入成功。
//返回 NGX_BUSY 意味着 key 值重复。
ngx_int_t ngx_hash_add_key(ngx_hash_keys_arrays_t *ha, ngx_str_t *key,
    void *value, ngx_uint_t flags);

/*
ha :   该结构的对象指针
key:   键值
value: 值
flags: 有两个标志位可以设置， NGX_HASH_WILDCARD_KEY 和 NGX_HASH_READONLY_KEY。同时要设置的使用逻辑与操作符就可以了。 NGX_HASH_READONLY_KEY 被设置的时候 ， 在 计 算 hash 值 的 时 候 ， key 的 值 不 会 被 转 成 小 写 字 符 ， 否 则 会 。NGX_HASH_WILDCARD_KEY 被设置的时候，说明 key 里面可能含有通配符，会进行相应的处理。如果两个标志位都不设置，传 0
*/
```



# 四 Nginx的配置指令和handler模块概述

nginx 的配置系统由一个主配置文件和其他一些辅助的配置文件构成。这些配置文件均是纯文本文件，全部位于 nginx 安装目录下的 conf 目录下 。配置文件中以#开始的行，或者是前面有若干空格或者 TAB，然后再跟#的行，都被认为是注释，也就是只对编辑查看文件的用户有意义，程序在读取这些注释行的时候，其实际的内容是被忽略的。由于除主配置文件 nginx.conf 以外的文件都是在某些情况下才使用的，而只有主配置文件是在任何情况下都被使用的。所以在这里我们就以主配置文件为例，来解释 nginx 的配置系统。在 nginx.conf 中，包含若干配置项。每个配置项由配置**指令**和**指令参数** 2 个部分构成。指令参数也就是配置指令对应的配置值。  

**配置指令是一个字符串，可以用单引号或者双引号括起来，也可以不括。但是如果配置指令包含空格，一定要引起来。**  

## 1. 指令参数

指令的参数使用一个或者多个空格或者 TAB 字符与指令分开。指令的参数有一个或者多个TOKEN 串组成。 TOKEN 串之间由空格或者 TAB 键分隔。TOKEN 串分为简单字符串或者是复合配置块。复合配置块即是由大括号括起来的一堆内容。一个复合配置块中可能包含若干其他的配置指令。如果一个配置指令的参数全部由简单字符串构成，也就是不包含复合配置块，那么我们就说这个配置指令是一个简单配置项，否则称之为复杂配置项。例如下面这个是一个简单配置项：  

```c
error_page 500 502 503 504 /50x.html;
```

复杂配置块:

```c
location / {
	root /home/nginx/build/html;
	index index.html index.htm;
}
```

## 2. 指令上下文

nginx.conf 中的配置信息，根据其逻辑上的意义，对它们进行了分类，也就是分成了多个作用域，或者称之为配置指令上下文。不同的作用域含有一个或者多个配置项。当前 nginx 支持的几个指令上下文：  

| 指令上下文   | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| **main**     | **nginx 在运行时与具体业务功能（比如 http 服务或者 email 服务代理）无关的一 些参数，比如工作进程数，运行的身份等。** |
| **http**     | **与提供 http 服务相关的一些配置参数。例如：是否使用 keepalive 啊，是否使用 gzip 进行压缩等** |
| **server**   | **http 服务上支持若干虚拟主机。每个虚拟主机一个对应的 server 配置项，配置项 里面包含该虚拟主机相关的配置。在提供 mail 服务的代理时，也可以建立若干 server.每个 server 通过监听的地址来区分。** |
| **location** | **http 服务中，某些特定的 URL 对应的一系列配置项**           |
| **mail**     | **实现 email 相关的 SMTP/IMAP/POP3 代理时，共享的一些配置项（因为可能实现 多个代理，工作在多个监听地址上）。** |

指令上下文，可能有包含的情况出现。例如：通常 http 上下文和 mail 上下文一定是出现在main 上下文里的。在一个上下文里，可能包含另外一种类型的上下文多次。例如：如果 http服务，支持了多个虚拟主机，那么在 http 上下文里，就会出现多个 server 上下文。我们来看一个示例配置：  

```tex
user nobody;
worker_processes 1;
error_log logs/error.log info;
events {
	worker_connections 1024;
}
http {
	server {
	listen 80;
	server_name www.linuxidc.com;
	access_log logs/linuxidc.access.log main;
	location / {
		index index.html;
		root /var/www/linuxidc.com/htdocs;
		}
	}
	server {
	listen 80;
	server_name www.Androidj.com;
	access_log logs/androidj.access.log main;
	location / {
		index index.html;
		root /var/www/androidj.com/htdocs;
		}
	}
}
mail {
	auth_http 127.0.0.1:80/auth.php;
	pop3_capabilities "TOP" "USER";
	imap_capabilities "IMAP4rev1" "UIDPLUS";
	server {
		listen 110;
		protocol pop3;
		proxy on;
	}
	server {
		listen 25;
		protocol smtp;
		proxy on;
		smtp_auth login plain;
		xclient off;
	}
}
```

## 3. 模块描述

nginx 将各功能模块组织成一条链，当有请求到达的时候，请求依次经过这条链上的部分或者全部模块，进行处理。每个模块实现特定的功能。例如，实现对请求解压缩的模块，实现SSI 的模块，实现与上游服务器进行通讯的模块，实现与 FastCGI 服务进行通讯的模块。有两个模块比较特殊，他们居于 nginx core 和各功能模块的中间。这两个模块就是 http 模块和 mail 模块。这 2 个模块在 nginx core 之上实现了另外一层抽象，处理与 HTTP 协议和 email相关协议`（SMTP/POP3/IMAP）`有关的事件，并且确保这些事件能被以正确的顺序调用其他的一些功能模块。目前 HTTP 协议是被实现在 http 模块中的，但是有可能将来被剥离到一个单独的模块中，以扩展 nginx 支持 SPDY 协议。  

**模块的分类**

nginx 的模块根据其功能基本上可以分为以下几种类型 :

| **event module**  | **搭建了独立于操作系统的事件处理机制的框架，及提供了各具体事件的处理。 包括 ngx_events_module， ngx_event_core_module 和 ngx_epoll_module 等。 nginx 具体使用何种事件处理模块，这依赖于具体的操作系统和编译选项。** |
| ----------------- | ------------------------------------------------------------ |
| **phase handler** | **此类型的模块也被直接称为 handler 模块。主要负责处理客户端请求并产生待 响应内容，比如 ngx_http_static_module 模块，负责客户端的静态页面请求处 理并将对应的磁盘文件准备为响应内容输出。** |
| **output filter** | **也称为 filter 模块，主要是负责对输出的内容进行处理，可以对输出进行修改。 例如，可以实现对输出的所有 html 页面增加预定义的 footbar 一类的工作，或 者对输出的图片的 URL 进行替换之类的工作。** |
| **upstream**      | **upstream 模块实现反向代理的功能，将真正的请求转发到后端服务器上，并从 后端服务器上读取响应，发回客户端。 upstream 模块是一种特殊的 handler， 只不过响应内容不是真正由自己产生的，而是从后端服务器上读取的。** |
| **load balancer** | **负载均衡模块，实现特定的算法，在众多的后端服务器中，选择一个服务器出 来作为某个请求的转发服务器** |

**Nginx请求处理**

nginx 使用一个多进程模型来对外提供服务，其中一个 master 进程，多个 worker 进程。master 进程负责管理 nginx 本身和其他 worker 进程。所有实际上的业务处理逻辑都在 worker进程。 worker 进程中有一个函数，执行无限循环，不断处理收到的来自客户端的请求，并进行处理，直到整个 nginx 服务被停止。 worker 进程中， `ngx_worker_process_cycle()`函数就是这个无限循环的处理函数。在这个函数中，一个请求的简单处理流程如下：  

1. 操作系统提供的机制（例如 epoll, kqueue 等）产生相关的事件。
2. 接收和处理这些事件，如是接受到数据，则产生更高层的 request 对象。
3. 处理 request 的 header 和 body。
4. 产生响应，并发送回客户端。
5. 完成 request 的处理。
6. 重新初始化定时器及其他事件。  

为了更好的了解 nginx 中请求处理过程，我们以 HTTP Request 为例，来做一下详细地说明。从 nginx 的内部来看，一个 HTTP Request 的处理过程涉及到以下几个阶段。  

1. 初始化 HTTP Request（读取来自客户端的数据，生成 HTTP Request 对象，该对象含有该请求所有的信息）。
2. 处理请求头。
3. 处理请求体。
4. 如果有的话，调用与此请求（URL 或者 Location）关联的 handler。
5. 依次调用各 phase handler 进行处理。  

在这里，我们需要了解一下 phase handler 这个概念。 phase 字面的意思，就是阶段。所以 phase handlers 也就好理解了，就是包含若干个处理阶段的一些 handler。在每一个阶段，包含有若干个 handler，再处理到某个阶段的时候，依次调用该阶段的 handler 对 HTTP Request进行处理。通常情况下，一个 phase handler 对这个 request 进行处理，并产生一些输出。通常 phase handler 是与定义在配置文件中的某个 location 相关联的。一个 phase handler 通常执行以下几项任务：  

1. 获取 location 配置。
2. 产生适当的响应。
3. 发送 response header。
4. 发送 response body。  

当 nginx 读取到一个 HTTP Request 的 header 的时候， nginx 首先查找与这个请求关联的虚拟主机的配置。如果找到了这个虚拟主机的配置，那么通常情况下，这个 HTTP Request 将会经过以下几个阶段的处理（phase handlers）：

```txt
NGX_HTTP_POST_READ_PHASE          //读取请求内容阶段
NGX_HTTP_SERVER_REWRITE_PHASE     //Server 请求地址重写阶段
NGX_HTTP_FIND_CONFIG_PHASE        //配置查找阶段
NGX_HTTP_REWRITE_PHASE            //Location 请求地址重写阶段
NGX_HTTP_POST_REWRITE_PHASE       // 请求地址重写提交阶段
NGX_HTTP_PREACCESS_PHASE          //访问权限检查准备阶段
NGX_HTTP_ACCESS_PHASE            //访问权限检查阶段
NGX_HTTP_POST_ACCESS_PHASE       //访问权限检查提交阶段
NGX_HTTP_TRY_FILES_PHASE         //配置项 try_files 处理阶段
NGX_HTTP_CONTENT_PHASE           // 内容产生阶段
NGX_HTTP_LOG_PHASE               //日志模块处理阶段
```

在内容产生阶段，为了给一个 request 产生正确的响应， nginx 必须把这个 request 交给一个合适的 content handler 去处理。如果这个 request 对应的 location 在配置文件中被明确指定了一个 content handler，那么 nginx 就可以通过对 location 的匹配，直接找到这个对应的 handler，并把这个 request 交给这个 content handler 去处理。这样的配置指令包括像，perl， flv， proxy_pass， mp4 等。    

如果一个 request 对应的 location 并没有直接有配置的 content handler，那么 nginx 依次尝试:  

1. 如果一个 location 里面有配置 random_index on，那么随机选择一个文件，发送给客户端。
2. 如果一个 location 里面有配置 index 指令，那么发送 index 指令指明的文件，给客户端。
3. 如果一个 location 里面有配置 autoindex on，那么就发送请求地址对应的服务端路径下的文件列表给客户端。
4. 如果这个 request 对应的 location 上有设置 gzip_static on，那么就查找是否有对应的.gz文件存在，有的话，就发送这个给客户端（客户端支持 gzip 的情况下）。
5. 请求的 URI 如果对应一个静态文件， static module 就发送静态文件的内容到客户端。  

内容产生阶段完成以后，生成的输出会被传递到 filter 模块去进行处理。 filter 模块也是与 location 相关的。所有的 fiter 模块都被组织成一条链。输出会依次穿越所有的 filter，直到有一个 filter 模块的返回值表明已经处理完成。这里列举几个常见的 filter 模块，例如：  

1. server-side includes。
2. XSLT filtering。
3. 图像缩放之类的。
4. gzip 压缩。  

在所有的 filter 中，有几个 filter 模块需要关注一下。按照调用的顺序依次说明如下：  

| **write:**    | **写输出到客户端，实际上是写到连接对应的 socket 上。**       |
| ------------- | ------------------------------------------------------------ |
| **postpone:** | **这个 filter 是负责 subrequest 的，也就是子请求的。**       |
| **copy:**     | **将一些需要复制的 buf(文件或者内存)重新复制一份然后交给剩余的 body filter 处理。** |

# 五 handler 模块

基本上大家开发者最可能开发的就是三种类型的模块，即 `handler`， `filter `和 `load-balancer `。 Handler 模块就是接受来自客户端的请求并产生输出的模块。有些地方说 upstream 模块实际上也是一种 handler 模块，只不过它产生的内容来自于从后端服务器获取的，而非在本机产生的 。配置文件中使用 location 指令可以配置 content handler 模块，当 Nginx系统启动的时候，每个 handler 模块都有一次机会把自己关联到对应的 location 上。如果有多个 handler 模块都关联了同一个 location，那么实际上只有一个 handler 模块真正会起作用。当然大多数情况下，模块开发人员都会避免出现这种情况。  handler 模块处理的结果通常有三种情况: 处理成功，处理失败（处理的时候发生了错误）或者是拒绝去处理。在拒绝处理的情况下，这个 location 的处理就会由默认的 handler 模块来进行处理。例如，当请求一个静态文件的时候，如果关联到这个 location 上的一个 handler模块拒绝处理，就会由默认的 ngx_http_static_module 模块进行处理，该模块是一个典型的handler 模块 。

## 1. 模块的基本结构

### 1. 模块的结构

基本上每个模块都会提供一些配置指令，以便于用户可以通过配置来控制该模块的行为。那么这些配置信息怎么存储呢？那就需要定义该模块的配置结构来进行存储。大家都知道Nginx 的配置信息分成了几个作用域(scope,有时也称作上下文)，这就是 main, server, 以及location。同样的每个模块提供的配置指令也可以出现在这几个作用域里。那对于这三个作用域的配置信息，每个模块就需要定义三个不同的数据结构去进行存储。当然，不是每个模块都会在这三个作用域都提供配置指令的。那么也就不一定每个模块都需要定义三个数据结构去存储这些配置信息了。视模块的实现而言，需要几个就定义几个。有一点需要特别注意的就是，在模块的开发过程中，我们最好使用 nginx 原有的命名习惯。这样跟原代码的契合度更高，看起来也更舒服。对于模块配置信息的定义，命名习惯是 `ngx_http_<module
name>_(main|srv|loc)_conf_t`。例如:

```c
typedef struct {
	ngx_str_t hello_string;
	ngx_int_t hello_counter;
}ngx_http_hello_loc_conf_t;
```

### 2. 模块的配置指令

一个模块的配置指令是定义在一个静态数组中的。例如:

```c
static ngx_command_t ngx_http_hello_commands[] = {
	{
		ngx_string("hello_string"),
		NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS|NGX_CONF_TAKE1,
		ngx_http_hello_string,
		NGX_HTTP_LOC_CONF_OFFSET,
		offsetof(ngx_http_hello_loc_conf_t, hello_string),
		NULL 
	},
	{
		ngx_string("hello_counter"),
		NGX_HTTP_LOC_CONF|NGX_CONF_FLAG,
		ngx_http_hello_counter,
		NGX_HTTP_LOC_CONF_OFFSET,
		offsetof(ngx_http_hello_loc_conf_t, hello_counter),
		NULL 
	},
	ngx_null_command
};
```

其实看这个定义，就基本能看出来一些信息。例如，我们是定义了两个配置指令，一个是叫hello_string，可以接受一个参数，或者是没有参数。另外一个命令是 hello_counter，接受一个 NGX_CONF_FLAG 类型的参数。除此之外，似乎看起来有点迷惑。没有关系，我们来详细看一下 `ngx_command_t`，一旦我们了解这个结构的详细信息，那么我相信上述这个定义所表达的所有信息就不言自明了。 `ngx_command_t` 的定义，位于 `src/core/ngx_conf_file.h` 中。

```c
typedef struct ngx_command_s         ngx_command_t;
struct ngx_command_s {
    ngx_str_t             name;  
    ngx_uint_t            type;
    char               *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
    ngx_uint_t            conf;
    ngx_uint_t            offset;
    void                 *post;
};
/*
name : 配置指令的名称。

type : 该配置的类型，其实更准确一点说，是该配置指令属性的集合。 nginx 提供了很多预定义的属性值（一些宏定义），通过逻辑或运算符可组合在一起，形成对这个配
置指令的详细的说明。下面列出可在这里使用的预定义属性值及说明。
#define NGX_CONF_NOARGS      0x00000001       配置指令不接受任何参数
#define NGX_CONF_TAKE1       0x00000002       配置指令接受 1 个参数
#define NGX_CONF_TAKE2       0x00000004       配置指令接受 2 个参数
#define NGX_CONF_TAKE3       0x00000008       配置指令接受 3 个参数
#define NGX_CONF_TAKE4       0x00000010       配置指令接受 4 个参数
#define NGX_CONF_TAKE5       0x00000020       配置指令接受 5 个参数
#define NGX_CONF_TAKE6       0x00000040       配置指令接受 6 个参数
#define NGX_CONF_TAKE7       0x00000080       配置指令接受 7 个参数
可以组合多个属性，比如一个指令即可以不填参数，也可以接受 1 个或者 2 个参数。那么就是 NGX_CONF_NOARGS|NGX_CONF_TAKE1|NGX_CONF_TAKE2。如果写上面三个属性在一起，你觉得麻烦，那么没有关系， nginx 提供了一些定义，使用起来更简洁。
#define NGX_CONF_TAKE12      (NGX_CONF_TAKE1|NGX_CONF_TAKE2)         配置指令接受 1 个或者 2 个参数
#define NGX_CONF_TAKE13      (NGX_CONF_TAKE1|NGX_CONF_TAKE3)         配置指令接受 1 个或者 3 个参数

#define NGX_CONF_TAKE23      (NGX_CONF_TAKE2|NGX_CONF_TAKE3)         配置指令接受 2 个或者 3 个参数

#define NGX_CONF_TAKE123     (NGX_CONF_TAKE1|NGX_CONF_TAKE2|NGX_CONF_TAKE3)  配置指令接受 1 个或者 2 个或者 3 参数
#define NGX_CONF_TAKE1234    (NGX_CONF_TAKE1|NGX_CONF_TAKE2|NGX_CONF_TAKE3   \  配置指令接受 1 个或者 2 个或者 3 个或者 4 个参数
                              |NGX_CONF_TAKE4)
#define NGX_CONF_1MORE       0x00000800                              配置指令接受至少一个参数
#define NGX_CONF_2MORE       0x00001000                              配置指令接受至少两个参数
#define NGX_CONF_ARGS_NUMBER 0x000000ff                              配置指令可以接受多个参数，即个数不定
#define NGX_CONF_BLOCK       0x00000100                              配置指令可以接受的值是一个配置信息块。也就是一对大括号括起
来的内容。里面可以再包括很多的配置指令。比如常见的 server 指令就是这个属性的。
#define NGX_CONF_FLAG        0x00000200                             配置指令可以接受的值是”on”或者”off”，最终会被转成 bool 值
#define NGX_CONF_ANY         0x00000400                             配置指令可以接受的任意的参数值。一个或者多个，或者”on”或者”off”，
或者是配置块
无论如何，nginx的配置指令的参数个数不可以超过NGX_CONF_MAX_ARGS个。目前这个值被定义为8，也就是不能超过8个参数.


set: 这是一个函数指针，当 nginx 在解析配置的时候，如果遇到这个配置指令，将会把读取到的值传递给这个函数进行分解处理。因为具体每个配置指令的值如何处理，只有定义这个配置指令的人是最清楚的。来看一下这个函数指针要求的函数原型。
char *(*set)(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
先看该函数的返回值，处理成功时，返回 NGX_OK，否则返回 NGX_CONF_ERROR 或者是一个自定义的错误信息的字符串。
cf: 该参数里面保存从配置文件读取到的原始字符串以及相关的一些信息。特别注意的是这个参数的 args 字段是一个 ngx_str_t 类型的数组，该数组的首个元素是这个配置指令本身，第二个元素是指令的第一个参数，第三个元素是第二个参数，依次类推。
cmd: 这个配置指令对应的 ngx_command_t 结构。
conf: 就是定义的存储这个配置值的结构体，比如在上面展示的那个ngx_http_hello_loc_conf_t。当解析这个 hello_string 变量的时候，传入的 conf 就指向一个 ngx_http_hello_loc_conf_t 类型的变量。用户在处理的时候可以使用类型转换，转换成自己知道的类型，再进行字段的赋值。
为了更加方便的实现对配置指令参数的读取， nginx 已经默认提供了对一些标准类型的参数进行读取的函数，可以直接赋值给 set 字段使用。下面来看一下这些已经实现的 set 类型函数。
 1. ngx_conf_set_flag_slot： 读取 NGX_CONF_FLAG 类型的参数
 2. ngx_conf_set_str_slot:读取字符串类型的参数。
 3. ngx_conf_set_str_array_slot: 读取字符串数组类型的参数。
 4. ngx_conf_set_keyval_slot： 读取键值对类型的参数。
 5. ngx_conf_set_num_slot: 读取整数类型(有符号整数 ngx_int_t)的参数。
 6. ngx_conf_set_size_slot:读取 size_t 类型的参数，也就是无符号数。
 7. ngx_conf_set_off_slot: 读取 off_t 类型的参数
 8. ngx_conf_set_msec_slot: 读取毫秒值类型的参数
 9. ngx_conf_set_sec_slot: 读取秒值类型的参数
 10. ngx_conf_set_bufs_slot： 读取的参数值是 2 个，一个是 buf 的个数，一个是 buf 的大小。例如： output_buffers 1 128k;
 11. ngx_conf_set_enum_slot: 读取枚举类型的参数，将其转换成整数 ngx_uint_t 类型
 12. ngx_conf_set_bitmask_slot: 读取参数的值，并将这些参数的值以 bit 位的形式存储。例如： HttpDavModule 模块的 dav_methods 指令。
 
 conf: 该字段被NGX_HTTP_MODULE类型模块所用 (我们编写的基本上都是NGX_HTTP_MOUDLE，只有一些 nginx 核心模块是非 NGX_HTTP_MODULE)，该字段指定当前配置项存储的内存位置。实际上是使用哪个内存池的问题。因为 http模块对所有 http 模块所要保存的配置信息，划分了 main, server 和 location 三个地方进行存储，每个地方都有一个内存池用来分配存储这些信息的内存。这里可能的值为NGX_HTTP_MAIN_CONF_OFFSET 、NGX_HTTP_SRV_CONF_OFFSET或NGX_HTTP_LOC_CONF_OFFSET。当然 也可以直接置为0，就是NGX_HTTP_MAIN_CONF_OFFSET。
 
 offset: 指定该配置项值的精确存放位置，一般指定为某一个结构体变量的字段偏移。因为对于配置信息的存储，一般我们都是定义个结构体来存储的。那么比如我们定义了一个结构体 A，该项配置的值需要存储到该结构体的 b 字段。那么在这里就可以填写为 offsetof(A, b)。对于有些配置项，它的值不需要保存或者是需要保存到更为复杂的结构中时，这里可以设置为0。
 
 post: 该字段存储一个指针。可以指向任何一个在读取配置过程中需要的数据，以便于进行配置读取的处理。大多数时候，都不需要，所以简单地设为 0 即可
*/
```

看到这里，应该就比较清楚了。 ngx_http_hello_commands 这个数组每 5 个元素为一组，用来描述一个配置项的所有情况。那么如果有多个配置项，只要按照需要再增加 5 个对应的元素对新的配置项进行说明。需要注意的是，就是在 ngx_http_hello_commands 这个数组定义的最后，都要加一个`ngx_null_command `作为结尾。  

### 3. 模块的上下文结构

这是一个 `ngx_http_module_t` 类型的静态变量。这个变量实际上是提供一组回调函数指针，这些函数有在创建存储配置信息的对象的函数，也有在创建前和创建后会调用的函数。这些函数都将被 nginx 在合适的时间进行调用。 

```c
typedef struct {
    ngx_int_t   (*preconfiguration)(ngx_conf_t *cf);
    ngx_int_t   (*postconfiguration)(ngx_conf_t *cf);

    void       *(*create_main_conf)(ngx_conf_t *cf);
    char       *(*init_main_conf)(ngx_conf_t *cf, void *conf);

    void       *(*create_srv_conf)(ngx_conf_t *cf);
    char       *(*merge_srv_conf)(ngx_conf_t *cf, void *prev, void *conf);

    void       *(*create_loc_conf)(ngx_conf_t *cf);
    char       *(*merge_loc_conf)(ngx_conf_t *cf, void *prev, void *conf);
} ngx_http_module_t;

/*
preconfiguration : 在创建和读取该模块的配置信息之前被调用
postconfiguration: 在创建和读取该模块的配置信息之后被调用
create_main_conf:  调用该函数创建本模块位于 http block 的配置信息存储结构。该函数成功的时候，返回创建的配置对象。失败的话，返回 NULL。
init_main_conf:  调用该函数初始化本模块位于 http block 的配置信息存储结构。该函数成功的时候，返回 NGX_CONF_OK。失败的话，返回 NGX_CONF_ERROR 或错误字符串。
create_srv_conf:  调用该函数创建本模块位于 http server block 的配置信息存储结构，每个 server block 会创建一个。该函数成功的时候，返回创建的配置对象。失败的话，返回 NULL。
merge_srv_conf: 因为有些配置指令既可以出现在 http block，也可以出现在 http server block 中。那么遇到这种情况，每个 server都会有自己存储结构来存储该 server 的配置，但是在这种情况下 http block 中的配置与 server block 中的配置信息发生冲突的时候，就需要调用此函数进行合并，该函数并非必须提供，当预计到绝对不会发生需要合并的情况的时候，就无需提供。当然为了安全起见还是建议提供。该函数执行成功的时候，返回 NGX_CONF_OK。失败的话，返回 NGX_CONF_ERROR 或错误字符串。
create_loc_conf ： 调用该函数创建本模块位于 location block 的配置信息存储结构。每个在配置中指明的 location 创建一个。该函数执行成功，返回创建的配置对象。失败的话，返回 NULL。
merge_loc_conf： 与 merge_srv_conf 类似，这个也是进行配置值合并的地方。该函数成功的时候，返回 NGX_CONF_OK。失败的话，返回 NGX_CONF_ERROR 或错误字符串。
*/
```

 Nginx 里面的配置信息都是上下一层层的嵌套的，对于具体某个 location 的话，对于同一个配置，如果当前层次没有定义，那么就使用上层的配置，否则使用当
前层次的配置。这些配置信息一般默认都应该设为一个未初始化的值，针对这个需求， Nginx 定义了一系列的宏定义来代表各种配置所对应数据类型的未初始化值，如下：    

```c
#define NGX_CONF_UNSET       -1
#define NGX_CONF_UNSET_UINT  (ngx_uint_t) -1
#define NGX_CONF_UNSET_PTR   (void *) -1
#define NGX_CONF_UNSET_SIZE  (size_t) -1
#define NGX_CONF_UNSET_MSEC  (ngx_msec_t) -1
```

又因为对于配置项的合并，逻辑都类似，也就是前面已经说过的，如果在本层次已经配置了，也就是配置项的值已经被读取进来了（那么这些配置项的值就不会等于上面已经定义的那些UNSET 的值），就使用本层次的值作为定义合并的结果，否则，使用上层的值，如果上层的值也是这些 UNSET 类的值，那就赋值为默认值，否则就使用上层的值作为合并的结果。对于这样类似的操作， Nginx 定义了一些宏操作来做这些事情，我们来看其中一个的定义 ：

```c
#define ngx_conf_merge_uint_value(conf, prev, default)                       \
    if (conf == NGX_CONF_UNSET_UINT) {                                       \
        conf = (prev == NGX_CONF_UNSET_UINT) ? default : prev;               \
    }
```

显而易见，这个逻辑确实比较简单，所以其它的宏定义也类似。

最后举例:

```c
static ngx_http_module_t ngx_http_hello_module_ctx = {
	NULL,                                              /* preconfiguration */
	ngx_http_hello_init,                               /* postconfiguration */
	NULL,                                              /* create main configuration*/
	NULL,                                              /* init main configuration */
	NULL,                                              /* create server configuration*/
	NULL, 										   /* merge server configuration*/
	ngx_http_hello_create_loc_conf, 				 /* create location configuration */
	NULL 										   /* merge location configuration*/
};
```

注意：这里并没有提供 merge_loc_conf 函数，因为我们这个模块的配置指令已经确定只出现在 NGX_HTTP_LOC_CONF 中这一个层次上，不会发生需要合并的情况。  

### 4. 模块定义

对于开发一个模块来说，我们都需要定义一个 `ngx_module_t `类型的变量来说明这个模块本身的信息，从某种意义上来说，这是这个模块最重要的一个信息，它告诉了 nginx 这个模块的一些信息，上面定义的配置信息，还有模块上下文信息，都是通过这个结构来告诉 nginx系统的，也就是加载模块的上层代码，都需要通过定义的这个结构，来获取这些信息。我们先来看下 `ngx_module_t` 的定义 ：

```c
typedef struct ngx_module_s          ngx_module_t;
struct ngx_module_s {
    ngx_uint_t            ctx_index;
    ngx_uint_t            index;

    char                 *name;

    ngx_uint_t            spare0;
    ngx_uint_t            spare1;

    ngx_uint_t            version;
    const char           *signature;

    void                 *ctx;
    ngx_command_t        *commands;
    ngx_uint_t            type;

    ngx_int_t           (*init_master)(ngx_log_t *log);

    ngx_int_t           (*init_module)(ngx_cycle_t *cycle);

    ngx_int_t           (*init_process)(ngx_cycle_t *cycle);
    ngx_int_t           (*init_thread)(ngx_cycle_t *cycle);
    void                (*exit_thread)(ngx_cycle_t *cycle);
    void                (*exit_process)(ngx_cycle_t *cycle);

    void                (*exit_master)(ngx_cycle_t *cycle);

    uintptr_t             spare_hook0;
    uintptr_t             spare_hook1;
    uintptr_t             spare_hook2;
    uintptr_t             spare_hook3;
    uintptr_t             spare_hook4;
    uintptr_t             spare_hook5;
    uintptr_t             spare_hook6;
    uintptr_t             spare_hook7;
};

#define NGX_MODULE_V1                                                         \
    NGX_MODULE_UNSET_INDEX, NGX_MODULE_UNSET_INDEX,                           \
    NULL, 0, 0, nginx_version, NGX_MODULE_SIGNATURE

#define NGX_MODULE_V1_PADDING  0, 0, 0, 0, 0, 0, 0, 0
```

再看下举例:

```
ngx_module_t ngx_http_hello_module = {
	NGX_MODULE_V1,
	&ngx_http_hello_module_ctx, /* module context */
	ngx_http_hello_commands, /* module directives */
	NGX_HTTP_MODULE, /* module type */
	NULL, /* init master */
	NULL, /* init module */
	NULL, /* init process */
	NULL, /* init thread */
	NULL, /* exit thread */
	NULL, /* exit process */
	NULL, /* exit master */
	NGX_MODULE_V1_PADDING
};
```

模块可以提供一些回调函数给 nginx，当 nginx 在创建进程线程或者结束进程线程时进行调用。但大多数模块在这些时刻并不需要做什么，所以都简单赋值为 NULL。  

## 2. handler 模块的基本结构  

除了上面说的模块的基本结构外，handler 模块必须提供一个真正的处理函数，这个函数负责对来自客户端请求的真正处理。这个函数的处理，既可以选择自己直接生成内容，也可以选择拒绝处理，由后续的 handler 去进行处理，或者是选择丢给后续的 filter 进行处理。来看一下这个函数的原型申明。

```c
typedef ngx_int_t (*ngx_http_handler_pt)(ngx_http_request_t *r);
```

r 是 http 请求。里面包含请求所有的信息，这里不详细说明了，可以参考别的章节的介绍。该函数处理成功返回NGX_OK，处理发生错误返回NGX_ERROR，拒绝处理（留给后续的handler进行处理）返回 NGX_DECLINE。 返回 NGX_OK 也就代表给客户端的响应已经生成好了，否则返回 NGX_ERROR 就发生错误了。  

## 3. handle模块的挂载

handler 模块真正的处理函数通过两种方式挂载到处理过程中，一种方式就是按处理阶段挂载;另外一种挂载方式就是按需挂载。  

### 1. 按处理阶段挂载

为了更精细地控制对于客户端请求的处理过程， nginx 把这个处理过程划分成了 11 个阶段。他们从前到后，依次列举如下：  

| **NGX_HTTP_POST_READ_PHASE**      | **读取请求内容阶段**          |
| --------------------------------- | ----------------------------- |
| **NGX_HTTP_SERVER_REWRITE_PHASE** | **Server 请求地址重写阶段**   |
| **NGX_HTTP_FIND_CONFIG_PHASE**    | **配置查找阶段**              |
| **NGX_HTTP_REWRITE_PHASE**        | **Location 请求地址重写阶段** |
| **NGX_HTTP_POST_REWRITE_PHASE**   | **请求地址重写提交阶段**      |
| **NGX_HTTP_PREACCESS_PHASE**      | **访问权限检查准备阶段**      |
| **NGX_HTTP_ACCESS_PHASE**         | **访问权限检查阶段**          |
| **NGX_HTTP_POST_ACCESS_PHASE**    | **访问权限检查提交阶段**      |
| **NGX_HTTP_TRY_FILES_PHASE**      | **配置项 try_files 处理阶段** |
| **NGX_HTTP_CONTENT_PHASE**        | **内容产生阶段**              |
| **NGX_HTTP_LOG_PHASE**            | **日志模块处理阶段**          |

一般情况下，我们自定义的模块，大多数是挂载在 `NGX_HTTP_CONTENT_PHASE` 阶段的。挂载的动作一般是在模块上下文调用的 postconfiguration 函数中。  注意：有几个阶段是特例，它不调用挂载地任何的 handler，也就是你就不用挂载到这几个阶段了：

NGX_HTTP_FIND_CONFIG_PHASE
NGX_HTTP_POST_ACCESS_PHASE
NGX_HTTP_POST_REWRITE_PHASE
NGX_HTTP_TRY_FILES_PHASE  

所以其实真正是有 7 个 phase 你可以去挂载 handler。  

挂载的代码如下 :

```c
static ngx_int_t ngx_http_hello_init(ngx_conf_t *cf)
{
	ngx_http_handler_pt *h;
	ngx_http_core_main_conf_t *cmcf;
	cmcf = ngx_http_conf_get_module_main_conf(cf, ngx_http_core_module);
	h = ngx_array_push(&cmcf->phases[NGX_HTTP_CONTENT_PHASE].handlers);
	if (h == NULL) {
		return NGX_ERROR;
	}
	*h = ngx_http_hello_handler;
	return NGX_OK;
}
```

  使用这种方式挂载的 handler 也被称为 **content phase handlers**。  

### 2. 按需挂载

以这种方式挂载的 handler 也被称为 content handler。当一个请求进来以后， nginx 从 NGX_HTTP_POST_READ_PHASE 阶段开始依次执行每个阶段中
所有 handler。执行到 NGX_HTTP_CONTENT_PHASE 阶段的时候，如果这个 location 有一个对应的 content handler 模块，那么就去执行这个 content handler 模块真正的处理函数。否则继续依次执行 NGX_HTTP_CONTENT_PHASE 阶段中所有 content phase handlers，直到某个函数处理返回 NGX_OK 或者 NGX_ERROR。换句话说，当某个location 处理到 NGX_HTTP_CONTENT_PHASE 阶段时，如果有 content handler模块，那么NGX_HTTP_CONTENT_PHASE挂载的所有content phase handlers都不会被执行了。但是使用这个方法挂载上去的 handler 有一个特点是必须在 NGX_HTTP_CONTENT_PHASE 阶段才能执行到。如果你想自己的 handler 在更早的阶段执行，那就不要使用这种挂载方式。那么在什么情况会使用这种方式来挂载呢？一般情况下，某个模块对某个 location 进行了处理以后，发现符合自己处理的逻辑，而且也没有必要再调用 NGX_HTTP_CONTENT_PHASE 阶段的其它 handler 进行处理的时候，就动态挂载上这个 handler。
下面来看一下使用这种挂载方式的具体例子  

```c
static char * ngx_http_circle_gif(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
	ngx_http_core_loc_conf_t *clcf;
	clcf = ngx_http_conf_get_module_loc_conf(cf, ngx_http_core_module);
	clcf->handler = ngx_http_circle_gif_handler;
	return NGX_CONF_OK;
}
```

## 4. handler的编写步骤

1. 编写模块基本结构。包括模块的定义，模块上下文结构，模块的配置结构等。
2. 实现 handler 的挂载函数。根据模块的需求选择正确的挂载方式。  
3. 编写 handler 处理函数。模块的功能主要通过这个函数来完成。  

我们看下完整的`hello handler module`模块的代码；在前面已经看到了这个 `hello handler module` 的部分重要的结构。该模块提供了 2 个配置指令，仅可以出现在 location 指令的作用域中。这两个指令是 `hello_string`, 该指令接受一个参数来设置显示的字符串。如果没有跟参数，那么就使用默认的字符串作为响应字符串。另一个指令是 `hello_counter`，如果设置为 on，则会在响应的字符串后面追加 Visited Times:的字样，以统计请求的次数。  

​	1.对于 flag 类型的配置指令，当值为 off 的时候，使用 ngx_conf_set_flag_slot 函数，会转化为 0，为 on，则转化为非 0。

​	2.另外一个是，我提供了 `merge_loc_conf `函数，但是却没有设置到模块的上下文定义中。这样有一个缺点，就是如果一个指令没有出现在配置文件中的时候，配置信息中的值，将永远会保持在 `create_loc_conf` 中的初始化的值。那如果，在类似 `create_loc_conf `这样的函数中，对创建出来的配置信息的值，没有设置为合理的值的话，后面用户又没有配置，就会出现问题。  

```c
#include <ngx_config.h>
#include <ngx_core.h>
#include <ngx_http.h>
typedef struct
{
	ngx_str_t hello_string;
	ngx_int_t hello_counter;
}ngx_http_hello_loc_conf_t;

static ngx_int_t ngx_http_hello_init(ngx_conf_t *cf);
static void *ngx_http_hello_create_loc_conf(ngx_conf_t *cf);
static char *ngx_http_hello_string(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);

static char *ngx_http_hello_counter(ngx_conf_t *cf, ngx_command_t *cmd, void *conf);
static ngx_command_t ngx_http_hello_commands[] = {
	{
        ngx_string("hello_string"),
        NGX_HTTP_LOC_CONF|NGX_CONF_NOARGS|NGX_CONF_TAKE1,
        ngx_http_hello_string,
        NGX_HTTP_LOC_CONF_OFFSET,
        offsetof(ngx_http_hello_loc_conf_t,	hello_string),
        NULL 
    },
	{
        ngx_string("hello_counter"),
        NGX_HTTP_LOC_CONF|NGX_CONF_FLAG,
        ngx_http_hello_counter,
        NGX_HTTP_LOC_CONF_OFFSET,
        offsetof(ngx_http_hello_loc_conf_t, hello_counter),
        NULL 
     },
	ngx_null_command
};

/*
static u_char ngx_hello_default_string[] = "Default String: Hello, world!";
*/
static int ngx_hello_visited_times = 0;
static ngx_http_module_t ngx_http_hello_module_ctx = {
	NULL, /* preconfiguration */
	ngx_http_hello_init, /* postconfiguration */
	NULL, /* create main configuration */
	NULL, /* init main configuration*/
    NULL, /* create server configuration */
	NULL, /* merge server configuration */
	ngx_http_hello_create_loc_conf, /* create location configuration */
	NULL /* merge location configuration */
};

ngx_module_t ngx_http_hello_module = {
	NGX_MODULE_V1,
	&ngx_http_hello_module_ctx, /* module context */
	ngx_http_hello_commands, /* module directives */
	NGX_HTTP_MODULE, /* module type */
	NULL, /* init master */
	NULL, /* init module */
	NULL, /* init process */
	NULL, /* init thread */
	NULL, /* exit thread */
	NULL, /* exit process */
	NULL, /* exit master */
	NGX_MODULE_V1_PADDING
};

static ngx_int_t ngx_http_hello_handler(ngx_http_request_t *r)
{
	ngx_int_t rc;
	ngx_buf_t *b;
	ngx_chain_t out;
	ngx_http_hello_loc_conf_t* my_conf;
	u_char ngx_hello_string[1024] = {0};
	ngx_uint_t content_length = 0;
	ngx_log_error(NGX_LOG_EMERG, r->connection->log, 0, "ngx_http_hello_handler is called!");
	my_conf = ngx_http_get_module_loc_conf(r, ngx_http_hello_module);
    
	if (my_conf->hello_string.len == 0 )
	{
		ngx_log_error(NGX_LOG_EMERG, r->connection->log, 0, "hello_string is empty!");
		return NGX_DECLINED;
	}
	if (my_conf->hello_counter == NGX_CONF_UNSET || my_conf->hello_counter == 0)
	{
		ngx_sprintf(ngx_hello_string, "%s",
		my_conf->hello_string.data);
	} else {
		ngx_sprintf(ngx_hello_string, "%s Visited Times:%d", my_conf->hello_string.data, ++ngx_hello_visited_times);
	}
	ngx_log_error(NGX_LOG_EMERG, r->connection->log, 0, "hello_string:%s", ngx_hello_string);
	content_length = ngx_strlen(ngx_hello_string);
    
	/* we response to 'GET' and 'HEAD' requests only */
	if (!(r->method & (NGX_HTTP_GET|NGX_HTTP_HEAD))) {
		return NGX_HTTP_NOT_ALLOWED;
	}
	/* discard request body, since we don't need it here */
	rc = ngx_http_discard_request_body(r);
	if (rc != NGX_OK) {
		return rc;
	}
	/* set the 'Content-type' header */
/*
	*r->headers_out.content_type.len = sizeof("text/html") - 1;
	*r->headers_out.content_type.data = (u_char *)"text/html";
*/
	ngx_str_set(&r->headers_out.content_type, "text/html");
    
    /* send the header only, if the request type is http 'HEAD' */
	if (r->method == NGX_HTTP_HEAD) {
		r->headers_out.status = NGX_HTTP_OK;
		r->headers_out.content_length_n = content_length;
		return ngx_http_send_header(r);
	}
	/* allocate a buffer for your response body */
	b = ngx_pcalloc(r->pool, sizeof(ngx_buf_t));
	if (b == NULL) {
		return NGX_HTTP_INTERNAL_SERVER_ERROR;
	}
	/* attach this buffer to the buffer chain */
	out.buf = b;
	out.next = NULL;
	/* adjust the pointers of the buffer */
	b->pos = ngx_hello_string;
	b->last = ngx_hello_string + content_length;
	b->memory = 1; /* this buffer is in memory */
	b->last_buf = 1; /* this is the last buffer in the
	buffer chain */
   
	/* set the status line */
	r->headers_out.status = NGX_HTTP_OK;
	r->headers_out.content_length_n = content_length;
	/* send the headers of your response */
	rc = ngx_http_send_header(r);
	if (rc == NGX_ERROR || rc > NGX_OK || r->header_only) {
		return rc;
	}
	/* send the buffer chain of your response */
	return ngx_http_output_filter(r, &out);
}

static void *ngx_http_hello_create_loc_conf(ngx_conf_t *cf) {
	ngx_http_hello_loc_conf_t* local_conf = NULL;
	local_conf = ngx_pcalloc(cf->pool, sizeof(ngx_http_hello_loc_conf_t));
	if (local_conf == NULL)
	{
		return NULL;
	}
	ngx_str_null(&local_conf->hello_string);
	local_conf->hello_counter = NGX_CONF_UNSET;
	return local_conf;
}

/*
static char *ngx_http_hello_merge_loc_conf(ngx_conf_t *cf, void *parent, void *child)
{
    ngx_http_hello_loc_conf_t* prev = parent;
    ngx_http_hello_loc_conf_t* conf = child;
    ngx_conf_merge_str_value(conf->hello_string,
    prev->hello_string, ngx_hello_default_string);
    ngx_conf_merge_value(conf->hello_counter,
    prev->hello_counter, 0);
    return NGX_CONF_OK;
}
*/

static char * ngx_http_hello_string(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
	ngx_http_hello_loc_conf_t* local_conf;
	local_conf = conf;
	char* rv = ngx_conf_set_str_slot(cf, cmd, conf);
	ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "hello_string:%s", local_conf->hello_string.data);
    return rv;
}

static char *ngx_http_hello_counter(ngx_conf_t *cf, ngx_command_t *cmd, void *conf)
{
	ngx_http_hello_loc_conf_t* local_conf;
	local_conf = conf;
	char* rv = NULL;
	rv = ngx_conf_set_flag_slot(cf, cmd, conf);
	ngx_conf_log_error(NGX_LOG_EMERG, cf, 0, "hello_counter:%d", local_conf->hello_counter);
	return rv;
}

static ngx_int_t ngx_http_hello_init(ngx_conf_t *cf) {
	ngx_http_handler_pt *h;
	ngx_http_core_main_conf_t *cmcf;
	cmcf = ngx_http_conf_get_module_main_conf(cf, ngx_http_core_module);
    
	h = ngx_array_push(&cmcf->phases[NGX_HTTP_CONTENT_PHASE].handlers);
	if (h == NULL) {
		return NGX_ERROR;
	}
	*h = ngx_http_hello_handler;
	return NGX_OK;
}
```

## 5. handler 模块的编译和使用  

### 1. config 文件的编写  

对于开发一个模块，我们是需要把这个模块的 C 代码组织到一个目录里，同时需要编写一个config 文件。这个 config 文件的内容就是告诉 nginx 的编译脚本，该如何进行编译。我们来看一下 `hello handler module` 的 config 文件的内容，然后再做解释。  

```c
ngx_addon_name=ngx_http_hello_module
HTTP_MODULES="$HTTP_MODULES ngx_http_hello_module"
NGX_ADDON_SRCS="$NGX_ADDON_SRCS
$ngx_addon_dir/ngx_http_hello_module.c"
```

其实文件很简单，几乎不需要做什么解释。大家一看都懂了。唯一需要说明的是，如果这个模块的实现有多个源文件，那么都在 NGX_ADDON_SRCS 这个变量里，依次写进去就可以。  

### 2.编译

对于模块的编译， nginx 并不像 apache 一样，提供了单独的编译工具，可以在没有 apache 源代码的情况下来单独编译一个模块的代码。 nginx 必须去到 nginx 的源代码目录里，通过 configure 指令的参数，来进行编译。下面看一下 hello module 的 configure 指令：   

```c
$ ./configure –prefix=/usr/local/nginx-1.4.1 –add-module= /home/xx/open_source/book_module
```

我们的这个示例模块的代码和 config 文件都放在/home/xx/open_source/book_module 这 个目录下。  

### 3. 使用

使用一个模块需要根据这个模块定义的配置指令来做。比如我们这个简单的 `hello handler module` 的使用就很简单。在我的测试服务器的配置文件里，就是在 http 里面的默认的 server 里面加入如下的配置：  

```
location /test {
	hello_string helloNginx;
	hello_counter on;
}
```

当我们访问这个地址的时候, http://localhost/test 的时候，就可以看到返回的结果。` helloNginx Visited Times:1  `当然你访问多次，这个次数是会增加的。  





# 死锁的检测与实现

死锁的发生是由于线程间资源争夺引起的，这里不对线程的死锁的概念和原理解释。主要介绍死锁的检测和具体实现，检测我们可以使用一个有向图来表示， 线程 A 获取线程 B 已占用的锁，则为线程 A 指向线程 B。 如何为线程 B 已占用的锁？运行过程线程 B 获取成功的锁。检测的原理采用另一个线程定时对图进程检测是否有环的存在。

数据结构定义:

```c++
typedef enum { PROCESS, RESOURCE} Type;  // 定义资源的类型

typedef struct source_type_t {  // 资源的数据结构
	uint64 id;
	Type type;
	
	uint64 lock_id;     //mutex的id 
	int degress;        //加锁的次数
}source_type;

typedef struct vertex_t {  // 图的一个节点
	source_type s;
	struct vertex_t *next;  //指向下个图
}vertex;

typedef struct task_graph_t { // 图的数据结构
	vertex list[MAX];         // 以一个链表的形式来存储
	int num;                  // 图节点的个数
	
	source_type locklist[MAX];
	int lockidx;             // 加锁成功后++
	
	pthread_mutex_t mutex;  //互斥锁
}task_graph;
```

  图算法，检测是否有成环:

```c
task_graph *tg = NULL;   //全局图变量
int path[MAX+1];      // 把第几个图的type 的序号记录到path里面   
int visited[MAX];     // 记录DFS 访问过的路径
int k = 0;            // 记录path的个数
int deadlock = 0;     //有环为1， 无为0


vertex *create_vertex(source_type type) {  //分配vertex的空间
	vertex *tex = (vertex*)malloc(sizeof(struct vertex_t));
	tex->s = type;
	tex->next = NULL;
	return tex;
}

int search_vertex(source_type type) { // 搜索图，找到就返回序号
	int i = 0;
	for (i = 0; i<tg->num; i++) {
		if (tg->list[i].s.type == type.type && tg->list[i].s.id == type.id) 
			return i;
	}
	return -1;
}

void add_vertex(source_type type) { // 添加vertex
	if (search_vertex(type) == -1) {
		tg->list[tg->num].s = type;
		tg->list[tg->num].next = NULL;
		tg->num++;
	}
}

int add_edge(source_type from, source_type to) {  //增加边
	//add_vertex(from);
	//add_vertex(to);

	vertex *v = &(tg->list[search_vertex(from)]);
	while( v->next != NULL) {
		v = v->next;
	}
	v->next = create_vertex(to);
}

int verify_edge(source_type i, source_type j) { // 验证边
	if (tg->num == 0) return 0;

	int idx = search_vertex(i);
	if (idx == -1) return 0;

	vertex *v = &(tg->list[idx]);
	while(v!=NULL) {
		if (v->s.id == j.id) return 1;
		v = v->next;
	}

	return 0;
}

int remove_edge(source_type from, source_type to) { //移除边
	int idxi = search_vertex(from);
	int idxj = search_vertex(to);

	if (idxi != -1 && idxj != -1) {
		vertex *v = &tg->list[idxi];
		vertex *remove;

		while (v->next != NULL) {
			if (v->next->s.id == to.id) {
				remove = v->next;
				v->next = v->next->next;
				free(remove);
				break;
			}
			v = v->next;
		}
	}
}

void print_deadlock(void) { //打印
	int i = 0;
	printf("deadlock: ");

	for ( i = 0; i<k-1; i++) {
		printf("%ld --> ", tg->list[path[i]].s.id);
	}
	printf("%ld\n", tg->list[path[i]].s.id);
}

int DFS(int idx) {
	 vertex *ver = &tg->list[idx];
	if (visited[idx] == 1) {  // 找过了

		path[k++] = idx;
		print_deadlock();
		deadlock = 1;
		
		return 0;
	}

	visited[idx] = 1;   //标记找过
	path[k++] = idx;

	while (ver->next != NULL) {

		DFS(search_vertex(ver->next->s));
		k --;
		
		ver = ver->next;

	}
	return 1;
}

int search_for_cycle(int idx) { //搜索是否有成环
	vertex *ver = &tg->list[idx];
	visited[idx] = 1;
	k = 0;
	path[k++] = idx;

	while (ver->next != NULL) {

		int i = 0;
		for (i = 0;i < tg->num;i ++) {
			if (i == idx) continue;
			
			visited[i] = 0;
		}

		for (i = 1;i <= MAX;i ++) {
			path[i] = -1;
		}
		k = 1;

		DFS(search_vertex(ver->next->s));
		ver = ver->next;
	}

}

int main() {


	tg = (struct task_graph*)malloc(sizeof(struct task_graph_t));
	tg->num = 0;

	struct source_type v1;
	v1.id = 1;
	v1.type = PROCESS;
	add_vertex(v1);

	struct source_type v2;
	v2.id = 2;
	v2.type = PROCESS;
	add_vertex(v2);

	struct source_type v3;
	v3.id = 3;
	v3.type = PROCESS;
	add_vertex(v3);

	struct source_type v4;
	v4.id = 4;
	v4.type = PROCESS;
	add_vertex(v4);

	
	struct source_type v5;
	v5.id = 5;
	v5.type = PROCESS;
	add_vertex(v5);


	add_edge(v1, v2);
	add_edge(v2, v3);
	add_edge(v3, v4);
	add_edge(v4, v5);
	add_edge(v3, v1);
	
	search_for_cycle(search_vertex(v1));

}

```

图算法的测试入口函数 :增加了线程

```c
void check_dead_lock(void) {
	int i = 0;
	deadlock = 0;
	for (i = 0; i < tg->num; i++) {
		if (deadlock == 1) break;
		search_for_cycle(i);
	}

	if (deadlock == 0)
		printf("no deadlock\n");
}

static void *thread_routine(void *args) {

	while (1) {

		sleep(5);
		check_dead_lock();

	}

}


void start_check(void) {

	tg = (task_graph *)malloc(sizeof(struct task_graph_t));
	tg->num = 0;
	tg->lockidx = 0;
	
	pthread_t tid;

	pthread_create(&tid, NULL, thread_routine, NULL);

}

int search_lock(uint64 lock) {

	int i = 0;
	
	for (i = 0;i < tg->lockidx;i ++) {
		
		if (tg->locklist[i].lock_id == lock) {
			return i;
		}
	}

	return -1;
}

int search_empty_lock(uint64 lock) {

	int i = 0;
	
	for (i = 0;i < tg->lockidx;i ++) {
		
		if (tg->locklist[i].lock_id == 0) {
			return i;
		}
	}

	return tg->lockidx;

}

int inc(int *value, int add) {  // 原子操作

	int old;

	__asm__ volatile(
		"lock;xaddl %2, %1;"
		: "=a"(old)
		: "m"(*value), "a" (add)
		: "cc", "memory"
	);
	
	return old;
}

void print_locklist(void) {

	int i = 0;

	printf("print_locklist: \n");
	printf("---------------------\n");
	for (i = 0;i < tg->lockidx;i ++) {
		printf("threadid : %ld, lockid: %ld\n", tg->locklist[i].id, tg->locklist[i].lock_id);
	}
	printf("---------------------\n\n\n");
}

void lock_before(uint64 thread_id, uint64 lockaddr) {


	int idx = 0;
	// list<threadid, toThreadid>

	for(idx = 0;idx < tg->lockidx;idx ++) {
		if ((tg->locklist[idx].lock_id == lockaddr)) {

			source_type from;
			from.id = thread_id;
			from.type = PROCESS;
			add_vertex(from);

			source_type to;
			to.id = tg->locklist[idx].id;
			tg->locklist[idx].degress++;
			to.type = PROCESS;
			add_vertex(to);
			
			if (!verify_edge(from, to)) {
				add_edge(from, to);
			}

		}
	}
}

void lock_after(uint64 thread_id, uint64 lockaddr) {

	int idx = 0;
	if (-1 == (idx = search_lock(lockaddr))) {  // lock list opera 

		int eidx = search_empty_lock(lockaddr);
		
		tg->locklist[eidx].id = thread_id;
		tg->locklist[eidx].lock_id = lockaddr;
		
		inc(&tg->lockidx, 1);
		
	} else {
		source_type from;
		from.id = thread_id;
		from.type = PROCESS;

		source_type to;
		to.id = tg->locklist[idx].id;
		tg->locklist[idx].degress --;
		to.type = PROCESS;

		if (verify_edge(from, to))
			remove_edge(from, to);

		tg->locklist[idx].id = thread_id;

	}
}

void unlock_after(uint64 thread_id, uint64 lockaddr) {


	int idx = search_lock(lockaddr);

	if (tg->locklist[idx].degress == 0) {
		tg->locklist[idx].id = 0;
		tg->locklist[idx].lock_id = 0;
		//inc(&tg->lockidx, -1);
	}
	
}

int pthread_mutex_lock(pthread_mutex_t *mutex) {

    pthread_t selfid = pthread_self(); //
    
	lock_before(selfid, (uint64)mutex);
    pthread_mutex_lock_f(mutex);
	lock_after(selfid, (uint64)mutex);

}

int pthread_mutex_unlock(pthread_mutex_t *mutex) {


	pthread_t selfid = pthread_self();

    pthread_mutex_unlock_f(mutex);
	unlock_after(selfid, (uint64)mutex);


}

static int init_hook() { //使用hook 重定向lock, unlock函数

    pthread_mutex_lock_f = dlsym(RTLD_NEXT, "pthread_mutex_lock");

    pthread_mutex_unlock_f = dlsym(RTLD_NEXT, "pthread_mutex_unlock");

}

pthread_mutex_t mutex_1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex_2 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex_3 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t mutex_4 = PTHREAD_MUTEX_INITIALIZER;

void *thread_rountine_1(void *args)
{
	pthread_t selfid = pthread_self(); //
	
    pthread_mutex_lock(&mutex_1);
    sleep(1);
    pthread_mutex_lock(&mutex_2);

    pthread_mutex_unlock(&mutex_2);
    pthread_mutex_unlock(&mutex_1);

    return (void *)(0);
}

void *thread_rountine_2(void *args)
{
	pthread_t selfid = pthread_self(); //
	
    pthread_mutex_lock(&mutex_2);
    sleep(1);
    pthread_mutex_lock(&mutex_3);

    pthread_mutex_unlock(&mutex_3);
    pthread_mutex_unlock(&mutex_2);

    return (void *)(0);
}

void *thread_rountine_3(void *args)
{
	pthread_t selfid = pthread_self(); //

    pthread_mutex_lock(&mutex_3);
    sleep(1);
    pthread_mutex_lock(&mutex_4);

    pthread_mutex_unlock(&mutex_4);
    pthread_mutex_unlock(&mutex_3);

    return (void *)(0);
}

void *thread_rountine_4(void *args)
{
	pthread_t selfid = pthread_self(); //
	
    pthread_mutex_lock(&mutex_4);
    sleep(1);
    pthread_mutex_lock(&mutex_3);

    pthread_mutex_unlock(&mutex_3);
    pthread_mutex_unlock(&mutex_4);

    return (void *)(0);
}


int main()
{

    
    init_hook();
	start_check();

    pthread_t tid1, tid2, tid3, tid4;
    pthread_create(&tid1, NULL, thread_rountine_1, NULL);
    pthread_create(&tid2, NULL, thread_rountine_2, NULL);
    pthread_create(&tid3, NULL, thread_rountine_3, NULL);
    pthread_create(&tid4, NULL, thread_rountine_4, NULL);

    pthread_join(tid1, NULL);
    pthread_join(tid2, NULL);
    pthread_join(tid3, NULL);
    pthread_join(tid4, NULL);

    return 0;
}


```

代码比较简单，就不详细解释了；



# 一. Redis的线程模型和异步机制

## 线程模型:

我们通常说，Redis 是单线程，主要是指 Redis 的**网络 IO** 和**键值对读写**是由一个线程来完成的，这也是 Redis 对外提供键值存储服务的主要流程。

 但 Redis 的其他功能，比如持久化、异步删除、集群数据同步等，其实是由额外的线程执行的。

**为什么使用单线程:**

多线程并发开销大，访问共享资源时，要确保资源的正确性，需要额外的机制保证正确性，额外的操作增加了系统开销。

在Redis 6.0之前，**Redis 在处理客户端的请求时，包括获取 (socket 读)、解析、执行、内容返回 (socket 写) 等都由一个顺序串行的主线程处理，这就是所谓的「单线程」**。其中执行命令阶段，由于 Redis 是单线程来处理命令的，所有每一条到达服务端的命令不会立刻执行，所有的命令都会进入一个 Socket 队列中，当 socket 可读则交给单线程事件分发器逐个被执行。

![image-20220424201957900](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220424201957900.png)

官方曾做过类似问题的回复：使用Redis时，几乎不存在CPU成为瓶颈的情况， Redis主要受限于内存和网络。例如在一个普通的Linux系统上，Redis通过使用pipelining每秒可以处理100万个请求，所以如果应用程序主要使用O(N)或O(log(N))的命令，它几乎不会占用太多CPU。

使用了单线程后，可维护性高。多线程模型虽然在某些方面表现优异，但是它却引入了程序执行顺序的不确定性，带来了并发读写的一系列问题，增加了系统复杂度、同时可能存在线程切换、甚至加锁解锁、死锁造成的性能损耗。Redis通过AE事件模型以及IO多路复用等技术，处理性能非常高，因此没有必要使用多线程。单线程机制使得 Redis 内部实现的复杂度大大降低，Hash 的惰性 Rehash、Lpush 等等 “线程不安全” 的命令都可以无锁进行。

Redis绝大部分操作是基于内存的，而且是纯kv（key-value）操作，所以命令执行速度非常快。我们可以大概理解成，redis中的数据存储在一张大HashMap中，HashMap的优势就是查找和写入的时间复杂度都是O(1)。Redis内部采用这种结构存储数据，就奠定了Redis高性能的基础。根据Redis官网描述，在理想情况下Redis每秒可以提交一百万次请求，每次请求提交所需的时间在纳秒的时间量级。既然每次的Redis操作都这么快，单线程就可以完全搞定了，那还何必要用多线程呢！

## Redis 6.0引入多线程

我们知道，单线程主要在一个CPU核进行工作，但是随着我们硬件的快速升级和大量业务的需求，**单个线程处理网络读写的速度跟不上底层网络硬件的速度**， 读写网络的read/write系统调用占用了redis执行期间大部分CPU时间，瓶颈主要在于网络的IO消耗，优化主要有两个方向:

- 提高网络 IO 性能，典型的实现比如使用 DPDK来替代内核网络栈的方式、零拷贝技术。
- 使用多线程充分利用多核，提高网络请求读写的并行度，典型的实现比如 Memcached。

零拷贝技术有其局限性，无法完全适配redis这一复杂的网络IO模型。而DPDK技术通过旁路网卡IO绕过内核协议栈的方式又太过于复杂以及需要内核甚至硬件的支持，所以我们只能从后者下手啦。主要注意的是，**redis多IO线程模型只用来处理网络读写请求，对于redis的读写命令，依然是单线程处理**。这是因为：

- 网络处理经常是瓶颈，需要通过多线程并行处理可提高性能
- 继续使用单线程执行读写命令，不需要为了保证LUA脚本、事务等开发多线程安全机制，实现更简单

![image-20220424202824284](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220424202824284.png)

**client：**客户端对象，Redis 是典型的 CS 架构（Client <—> Server），客户端通过 socket 与服务端建立网络通道然后发送请求命令，服务端执行请求的命令并回复。Redis 使用结构体 client 存储客户端的所有相关信息，包括但不限于封装的套接字连接 – *conn，当前选择的数据库指针 – *db，读入缓冲区 – querybuf，写出缓冲区 – buf，写出数据链表 – reply等。

**aeApiPoll**：I/O 多路复用 API，是基于 epoll_wait/select/kevent 等系统调用的封装，监听等待读写事件触发，然后处理，它是事件循环（Event Loop）中的核心函数，是事件驱动得以运行的基础。

**acceptTcpHandler：**连接应答处理器，底层使用系统调用 accept 接受来自客户端的新连接，并为新连接注册绑定命令读取处理器，以备后续处理新的客户端 TCP 连接；除了这个处理器，还有对应的 acceptUnixHandler 负责处理 Unix Domain Socket 以及 acceptTLSHandler 负责处理 TLS 加密连接。

**readQueryFromClient**：命令读取处理器，解析并执行客户端的请求命令。

**beforeSleep：**事件循环中进入 aeApiPoll 等待事件到来之前会执行的函数，其中包含一些日常的任务，比如把 client->buf 或者 client->reply （后面会解释为什么这里需要两个缓冲区）中的响应写回到客户端，持久化 AOF 缓冲区的数据到磁盘等，相对应的还有一个 afterSleep 函数，在 aeApiPoll 之后执行。

**sendReplyToClient：**命令回复处理器，当一次事件循环之后写出缓冲区中还有数据残留，则这个处理器会被注册绑定到相应的连接上，等连接触发写就绪事件时，它会将写出缓冲区剩余的数据回写到客户端。

![image-20220424204137925](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220424204137925.png)

**流程简述如下：**

1、主线程负责接收建立连接请求，获取 socket 放入全局等待读处理队列
2、主线程处理完读事件之后，通过 RR(Round Robin) 将这些连接分配给这些 IO 线程
3、主线程阻塞等待 IO 线程读取 socket 完毕
4、主线程通过单线程的方式执行请求命令，请求数据读取并解析完成，但并不执行
5、主线程阻塞等待 IO 线程将数据回写 socket 完毕
6、解除绑定，清空等待队列

该设计有如下特点：

- IO 线程要么同时在读 socket，要么同时在写，不会同时读或写
- IO 线程只负责读写 socket 解析命令，不负责命令处理

**开启多线程后，是否会存在线程并发安全问题？**

不存在。从实现机制可以看出，redis的多线程部分只是用来处理网络数据的读写和协议解析，执行命令仍然是单线程顺序执行所以我们不需要去考虑控制 Key、Lua、事务，LPUSH/LPOP 等等的并发及线程安全问题。

**Redis6.0与Memcached多线程模型对比：**

相同点：都采用了master线程—worker线程的模型

不同点：

- memcached执行主逻辑也是在worker线程里面，模型更进简单，实现了真正的线程隔离
- redis把处理逻辑交还给了master线程，虽然在一定程序上增加了模型复杂度，但也解决了线程并发安全等问题

**Redis 6.0 默认是否开启了多线程?**

redis 6.0 的多线程默认是禁用的，只使用主线程。如需开启在redis.conf文件进行配置

```txt
# 在 redis.conf 中
# if you have a four cores boxes, try to use 2 or 3 I/O threads, if you have
a 8 cores, try to use 6 threads.
io-threads 4
# 默认只开启 encode 也就是redis发送给客户端的协议压缩工作；也可开启io-threads-do-reads
yes来实现 decode;
# 一般发送给redis的命令数据包都比较少，所以不需要开启 decode 功能；
# io-threads-do-reads no
```

## 异步机制

创建线程:

Redis 主线程启动后，会使用操作系统提供的 pthread_create 函数创建 **3 个子线程**，分别由它们负责 **AOF 日志写操作**、**key-value删除**以及**文件关闭的异步执行**。

主线程通过一个**链表形式的任务队列**和子线程进行交互。

当Redis实例收到key-value删除和清空数据库的操作时，主线程会把这个操作封装成一个任务，放入到任务队列中，然后给客户端返回一个完成信息，表明删除已经完成。

  这个时候删除操作还没有执行，等到后台子线程从任务队列中读取任务后，才开始实际删除键值对，并释放相应的内存空间。因此，我们把这种异步删除也称为惰性删除（lazy free）。此时，删除或清空操作不会阻塞主线程，这就避免了对主线程的性能影响。

同步连接方案采用阻塞io来实现；优点是代码书写是同步的，业务逻辑没有割裂；缺点是阻塞当前线程，直至redis返回结果；通常用多个线程来实现线程池来解决效率问题；异步连接方案采用非阻塞io来实现；优点是没有阻塞当前线程，redis 没有返回，依然可以往redis发送命令；缺点是代码书写是异步的（回调函数），业务逻辑割裂，可以通过协程解决
（openresty，skynet）；配合redis6.0以后的io多线程（前提是有大量并发请求），异步连接池，能更好解决应用层的数据访问性能；  

## **Redis pipeline技术**

**redis pipeline 是一个客户端提供的，而不是服务端提供的；**  

当我们使用客户端对 Redis 进行一次操作时，如下图所示，客户端将请求传送给服务器，服务器处理完毕后，再将响应回复给客户端。这要花费一个网络数据包来回的时间。

![image-20220424224751583](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220424224751583.png)

如果连续执行多条指令，那就会花费多个网络数据包来回的时间。
如下图所示。

![image-20220424224808523](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220424224808523.png)

回到客户端代码层面，客户端是经历了读-写-读-写四个操作才完整地执行了两条指令。

现在如果我们调整读写顺序，改成写—写-读-读，这两个指令同样可以正常完成。

两个连续的写操作和两个连续的读操作总共只会花费一次网络来回，就好比连续的 write 操作合并了，连续的 read 操作也合并了一样。

![image-20220424224906813](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220424224906813.png)

这便是管道操作的本质， 服务器根本没有任何区别对待， 还是收到一条消息， 执行一条消息， 回复一条消息的正常的流程。 客户端通过对管道中的指令列表改变读写顺序就可以大幅节省 IO 时间。 管道中指令越多，效果越好。对于request操作，只是将数据写到fd对应的写缓冲区，时间非常快，真正耗时操作在读取response。  

## Redis 事务

MULTI 开启事务，事务执行过程中，单个命令是入队列操作，直到调用 EXEC 才会一起执行；  

> MULTI 开启事务
>
> EXEC 提交事务
>
> DISCARD 取消事务
>
> WATCH  检测key的变动，若在事务执行中，key变动则取消事务；在事务开启前调用，乐观锁实现（cas）;
> 若被取消则事务返回 nil ;  

但是事务在执行的过程中，却不是原子性的，所以我们可以使用lua脚本来实现redis的原子性。

>redis中加载了一个lua虚拟机；用来执行redis lua脚本；redis lua 脚本的执行是原子性的；当某个
>脚本正在执行的时候，不会有其他命令或者脚本被执行；
>lua脚本当中的命令会直接修改数据状态；  



> cat test1.lua | redis-cli script load --pipe
> \# 加载 lua脚本字符串 生成 sha1
> \> script load 'local val = KEYS[1]; return val'
> "b8059ba43af6ffe8bed3db65bac35d452f8115d8"
> \# 检查脚本缓存中，是否有该 sha1 散列值的lua脚本
> \> script exists "b8059ba43af6ffe8bed3db65bac35d452f8115d8"
> \1) (integer) 1
> \# 清除所有脚本缓存
> \> script flush
> OK
> \# 如果当前脚本运行时间过长，可以通过 script kill 杀死当前运行的脚本
> \> script kill
> (error) NOTBUSY No scripts in execution right now.  

**EVAL**  

> \# 测试使用
> EVAL script numkeys key [key ...] arg [arg ...]  

**EVALSHA**  

> \# 线上使用
> EVALSHA sha1 numkeys key [key ...] arg [arg ...]  

## ACID特性分析

> A 原子性；事务是一个不可分割的工作单位，事务中的操作要么全部成功，要么全部失败；redis
> 不支持回滚；即使事务队列中的某个命令在执行期间出现了错误，整个事务也会继续执行下去，直
> 到将事务队列中的所有命令都执行完毕为止。
> C 一致性；事务使数据库从一个一致性状态到另外一个一致性状态；这里的一致性是指预期的一
> 致性而不是异常后的一致性；所以redis也不满足；
> I 隔离性；事务的操作不被其他用户操作所打断；redis命令执行是串行的，redis事务天然具备隔
> 离性；
> D 持久性；redis只有在 aof 持久化策略的时候，并且需要在 redis.conf 中
> appendfsync=always 才具备持久性；实际项目中几乎不会使用 aof 持久化策略  

## redis 发布订阅  

> 为了支持消息的多播机制，redis引入了发布订阅模块；disque 消息队列  

> \# 订阅频道
> subscribe 频道
> \# 订阅模式频道
> psubscribe 频道
> \# 取消订阅频道
> unsubscribe 频道
> \# 取消订阅模式频道
> punsubscribe 频道
> \# 发布具体频道或模式频道的内容
> publish 频道 内容
> \# 客户端收到具体频道内容
> message 具体频道 内容
> \# 客户端收到模式频道内容
> pmessage 模式频道 具体频道 内容  

发布订阅功能一般要区别命令连接重新开启一个连接；因为命令连接严格遵循请求回应模式；而pubsub能收到redis主动推送的内容；所以实际项目中如果支持pubsub的话，需要另开一条连接用于处理发布订阅 。

**缺点:**

> 发布订阅的生产者传递过来一个消息，redis会直接找到相应的消费者并传递过去；假如没有消费
> 者，消息直接丢弃；假如开始有2个消费者，一个消费者突然挂掉了，另外一个消费者依然能收到
> 消息，但是如果刚挂掉的消费者重新连上后，在断开连接期间的消息对于该消费者来说彻底丢失
> 了；
> 另外，redis停机重启，pubsub的消息是不会持久化的，所有的消息被直接丢弃；  





# 二.  Redis 源码分析和存储原理

## 1 . SDS

Redis 没有直接使用c语言传统的字符串表示，而是自己构建一套一种名为简单动态字符串的，简称SDS.

**SDS的定义**

```c
/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */  // 记录buf数组中已使用字节的数量
    uint8_t alloc; /* excluding the header and null terminator */ //记录buf中总分配的大小
    unsigned char flags; /* 3 lsb of type, 5 unused bits */ //判断是哪一种类型8, 16, 32, 64
    char buf[]; //字节数组，用于保存字符串
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

根据传统，C语言使用长度为N+1的字符数组来表示长度为N的字符串，并且字符数组的最后一个元素总是空字符'\0'。C语言使用的这种简单的字符串表示方式，并不能满足Redis对字符串在安全性、效率以及功能方面的要求。而Redisz使用结构体中的alloc来分配和计算长度，保证了字符串如果中间有结束字符，并不会结束。通过使用SDS而不是C字符串，Redis将获取字符串长度所需的复杂度从O（N）降低到了O（1），这确保了获取字符串长度的工作不会成为Redis的性能瓶颈。例如，因为字符串键在底层使用SDS来实现，所以即使我们对一个非常长的字符串键反复执行STRLEN命令，也不会对系统性能造成任何影响，因为STRLEN命令的复杂度仅为O（1）。

减少修改字符串时带来的内存重分配次数

因为C字符串并不记录自身的长度，所以对于一个包含了N个字符的C字符串来说，这个C字符串的底层实现总是一个N+1个字符长的数组（额外的一个字符空间用于保存空字符）。因为C字符串的长度和底层数组的长度之间存在着这种关联性，所以每次增长或者缩短一个C字符串，程序都总要对保存这个C字符串的数组进行一次内存重分配操作：

**而SDS实现了空间预分配和惰性空间释放两种优化策略：**

**空间预分配**：

- 如果对SDS进行修改之后，SDS的长度（也即是len属性的值）将小于1MB，那么程序分配和len属性同样大小的未使用空间，这时SDS len属性的值将和free属性的值相同。举个例子，如果进行修改之后，SDS的len将变成13字节，那么程序也会分配13字节的未使用空间，SDS的buf数组的实际长度将变成13+13+1=27字节（额外的一字节用于保存空字符）。
- 如果对SDS进行修改之后，SDS的长度将大于等于1MB，那么程序会分配1MB的未使用空间。举个例子，如果进行修改之后，SDS的len将变成30MB，那么程序会分配1MB的未使用空间，SDS的buf数组的实际长度将为30MB+1MB+1byte

**惰性空间释放:**

- 惰性空间释放用于优化SDS的字符串缩短操作：当SDS的API需要缩短SDS保存的字符串时，程序并不立即使用内存重分配来回收缩短后多出来的字节，而是使用free属性将这些字节的数量记录起来，并等待将来使用。



总结和C字符串的区别

![image-20220502153027135](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502153027135.png)



## 2. 链表

链表提供了高效的节点重排能力，以及顺序性的节点访问方式，并且可以通过增删节点来灵活地调整链表的长度。作为一种常用数据结构，链表内置在很多高级的编程语言里面，因为Redis使用的C语言并没有内置这种数据结构，所以Redis构建了自己的链表实现。

```c
typedef struct listNode {
    struct listNode *prev;  //前置节点
    struct listNode *next;  //后置节点
    void *value;   //节点的值
} listNode;
```

多个listNode可以通过prev和next指针组成双端链表

![image-20220502153325564](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502153325564.png)

虽然仅仅使用多个listNode结构就可以组成链表，但使用adlist.h/list来持有链表的话，操作起来会更方便：

```c
typedef struct list {
    listNode *head;  //表头节点
    listNode *tail; //表尾节点
    void *(*dup)(void *ptr); //节点值复制函数
    void (*free)(void *ptr); //节点释放函数
    int (*match)(void *ptr, void *key); //节点值对比函数
    unsigned long len;  //链表所包含的节点数量
} list;
```

list结构为链表提供了表头指针head、表尾指针tail，以及链表长度计数器len，而dup、free和match成员则是用于实现多态链表所需的类型特定函数：

- dup函数用于复制链表节点所保存的值；
- free函数用于释放链表节点所保存的值；
- match函数则用于对比链表节点所保存的值和另一个输入值是否相等

下图是由一个list结构和三个listNode结构组成的链表。

![image-20220502164448173](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502164448173.png)

Redis的链表实现的特性可以总结如下:

- 双端：链表节点带有prev和next指针，获取某个节点的前置节点和后置节点的复杂度都是O（1）。
- 无环：表头节点的prev指针和表尾节点的next指针都指向NULL，对链表的访问以NULL为终点。
- 带表头指针和表尾指针：通过list结构的head指针和tail指针，程序获取链表的表头节点和表尾节点的复杂度为O（1）。
- 带链表长度计数器：程序使用list结构的len属性来对list持有的链表节点进行计数，程序获取链表中节点数量的复杂度为O（1）。
- 多态：链表节点使用void*指针来保存节点值，并且可以通过list结构的dup、free、match三个属性为节点值设置类型特定函数，所以链表可以用于保存各种不同类型的值。

## 3. 字典(dict)

字典，又称为符号表（symbol table）、关联数组（associative array）或映射（map），是一种用于保存键值对（key-value pair）的抽象数据结构。在字典中，一个键（key）可以和一个值（value）进行关联（或者说将键映射为值），这些关联的键和值就称为键值对。字典中的每个键都是独一无二的，程序可以在字典中根据键查找与之关联的值，或者通过键来更新值，又或者根据键来删除整个键值对，等等。字典在Redis中的应用相当广泛，比如Redis的数据库就是使用字典来作为底层实现的，对数据库的增、删、查、改操作也是构建在对字典的操作之上的。

Redis的字典使用哈希表作为底层实现，一个哈希表里面可以有多个哈希表节点，而每个哈希表节点就保存了字典中的一个键值对。

```c
/* This is our hash table structure. Every dictionary has two of this as we
 * implement incremental rehashing, for the old to the new table. */
typedef struct dictht {
    dictEntry **table;  //哈希表数组
    unsigned long size; //哈希表大小
    unsigned long sizemask; //哈希表大小掩码，用于计算索引值，总是等于size-1
    unsigned long used; //哈希表已有节点的数量
} dictht;
```

table属性是一个数组，数组中的每个元素都是一个指向dict.h/dictEntry结构的指针，每个dictEntry结构保存着一个键值对。size属性记录了哈希表的大小，也即是table数组的大小，而used属性则记录了哈希表目前已有节点（键值对）的数量。sizemask属性的值总是等于size-1，这个属性和哈希值一起决定一个键应该被放到table数组的哪个索引上面。

![image-20220502164831033](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502164831033.png)

哈希表节点使用dictEntry结构表示，每个dictEntry结构都保存着一个键值对:

```c
typedef struct dictEntry {
    void *key;  //键
    union {   //不同类型的值
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    struct dictEntry *next;  //指向下个哈希表节点，形成链表
} dictEntry;
```

key属性保存着键值对中的键，而v属性则保存着键值对中的值，其中键值对的值可以是一个指针，或者是一个uint64_t整数，又或者是一个int64_t整数。

next属性是指向另一个哈希表节点的指针，这个指针可以将多个哈希值相同的键值对连接在一次，以此来解决键冲突（collision）的问题。

下图就展示了如何通过next指针，将两个索引值相同的键k1和k0连接在一起。 

![image-20220502165040780](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502165040780.png)



**Redis中的字典由dict.h/dict结构表示**

```c
typedef struct dict {
    dictType *type;  //类型特定函数
    void *privdata; //私有数据
    dictht ht[2]; //哈希表
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    int16_t pauserehash; /* If >0 rehashing is paused (<0 indicates coding error) */
} dict;
```

type属性和privdata属性是针对不同类型的键值对，为创建多态字典而设置的：

- type属性是一个指向dictType结构的指针，每个dictType结构保存了一簇用于操作特定类型键值对的函数，Redis会为用途不同的字典设置不同的类型特定函数。-
- 而privdata属性则保存了需要传给那些类型特定函数的可选参数。



```c
typedef struct dictType {
    uint64_t (*hashFunction)(const void *key);
    void *(*keyDup)(void *privdata, const void *key);
    void *(*valDup)(void *privdata, const void *obj);
    int (*keyCompare)(void *privdata, const void *key1, const void *key2);
    void (*keyDestructor)(void *privdata, void *key);
    void (*valDestructor)(void *privdata, void *obj);
    int (*expandAllowed)(size_t moreMem, double usedRatio);
} dictType;
```

ht属性是一个包含两个项的数组，数组中的每个项都是一个dictht哈希表，一般情况下，字典只使用ht[0]哈希表，ht[1]哈希表只会在对ht[0]哈希表进行rehash时使用。除了ht[1]之外，另一个和rehash有关的属性就是rehashidx，它记录了rehash目前的进度，如果目前没有在进行rehash，那么它的值为-1。

下图展示了一个普通状态下（没有进行rehash）的字典。

![image-20220502165620986](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502165620986.png)



**解决键冲突:** 

当有两个或以上数量的键被分配到了哈希表数组的同一个索引上面时，我们称这些键发生了冲突（collision）。Redis的哈希表使用链地址法（separate chaining）来解决键冲突，每个哈希表节点都有一个next指针，多个哈希表节点可以用next指针构成一个单向链表，被分配到同一个索引上的多个节点可以用这个单向链表连接起来，这就解决了键冲突的问题。举个例子，假设程序要将键值对k2和v2添加到图所示的哈希表里面，并且计算得出k2的索引值为2，那么键k1和k2将产生冲突，而解决冲突的办法就是使用next指针将键k2和k1所在的节点连接起来，如图4-7所示。因为dictEntry节点组成的链表没有指向链表表尾的指针，所以为了速度考虑，程序总是将新节点添加到链表的表头位置（复杂度为O（1）），排在其他已有节点的前面。

![image-20220502165814623](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502165814623.png)



**rehash**

随着操作的不断执行，哈希表保存的键值对会逐渐地增多或者减少，为了让哈希表的负载因子（loadfactor）维持在一个合理的范围之内，当哈希表保存的键值对数量太多或者太少时，程序需要对哈希表的大小进行相应的扩展或者收缩,扩展和收缩哈希表的工作可以通过执行rehash（重新散列）操作来完成，Redis对字典的哈希表执行rehash的步骤如下：

1. 为字典的ht[1]哈希表分配空间，这个哈希表的空间大小取决于要执行的操作，以及ht[0]当前包含的键值对数量（也即是ht[0].used属性的值）：
   - 如果执行的是扩展操作，那么ht[1]的大小为第一个大于等于ht[0].used*2的$2^n$；
   - 如果执行的是收缩操作，那么ht[1]的大小为第一个大于等于ht[0].used的$2^n$。
2. 将保存在ht[0]中的所有键值对rehash到ht[1]上面：rehash指的是重新计算键的哈希值和索引值，然后将键值对放置到ht[1]哈希表的指定位置上。
3. 当ht[0]包含的所有键值对都迁移到了ht[1]之后（ht[0]变为空表），释放ht[0]，将ht[1]设置为ht[0]，并在ht[1]新创建一个空白哈希表，为下一次rehash做准备。

举个例子，假设程序要对图所示字典的ht[0]进行扩展操作，那么程序将执行以下步骤:

![image-20220502170105487](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502170105487.png)

1. ht[0].used当前的值为4，4*2=8，而8（$2^3$）恰好是第一个大于等于4的2的n次方，所以程序会将ht[1]哈希表的大小设置为8。下图展示了ht[1]在分配空间之后，字典的样子。

![](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502170237230.png)

2. 将ht[0]包含的四个键值对都rehash到ht[1]，如图所示

   ![image-20220502170330699](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502170330699.png)

3. 释放ht[0]，并将ht[1]设置为ht[0]，然后为ht[1]分配一个空白哈希表，如下图所示。至此，对哈希表的扩展操作执行完毕，程序成功将哈希表的大小从原来的4改为了现在的8。

![image-20220502170413772](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502170413772.png)

哈希表的扩展与收缩当以下条件中的任意一个被满足时，程序会自动开始对哈希表执行扩展操作：

- 服务器目前没有在执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于1。
- 服务器目前正在执行BGSAVE命令或者BGREWRITEAOF命令，并且哈希表的负载因子大于等于5。

其中哈希表的负载因子可以通过公式：

> 负载因子 = 哈希表已保存节点数量 / 哈希表的大小
>
> load_factor = ht[0].used / ht[0].size

根据BGSAVE命令或BGREWRITEAOF命令是否正在执行，服务器执行扩展操作所需的负载因子并不相同，这是因为在执行BGSAVE命令或BGREWRITEAOF命令的过程中，Redis需要创建当前服务器进程的子进程，而大多数操作系统都采用写时复制（copy-on-write）技术来优化子进程的使用效率，所以在子进程存在期间，服务器会提高执行扩展操作所需的负载因子，从而尽可能地避免在子进程存在期间进行哈希表扩展操作，这可以避免不必要的内存写入操作，最大限度地节约内存。

另一方面，当哈希表的负载因子小于0.1时，程序自动开始对哈希表执行收缩操作。

但是，这个rehash动作并不是一次性、集中式地完成的，而是分多次、渐进式地完成的。这样做的原因在于，如果ht[0]里只保存着四个键值对，那么服务器可以在瞬间就将这些键值对全部rehash到ht[1]；但是，如果哈希表里保存的键值对数量不是四个，而是四百万、四千万甚至四亿个键值对，那么要一次性将这些键值对全部rehash到ht[1]的话，庞大的计算量可能会导致服务器在一段时间内停止服务。因此，为了避免rehash对服务器性能造成影响，服务器不是一次性将ht[0]里面的所有键值对全部rehash到ht[1]，而是分多次、渐进式地将ht[0]里面的键值对慢慢地rehash到ht[1]。

因此，为了避免rehash对服务器性能造成影响，服务器不是一次性将ht[0]里面的所有键值对全部rehash到ht[1]，而是分多次、渐进式地将ht[0]里面的键值对慢慢地rehash到ht[1]。

以下是哈希表渐进式rehash的详细步骤：

1. 为ht[1]分配空间，让字典同时持有ht[0]和ht[1]两个哈希表。
2. 在字典中维持一个索引计数器变量rehashidx，并将它的值设置为0，表示rehash工作正式开始。
3. 在rehash进行期间，每次对字典执行添加、删除、查找或者更新操作时，程序除了执行指定的操作以外，还会顺带将ht[0]哈希表在rehashidx索引上的所有键值对rehash到ht[1]，当rehash工作完成之后，程序将rehashidx属性的值增一。
4. 随着字典操作的不断执行，最终在某个时间点上，ht[0]的所有键值对都会被rehash至ht[1]，这时程序将rehashidx属性的值设为-1，表示rehash操作已完成。

渐进式rehash的好处在于它采取分而治之的方式，将rehash键值对所需的计算工作均摊到对字典的每个添加、删除、查找和更新操作上，从而避免了集中式rehash而带来的庞大计算量。

**大KEY**

在redis实例中形成了很大的对象，比如一个很大的hash或很大的zset，这样的对象在扩容的时候，会一次性申请更大的一块内存，这会导致卡顿；如果这个大key被删除，内存会一次性回收，卡顿现象会再次产生；如果观察到redis的内存大起大落，极有可能因为大key导致的；  

> \# 每隔0.1秒 执行100条scan命令
> redis-cli -h 127.0.0.1 --bigkeys -i 0.1  

## 4. 跳表(skiplist)

跳表（多层级有序链表）结构用来实现有序集合；鉴于redis需要实现 zrange 以及 zrevrange功能；需要节点间最好能直接相连并且增删改操作后结构依然有序；B+树时间复杂度为$h * O(log_2{n})$鉴; 于B+复杂的节点分裂操作；考虑其他数据结构;

有序数组通过二分查找能获得$O(log_2{n})$时间复杂度， 平衡二叉树也能获得$O(log_2{n})$时间复杂度；

在大部分情况下，跳跃表的效率可以和平衡树相媲美，并且因为跳跃表的实现比平衡树要来得更为简单，所以有不少程序都使用跳跃表来代替平衡树。Redis使用跳跃表作为有序集合键的底层实现之一，如果一个有序集合包含的元素数量比较多，又或者有序集合中元素的成员（member）是比较长的字符串时，Redis就会使用跳跃表来作为有序集合键的底层实现。

和链表、字典等数据结构被广泛地应用在Redis内部不同，Redis只在两个地方用到了跳跃表，一个是实现有序集合键，另一个是在集群节点中用作内部数据结构，除此之外，跳跃表在Redis里面没有其他用途。

Redis的跳跃表由redis.h/zskiplistNode和redis.h/zskiplist两个结构定义，其中zskiplistNode结构用于表示跳跃表节点，而zskiplist结构则用于保存跳跃表节点的相关信息，比如节点的数量，以及指向表头节点和表尾节点的指针等等。

从节约内存出发，redis 考虑牺牲一点时间复杂度让跳表结构更加变扁平，就像二叉堆改成四叉堆
结构；并且redis 还限制了跳表的最高层级为 32 ；节点数量大于 128 或者有一个字符串长度大于 64 ，则使用跳表（ skiplist ）；  

```c
/* ZSETs use a specialized version of Skiplists */
typedef struct zskiplistNode {
    sds ele;    //成员对象，以sds实现
    double score; // 节点按各自所保存的分值从小到大排列。
    struct zskiplistNode *backward;  // 从后向前遍历的指针
    struct zskiplistLevel {
        struct zskiplistNode *forward; // 前进指针
        unsigned long span; // 跨度
    } level[];
} zskiplistNode;
```

![image-20220502171657497](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502171657497.png)

上图展示了一个跳跃表示例，位于图片最左边的是zskiplist结构，该结构包含以下属性:

- header：指向跳跃表的表头节点
- tail：指向跳跃表的表尾节点
- level：记录目前跳跃表内，层数最大的那个节点的层数（表头节点的层数不计算在内）。
- length：记录跳跃表的长度，也即是，跳跃表目前包含节点的数量（表头节点不计算在内）。

位于zskiplist结构右方的是四个zskiplistNode结构，该结构包含以下属性：

- 层（level）：节点中用L1、L2、L3等字样标记节点的各个层，L1代表第一层，L2代表第二层，以此类推。每个层都带有两个属性：前进指针和跨度。前进指针用于访问位于表尾方向的其他节点，而跨度则记录了前进指针所指向节点和当前节点的距离。在上面的图片中，连线上带有数字的箭头就代表前进指针，而那个数字就是跨度。当程序从表头向表尾进行遍历时，访问会沿着层的前进指针进行。
- 后退（backward）指针：节点中用BW字样标记节点的后退指针，它指向位于当前节点的前一个节点。后退指针在程序从表尾向表头遍历时使用。
- 分值（score）：各个节点中的1.0、2.0和3.0是节点所保存的分值。在跳跃表中，节点按各自所保存的分值从小到大排列。在同一个跳跃表中，各个节点保存的成员对象必须是唯一的，但是多个节点保存的分值却可以是相同的：分值相同的节点将按照成员对象在字典序中的大小来进行排序，成员对象较小的节点会排在前面（靠近表头的方向），而成员对象较大的节点则会排在后面（靠近表尾的方向）
- 成员对象（sds）：各个节点中的o1、o2和o3是节点所保存的成员对象。

- 层的跨度（level[i].span属性）用于记录两个节点之间的距离：
  - 两个节点之间的跨度越大，它们相距得就越远。
  - 指向NULL的所有前进指针的跨度都为0，因为它们没有连向任何节点。
  - 初看上去，很容易以为跨度和遍历操作有关，但实际上并不是这样，遍历操作只使用前进指针就可以完成了，跨度实际上是用来计算排位（rank）的：在查找某个节点的过程中，将沿途访问过的所有层的跨度累计起来，得到的结果就是目标节点在跳跃表中的排位。
  - 举个例子，图5-4用虚线标记了在跳跃表中查找分值为3.0、成员对象为o3的节点时，沿途经历的层：查找的过程只经过了一个层，并且层的跨度为3，所以目标节点在跳跃表中的排位为3
  - ![image-20220502172329704](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502172329704.png)



仅靠多个跳跃表节点就可以组成一个跳跃表，如图所示。但通过使用一个zskiplist结构来持有这些节点，程序可以更方便地对整个跳跃表进行处理，比如快速访问跳跃表的表头节点和表尾节点，或者快速地获取跳跃表节点的数量（也即是跳跃表的长度）等信息，如图所示。

```c
typedef struct zskiplist {
    struct zskiplistNode *header, *tail;
    unsigned long length;
    int level;
} zskiplist;
```

![image-20220502172552048](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502172552048.png)

header和tail指针分别指向跳跃表的表头和表尾节点，通过这两个指针，程序定位表头节点和表尾节点的复杂度为$O(1)$。通过使用length属性来记录节点的数量，程序可以在$O(1)$复杂度内返回跳跃表的长度。level属性则用于在$O(1)$复杂度内获取跳跃表中层高最大的那个节点的层数量，注意表头节点的层高并不计算在内。



## 5. 整数集合

整数集合（intset）是集合键的底层实现之一，当一个集合只包含整数值元素，并且这个集合的元素数量不多时，Redis就会使用整数集合作为集合键的底层实现。如果我们创建一个集合键，并且集合中的所有元素都是整数值，那么这个集合键的底层实现就会是整数集合：

整数集合（intset）是Redis用于保存整数值的集合抽象数据结构，它可以保存类型为int16_t、int32_t或者int64_t的整数值，并且保证集合中不会出现重复元素。每个intset.h/intset结构表示一个整数集合：

```c
typedef struct intset {
    uint32_t encoding;  // 编码方式
    uint32_t length;   //包含元素的数量
    int8_t contents[];  // 保存元素的数组
} intset;
```

contents数组是整数集合的底层实现：整数集合的每个元素都是contents数组的一个数组项（item），各个项在数组中按值的大小从小到大有序地排列，并且数组中不包含任何重复项。length属性记录了整数集合包含的元素数量，也即是contents数组的长度。虽然intset结构将contents属性声明为int8_t类型的数组，但实际上contents数组并不保存任何int8_t类型的值，contents数组的真正类型取决于encoding属性的值

- 如果encoding属性的值为INTSET_ENC_INT16，那么contents就是一个int16_t类型的数组，数组里的每个项都是一个int16_t类型的整数值（最小值为-32768，最大值为32767）。
- 如果encoding属性的值为INTSET_ENC_INT32，那么contents就是一个int32_t类型的数组，数组里的每个项都是一个int32_t类型的整数值（最小值为-2147483648，最大值为2147483647）。
- 如果encoding属性的值为INTSET_ENC_INT64，那么contents就是一个int64_t类型的数组，数组里的每个项都是一个int64_t类型的整数值（最小值为-9223372036854775808，最大值为9223372036854775807）。

展示了一个整数集合示例: 

![image-20220502175543537](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502175543537.png)



- encoding属性的值为INTSET_ENC_INT16，表示整数集合的底层实现为int16_t类型的数组，而集合保存的都是int16_t类型的整数值。
- length属性的值为5，表示整数集合包含五个元素。
- contents数组按从小到大的顺序保存着集合中的五个元素。
- 因为每个集合元素都是int16_t类型的整数值，所以contents数组的大小等于`sizeof（int16_t）*5=16*5=80`位。



## 6. 压缩列表(ziplist)

压缩列表（ziplist）是列表键和哈希键的底层实现之一。当一个列表键只包含少量列表项，并且每个列表项要么就是小整数值，要么就是长度比较短的字符串，那么Redis就会使用压缩列表来做列表键的底层实现。

压缩列表是Redis为了节约内存而开发的，是由一系列特殊编码的连续内存块组成的顺序型（sequential）数据结构。一个压缩列表可以包含任意多个节点（entry），每个节点可以保存一个字节数组或者一个整数值。

```c
/* Each entry in the ziplist is either a string or an integer. */
typedef struct {
    /* When string is used, it is provided with the length (slen). */
    unsigned char *sval;
    unsigned int slen;
    /* When integer is used, 'sval' is NULL, and lval holds the value. */
    long long lval;
} ziplistEntry;
```

每个压缩列表节点可以保存一个字节数组或者一个整数值，其中，字节数组可以是以下三种长度的其中一种：

- 长度小于等于63（$2^6–1$）字节的字节数组；
- 长度小于等于16383（$2^{14}–1$）字节的字节数组；
- 长度小于等于4294967295（$2^{32}–1$）字节的字节数组

而整数值则可以是以下六种长度的其中一种

- 4位长，介于0至12之间的无符号整数；
- 1字节长的有符号整数；
- 3字节长的有符号整数；
- int16_t类型整数；
- int32_t类型整数；
- int64_t类型整数

每个压缩列表节点都由previous_entry_length、encoding、content三个部分组成：

![image-20220502180256068](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220502180256068.png)

- previous_entry_length: 节点的previous_entry_length属性以字节为单位，记录了压缩列表中前一个节点的长度。previous_entry_length属性的长度可以是1字节或者5字节：
  - 如果前一节点的长度小于254字节，那么previous_entry_length属性的长度为1字节：前一节点的长度就保存在这一个字节里面。
  - 如果前一节点的长度大于等于254字节，那么previous_entry_length属性的长度为5字节：其中属性的第一字节会被设置为0xFE（十进制值254），而之后的四个字节则用于保存前一节点的长度。
- 节点的encoding属性记录了节点的content属性所保存数据的类型以及长度：
  - 一字节、两字节或者五字节长，值的最高位为00、01或者10的是字节数组编码：这种编码表示节点的content属性保存着字节数组，数组的长度由编码除去最高两位之后的其他位记录；
  - 一字节长，值的最高位以11开头的是整数编码：这种编码表示节点的content属性保存着整数值，整数值的类型和长度由编码除去最高两位之后的其他位记录；
- 节点的content属性负责保存节点的值，节点值可以是一个字节数组或者整数，值的类型和长度由节点的encoding属性决定。





# 三. Redis 集群和持久化

## 1. RDB持久化

因为Redis是内存数据库(也是一个键值对数据库)，它将自己的数据库状态储存在内存里面，所以如果不想办法将储存在内存中的数据库状态保存到磁盘里面，那么一旦服务器进程退出，服务器中的数据库状态也会消失不见。为了解决这个问题，Redis提供了RDB持久化功能，这个功能可以将Redis在内存中的数据库状态以压缩的二进制形式保存到磁盘里面，避免数据意外丢失， 并且通过该文件可以还原生成RDB文件时的数据库状态。

### a. RDB文件的创建和载入

有两个Redis命令可以用于生成RDB文件，一个是SAVE，另一个是BGSAVE。

SAVE命令会阻塞Redis服务器进程，直到RDB文件创建完毕为止，在服务器进程阻塞期间，服务器不能处理任何命令请求：

> redis > SAVA                //等待知道RDB文件创建完毕
>
> OK

和SAVE命令直接阻塞服务器进程的做法不同，BGSAVE命令会派生出一个子进程，然后由子进程负责创建RDB文件，服务器进程（父进程）继续处理命令请求

> redis > BGSAVA                //派生子进程， 并由子进程创建RDB文件
>
> Background saving started

![image-20220514164130024](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514164130024.png)

创建RDB文件的实际工作由rdb.c/rdbSave函数完成。和使用SAVE命令或者BGSAVE命令创建RDB文件不同，RDB文件的载入工作是在服务器启动时自动执行的，所以Redis并没有专门用于载入RDB文件的命令，只要Redis服务器在启动时检测到RDB文件存在，它就会自动载入RDB文件。

另外值得一提的是，因为AOF文件的更新频率通常比RDB文件的更新频率高，所以：

- 如果服务器开启了AOF持久化功能，那么服务器会优先使用AOF文件来还原数据库状态。
- 只有在AOF持久化功能处于关闭状态时，服务器才会使用RDB文件来还原数据库状态。

载入RDB文件的实际工作由rdb.c/rdbLoad函数完成。

因为BGSAVE命令的保存工作是由子进程执行的，所以在子进程创建RDB文件的过程中，Redis服务器仍然可以继续处理客户端的命令请求，但是，在BGSAVE命令执行期间，服务器处理SAVE、BGSAVE、BGREWRITEAOF三个命令的方式会和平时有所不同。

- 如果BGSAVE命令正在执行，客户端发送的BGSAVE命令会被服务器拒绝，因为同时执行两个BGSAVE命令也会产生竞争条件。

- 如果BGSAVE命令正在执行，那么客户端发送的BGREWRITEAOF命令会被延迟到BGSAVE命令执行完毕之后执行。
- 如果BGREWRITEAOF命令正在执行，那么客户端发送的BGSAVE命令会被服务器拒绝

服务器在载入RDB文件期间，会一直处于阻塞状态，直到载入工作完成为止。 BGSAVE命令可以在不阻塞服务器进程的情况下执行，所以Redis允许用户通过设置服务器配置的save选项，让服务器每隔一段时间自动执行一次BGSAVE命令，用户可以通过save选项设置多个保存条件，但只要其中任意一个条件被满足，服务器就会执行BGSAVE命令。

> \# redis 默认策略如下：
> \# 注意：写了多个 save 策略，只需要满足一个则开启rdb持久化
> \# 3600 秒内有以1次修改
> save 3600 1
> \# 300 秒内有100次修改
> save 300 100
> \# 60 秒内有10000次修改
> save 60 10000  

### b. RDB文件结构

![image-20220514171846865](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514171846865.png)

1. RDB文件的最开头是REDIS部分，这个部分的长度为5字节，保存着“REDIS”五个字符。通过这五个字符，程序可以在载入文件时，快速检查所载入的文件是否RDB文件；
2. db_version长度为4字节，它的值是一个字符串表示的整数，这个整数记录了RDB文件的版本号，比如"0006"就代表RDB文件的版本为第六版
3. databases部分包含着零个或任意多个数据库，以及各个数据库中的键值对数据
4. databases部分包含着零个或任意多个数据库，以及各个数据库中的键值对数据
5. check_sum是一个8字节长的无符号整数，保存着一个校验和，这个校验和是程序通过对REDIS、db_version、databases、EOF四个部分的内容进行计算得出的。服务器在载入RDB文件时，会将载入数据所计算出的校验和与check_sum所记录的校验和进行对比，以此来检查RDB文件是否有出错或者损坏的情况出现。

因为Redis本身带有RDB文件检查工具redis-check-dump，网上也能找到很多处理RDB文件的工具，所以人工分析RDB文件的内容并不是学习Redis所必须掌握的技能。

### c. 配置redis.conf

> \# 关闭 aof 同时也关闭了 aof复写
> appendonly no
> \# 关闭 aof复写
> auto-aof-rewrite-percentage 0
> \# 关闭 混合持久化
> aof-use-rdb-preamble no
> \# 开启 rdb 也就是注释 save ""
> \# save ""
> \# save 3600 1
> \# save 300 100
> \# save 60 10000  

### d. 缺点

若采用 rdb 持久化，一旦 redis 宕机，redis将丢失一段时间的数据；RDB 需要经常 fork 子进程来保存数据集到硬盘上，当数据集比较大的时候，fork 的过程是非常耗时的，可能会导致 Redis 在一些毫秒级内不能响应客户端的请求。如果数据集巨大并且 CPU 性能不是很好的情况下，这种情况会持续1秒，AOF 也需要 fork，但是你可以调节重写日志文件的频率
来提高数据集的耐久度。  

## 2. AOF持久化

除了RDB持久化功能之外，Redis还提供了AOF（Append Only File）持久化功能。与RDB持久化通过保存数据库中的键值对来记录数据库状态不同，AOF持久化是通过保存Redis服务器所执行的写命令来记录数据库状态的。

![image-20220514172318803](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514172318803.png)



举个例子，如果我们对空白的数据库执行以下写命令，那么数据库中将包含三个键值对。

>redis > SET msg "hello"
>
>OK
>
>redis > SADD fruits "apple"  "banana" "cherry"
>
>(integer) 3
>
>redis > RPUSH numbers 128 256 512
>
>(integer) 3

RDB持久化保存数据库状态的方法是将msg、fruits、numbers三个键的键值对保存到RDB文件中，而AOF持久化保存数据库状态的方法则是将服务器执行的SET、SADD、RPUSH三个命令保存到AOF文件中。AOF持久化功能的实现可以分为命令追加（append）、文件写入、文件同步（sync）三个步骤。

> append only file
> aof 日志存储的是 Redis 服务器的顺序指令序列，aof 日志只记录对内存修改的指令记录；  

### a. 追加

当AOF持久化功能处于打开状态时，服务器在执行完一个写命令之后，会以协议格式将被执行的写命令追加到服务器状态的aof_buf缓冲区的末尾：

### b. 写入和同步

Redis的服务器进程就是一个事件循环（loop），这个循环中的文件事件负责接收客户端的命令请求，以及向客户端发送命令回复，而时间事件则负责执行像serverCron函数这样需要定时运行的函数。因为服务器在处理文件事件时可能会执行写命令，使得一些内容被追加到aof_buf缓冲区里面，所以在服务器每次结束一个事件循环之前，它都会调用flushAppendOnlyFile函数，考虑是否需要将aof_buf缓冲区中的内容写入和保存到AOF文件里面。flushAppendOnlyFile函数的行为由服务器配置的appendfsync选项的值来决定， 如果用户没有主动为appendfsync选项设置值，那么appendfsync选项的默认值为everysec。 

![image-20220514172957154](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514172957154.png)

- 当appendfsync的值为always时，服务器在每个事件循环都要将aof_buf缓冲区中的所有内容写入到AOF文件，并且同步AOF文件，所以always的效率是appendfsync选项三个值当中最慢的一个，但从安全性来说，always也是最安全的，因为即使出现故障停机，AOF持久化也只会丢失一个事件循环中所产生的命令数据。
- 当appendfsync的值为everysec时，服务器在每个事件循环都要将aof_buf缓冲区中的所有内容写入到AOF文件，并且每隔一秒就要在子线程中对AOF文件进行一次同步。从效率上来讲，everysec模式足够快，并且就算出现故障停机，数据库也只丢失一秒钟的命令数据
- 当appendfsync的值为no时，服务器在每个事件循环都要将aof_buf缓冲区中的所有内容写入到AOF文件，至于何时对AOF文件进行同步，则由操作系统控制。因为处于no模式下的flushAppendOnlyFile调用无须执行同步操作，所以该模式下的AOF文件写入速度总是最快的，不过因为这种模式会在系统缓存中积累一段时间的写入数据，所以该模式的单次同步时长通常是三种模式中时间最长的。从平摊操作的角度来看，no模式和everysec模式的效率类似，当出现故障停机时，使用no模式的服务器将丢失上次同步AOF文件之后的所有写命令数据。

为了提高文件的写入效率，在现代操作系统中，当用户调用write函数，将一些数据写入到文件的时候，操作系统通常会将写入数据暂时保存在一个内存缓冲区里面，等到缓冲区的空间被填满、或者超过了指定的时限之后，才真正地将缓冲区中的数据写入到磁盘里面。这种做法虽然提高了效率，但也为写入数据带来了安全问题，因为如果计算机发生停机，那么保存在内存缓冲区里面的写入数据将会丢失。为此，系统提供了fsync和fdatasync两个同步函数，它们可以强制让操作系统立即将缓冲区中的数据写入到硬盘里面，从而确保写入数据的安全性。

### c. AOF文件的载入和数据还原

因为AOF文件里面包含了重建数据库状态所需的所有写命令，所以服务器只要读入并重新执行一遍AOF文件里面保存的写命令，就可以还原服务器关闭之前的数据库状态。

详细步骤如下:

- 创建一个不带网络连接的伪客户端（fake client）：因为Redis的命令只能在客户端上下文中执行，而载入AOF文件时所使用的命令直接来源于AOF文件而不是网络连接，所以服务器使用了一个没有网络连接的伪客户端来执行AOF文件保存的写命令，伪客户端执行命令的效果和带网络连接的客户端执行命令的效果完全一样。
- 从AOF文件中分析并读取出一条写命令。
- 使用伪客户端执行被读出的写命令
- 一直执行步骤2和步骤3，直到AOF文件中的所有写命令都被处理完毕为止。

![image-20220514173419130](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514173419130.png)

### d. aof重写

因为AOF持久化是通过保存被执行的写命令来记录数据库状态的，所以随着服务器运行时间的流逝，AOF文件中的内容会越来越多，文件的体积也会越来越大，如果不加以控制的话，体积过大的AOF文件很可能对Redis服务器、甚至整个宿主计算机造成影响，并且AOF文件的体积越大，使用AOF文件来进行数据还原所需的时间就越多。为了解决AOF文件体积膨胀的问题，Redis提供了AOF文件重写（rewrite）功能。通过该功能，Redis服务器可以创建一个新的AOF文件来替代现有的AOF文件，新旧两个AOF文件所保存的数据库状态相同，但新AOF文件不会包含任何浪费空间的冗余命令，所以新AOF文件的体积通常会比旧AOF文件的体积要小得多。

> lpush list xiaoming1
> lpush list xiaoming2
> lpush list xiaoming3
> bgrewriteaof
> \# 此时会将上面三个命令进行合并成为一个命令
> \# 合并策略：会先检测键所包含的元素数量，如果超过 64 个会使用多个命令来记录键的值；
> hset hash da1 10001
> hset hash da2 10002
> hset hash da3 10003
> hdel hash da1 
> bgrewriteaof
> \# 此时aof中不会出现da1 ，设置da1 跟删除da1 变得像从来没操作过  

aof_rewrite函数可以很好地完成创建一个新AOF文件的任务，但是，因为这个函数会进行大量的写入操作，所以调用这个函数的线程将被长时间阻塞，因为Redis服务器使用单个线程来处理命令请求，所以如果由服务器直接调用aof_rewrite函数的话，那么在重写AOF文件期间，服务期将无法处理客户端发来的命令请求。

很明显，作为一种辅佐性的维护手段，Redis不希望AOF重写造成服务器无法处理请求，所以Redis决定将AOF重写程序放到子进程里执行，这样做可以同时达到两个目的：

- 子进程进行AOF重写期间，服务器进程（父进程）可以继续处理命令请求。
- 子进程带有服务器进程的数据副本，使用子进程而不是线程，可以在避免使用锁的情况下，保证数据的安全性。

不过，使用子进程也有一个问题需要解决，因为子进程在进行AOF重写期间，服务器进程还需要继续处理命令请求，而新的命令可能会对现有的数据库状态进行修改，从而使得服务器当前的数据库状态和重写后的AOF文件所保存的数据库状态不一致。为了解决这种数据不一致问题，Redis服务器设置了一个AOF重写缓冲区，这个缓冲区在服务器创建子进程之后开始使用，当Redis服务器执行完一个写命令之后，它会同时将这个写命令发送给AOF缓冲区和AOF重写缓冲区。

![image-20220514174054148](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514174054148.png)

### e. 配置

> \# 开启 aof
> appendonly yes
> \# 开启 aof复写
> auto-aof-rewrite-percentage 100
> auto-aof-rewrite-min-size 64mb
> \# 关闭 混合持久化
> aof-use-rdb-preamble no
> \# 关闭 rdb
> save ""  

> \# 1. redis 会记录上次aof复写时的size，如果之后累计超过了原来的size，则会发生aof复写；
> auto-aof-rewrite-percentage 100
> \# 2. 为了避免策略1中，小数据量时产生多次发生aof复写，策略2在满足策略1的前提下需要超过 64mb
> 才会发生aof复写；
> auto-aof-rewrite-min-size 64mb  

aof复写在 aof 基础上实现了瘦身，但是 aof 复写的数据量仍然很大；加载会非常慢  



## 3. 混合持久化  

从上面知道，rdb 文件小且加载快但丢失多，aof 文件大且加载慢但丢失少；混合持久化是吸取rdb 和 aof 两者优点的一种持久化方案；aof 复写的时候实际持久化的内容是 rdb，等持久化后，持久化期间修改的数据以 aof 的形式附加到文件的尾部；混合持久化实际上是在 aof rewrite 基础上进行优化；所以需要先开启 aof rewrite；  

![image-20220514174326358](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514174326358.png)

### 配置

> \# 开启 aof
> appendonly yes
> \# 开启 aof复写
> auto-aof-rewrite-percentage 100
> auto-aof-rewrite-min-size 64mb
> \# 开启 混合持久化
> aof-use-rdb-preamble yes
> \# 关闭 rdb
> save ""
> \# save 3600 1
> \# save 300 100
> \# save 60 10000  

### 应用

> 1. MySQL 缓存方案中，redis 不开启持久化，redis 只存储热点数据，数据的依据来源于
>    MySQL；若某些数据经常访问需要开启持久化，此时可以选择 rdb 持久化方案，也就是允许
>    丢失一段时间数据；
> 2. 对数据可靠性要求高，在机器性能，内存也安全 (fork 写时复制 最差的情况下 96G)的情况
>    下，可以让 redis 同时开启 aof 和 rdb，注意此时不是混合持久化；redis 重启优先从 aof 加
>    载数据，理论上 aof 包含更多最新数据；如果只开启一种，那么使用混合持久化；
> 3. 在允许丢失的情况下，亦可采用主redis不持久化（96G 90G），从redis进行持久化；
> 4. 伪装从数据  

### 数据安全策略

问题：拷贝持久化文件是否安全？

> 是安全的，持久化 文件一旦被创建， 就不会进行任何修改。 当服务 器要创建一个新的持久化文件
> 时， 它先将文件的内容保存在一个临时文件里面， 当临时文件写入完毕时， 程序才使用
> rename(2) 原子地用临时文件替换原来的持久化文件。
> 数据安全要考虑两个问题：
>
> 1. 节点宕机（redis 是内存数据库，宕机数据会丢失）
> 2. 磁盘故障  (很少发生)

- 创建一个定期任务（cron job）， 每小时将一个 RDB 文件备份到一个文件夹， 并且每天将一个
  RDB 文件备份到另一个文件夹。
- 确保快照的备份都带有相应的日期和时间信息， 每次执行定期任务脚本时， 使用 find 命令来删除
  过期的快照： 比如说， 你可以保留最近 48 小时内的每小时快照， 还可以保留最近一两个月的每
  日快照。
- 至少每天一次， 将 RDB 备份到你的数据中心之外， 或者至少是备份到你运行 Redis 服务器的物理
  机器之外  

## 4. Redis 主从复制

在Redis中，用户可以通过执行SLAVEOF(Redis5.0 以前， 5.0以后使用replicaof  )命令或者设置slaveof选项，让一个服务器去复制（replicate）另一个服务器，我们称呼被复制的服务器为主服务器（master），而对主服务器进行复制的服务器则被称为从服务器（slave）

> \# redis.conf
> replicaof 127.0.0.1 7002
> info replication  

执行上面的命令，服务器将成为127.0.0.1:7002的从服务器， 127.0.0.1:7002将成为主服务器。

如果我们在主服务器上执行以下命令:

> redis > SET msg "hello world"
>
> OK

我们既可以在主服务器获取msg的值，也可以在从服务器上获取。

我们使用PSYNC命令来执行复制时的同步操作: 

PSYNC命令具有完整重同步（full resynchronization）和部分重同步（partialresynchronization）两种模式：

- 其中完整重同步用于处理初次复制情况：完整重同步的执行步骤和SYNC命令的执行步骤基本一样，它们都是通过让主服务器创建并发送RDB文件，以及向从服务器发送保存在缓冲区里面的写命令来进行同步。
- 而部分重同步则用于处理断线后重复制情况：当从服务器在断线后重新连接主服务器时，如果条件允许，主服务器可以将主从服务器连接断开期间执行的写命令发送给从服务器，从服务器只要接收并执行这些写命令，就可以将数据库更新至主服务器当前所处的状态。

PSYNC命令的部分重同步模式解决了旧版复制功能在处理断线后重复制时出现的低效情况. 



- 完整重同步步骤

![image-20220514175406492](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514175406492.png)



- 部分重同步步骤

![image-20220514175418271](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514175418271.png)

#### a. 部分重同步的实现(增量复制)

部分重同步功能由以下三个部分构成:

- 主服务器的复制偏移量（replication offset）和从服务器的复制偏移量
- 主服务器的复制积压缓冲区（replication backlog）(环形缓冲区)
- 服务器的运行ID（run ID）

**复制偏移量**

执行复制的双方——主服务器和从服务器会分别维护一个复制偏移量

- 主服务器每次向从服务器传播N个字节的数据时，就将自己的复制偏移量的值加上N。
- 从服务器每次收到主服务器传播来的N个字节的数据时，就将自己的复制偏移量的值加上N。

通过对比主从服务器的复制偏移量，程序可以很容易地知道主从服务器是否处于一致状态：

- 如果主从服务器处于一致状态，那么主从服务器两者的偏移量总是相同的。
- 相反，如果主从服务器两者的偏移量并不相同，那么说明主从服务器并未处于一致状态。

**复制积压缓冲区(环形缓冲区)**

复制积压缓冲区是由主服务器维护的一个固定长度（fixed-size）先进先出（FIFO）队列，默认大小为1MB。

当主服务器进行命令传播时，它不仅会将写命令发送给所有从服务器，还会将写命令入队到复制积压缓冲区里面

![image-20220514183212519](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514183212519.png)



因此，主服务器的复制积压缓冲区里面会保存着一部分最近传播的写命令，并且复制积压缓冲区会为队列中的每个字节记录相应的复制偏移量; 当从服务器重新连上主服务器时，从服务器会通过PSYNC命令将自己的复制偏移量offset发送给主服务器，主服务器会根据这个复制偏移量来决定对从服务器执行何种同步操作：

- 如果offset偏移量之后的数据（也即是偏移量offset+1开始的数据）仍然存在于复制积压缓冲区里面，那么主服务器将对从服务器执行部分重同步操作
- 相反，如果offset偏移量之后的数据已经不存在于复制积压缓冲区，那么主服务器将对从服务器执行完整重同步操作

Redis为复制积压缓冲区设置的默认大小为1MB，如果主服务器需要执行大量写命令，又或者主从服务器断线后重连接所需的时间比较长，那么这个大小也许并不合适。如果复制积压缓冲区的大小设置得不恰当，那么PSYNC命令的复制重同步模式就不能正常发挥作用，因此，正确估算和设置复制积压缓冲区的大小非常重要。复制积压缓冲区的最小大小可以根据公式`second*write_size_per_second`来估算：

- 其中second为从服务器断线后重新连接上主服务器所需的平均时间（以秒计算）
- 而write_size_per_second则是主服务器平均每秒产生的写命令数据量（协议格式的写命令的长度总和）。

例如，如果主服务器平均每秒产生1 MB的写数据，而从服务器断线之后平均要5秒才能重新连接上主服务器，那么复制积压缓冲区的大小就不能低于5MB。为了安全起见，可以将复制积压缓冲区的大小设为`2*second*write_size_per_second`，这样可以保证绝大部分断线情况都能用部分重同步来处理。

> \# redis.conf
> repl-backlog-size 1mb
> \# 如果所有从库断开连接 3600 秒后没有从库连接，则释放环形缓冲区
> repl-backlog-ttl 3600  

**服务器运行ID**

除了复制偏移量和复制积压缓冲区之外，实现部分重同步还需要用到服务器运行ID（run ID）：

- 每个Redis服务器，不论主服务器还是从服务，都会有自己的运行ID。
- 运行ID在服务器启动时自动生成，由40个随机的十六进制字符组成，例如53b9b28df8042fdc9ab5e3fcbbbabff1d5dce2b3。

当从服务器对主服务器进行初次复制时，主服务器会将自己的运行ID传送给从服务器，而从服务器则会将这个运行ID保存起来。当从服务器断线并重新连上一个主服务器时，从服务器将向当前连接的主服务器发送之前保存的运行ID：

- 如果从服务器保存的运行ID和当前连接的主服务器的运行ID相同，那么说明从服务器断线之前复制的就是当前连接的这个主服务器，主服务器可以继续尝试执行部分重同步操作。
- 相反地，如果从服务器保存的运行ID和当前连接的主服务器的运行ID并不相同，那么说明从服务器断线之前复制的主服务器并不是当前连接的这个主服务器，主服务器将对从服务器执行完整重同步操作。

**PSYNC命令的实现**

PSYNC命令的调用方法有两种:

- 如果从服务器以前没有复制过任何主服务器，或者之前执行过SLAVEOF no one命令，那么从服务器在开始一次新的复制时将向主服务器发送PSYNC ? -1命令，主动请求主服务器进行**完整重同步**（因为这时不可能执行部分重同步）。
- 相反地，如果从服务器已经复制过某个主服务器，那么从服务器在开始一次新的复制时将向主服务器发送PSYNC ＜runid＞ ＜offset＞命令：其中runid是上一次复制的主服务器的运行ID，而offset则是从服务器当前的复制偏移量，接收到这个命令的主服务器会通过这两个参数来判断应该对从服务器执行哪种同步操作。

根据情况，接收到PSYNC命令的主服务器会向从服务器返回以下三种回复的其中一种

- 如果主服务器返回+FULLRESYNC ＜runid＞ ＜offset＞回复，那么表示主服务器将与从服务器执行完整重同步操作：其中runid是这个主服务器的运行ID，从服务器会将这个ID保存起来，在下一次发送PSYNC命令时使用；而offset则是主服务器当前的复制偏移量，从服务器会将这个值作为自己的初始化偏移量。
- 如果主服务器返回+CONTINUE回复，那么表示主服务器将与从服务器执行部分重同步操作，从服务器只要等着主服务器将自己缺少的那部分数据发送过来就可以了。
- 如果主服务器返回-ERR回复，那么表示主服务器的版本低于Redis 2.8，它识别不了PSYNC命令，从服务器将向主服务器发送SYNC命令，并与主服务器执行完整同步操作。

![image-20220514221734798](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514221734798.png)



## 5. Redis 哨兵模式  

哨兵模式是Redis可用性的解决方案；它由一个或多个 sentinel 实例构成 sentinel 系统；该系统可以监视任意多个主库以及这些主库所属的从库；当主库处于下线状态，自动将该主库所属的某个从库升级为新的主库；客户端来连接集群时，会首先连接 sentinel，通过 sentinel 来查询主节点的地址，然后再连接主节点进行数据交互。当主节点发生故障时，客户端会重新向 sentinel 索要主库地址，sentinel 会将最新的主库地址告诉客户端。通过这样客户端无须重启即可自动完成节点切换。哨兵模式当中涉及多个选举流程采用的时 Raft 算法的领头选举方法的实现；  

**配置:**

> \# sentinel.cnf
> \# sentinel 只需指定检测主节点就行了，通过主节点自动发现从节点
> sentinel monitor mymaster 127.0.0.1 6379 2
> \# 判断主观下线时长
> sentinel down-after-milliseconds mymaster 30000
> \# 指定可以有多少个Redis服务同步新的主机，一般而言，这个数字越小同步时间越长，而越大，则对网
> 络资源要求越高
> sentinel parallel-syncs mymaster 1
> \# 指定故障切换允许的毫秒数，超过这个时间，就认为故障切换失败，默认为3分钟
> sentinel failover-timeout mymaster 180000  

**检测异常**  

**主观下线:**

> sentinel 会以每秒一次的频率向所有节点（其他sentinel、主节点、以及从节点）发送 ping 消
> 息，然后通过接收返回判断该节点是否下线；如果在配置指定 down-after-milliseconds 时间内
> 则被判断为主观下线；  

**客观下线:**

> 当一个 sentinel 节点将一个主节点判断为主观下线之后，为了确认这个主节点是否真的下线，它
> 会向其他sentinel 节点进行询问，如果收到一定数量的已下线回复，sentinel 会将主节点判定为客
> 观下线，并通过领头 sentinel 节点对主节点执行故障转移；  

**故障转移:**

> 主节点被判定为客观下线后，开始领头 sentinel 选举，需要一半以上的 sentinel 支持，选举领头sentinel后，开始执行对主节点故障转移；
>
> 从从节点中选举一个从节点作为新的主节点
>
> 通知其他从节点复制连接新的主节点
>
> 若故障主节点重新连接，将作为新的主节点的从节点  

**缺点:**

> redis 采用异步复制的方式，意味着当主节点挂掉时，从节点可能没有收到全部的同步消息，这部分未同步的消息将丢失。如果主从延迟特别大，那么丢失可能会特别多。sentinel 无法保证消息完全不丢失，但是可以通过配置来尽量保证少丢失。  
>
> 同时，它的致命缺点是不能进行横向扩展；  

> \# 主库必须有一个从节点在进行正常复制，否则主库就停止对外写服务，此时丧失了可用性
>
> min-slaves-to-write 1
> \# 这个参数用来定义什么是正常复制，该参数表示如果在10s内没有收到从库反馈，就意味着从库
> 同步不正常；
> min-slaves-max-lag 10  

## 6. Redis cluster集群  

Redis集群是Redis提供的分布式数据库方案，集群通过分片（sharding）来进行数据共享，并提供复制和故障转移功能。

### a. 节点

一个Redis集群通常由多个节点（node）组成，在刚开始的时候，每个节点都是相互独立的，它们都处于一个只包含自己的集群当中，要组建一个真正可工作的集群，我们必须将各个独立的节点连接起来，构成一个包含多个节点的集群。

连接各个节点的工作可以使用CLUSTER MEET命令来完成，该命令的格式如下

> CLUSTER MEET <ip> <port>

向一个节点node发送CLUSTER MEET命令，可以让node节点与ip和port所指定的节点进行握手（handshake），当握手成功时，node节点就会将ip和port所指定的节点添加到node节点当前所在的集群中。

**启动节点**

一个节点就是一个运行在集群模式下的Redis服务器，Redis服务器在启动时会根据cluster-enabled配置选项是否为yes来决定是否开启服务器的集群模式，如图所示。

![image-20220514225213157](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514225213157.png)

节点（运行在集群模式下的Redis服务器）会继续使用所有在单机模式中使用的服务器组件，比如说：

- 节点会继续使用文件事件处理器来处理命令请求和返回命令回复
- 节点会继续使用时间事件处理器来执行serverCron函数，而serverCron函数又会调用集群模式特有的clusterCron函数。clusterCron函数负责执行在集群模式下需要执行的常规操作，例如向集群中的其他节点发送Gossip消息，检查节点是否断线，或者检查是否需要对下线节点进行自动故障转移等
- 节点会继续使用数据库来保存键值对数据，键值对依然会是各种不同类型的对象
- 节点会继续使用RDB持久化模块和AOF持久化模块来执行持久化工作。
- 节点会继续使用发布与订阅模块来执行PUBLISH、SUBSCRIBE等命令
- 节点会继续使用复制模块来进行节点的复制工作。
- 节点会继续使用Lua脚本环境来执行客户端输入的Lua脚本

除此之外，节点会继续使用redisServer结构来保存服务器的状态，使用redisClient结构来保存客户端的状态，至于那些只有在集群模式下才会用到的数据，节点将它们保存到了cluster.h/clusterNode结构、cluster.h/clusterLink结构，以及cluster.h/clusterState结构里面。

**集群数据结构**

clusterNode结构保存了一个节点的当前状态，比如节点的创建时间、节点的名字、节点当前的配置纪元、节点的IP地址和端口号等等。每个节点都会使用一个clusterNode结构来记录自己的状态，并为集群中的所有其他节点（包括主节点和从节点）都创建一个相应的clusterNode结构，以此来记录其他节点的状态：

```c++
typedef struct clusterNode {
    mstime_t ctime; /* Node object creation time. */
    char name[CLUSTER_NAMELEN]; /* Node name, hex string, sha1-size */
    int flags;      /* CLUSTER_NODE_... */
    uint64_t configEpoch; /* Last configEpoch observed for this node 用于实现故障转移 */
    unsigned char slots[CLUSTER_SLOTS/8]; /* slots handled by this node */
    sds slots_info; /* Slots info represented by string. */
    int numslots;   /* Number of slots handled by this node */
    int numslaves;  /* Number of slave nodes, if this is a master */
    struct clusterNode **slaves; /* pointers to slave nodes */
    struct clusterNode *slaveof; /* pointer to the master node. Note that it
                                    may be NULL even if the node is a slave
                                    if we don't have the master node in our
                                    tables. */
    mstime_t ping_sent;      /* Unix time we sent latest ping */
    mstime_t pong_received;  /* Unix time we received the pong */
    mstime_t data_received;  /* Unix time we received any data */
    mstime_t fail_time;      /* Unix time when FAIL flag was set */
    mstime_t voted_time;     /* Last time we voted for a slave of this master */
    mstime_t repl_offset_time;  /* Unix time we received offset for this node */
    mstime_t orphaned_time;     /* Starting time of orphaned master condition */
    long long repl_offset;      /* Last known repl offset for this node. */
    char ip[NET_IP_STR_LEN];  /* Latest known IP address of this node */
    int port;                   /* Latest known clients port (TLS or plain). */
    int pport;                  /* Latest known clients plaintext port. Only used
                                   if the main clients port is for TLS. */
    int cport;                  /* Latest known cluster port of this node. */
    clusterLink *link;          /* TCP/IP link with this node */
    list *fail_reports;         /* List of nodes signaling this as failing */
} clusterNode;

```

clusterNode结构的link属性是一个clusterLink结构，该结构保存了连接节点所需的有关信息，比如套接字描述符，输入缓冲区和输出缓冲区：

```c++
typedef struct clusterLink {
    mstime_t ctime;             /* Link creation time */
    connection *conn;           /* Connection to remote node */
    sds sndbuf;                 /* Packet send buffer */
    char *rcvbuf;               /* Packet reception buffer */
    size_t rcvbuf_len;          /* Used size of rcvbuf */
    size_t rcvbuf_alloc;        /* Allocated size of rcvbuf */
    struct clusterNode *node;   /* Node related to this link if any, or NULL */
} clusterLink;
```

redisClient结构和clusterLink结构都有自己的套接字描述符和输入、输出缓冲区，这两个结构的区别在于，redisClient结构中的套接字和缓冲区是用于连接客户端的，而clusterLink结构中的套接字和缓冲区则是用于连接节点的。 

最后，每个节点都保存着一个clusterState结构，这个结构记录了在当前节点的视角下，集群目前所处的状态，例如集群是在线还是下线，集群包含多少个节点，集群当前的配置纪元，诸如此类：

```c
typedef struct clusterState {
    clusterNode *myself;  /* This node */
    uint64_t currentEpoch;
    int state;            /* CLUSTER_OK, CLUSTER_FAIL, ... */
    int size;             /* Num of master nodes with at least one slot */
    dict *nodes;          /* Hash table of name -> clusterNode structures */
    dict *nodes_black_list; /* Nodes we don't re-add for a few seconds. */
    clusterNode *migrating_slots_to[CLUSTER_SLOTS];
    clusterNode *importing_slots_from[CLUSTER_SLOTS];
    clusterNode *slots[CLUSTER_SLOTS];
    uint64_t slots_keys_count[CLUSTER_SLOTS];
    rax *slots_to_keys;
    ...
}clusterState
```

**CLUSTER MEET命令的实现**

通过向节点A发送CLUSTER MEET命令，客户端可以让接收命令的节点A将另一个节点B添加到节点A当前所在的集群里面：

> CLUSTER MEET <ip> <port>

收到命令的节点A将与节点B进行握手（handshake），以此来确认彼此的存在，并为将来的进一步通信打好基础：

- 节点A会为节点B创建一个clusterNode结构，并将该结构添加到自己的clusterState.nodes字典里面。
- 之后，节点A将根据CLUSTER MEET命令给定的IP地址和端口号，向节点B发送一条MEET消息（message）。
- 如果一切顺利，节点B将接收到节点A发送的MEET消息，节点B会为节点A创建一个clusterNode结构，并将该结构添加到自己的clusterState.nodes字典里面
- 之后，节点B将向节点A返回一条PONG消息。
- 如果一切顺利，节点A将接收到节点B返回的PONG消息，通过这条PONG消息节点A可以知道节点B已经成功地接收到了自己发送的MEET消息。
- 之后，节点A将向节点B返回一条PING消息。
- 如果一切顺利，节点B将接收到节点A返回的PING消息，通过这条PING消息节点B可以知道节点A已经成功地接收到了自己返回的PONG消息，握手完成。



![image-20220514230046421](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514230046421.png)

之后，节点A会将节点B的信息通过Gossip协议传播给集群中的其他节点，让其他节点也与节点B进行握手，最终，经过一段时间之后，节点B会被集群中的所有节点认识.

### b. 槽指派

Redis集群通过分片的方式来保存数据库中的键值对：集群的整个数据库被分为16384($2^{14}$)个槽（slot），数据库中的每个键都属于这16384个槽的其中一个，集群中的每个节点可以处理0个或最多16384个槽。

当数据库中的16384个槽都有节点在处理时，集群处于上线状态（ok）；相反地，如果数据库中有任何一个槽没有得到处理，那么集群处于下线状态（fail）。

通过向节点发送CLUSTER ADDSLOTS命令，我们可以将一个或多个槽指派（assign）给节点负责：

> CLUSTER ADDSLOTS  <slot> [slot.....]

举个例子，执行以下命令可以将槽0至槽5000指派给节点一个节点负责：

> CLUSTER ADDSLOTS   0 1 2 3 4 ......5000

当数据库中的16384个槽都已经被指派给了相应的节点，集群进入上线状态：

**记录节点的槽指派信息**

clusterNode结构的slots属性和numslot属性记录了节点负责处理哪些槽：

```c
typedef struct clusterNode {
	unsigned char slots[CLUSTER_SLOTS/8]; /* slots handled by this node */
    int numslots;   /* Number of slots handled by this node */
}clusterNode;
```

slots属性是一个二进制位数组（bit array），这个数组的长度为16384/8=2048个字节，共包含16384个二进制位。Redis以0为起始索引，16383为终止索引，对slots数组中的16384个二进制位进行编号，并根据索引i上的二进制位的值来判断节点是否负责处理槽i:

- 如果slots数组在索引i上的二进制位的值为1，那么表示节点负责处理槽i。
- 如果slots数组在索引i上的二进制位的值为0，那么表示节点不负责处理槽i

因为取出和设置slots数组中的任意一个二进制位的值的复杂度仅为O（1），所以对于一个给定节点的slots数组来说，程序检查节点是否负责处理某个槽，又或者将某个槽指派给节点负责，这两个动作的复杂度都是O（1）。至于numslots属性则记录节点负责处理的槽的数量，也即是slots数组中值为1的二进制位的数量。

**传播节点的槽指派信息**

一个节点除了会将自己负责处理的槽记录在clusterNode结构的slots属性和numslots属性之外，它还会将自己的slots数组通过消息发送给集群中的其他节点，以此来告知其他节点自己目前负责处理哪些槽。

当节点A通过消息从节点B那里接收到节点B的slots数组时，节点A会在自己的clusterState.nodes字典中查找节点B对应的clusterNode结构，并对结构中的slots数组进行保存或者更新。因为集群中的每个节点都会将自己的slots数组通过消息发送给集群中的其他节点，并且每个接收到slots数组的节点都会将数组保存到相应节点的clusterNode结构里面，因此，集群中的每个节点都会知道数据库中的16384个槽分别被指派给了集群中的哪些节点。

**记录集群所有槽的指派信息**

clusterState结构中的slots数组记录了集群中所有16384个槽的指派信息：

```c
typedef struct clusterState {
	clusterNode *slots[16384];
} clusterState;
```

slots数组包含16384个项，每个数组项都是一个指向clusterNode结构的指针：

- 如果slots[i]指针指向NULL，那么表示槽i尚未指派给任何节点
- 如果slots[i]指针指向一个clusterNode结构，那么表示槽i已经指派给了clusterNode结构所代表的节点。

如果只将槽指派信息保存在各个节点的clusterNode.slots数组里，会出现一些无法高效地解决的问题，而clusterState.slots数组的存在解决了这些问题：

- 如果节点只使用clusterNode.slots数组来记录槽的指派信息，那么为了知道槽i是否已经被指派，或者槽i被指派给了哪个节点，程序需要遍历clusterState.nodes字典中的所有clusterNode结构，检查这些结构的slots数组，直到找到负责处理槽i的节点为止，这个过程的复杂度为O（N），其中N为clusterState.nodes字典保存的clusterNode结构的数量。![image-20220514231001814](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514231001814.png)

- 而通过将所有槽的指派信息保存在clusterState.slots数组里面，程序要检查槽i是否已经被指派，又或者取得负责处理槽i的节点，只需要访问clusterState.slots[i]的值即可，这个操作的复杂度仅为O（1）。

举个例子，对于图所示的slots数组来说，如果程序需要知道槽10002被指派给了哪个节点，那么只要访问数组项slots[10002]，就可以马上知道槽10002被指派给了节点7002。

![image-20220514231112012](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514231112012.png)

要说明的一点是，虽然clusterState.slots数组记录了集群中所有槽的指派信息，但使用clusterNode结构的slots数组来记录单个节点的槽指派信息仍然是有必要的：

- 因为当程序需要将某个节点的槽指派信息通过消息发送给其他节点时，程序只需要将相应节点的clusterNode.slots数组整个发送出去就可以了。
- 另一方面，如果Redis不使用clusterNode.slots数组，而单独使用clusterState.slots数组的话，那么每次要将节点A的槽指派信息传播给其他节点时，程序必须先遍历整个clusterState.slots数组，记录节点A负责处理哪些槽，然后才能发送节点A的槽指派信息，这比直接发送clusterNode.slots数组要麻烦和低效得多

clusterState.slots数组记录了集群中所有槽的指派信息，而clusterNode.slots数组只记录了clusterNode结构所代表的节点的槽指派信息，这是两个slots数组的关键区别所在。 

**CLUSTER ADDSLOTS命令的实现**

CLUSTER ADDSLOTS命令接受一个或多个槽作为参数，并将所有输入的槽指派给接收该命令的节点负责：

> CLUSTER ADDSLOTS  <slot>  [slot.....]

图展示了一个节点的clusterState结构，clusterState.slots数组中的所有指针都指向NULL，并且clusterNode.slots数组中的所有二进制位的值都是0，这说明当前节点没有被指派任何槽，并且集群中的所有槽都是未指派的。

![image-20220514231404692](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514231404692.png)

当客户端对图所示的节点执行命令:

> CLUSTER ADDSLOTS  1 2

将槽1和槽2指派给节点之后，节点的clusterState结构将被更新成图所示的样子：

- clusterState.slots数组在索引1和索引2上的指针指向了代表当前节点的clusterNode结构。
- 并且clusterNode.slots数组在索引1和索引2上的位被设置成了1。

![image-20220514231523761](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514231523761.png)

最后，在CLUSTER ADDSLOTS命令执行完毕之后，节点会通过发送消息告知集群中的其他节点，自己目前正在负责处理哪些槽。

### c. 在集群中执行命令

在对数据库中的16384个槽都进行了指派之后，集群就会进入上线状态，这时客户端就可以向集群中的节点发送数据命令了。当客户端向节点发送与数据库键有关的命令时，接收命令的节点会计算出命令要处理的数据库键属于哪个槽，并检查这个槽是否指派给了自己：

- 如果键所在的槽正好就指派给了当前节点，那么节点直接执行这个命令。
- 如果键所在的槽并没有指派给当前节点，那么节点会向客户端返回一个MOVED错误，指引客户端转向（redirect）至正确的节点，并再次发送之前想要执行的命令。

![image-20220514232537764](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514232537764.png)

**计算键属于哪个槽**

节点使用以下算法来计算给定键key属于哪个槽：

```python
def slot_number(key):
	return CRC16(key) & 16383
```

其中CRC16（key）语句用于计算键key的CRC-16校验和，而&16383语句则用于计算出一个介于0至16383之间的整数作为键key的槽号。

使用`CLUSTER KEYSLOT＜key＞` 命令可以查看一个给定键属于哪个槽：

**判断槽是否由当前节点负责处理**

当节点计算出键所属的槽i之后，节点就会检查自己在clusterState.slots数组中的项i，判断键所在的槽是否由自己负责：

- 如果clusterState.slots[i]等于clusterState.myself，那么说明槽i由当前节点负责，节点可以执行客户端发送的命令。
- 如果clusterState.slots[i]不等于clusterState.myself，那么说明槽i并非由当前节点负责，节点会根据clusterState.slots[i]指向的clusterNode结构所记录的节点IP和端口号，向客户端返回MOVED错误，指引客户端转向至正在处理槽i的节点。

**MOVED错误**

当节点发现键所在的槽并非由自己负责处理的时候，节点就会向客户端返回一个MOVED错误，指引客户端转向至正在负责槽的节点。MOVED错误的格式为：

> MOVED <slot> <ip>:<port>

其中slot为键所在的槽，而ip和port则是负责处理槽slot的节点的IP地址和端口号。

一个集群客户端通常会与集群中的多个节点创建套接字连接，而所谓的节点转向实际上就是换一个套接字来发送命令。如果客户端尚未与想要转向的节点创建套接字连接，那么客户端会先根据MOVED错误提供的IP地址和端口号来连接节点，然后再进行转向。

集群模式的redis-cli客户端在接收到MOVED错误时，并不会打印出MOVED错误，而是根据MOVED错误自动进行节点转向，并打印出转向信息，所以我们是看不见节点返回的MOVED错误的；但是，如果我们使用单机（stand alone）模式的redis-cli客户端，再次向节点7000发送相同的命令，那么MOVED错误就会被客户端打印出来。这是因为单机模式的redis-cli客户端不清楚MOVED错误的作用，所以它只会直接将MOVED错误直接打印出来，而不会进行自动转向。

**节点数据库的实现**

节点和单机服务器在数据库方面的一个区别是，节点只能使用0号数据库，而单机Redis服务器则没有这一限制。

另外，除了将键值对保存在数据库里面之外，节点还会用clusterState结构中的slots_to_keys跳跃表来保存槽和键之间的关系：

```c
typedef struct clusterState {
	rax *slots_to_keys;
    ...
}clusterState
```

![image-20220514233710991](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514233710991.png)

slots_to_keys跳跃表每个节点的分值（score）都是一个槽号，而每个节点的成员（member）都是一个数据库键：

- 每当节点往数据库中添加一个新的键值对时，节点就会将这个键以及键的槽号关联到slots_to_keys跳跃表。
- 当节点删除数据库中的某个键值对时，节点就会在slots_to_keys跳跃表解除被删除键与槽号的关联。

通过在slots_to_keys跳跃表中记录各个数据库键所属的槽，节点可以很方便地对属于某个或某些槽的所有数据库键进行批量操作，例如命令`CLUSTER GETKEYSINSLOT＜slot＞＜count＞`命令可以返回最多count个属于槽slot的数据库键，而这个命令就是通过遍历slots_to_keys跳跃表来实现的。

![image-20220514233803672](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514233803672.png)



### d. 重新分片

Redis集群的重新分片操作可以将任意数量已经指派给某个节点（源节点）的槽改为指派给另一个节点（目标节点），并且相关槽所属的键值对也会从源节点被移动到目标节点。重新分片操作可以在线（online）进行，在重新分片的过程中，集群不需要下线，并且源节点和目标节点都可以继续处理命令请求。

**重新分片的实现原理**

Redis集群的重新分片操作是由Redis的集群管理软件redis-trib负责执行的，Redis提供了进行重新分片所需的所有命令，而redis-trib则通过向源节点和目标节点发送命令来进行重新分片操作。

redis-trib对集群的单个槽slot进行重新分片的步骤如下：

- redis-trib对目标节点发送CLUSTER SETSLOT＜slot＞IMPORTING＜source_id＞命令，让目标节点准备好从源节点导入（import）属于槽slot的键值对。
- redis-trib对源节点发送CLUSTER SETSLOT＜slot＞MIGRATING＜target_id＞命令，让源节点准备好将属于槽slot的键值对迁移（migrate）至目标节点。
- redis-trib向源节点发送CLUSTER GETKEYSINSLOT＜slot＞＜count＞命令，获得最多count个属于槽slot的键值对的键名（key name）。
- 对于步骤3获得的每个键名，redis-trib都向源节点发送一个MIGRATE＜target_ip＞＜target_port＞＜key_name＞0＜timeout＞命令，将被选中的键原子地从源节点迁移至目标节点。
- 重复执行步骤3和步骤4，直到源节点保存的所有属于槽slot的键值对都被迁移至目标节点为止。
- redis-trib向集群中的任意一个节点发送CLUSTER SETSLOT＜slot＞NODE＜target_id＞命令，将槽slot指派给目标节点，这一指派信息会通过消息发送至整个集群，最终集群中的所有节点都会知道槽slot已经指派给了目标节点。

![image-20220514234000917](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514234000917.png)

如果重新分片涉及多个槽，那么redis-trib将对每个给定的槽分别执行上面给出的步骤。

![image-20220514234021043](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514234021043.png)

### e. ASK错误

在进行重新分片期间，源节点向目标节点迁移一个槽的过程中，可能会出现这样一种情况：属于被迁移槽的一部分键值对保存在源节点里面，而另一部分键值对则保存在目标节点里面。当客户端向源节点发送一个与数据库键有关的命令，并且命令要处理的数据库键恰好就属于正在被迁移的槽时：

- 源节点会先在自己的数据库里面查找指定的键，如果找到的话，就直接执行客户端发送的命令。
- 相反地，如果源节点没能在自己的数据库里面找到指定的键，那么这个键有可能已经被迁移到了目标节点，源节点将向客户端返回一个ASK错误，指引客户端转向正在导入槽的目标节点，并再次发送之前想要执行的命令。

![image-20220514234142749](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514234142749.png)

和接到MOVED错误时的情况类似，集群模式的redis-cli在接到ASK错误时也不会打印错误，而是自动根据错误提供的IP地址和端口进行转向动作。如果想看到节点发送的ASK错误的话，可以使用单机模式的redis-cli客户端：

**CLUSTER SETSLOT IMPORTING命令的实现**

clusterState结构的importing_slots_from数组记录了当前节点正在从其他节点导入的槽：

```c
typedef struct clusterState {
	  clusterNode *importing_slots_from[16384];
} clusterState;
```

如果importing_slots_from[i]的值不为NULL，而是指向一个clusterNode结构，那么表示当前节点正在从clusterNode所代表的节点导入槽i。在对集群进行重新分片的时候，向目标节点发送命令

> CLUSTER SETSLOT <i> IMPORTING <source_id>

可以将目标节点clusterState.importing_slots_from[i]的值设置为source_id所代表节点的clusterNode结构。

**CLUSTER SETSLOT MIGRATING命令的实现**

clusterState结构的migrating_slots_to数组记录了当前节点正在迁移至其他节点的槽：

```c
typedef struct clusterState {
    clusterNode *migrating_slots_to[16384];
} clusterState;
```

如果migrating_slots_to[i]的值不为NULL，而是指向一个clusterNode结构，那么表示当前节点正在将槽i迁移至clusterNode所代表的节点。在对集群进行重新分片的时候，向源节点发送命令：

> CLUSTER SETSLOT <i> MIGRATING<source_id>

可以将源节点clusterState.migrating_slots_to[i]的值设置为target_id所代表节点的clusterNode结构。

**ASK错误**

如果节点收到一个关于键key的命令请求，并且键key所属的槽i正好就指派给了这个节点，那么节点会尝试在自己的数据库里查找键key，如果找到了的话，节点就直接执行客户端发送的命令。与此相反，如果节点没有在自己的数据库里找到键key，那么节点会检查自己的clusterState.migrating_slots_to[i]，看键key所属的槽i是否正在进行迁移，如果槽i的确在进行迁移的话，那么节点会向客户端发送一个ASK错误，引导客户端到正在导入槽i的节点去查找键key。

接到ASK错误的客户端会根据错误提供的IP地址和端口号，转向至正在导入槽的目标节点，然后首先向目标节点发送一个ASKING命令，之后再重新发送原本想要执行的命令。

**ASKING命令**

ASKING命令唯一要做的就是打开发送该命令的客户端的REDIS_ASKING标识, 在一般情况下，如果客户端向节点发送一个关于槽i的命令，而槽i又没有指派给这个节点的话，那么节点将向客户端返回一个MOVED错误；但是，如果节点的clusterState.importing_slots_from[i]显示节点正在导入槽i，并且发送命令的客户端带有REDIS_ASKING标识，那么节点将破例执行这个关于槽i的命令一次，图展示了这个判断过程。

![image-20220514234708654](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220514234708654.png)

当客户端接收到ASK错误并转向至正在导入槽的节点时，客户端会先向节点发送一个ASKING命令，然后才重新发送想要执行的命令，这是因为如果客户端不发送ASKING命令，而直接发送想要执行的命令的话，那么客户端发送的命令将被节点拒绝执行，并返回MOVED错误。



### f. 复制与故障转移

Redis集群中的节点分为主节点（master）和从节点（slave），其中主节点用于处理槽，而从节点则用于复制某个主节点，并在被复制的主节点下线时，代替下线主节点继续处理命令请求

**设置从节点**

> CLUSTER REPLICATE <node_id>

可以让接收命令的节点成为node_id所指定节点的从节点，并开始对主节点进行复制：

- 接收到该命令的节点首先会在自己的clusterState.nodes字典中找到node_id所对应节点的clusterNode结构，并将自己的clusterState.myself.slaveof指针指向这个结构，以此来记录这个节点正在复制的主节点：

```c
typedef struct clusterNode {
//如果这是一个从节点，那么指向主节点
   struct clusterNode *slaveof; 
}clusterNode;
```

- 然后节点会修改自己在clusterState.myself.flags中的属性，关闭原本的REDIS_NODE_MASTER标识，打开REDIS_NODE_SLAVE标识，表示这个节点已经由原来的主节点变成了从节点。
- 最后，节点会调用复制代码，并根据clusterState.myself.slaveof指向的clusterNode结构所保存的IP地址和端口号，对主节点进行复制。因为节点的复制功能和单机Redis服务器的复制功能使用了相同的代码，所以让从节点复制主节点相当于向从节点发送命令SLAVEOF。

一个节点成为从节点，并开始复制某个主节点这一信息会通过消息发送给集群中的其他节点，最终集群中的所有节点都会知道某个从节点正在复制某个主节点。集群中的所有节点都会在代表主节点的clusterNode结构的slaves属性和numslaves属性中记录正在复制这个主节点的从节点名单：

```c
typedef struct clusterNode {
//正在复制这个主节点的从节点数量
   int numslaves;  
   //每个数组项指向一个正在复制这个主节点的从节点的clusterNode结构
   struct clusterNode **slaves; 
}clusterNode;
```

**故障检测**

集群中的每个节点都会定期地向集群中的其他节点发送PING消息，以此来检测对方是否在线，如果接收PING消息的节点没有在规定的时间内，向发送PING消息的节点返回PONG消息，那么发送PING消息的节点就会将接收PING消息的节点标记为疑似下线（probable fail，PFAIL）。

集群中的各个节点会通过互相发送消息的方式来交换集群中各个节点的状态信息，例如某个节点是处于在线状态、疑似下线状态（PFAIL），还是已下线状态（FAIL）。当一个主节点A通过消息得知主节点B认为主节点C进入了疑似下线状态时，主节点A会在自己的clusterState.nodes字典中找到主节点C所对应的clusterNode结构，并将主节点B的下线报告（failure report）添加到clusterNode结构的fail_reports链表里面：

```c
typedef struct clusterNode {
    list *fail_reports;         /* List of nodes signaling this as failing */
}clusterNode;
```

每个下线报告由一个clusterNodeFailReport结构表示：

```c
/* This structure represent elements of node->fail_reports. */
typedef struct clusterNodeFailReport {
    struct clusterNode *node;  /* Node reporting the failure condition. */
    mstime_t time;             /* Time of the last report from this node. */
} clusterNodeFailReport;
```

如果在一个集群里面，半数以上负责处理槽的主节点都将某个主节点x报告为疑似下线，那么这个主节点x将被标记为已下线（FAIL），将主节点x标记为已下线的节点会向集群广播一条关于主节点x的FAIL消息，所有收到这条FAIL消息的节点都会立即将主节点x标记为已下线。

**故障转移**

当一个从节点发现自己正在复制的主节点进入了已下线状态时，从节点将开始对下线主节点进行故障转移，以下是故障转移的执行步骤：

- 复制下线主节点的所有从节点里面，会有一个从节点被选中。
- 被选中的从节点会执行SLAVEOF no one命令，成为新的主节点。
- 新的主节点会撤销所有对已下线主节点的槽指派，并将这些槽全部指派给自己。
- 新的主节点向集群广播一条PONG消息，这条PONG消息可以让集群中的其他节点立即知道这个节点已经由从节点变成了主节点，并且这个主节点已经接管了原本由已下线节点负责处理的槽。
- 新的主节点开始接收和自己负责处理的槽有关的命令请求，故障转移完成。

**选举新的主节点**

新的主节点是通过选举产生的。

- 集群的配置纪元是一个自增计数器，它的初始值为0
- 当集群里的某个节点开始一次故障转移操作时，集群配置纪元的值会被增一。
- 对于每个配置纪元，集群里每个负责处理槽的主节点都有一次投票的机会，而第一个向主节点要求投票的从节点将获得主节点的投票。
- 当从节点发现自己正在复制的主节点进入已下线状态时，从节点会向集群广播一条CLUSTERMSG_TYPE_FAILOVER_AUTH_REQUEST消息，要求所有收到这条消息、并且具有投票权的主节点向这个从节点投票。
- 如果一个主节点具有投票权（它正在负责处理槽），并且这个主节点尚未投票给其他从节点，那么主节点将向要求投票的从节点返回一条CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK消息，表示这个主节点支持从节点成为新的主节点。
- 每个参与选举的从节点都会接收CLUSTERMSG_TYPE_FAILOVER_AUTH_ACK消息，并根据自己收到了多少条这种消息来统计自己获得了多少主节点的支持。
- 如果集群里有N个具有投票权的主节点，那么当一个从节点收集到大于等于N/2+1张支持票时，这个从节点就会当选为新的主节点。
- 因为在每一个配置纪元里面，每个具有投票权的主节点只能投一次票，所以如果有N个主节点进行投票，那么具有大于等于N/2+1张支持票的从节点只会有一个，这确保了新的主节点只会有一个。
- 如果在一个配置纪元里面没有从节点能收集到足够多的支持票，那么集群进入一个新的配置纪元，并再次进行选举，直到选出新的主节点为止。
- 



# 查找与排序-KMP算法栈队列

[(1条消息) 查找与排序-KMP算法_birate_小小人生的博客-CSDN博客](https://blog.csdn.net/u014183456/article/details/111305261)

![在这里插入图片描述](https://img-blog.csdnimg.cn/20201216230927207.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTQxODM0NTY=,size_16,color_FFFFFF,t_70)

## 希尔排序

希尔排序，也称递减增量排序算法，是插入排序的一种更高效的改进版本。但希尔排序是非稳定排序算法。

希尔排序是基于插入排序的以下两点性质而提出改进方法的：

- 插入排序在对几乎已经排好序的数据操作时，效率高，即可以达到线性排序的效率；
- 但插入排序一般来说是低效的，因为插入排序每次只能将数据移动一位；

希尔排序的基本思想是：先将整个待排序的记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录"基本有序"时，再对全体记录进行依次直接插入排序。

```c++
int shell_sort(int *data, int len) {
    int i = 0, j = 0;
    int gap = 0;
    int temp = 0;
    for (gap = len /2; gap >=1 ; gap /= 2) {
        for (int i = gap; i < len; i++) {
            temp = data[i];
            for (j = i - gap; j >=0 && temp < data[j]; j = j - gap) {
                data[j+gap] = data[j];
            }
            data[j+gap] = temp;
        }
    }
    return 0;
}
```



## 归并排序

归并排序，是创建在归并操作上的一种有效的排序算法。算法是采用分治法（Divide and Conquer）的一个非常典型的应用，且各层分治递归可以同时进行。归并排序思路简单，速度仅次于快速排序，为稳定排序算法，一般用于对总体无序，但是各子项相对有序的数列。



```c++
void merge(int *data, int *temp,  int start, int middle, int end) {
    int i = start, j = middle + 1, k = start;
    while (i <= midlle && j <= end) {
        if (data[i] > data[j]) temp[k++] = data[j++];
        else temp[k++] = data[i++];
    }
    
    while (i <= middle) {
        temp[k++] = data[i++];
    }
    while (j <= end) {
        temp[k++] = data[j++];
    }
    for (i = start; i < end; i++) 
        data[i] = temp[i];
}

int merge_sort(int *data, int *temp, int start, int end) {
    int middle;
    if (start < end) {
        middle = start + (end - start)/2;
        merge_sort(data, temp,  start,  middle);
        merge_sort(data, temp,  middle + 1,  end);
        merge(data, temp,  start, middle, end);
    } 
    return 0;
}
```



## 快速排序

该方法的基本思想是：

- 1．先从数列中取出一个数作为基准数。
- 2．分区过程，将比这个数大的数全放到它的右边，小于或等于它的数全放到它的左边。
- 3．再对左右区间重复第二步，直到各区间只有一个数。

```c++
void sort(int * data, int left, int right) {
    int i = left, j = right;
    int temp = data[i];
    while (i < j) {
        while (i < j && temp <= data[j]) j--;
        data[i] = data[j];;
        while (i < j && temp >= data[i]) i++;
        data[j] = data[i];
    }
    data[i] = temp;
    sort(data, left, i - 1);
    sort(data, i + 1, right);
}


int quick_sort(int *data, int len) {
    sort(data, 0, length-1);
}
```

  

## KMP 算法

![image-20230312224213783](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230312224213783.png)



![image-20230312224226451](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230312224226451.png)

```c++
#include <stdio.h>
#include <stdlib.h>
#include <string.h>


void make_next(const char *pattern, int *next) {

	int q, k;
	int m = strlen(pattern);

	next[0] = 0;
	for (q = 1,k = 0;q < m; q ++) { //q 后缀， k 前缀

		while (k > 0 && pattern[q] != pattern[k])
			k = next[k-1];

		if (pattern[q] == pattern[k]) {
			k ++;
		}

		next[q] = k;

	}

	// next[0] = 0;
	// q=1, k=0, pattern[q]:pattern[k] = b:a, next[1] = 0;
	// q=2, k=0, pattern[q]:pattern[k] = c:a, next[2] = 0;
	// q=3, k=0, pattern[q]:pattern[k] = a:a, k++, next[3] = 1;
	// q=4, k=1, pattern[q]:pattern[k] = b:b, k++, next[4] = 2;
	// q=5, k=2, pattern[q]:pattern[k] = c:c, k++, next[5] = 3;
	// q=6, k=3, pattern[q]:pattern[k] = d:a, k=next[k-1] -> k=0; next[6] = 0;

}


int kmp(const char *text, const char *pattern, int *next) {

	int n = strlen(text);
	int m = strlen(pattern);

	make_next(pattern, next);
	
	int i, q;
	for (i = 0, q = 0;i < n;i ++) {

		while (q > 0 && pattern[q] != text[i]) {
			q = next[q-1];
		}

		if (pattern[q] == text[i]) {
			q ++;
		}

		if (q == m) {
			//printf("Pattern occurs with shift: %d\n", (i-m+1));
			break;
		}
	}

	return i-q+1;
}

int main() {

	int i;
	int next[20] = {0};

	char *text = "ababxbababababcdababcabddcadfdsss";
	char *pattern = "abcabd";

	int idx = kmp(text, pattern, next);
	printf("match pattern : %d\n", idx);

	for (i = 0;i < sttern);i ++) {
		printf("%4d", next[i]);
	}
	printf("\n");

	return 0;

}
```



# CMake 

> 教程: [Introduction · Modern CMake (modern-cmake-cn.github.io)](https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/)

我们使用的cmake版本为3.x 以上。 

首先我们熟悉基本的命令:

> cmake -B build                                // 在源码目录用 -B 直接创建 build 目录并生成 build/Makefile
>
> cmake --build build -j4                // 自动调用本地的构建系统在 build 里构建，即：make -C build -j4
>
> sudo cmake --build build --target install    // 调用本地的构建系统执行 install 这个目标，即安装
>
> 结论：从现在开始，如果在命令行操作 cmake，请使用更方便的 -B 和 --build 命令。

**-D** **选项：指定配置变量（又称缓存变量）**

CMake 项目的构建分为两步：

- 第一步是 cmake -B build，称为配置阶段（configure），这时只检测环境并生成构建规则, 会在 build 目录下生成本地构建系统能识别的项目文件（Makefile 或是 .sln）
- 第二步是 cmake --build build，称为构建阶段（build），这时才实际调用编译器来编译代码



在配置阶段可以通过 -D 设置缓存变量。第二次配置时，之前的 -D 添加仍然会被保留。

- cmake -B build **-DCMAKE_INSTALL_PREFIX=/opt/openvdb-8.0**设置安装路径为 /opt/openvdb-8.0（会安装到 /opt/openvdb-8.0/lib/libopenvdb.so. 

设置构建模式为发布模式（开启全部优化）

- cmake -B build **-DCMAKE_BUILD_TYPE=Release**

一般我们使用这个参数比较多。

- cmake -B build  第二次配置时没有 -D 参数，但是之前的 -D 设置的变量都会被保留（此时缓存里仍有你之前定义的 CMAKE_BUILD_TYPE 和 CMAKE_INSTALL_PREFIX）

**-G** **选项：指定要用的生成器**

众所周知，CMake 是一个跨平台的构建系统，可以从 CMakeLists.txt 生成不同类型的构建系统（比如 Linux 的 make，Windows 的 MSBuild），从而让构建规则可以只写一份，跨平台使用。过去的软件（例如 TBB）要跨平台，只好 Makefile 的构建规则写一份，MSBuild 也写一份。现在只需要写一次 CMakeLists.txt，他会视不同的操作系统，生成不同构建系统的规则文件。那个和操作系统绑定的构建系统（make、MSBuild）称为本地构建系统（native buildsystem）。负责从 CMakeLists.txt 生成本地构建系统构建规则文件的，称为生成器（generator）。

Linux 系统上的 CMake 默认用是 Unix Makefiles 生成器；Windows 系统默认是 Visual Studio 2019 生成器；MacOS 系统默认是 Xcode 生成器。可以用 -G 参数改用别的生成器，例如 cmake -GNinja 会生成 Ninja 这个构建系统的构建规则。Ninja 是一个高性能，跨平台的构建系统，Linux、Windows、MacOS 上都可以用。Ninja 可以从包管理器里安装，没有包管理器的 Windows 可以用 Python 的包管理器安装：pip install ninja事实上，MSBuild 是单核心的构建系统，Makefile 虽然多核心但因历史兼容原因效率一般。而 Ninja 则是专为性能优化的构建系统，他和 CMake 结合都是行业标准了。

![image-20220426231757371](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220426231757371.png)



性能上：Ninja > Makefile > MSBuild

Makefile 启动时会把每个文件都检测一遍，浪费很多时间。特别是有很多文件，但是实际需要构建的只有一小部分，从而是 I/O Bound 的时候，Ninja 的速度提升就很明显。

## 1. 最简单的CMake

CMakeLists.txt

```txt
cmake_minimum_required(VERSION 3.15)   //要求最小的camke版本
project(hellocmake LANGUAGES CXX)   // 设置项目的名称 使用c++
set(CMAKE_BUILD_TYPE Release)       // 设置我们的构建类型为release 
add_executable(main main.cpp)       // 生成可执行文件
```

> set(CMAKE_BUILD_TYPE Release)

 CMAKE_BUILD_TYPE 是 CMake 中一个特殊的变量，用于控制构建类型，他的值可以是

- Debug 调试模式，完全不优化，生成调试信息，方便调试程序
- Release 发布模式，优化程度最高，性能最佳，但是编译比 Debug 慢
- MinSizeRel 最小体积发布，生成的文件比 Release 更小，不完全优化，减少二进制体积
- RelWithDebInfo 带调试信息发布，生成的文件比 Release 更大，因为带有调试的符号信息, 默认情况下 CMAKE_BUILD_TYPE 为空字符串，这时相当于 Debug。

在Release模式下，追求的是程序的最佳性能表现，在此情况下，编译器会对程序做最大的代码优化以达到最快运行速度。另一方面，由于代码优化后不与源代码一致，此模式下一般会丢失大量的调试信息。

> Debug: `-O0 -g`
>
> Release: `-O3 -DNDEBUG`
>
> MinSizeRel: `-Os -DNDEBUG`
>
> RelWithDebInfo: `-O2 -g -DNDEBUG`
>
> 此外，注意定义了 NDEBUG 宏会使 assert 被去除掉。

如何让 CMAKE_BUILD_TYPE 在用户没有指定的时候为 Release，指定的时候保持用户指定的值不变呢。就是说 CMake 默认情况下 CMAKE_BUILD_TYPE 是一个空字符串。因此这里通过 if (NOT CMAKE_BUILD_TYPE) 判断是否为空，如果空则自动设为 Release 模式。大多数 CMakeLists.txt 的开头都会有这样三行，为的是让默认的构建类型为发布模式（高度优化）而不是默认的调试模式（不会优化）。

```txt
if (NOT CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release)
endif()
```

## 2. 添加message信息

CMakeLists.txt

```
cmake_minimum_required(VERSION 3.15)
project(hellocmake)

message("PROJECT_NAME: ${PROJECT_NAME}")
message("PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message("PROJECT_BINARY_DIR: ${PROJECT_BINARY_DIR}")
message("CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}")
message("CMAKE_CURRENT_BINARY_DIR: ${CMAKE_CURRENT_BINARY_DIR}")
add_executable(main main.cpp)

add_subdirectory(mylib)

```

main.cpp

```c
#include <cstdio>

int main() {
    printf("Hello, world!\n");
}

```

mylib/CMakeLists.txt

```txt
message("mylib got PROJECT_NAME: ${PROJECT_NAME}")
message("mylib got PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message("mylib got PROJECT_BINARY_DIR: ${PROJECT_BINARY_DIR}")
message("mylib got CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}")
message("mylib got CMAKE_CURRENT_BINARY_DIR: ${CMAKE_CURRENT_BINARY_DIR}")

```

此时我们的项目目录结构为:

```txt
├── CMakeLists.txt
├── main.cpp
└── mylib
    └── CMakeLists.txt
```

**project**：初始化项目信息，并把当前**CMakeLists.txt** **所在位置作为根目录**

**message**: 显示打印信息，这里面有些参数:

- PROJECT_NAME:  我们指定的project里面的name: hellocmake
- CMAKE_PROJECT_NAME：根项目的项目名
- PROJECT_SOURCE_DIR:  当前项目源码路径（存放main.cpp的地方）
- PROJECT_BINARY_DIR:  当前项目输出路径（存放main.exe的地方）
- CMAKE_SOURCE_DIR:  根项目源码路径（存放main.cpp的地方）
- CMAKE_BINARY_DIR：根项目输出路径（存放main.exe的地方）
- CMAKE_CURRENT_SOURCE_DIR:  表示当前源码目录的位置
- CMAKE_CURRENT_BINARY_DIR :表示当前输出目录的位置
- PROJECT_IS_TOP_LEVEL：BOOL类型，表示当前项目是否是（最顶层的）根项目

**message可以打印出状态，错误警告等一系列的信息:**

- **message(STATUS “...”)** **表示信息类型是状态信息，有** **--** **前缀**
- **message(WARNING “...”)** **表示是警告信息**
- **message(AUTHOR_WARNING “...”)** **表示是仅仅给项目作者看的警告信息**,  可以通过 cmake -B build -Wno-dev 关闭
- **message(FATAL_ERROR “...”)** **表示是错误信息，会终止** **CMake** **的运行**
- **message(SEND_ERROR “...”)** **表示是错误信息，但之后的语句仍继续执行**



**PROJECT_x_DIR** **和** **CMAKE_CURRENT_x_DIR** **的区别**

- PROJECT_SOURCE_DIR 表示最近一次调用 project 的 CMakeLists.txt 所在的源码目录。CMAKE_CURRENT_SOURCE_DIR 表示当前 CMakeLists.txt 所在的源码目录。
- CMAKE_SOURCE_DIR 表示最为外层 CMakeLists.txt 的源码根目录。利用 PROJECT_SOURCE_DIR 可以实现从子模块里直接获得项目最外层目录的路径。不建议用 CMAKE_SOURCE_DIR，那样会让你的项目无法被人作为子模块使用。

> 详见: https://cmake.org/cmake/help/latest/command/project.html

**子模块里也可以用** **project** **命令，将当前目录作为一个独立的子项目**

这样一来 PROJECT_SOURCE_DIR 就会是子模块的源码目录而不是外层了。这时候 CMake 会认为这个子模块是个独立的项目，会额外做一些初始化。他的构建目录 PROJECT_BINARY_DIR 也会变成 build/<源码相对路径>。

**project 的初始化：LANGUAGES字段**

> project(项目名 LANGUAGES 使用的语言列表...) 指定了该项目使用了哪些编程语言。
>
> 目前支持的语言包括：
>
> C：C语言
>
> CXX：C++语言
>
> ASM：汇编语言
>
> Fortran：老年人的编程语言
>
> CUDA：英伟达的 CUDA（3.8 版本新增）
>
> OBJC：苹果的 Objective-C（3.16 版本新增）
>
> OBJCXX：苹果的 Objective-C++（3.16 版本新增）
>
> ISPC：一种因特尔的自动 SIMD 编程语言（3.18 版本新增）
>
> 如果不指定 LANGUAGES，默认为 C 和 CXX。

**常见问题：**LANGUAGES** **中没有启用** **C** **语言，但是却用到了** **C** **语言**

例如: CMakeLists.txt里面是:

```txt
cmake_minimum_required(VERSION 3.15)
project(hellocmake LANGUAGES CXX) // 这里启用的是CXX
add_executable(main main.c)   //这里使用的是main.c而不是main.cpp
```

**解决：改成** project(项目名LANGUAGES C CXX) 即可

```
cmake_minimum_required(VERSION 3.15)
project(hellocmake LANGUAGES C CXX) // 这里启用的是CXX
add_executable(main main.c)   //这里使用的是main.c而不是main.cpp
```

**project** **的初始化：VERSION字段**

```txt
cmake_minimum_required(VERSION 3.15)
project(hellocmake VERSION 0.2.3)

message("PROJECT_NAME: ${PROJECT_NAME}")
message("PROJECT_VERSION: ${PROJECT_VERSION}")
message("PROJECT_SOURCE_DIR: ${PROJECT_SOURCE_DIR}")
message("PROJECT_BINARY_DIR: ${PROJECT_BINARY_DIR}")
message("hellocmake_VERSION: ${hellocmake_VERSION}")
message("hellocmake_SOURCE_DIR: ${hellocmake_SOURCE_DIR}")
message("hellocmake_BINARY_DIR: ${hellocmake_BINARY_DIR}")
add_executable(main main.cpp)

```

- project(项目名 VERSION x.y.z) 可以把当前项目的版本号设定为 x.y.z。之后可以通过 PROJECT_VERSION 来获取当前项目的版本号。

- PROJECT_VERSION_MAJOR 获取 x（主版本号）。

- PROJECT_VERSION_MINOR 获取 y（次版本号）。

- PROJECT_VERSION_PATCH 获取 z（补丁版本号）。

![image-20220427001312599](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220427001312599.png)



小技巧: **CMake** **的** **${}** **表达式可以嵌套**

因为` ${PROJECT_NAME} `求值的结果是 hellocmake

所以 `${${PROJECT_NAME}_VERSION}`相当于 `${hellocmake_VERSION}`

进一步求值的结果也就是刚刚指定的 0.2.3 了。



假如我们的工程很大，需要很多个CPP文件，总不能一个一个添加吧，对于这种情况 ，cmake提供了一个能够自动获取当前目录下所有CPP的函数：

> aux_source_directory（目录 存放文件列表的变量）

> add_subdirectory(目录)

将子目录添加到CMake中，会在子目录里面查找和编译

**设置 C++ 标准：CMAKE_CXX_STANDARD变量**

```txt
cmake_minimum_required(VERSION 3.15)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

project(hellocmake LANGUAGES CXX)

add_executable(main main.cpp)

```

- CMAKE_CXX_STANDARD 是一个整数，表示要用的 C++ 标准。比如需要 C++17 那就设为 17，需要 C++23 就设为 23。

- CMAKE_CXX_STANDARD_REQUIRED 是 BOOL 类型，可以为 ON 或 OFF，默认 OFF。他表示是否一定要支持你指定的 C++ 标准：如果为 OFF 则 CMake 检测到编译器不支持 C++17 时不报错，而是默默调低到 C++14 给你用；为 ON 则发现不支持报错，更安全。

## 3. 添加链接库

**a. main.cpp 调用mylib.cpp里面的函数**

CMakeLists.txt

```txt
add_executable(main main.cpp mylib.cpp)
```

main.cpp

```c
#include "mylib.h"

int main() {
    say_hello();
}
```

mylib.cpp

```c
#include "mylib.h"
#include <cstdio>

void say_hello() {
    printf("hello, mylib!\n");
}
```

mylib.h

```c
#pragma once

void say_hello();

```

**b. mylib作为与一个静态库**

修改CMakeLists.txt

```txt
add_library(mylib STATIC mylib.cpp) // static 编译为静态库libmylib.a

add_executable(main main.cpp)

target_link_libraries(main PUBLIC mylib)

```

**c. mylib作为与一个动态库**

修改CMakeLists.txt

```txt
add_library(mylib SHARED mylib.cpp) // 编译成 .so文件

add_executable(main main.cpp)

target_link_libraries(main PUBLIC mylib)

```

**d. mylib作为与一个对象库**

对象库类似于静态库，但不生成 .a 文件，只由 CMake 记住该库生成了哪些对象文件, 对象库是 CMake 自创的，绕开了编译器和操作系统的各种繁琐规则，保证了跨平台统一性。在自己的项目中，我推荐全部用对象库(OBJECT)替代静态库(STATIC)避免跨平台的麻烦。对象库仅仅作为组织代码的方式，而实际生成的可执行文件只有一个，减轻了部署的困难。 

修改CMakeLists.txt

```txt
add_library(mylib OBJECT mylib.cpp)

add_executable(main main.cpp)

target_link_libraries(main PUBLIC mylib)
```

**e. add_library无参数时，是静态库还是动态库?**

会根据 BUILD_SHARED_LIBS 这个变量的值决定是动态库还是静态库。ON 则相当于 SHARED，OFF 则相当于 STATIC。

如果未指定 BUILD_SHARED_LIBS 变量，则默认为 STATIC。因此，如果发现一个项目里的 add_library 都是无参数的，意味着你可以用：**cmake -B build -DBUILD_SHARED_LIBS:BOOL=ON** 来让他全部生成为动态库。

要让 BUILD_SHARED_LIBS 默认为 ON，可以用下图这个方法：如果该变量没有定义，则设为 ON，否则保持用户指定的值不变。这样当用户没有指定 BUILD_SHARED_LIBS 这个变量时，会默认变成 ON。也就是说除非用户指定了 -DBUILD_SHARED_LIBS:BOOL=OFF 才会生成静态库，否则默认是生成动态库。

```c
if (NOT DEFINED BUILD_SHARED_LIBS)
    set(BUILD_SHARED_LIBS ON)
endif()

add_library(mylib mylib.cpp)

add_executable(main main.cpp)

target_link_libraries(main PUBLIC mylib)

```

**f. 想动态库链接静态库怎么处理。**

我们一般使用动态库链接动态库时，可以这样处理:

```
add_library(otherlib STATIC otherlib.cpp)

add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)

```

但是会报错，解决: **让静态库编译时也生成位置无关的代码(PIC)，这样才能装在动态库里**

```
add_library(otherlib STATIC otherlib.cpp)
set_property(TARGET otherlib PROPERTY POSITION_INDEPENDENT_CODE ON) //只针对otherlib库

add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)

```

或者全局设置

```c
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
add_library(otherlib STATIC otherlib.cpp)

add_library(mylib SHARED mylib.cpp)
target_link_libraries(mylib PUBLIC otherlib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)
```

除了**POSITION_INDEPENDENT_CODE** **还有哪些这样的属性**

1. 采用 C++17 标准进行编译（默认 11）

> set_property(TARGET main PROPERTY CXX_STANDARD 17) 

2. 如果编译器不支持 C++17，则直接报错（默认 OFF）

> set_property(TARGET main PROPERTY CXX_STANDARD_REQUIRED ON)

3. 在 Windows 系统中，运行时不启动控制台窗口，只有 GUI 界面（默认 OFF）

> set_property(TARGET main PROPERTY WIN32_EXECUTABLE ON)

4. 告诉编译器不要自动剔除没有引用符号的链接库（默认 OFF）

> set_property(TARGET main PROPERTY LINK_WHAT_YOU_USE ON)

5. 设置动态链接库的输出路径（默认 ${CMAKE_BINARY_DIR}）

> ​	set_property(TARGET main PROPERTY LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)

6. 设置静态链接库的输出路径（默认 ${CMAKE_BINARY_DIR}）

> ​	set_property(TARGET main PROPERTY ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)

7. 设置可执行文件的输出路径（默认 ${CMAKE_BINARY_DIR}）

> set_property(TARGET main PROPERTY RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin)

**另一种方式：set_target_properties批量设置多个属性**

```
set_target_properties(main PROPERTIES
    CXX_STANDARD 17           # 采用 C++17 标准进行编译（默认 11）
    CXX_STANDARD_REQUIRED ON  # 如果编译器不支持 C++17，则直接报错（默认 OFF）
    WIN32_EXECUTABLE ON       # 在 Windows 系统中，运行时不启动控制台窗口，只有 GUI 界面（默认 OFF）
    LINK_WHAT_YOU_USE ON      # 告诉编译器不要自动剔除没有引用符号的链接库（默认 OFF）
    LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib   # 设置动态链接库的输出路径（默认 ${CMAKE_BINARY_DIR}）
    ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib   # 设置静态链接库的输出路径（默认 ${CMAKE_BINARY_DIR}）
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin   # 设置可执行文件的输出路径（默认 ${CMAKE_BINARY_DIR}）
    )

```

**另一种方式：通过全局的变量，让之后创建的所有对象都享有同样的属性**

> set(CMAKE_CXX_STANDARD 17)
>
> set(CMAKE_CXX_STANDARD_REQUIRED ON)
>
> set(CMAKE_WIN32_EXECUTABLE ON) 
>
> set(CMAKE_LINK_WHAT_YOU_USE ON)  
>
> set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
>
> set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
>
> set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/bin) 



再看一个链接动态库的例子:

现在我们的目录结构是这样的:

> .
> ├── CMakeLists.txt
> ├── main.cpp
> └── mylib
>     ├── CMakeLists.txt
>     ├── mylib.cpp
>     └── mylib.h

最顶层的CMakeLists.txt需要链接底层的mylb生成的动态库，我们要怎么做:

顶层CMakeLists.txt

```c
cmake_minimum_required(VERSION 3.15)

add_subdirectory(mylib)

add_executable(main main.cpp)
target_link_libraries(main PUBLIC mylib)
```

底层CMakeLists.txt

```c
add_library(mylib SHARED mylib.cpp mylib.h)
```

## 4. 链接第三方库

> 用 CMake 的 find_package 命令。
>
> find_package(TBB REQUIRED) 会查找 /usr/lib/cmake/*.cmake 这个配置文件，并根据里面的配置信息创建 伪对象，之后通过 target_link_libraries 链接 对象就可以正常工作了。

例如我们链接tbb对象:

CMakeLists.txt

> add_executable(main main.cpp)
>
> find_package(TBB REQUIRED)
> target_link_libraries(main PUBLIC TBB::tbb)

有些包有多个组件，需要指定:

find_package 生成的伪对象(imported target)都按照“包名::组件名”的格式命名。你可以在 find_package 中通过 COMPONENTS 选项，后面跟随一个列表表示需要用的组件。

![image-20220428231622887](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220428231622887.png)

![image-20220428231601451](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220428231601451.png)



## 5. 变量与缓存

**重复执行** **cmake** **-B build** **会有什么区别？**

可以看到第二次的输出少了很多，这是因为 CMake 第一遍需要检测编译器和 C++ 特性等比较耗时，检测完会把结果存储到**缓存**中，这样第二遍运行cmake -B build 时就可以直接用缓存的值，就不需要再检测一遍了。

![image-20220429000559438](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220429000559438.png)

**如何清除缓存？**

最简单粗暴的方法是`rm -rf build/ ` 删除build 目录，其实我们的缓存都存在**CMakeCache.txt** 这个文件里面，所以我们只需要删除这个文件即可。

**设置缓存变量**

变量缓存的意义在于能够把 find_package 找到的库文件位置等信息，储存起来。这样下次执行 find_package 时，就会利用上次缓存的变量，直接返回。避免重复执行 cmake -B 时速度变慢的问题。

语法: 

> set(变量名 “变量值” CACHE 变量类型 “注释”)

![image-20220429000927162](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220429000927162.png)

**缓存的** **myvar** **会出现在** **build/CMakeCache.txt** **里**

但是我们经常会遇到一个问题: **我修改了** **CMakeLists.txt** **里** **set** **的值，却没有更新？**为了更新缓存变量，有的同学偷懒直接修改 CMakeLists.txt 里的值，这是没用的。因为 set(... CACHE ...) 在缓存变量已经存在时，不会更新缓存的值！CMakeLists.txt 里 set 的被认为是“默认值”因此不会在第二次 set 的时候更新。

因此我们必须通过命令行-D参数， 例如： cmake -B build -Dmyvar=world

**缓存变量除了** **STRING** **还有哪些类型？**

> STRING 字符串，例如 “hello, world”
>
> FILEPATH 文件路径，例如 “C:/vcpkg/scripts/buildsystems/vcpkg.cmake”
>
> PATH 目录路径，例如 “C:/Qt/Qt5.14.2/msvc2019_64/lib/cmake/”
>
> BOOL 布尔值，只有两个取值：ON 或 OFF。
>
> 注意：TRUE 和 ON 等价，FALSE 和 OFF 等价；YES 和 ON 等价，NO 和 OFF 等价

**案例：添加一个** **BOOL** **类型的缓存变量，用于控制要不要启用某特性**

```c
cmake_minimum_required(VERSION 3.15)
project(hellocmake LANGUAGES CXX)

add_executable(main main.cpp)

set(WITH_TBB ON CACHE BOOL "set to ON to enable TBB, OFF to disable TBB.")
if (WITH_TBB)
    target_compile_definitions(main PUBLIC WITH_TBB)
    find_package(TBB REQUIRED)
    target_link_libraries(main PUBLIC TBB::tbb)
endif()

```

**CMake** **对** **BOOL** **类型缓存的** **set**  指令提供了一个简写：option

option(变量名 “描述” 变量值)

等价于：

set(变量名 CACHE BOOL 变量值 “描述”)

```
cmake_minimum_required(VERSION 3.15)
project(hellocmake LANGUAGES CXX)

add_executable(main main.cpp)

option(WITH_TBB "set to ON to enable TBB, OFF to disable TBB." ON)
if (WITH_TBB)
    target_compile_definitions(main PUBLIC WITH_TBB)
    find_package(TBB REQUIRED)
    target_link_libraries(main PUBLIC TBB::tbb)
endif()

```

我们可以在.cpp文件中使用WITH_TBB变量:

```c
#include <cstdio>

int main() {
#ifdef WITH_TBB
    printf("TBB enabled!\n");
#endif
    printf("Hello, world!\n");
}
```

## 6. 跨平台和编译器

**在 CMake 中给 .cpp 定义一个宏MY_MACRO**

CMakeList.txt

```txt
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})
target_compile_definitions(main PUBLIC MY_MACRO=233)  
```

main.cpp

```cpp
#include <cstdio>

int main() {
#ifdef MY_MACRO
    printf("MY_MACRO defined! value: %d\n", MY_MACRO);
#else
    printf("MY_MACRO not defined!\n");
#endif
}
```

**根据不同的操作系统，把宏定义成不同的值**

```
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})

if (CMAKE_SYSTEM_NAME MATCHES "Windows")
    target_compile_definitions(main PUBLIC MY_NAME="Bill Gates")
elseif (CMAKE_SYSTEM_NAME MATCHES "Linux")
    target_compile_definitions(main PUBLIC MY_NAME="Linus Torvalds")
elseif (CMAKE_SYSTEM_NAME MATCHES "Darwin")
    target_compile_definitions(main PUBLIC MY_NAME="Steve Jobs")
endif()

```

**CMake还提供了一些简写变量：WIN32, APPLE, UNIX, ANDROID, IOS等**

```txt
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})

if (WIN32)
    target_compile_definitions(main PUBLIC MY_NAME="Bill Gates")
elseif (UNIX AND NOT APPLE)
    target_compile_definitions(main PUBLIC MY_NAME="Linus Torvalds")
elseif (APPLE)
    target_compile_definitions(main PUBLIC MY_NAME="Steve Jobs")
endif()
```

**使用生成器表达式，简化成一条指令**

语法：`$<$<类型:值>:为真时的表达式>`

比如 `$<$<PLATFORM_ID:Windows>:MY_NAME=”Bill Gates”>`

在 Windows 平台上会变为 MY_NAME=”Bill Gates”

其他平台上则表现为空字符串

```
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})

target_compile_definitions(main PUBLIC
    $<$<PLATFORM_ID:Windows>:MY_NAME="Bill Gates">
    $<$<PLATFORM_ID:Linux>:MY_NAME="Linus Torvalds">
    $<$<PLATFORM_ID:Darwin>:MY_NAME="Steve Jobs">
    )
```

**生成器表达式：如需多个平台可以用逗号分割**

```txt
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})

target_compile_definitions(main PUBLIC
    $<$<PLATFORM_ID:Windows>:MY_NAME="DOS-like">
    $<$<PLATFORM_ID:Linux,Darwin,FreeBSD>:MY_NAME="Unix-like">
    )
```

**判断当前用的是哪一款** **C++** **编译器**

```txt
add_executable(main)
file(GLOB sources CONFIGURE_DEPENDS *.cpp *.h)
target_sources(main PUBLIC ${sources})

if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_definitions(main PUBLIC MY_NAME="gcc")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "NVIDIA")
    target_compile_definitions(main PUBLIC MY_NAME="nvcc")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
    target_compile_definitions(main PUBLIC MY_NAME="clang")
elseif (CMAKE_CXX_COMPILER_ID MATCHES "MSVC")
    target_compile_definitions(main PUBLIC MY_NAME="msvc")
endif()
```

## 7. 分支和判断

通常来说 BOOL 类型的变量只有 ON/OFF 两种取值。但是由于历史原因，TRUE/FALSE 和 YES/NO 也可以表示 BOOL 类型。

**if** **的特点：不需要加** **${}，会自动尝试作为变量名求值**

由于历史原因，if 的括号中有着特殊的语法，如果是一个字符串，比如 MYVAR，则他会先看是否有 ${MYVAR} 这个变量。如果有这个变量则会被替换为变量的值来进行接下来的比较，否则保持原来字符串不变。

```txt
cmake_minimum_required(VERSION 3.15)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)

project(hellocmake LANGUAGES C CXX)


set(MYVAR OFF)
if (MYVAR)
    message("MYVAR is true")
else()
    message("MYVAR is false")
endif()

add_executable(main main.cpp)
```

## 8. 变量和作用域

**变量的传播规则：父会传给子**， **子不传给父**



## 9. 一些经验

- 最少的三行

  > - cmake_minimum_required(VERSION 3.15)
  > - project(hellocmake)
  > - add_executable(main main.cpp)

- 添加c++11的标准

  > - set(CMAKE_CXX_STANDARD 11)
  > - set(CMAKE_CXX_STANDAND_REQUIRED True)

- 添加版本信息和编译的头文件

  > - project(Tutorial VERSION 1.0)
  > - configure_file(TutorialConfig.h.in TutorialConfig.h)
  > - target_include_directories(Tutorial PUBLIC                 // 找TutorialConfig.h 文件
  >                              "${PROJECT_BINARY_DIR}"
  >                              )
  >
  > 在头文件中添加版本信息
  >
  > - ```c
  >   // the configured options and settings for Tutorial
  >   #define Tutorial_VERSION_MAJOR @Tutorial_VERSION_MAJOR@
  >   #define Tutorial_VERSION_MINOR @Tutorial_VERSION_MINOR@
  >   ```
  >
  >   
  >
  > 在.cpp文件包含这个头文件并打印
  >
  > - ```c
  >     if (argc < 2) {
  >       // report version
  >       std::cout << argv[0] << " Version " << Tutorial_VERSION_MAJOR << "."
  >                 << Tutorial_VERSION_MINOR << std::endl;
  >       std::cout << "Usage: " << argv[0] << " number" << std::endl;
  >       return 1;
  >     }
  >   ```

- 目录格式

  > - `CMakeLists.txt`
  > - `tutorial.cxx`
  > - `MathFunctions/CMakeLists.txt`

  在`MathFunctions/CMakeLists.txt` 添加add_library(MathFunctions mysqrt.cxx)

  在top level CMakeLists.txt 里面包含

  > ```c
  > add_subdirectory(MathFunctions)
  > ```

​	  链接可执行目标使用

> ```
> add_subdirectory(MathFunctions)
> target_link_libraries(Tutorial PUBLIC MathFunctions)
> ```

​      包含头文件

> ```cmake
> add_subdirectory(MathFunctions)
> target_link_libraries(Tutorial PUBLIC MathFunctions)
> target_include_directories(Tutorial PUBLIC
>                           "${PROJECT_BINARY_DIR}"
>                           "${PROJECT_SOURCE_DIR}/MathFunctions"
>                           )
> ```
>
> 

​      这样在tutorial.cxx 可以包含

```c
#include "MathFunctions.h"
```

- 使用option() 命令给用户一个选择

​	在top-level CMakeLists.txt

> ```cmake
> option(USE_MYMATH "Use tutorial provided math implementation" ON)
> ```

​     使用if 来判断

> ```cmake
> if(USE_MYMATH)
>   add_subdirectory(MathFunctions)
>   list(APPEND EXTRA_LIBS MathFunctions)  //MathFunctions 叫EXTRA_LIBS
>   list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
> endif()
> ```

使用它

> ```cmake
> target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})
> target_include_directories(Tutorial PUBLIC
>                            "${PROJECT_BINARY_DIR}"
>                            ${EXTRA_INCLUDES}
>                            )
> ```

在头文件里面

> ```c
> #ifdef USE_MYMATH
> #include "MathFunctions.h"
> #endif
> ```

 编译的时候也可以关闭：

```
cmake ../Step2 -DUSE_MYMATH=OFF
```



目录结构：

> - `MathFunctions/CMakeLists.txt`
> - `CMakeLists.txt`

添加：

> ```cmake
> #MathFunctions/CMakeLists.txt
> target_include_directories(MathFunctions
>           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR}
>           )
> ```

> #
>
> ```cmake
> #CMakeLists.txt
> if(USE_MYMATH)
>   add_subdirectory(MathFunctions)
>   list(APPEND EXTRA_LIBS MathFunctions)
> endif()
> target_include_directories(Tutorial PUBLIC
>                            "${PROJECT_BINARY_DIR}"
>                            )
> ```

When run in [`cmake -P`](https://cmake.org/cmake/help/v3.25/manual/cmake.1.html#cmdoption-cmake-P) script mode, CMake sets the variables [`CMAKE_BINARY_DIR`](https://cmake.org/cmake/help/v3.25/variable/CMAKE_BINARY_DIR.html#variable:CMAKE_BINARY_DIR), [`CMAKE_SOURCE_DIR`](https://cmake.org/cmake/help/v3.25/variable/CMAKE_SOURCE_DIR.html#variable:CMAKE_SOURCE_DIR), [`CMAKE_CURRENT_BINARY_DIR`](https://cmake.org/cmake/help/v3.25/variable/CMAKE_CURRENT_BINARY_DIR.html#variable:CMAKE_CURRENT_BINARY_DIR) and [`CMAKE_CURRENT_SOURCE_DIR`](https://cmake.org/cmake/help/v3.25/variable/CMAKE_CURRENT_SOURCE_DIR.html#variable:CMAKE_CURRENT_SOURCE_DIR) to the current working directory.

使用INTERFACE library 去指定c++标准

首先删除set

> ```cmake
> set(CMAKE_CXX_STANDARD 11)
> set(CMAKE_CXX_STANDARD_REQUIRED True)
> ```

然后添加一个library, 并添加编译的标准

> ```
> add_library(tutorial_compiler_flags INTERFACE)
> target_compile_features(tutorial_compiler_flags INTERFACE cxx_std_11)
> ```

链接我们的执行目标和库

> ```
> target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS} tutorial_compiler_flags)
> target_link_libraries(MathFunctions tutorial_compiler_flags)
> ```

Install the `Tutorial` executable and the `MathFunctions` library.

使用 cmake --install .    or  cmake --install . --prefix "/home/myuser/installdir"

> 对于MathFunctions， 我们想安装库和头文件到lib 和include 目录
>
> 对于Tutorial 可执行文件，我们想安装可执行文件到bin 或者 inlcude

> #MathFunctions/CMakeLists.txt
>
> ```cmake
> set(installable_libs MathFunctions tutorial_compiler_flags)
> install(TARGETS ${installable_libs} DESTINATION lib)
> install(FILES MathFunctions.h DESTINATION include)
> ```

> CMakeLists.txt
>
> ```cmake
> install(TARGETS Tutorial DESTINATION bin)
> install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
>   DESTINATION include
>   )
> ```

CTest为您的项目提供了一种轻松管理测试的方法。可以通过add _ test ( )命令添加测试。虽然本教程中没有明确涉及，但CTest与其他测试框架如Google Test有很多兼容性。

Navigate to the build directory and rebuild the application. then, run the `ctest` executable: [`ctest -N`](https://cmake.org/cmake/help/v3.25/manual/ctest.1.html#cmdoption-ctest-N) and [`ctest -VV`](https://cmake.org/cmake/help/v3.25/manual/ctest.1.html#cmdoption-ctest-VV). 

对于debug: ctest -C Debug -VV   release: ctest -C Release -VV

>  CMakeLists.txt
>
> ```
> enable_testing()
> #首先，我们使用add _ test ( )创建一个测试，该测试运行参数为25的Tutorial可执行文件。
> add_test(NAME Runs COMMAND Tutorial 25)
> #接下来，我们使用PASS_REGULAR_EXPRESSION测试属性来验证测试的输出包含某些字符串。打印消息当提供不正确数量的参数时
> add_test(NAME Usage COMMAND Tutorial)
> set_tests_properties(Usage
>   PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number"
>   )
>   
>   #
>  add_test(NAME StandardUse COMMAND Tutorial 4)
> set_tests_properties(StandardUse
>   PROPERTIES PASS_REGULAR_EXPRESSION "4 is 2"
>   )
>   
>   ## 添加更多的测试
>   function(do_test target arg result)
>   add_test(NAME Comp${arg} COMMAND ${target} ${arg})
>   set_tests_properties(Comp${arg}
>     PROPERTIES PASS_REGULAR_EXPRESSION ${result}
>     )
> endfunction()
> 
> # do a bunch of result based tests
> do_test(Tutorial 4 "4 is 2")
> do_test(Tutorial 9 "9 is 3")
> do_test(Tutorial 5 "5 is 2.236")
> do_test(Tutorial 7 "7 is 2.645")
> do_test(Tutorial 25 "25 is 5")
> do_test(Tutorial -25 "-25 is (-nan|nan|0)")
> do_test(Tutorial 0.0001 "0.0001 is 0.01")
> ```

提交测试结果到dashboard

> CMakeLists.txt
>
> ```
> enable_testing()
> #替换为
> include(CTest)
> ```

添加一个CTestConfig.cmake 文件

> ```
> set(CTEST_PROJECT_NAME "CMakeTutorial")
> set(CTEST_NIGHTLY_START_TIME "00:00:00 EST")
> 
> set(CTEST_DROP_METHOD "http")
> set(CTEST_DROP_SITE "my.cdash.org")
> set(CTEST_DROP_LOCATION "/submit.php?project=CMakeTutorial")
> set(CTEST_DROP_SITE_CDASH TRUE)
> ```

Ex:

```cmake
cmake_minimum_required(VERSION 3.1)

project (C++11)
# 设置C++标准为 C++ 11
# set(CMAKE_CXX_STANDARD 11)
# 设置C++标准为 C++ 14
set(CMAKE_CXX_STANDARD 14)
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
set(CMAKE_BUILD_TYPE "Debug")
set(CMAKE_CXX_FLAGS_DEBUG "$ENV{CXXFLAGS} -O0 -Wall -g")

add_executable(1-thread  1-thread.cpp)
target_link_libraries(1-thread pthread)
```



# GoogleTest

## 1. 一个好的测试

- 测试应该是**独立的和可重复的**。调试一个由于其他测试而成功或失败的测试是一件痛苦的事情。
  googletest通过在不同的对象上运行测试来隔离测试。当测试失败时，googletest允许您单独运行
  它以快速调试。  
- 测试应该很好地“组织”，并反映出**测试代码的结构**。googletest将相关测试分组到可以共享数据和
  子例程的测试套件中。这种通用模式很容易识别，并使测试易于维护。当人们切换项目并开始在新
  的代码库上工作时，这种一致性尤其有用。  
- 测试应该是“**可移植的”和“可重用的”**。谷歌有许多与平台无关的代码;它的测试也应该是平台中立
  的。googletest可以在不同的操作系统上工作，使用不同的编译器，所以googletest测试可以在多
  种配置下工作。  
- 当测试失败时，他们应该提供尽可能多的关于问题的“信息”。谷歌测试不会在第一次测试失败时停
  止。相反，它只停止当前的测试并继续下一个测试。还可以设置报告非致命失败的测试，在此之后
  当前测试将继续进行。因此，您可以在一个运行-编辑-编译周期中检测和修复多个错误。  
- 测试框架应该将测试编写者从日常琐事中解放出来，让他们专注于测试“内容”。Googletest自动跟
  踪所有定义的测试，并且不要求用户为了运行它们而枚举它们。  
- 测试应该是“快速的”。使用googletest，您可以在测试之间重用共享资源，并且只需要为设置/拆除
  支付一次费用，而无需使测试彼此依赖  

![image-20220522193803949](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220522193803949.png)

## 2. 环境准备

下载:

> git clone https://github.com/google/googletest.git
> \# 或者
> wget https://github.com/google/googletest/releases/tag/release-1.11.0  

安装:

> cd googletest
>
> cmake CMakeLists.txt
> make
> sudo make install  

重要的文件:

**googletest** 

头文件: #include "gtest/gtest.h"

链接静态库: libgtest.a  libgtest_main.a    

当不想写main函数的时候，可以直接链接到libgtest_main.a  

>g++ sample.cc -o sample -lgtest -lgtest_main -lpthread
>g++ sample.cc -o sample -lgmock -lgmock_main -lpthread  

或者自己有main函数:

>g++ sample.cc -o sample -lgtest -lpthread  

**googlemock**  

头文件: #include "gmock/gmock.h"

静态库文件: libgmock.a   libgmock_main.a  



## 3 断言

googletest中大量使用断言来决定测试的成败，所以我们有必要了解这些断言的含义。断言成对出现，它们测试相同的东西，但对当前函数有不同的影响。 ASSERT_ 版本在失败时产生致命失败，并中止当前函数。 EXPECT_ 版本生成非致命失败，它不会中止当前函数。通常首选EXPECT_ ，因为它们允许在测试中报告一个以上的失败。但是，如果在有问题的断言失败时继续没有意义，则应该使用 ASSERT_* 。所有断言宏都支持输出流，也就是当出现错误的时候，我们可以通过流输出更详细的信息；注意编码问题，经流输出的信息会自动转换为 UTF-8;

| 致命断言                   | 非致命断言                 | 验证                 |
| -------------------------- | -------------------------- | -------------------- |
| `ASSERT_TRUE(condition);`  | `EXPECT_TRUE(condition);`  | `condition` 是 true  |
| `ASSERT_FALSE(condition);` | `EXPECT_FALSE(condition);` | `condition` 是 false |



**二元比较**  

这个部分描述比较两个值的断言。

致命断言 非致命断言 验证   

| 致命断言               | **非致命断言**         |      | 验证         |
| ---------------------- | ---------------------- | ---- | ------------ |
| ASSERT_EQ(val1, val2); | EXPECT_EQ(val1, val2); |      | val1 == val2 |
| ASSERT_NE(val1, val2); | EXPECT_NE(val1, val2); |      | val1 != val2 |
| ASSERT_LT(val1, val2); | EXPECT_LT(val1, val2); |      | val1 < val2  |
| ASSERT_LE(val1, val2); | EXPECT_LE(val1, val2); |      | val1 <= val2 |
| ASSERT_GT(val1, val2); | EXPECT_GT(val1, val2); |      | val1 > val2  |
| ASSERT_GE(val1, val2); | EXPECT_GE(val1, val2); |      | val1 >= val2 |

当可能时，`ASSERT_EQ(actual, expected)` 好于 `ASSERT_TRUE(actual == expected)`，因为它在失败时告诉你 `actual` 和 `expected` 的值。`ASSERT_EQ()` 对指针执行相等性操作。如果使用两个 C 字符串，它测试它们是否位于相同的内存位置，而不是它们是否具有相同的值。因此，如果你想比较 C 字符串的值（比如 `const char*`），使用 `ASSERT_STREQ()`，特别地，要断言 C 字符串是 `NULL`，则使用 `ASSERT_STREQ(c_string, NULL)`，如果支持 c++11 则考虑使用 `ASSERT_EQ(c_string, nullptr)`。要比较两个 `string` 对象，你应该使用 `ASSERT_EQ`。当执行指针比较时使用 `*_EQ(ptr, nullptr)` 和 `*_NE(ptr, nullptr)` 而不是 `*_EQ(ptr, NULL)` 和 `*_NE(ptr, NULL)`。这是因为 `nullptr` 是类型安全的而 `NULL` 不是。



# Docker 

## 1. **什么是docker**

docker是一个用Go语言实现的开源项目，可以让我们方便的创建和使用容器，docker将程序以及程序所有的依赖都打包到docker container，这样你的程序可以在任何环境都会有一致的表现，这里程序运行的依赖也就是容器就好比集装箱，容器所处的操作系统环境就好比货船或港口，**程序的表现只和集装箱有关系(容器)，和集装箱放在哪个货船或者哪个港口(操作系统)没有关系**。

因此我们可以看到docker可以屏蔽环境差异，也就是说，只要你的程序打包到了docker中，那么无论运行在什么环境下程序的行为都是一致的，程序员再也无法施展表演才华了，**不会再有“在我的环境上可以运行”**，真正实现“build once, run everywhere”。

此外docker的另一个好处就是**快速部署**，这是当前互联网公司最常见的一个应用场景，一个原因在于容器启动速度非常快，另一个原因在于只要确保一个容器中的程序正确运行，那么你就能确信无论在生产环境部署多少都能正确运行。

docker出现之前，部署软件到不同环境所需的工作量巨大:

![image-20220529212511240](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220529212511240.png)

**Docker的好处**  

- 替代虚拟机：用户只需要关心应用程序，而不是操作系统。
- 软件原型：快速体验软件，同时避免干扰目前的设置或配备一个虚拟机的麻烦。比如要测试不同版本的 redis，可以使用docker开启多个ubuntu镜像，然后在不同的ubuntu镜像测试不同的redis版本。
- 打包软件：docker镜像对Linux用户没有依赖，可以将构建的镜像运行在不同的Linux机器上。
  让微服务成为可能：docker有助于将一个复杂的系统分解成一系列可组合的部分，有利于用户更好
  地思考其服务。
- 网络建模：可以在一台机器上启动数百个隔离的容器，因而对网络进行建模轻而易举。这对于显示
  世界场景的测试非常有用。
- 降低调试支出：可以让生产、测试、部署统一环境，而不因不同环境：失效的库、有问题的依赖、更新被错误实施或是执行顺序有误，甚至可能根本没有执行以及无法出现的错误等等。启用持续交付：更利于构建一个基于流水线的软件交付模型。  

**容器和虚拟机的区别**  

![image-20220529212615201](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220529212615201.png)

vm（虚拟机）与docker（容器)框架，直观上来讲vm多了一层guest OS，同时Hypervisor会对硬件资源进行虚拟化，docker直接使用硬件资源，所以资源利用率相对docker低也是比较容易理解的。服务器虚拟化解决的核心问题是资源调配，而容器解决的核心问题是应用开发、测试和部署。虚拟机技术通过Hypervisor层抽象底层基础设施资源，提供相互隔离的虚拟机，通过统一配置、统一管理，计算资源的可运维性，以及资源利用率都能够得到有效的提升。同时，虚拟机提供客户机操作系统，客户机变化不会影响宿主机，能够提供可控的测试环境，更能够屏蔽底层硬件甚至基础软件的差异
性，让应用做到的广泛兼容。然而，再牛逼的虚拟化技术，都不可避免地出现计算、IO、网络性能损失，毕竟多了一层软件，毕竟要运行一个完整的客户机操作系统。容器技术严格来说并不是虚拟化，没有客户机操作系统，是共享内核的。容器可以视为软件供应链的集装箱，能够把应用需要的运行环境、缓存环境、数据库环境等等封装起来，以最简洁的方式支持应用运行，轻装上阵，当然是性能更佳。Docker镜像特性则让这种方式简单易行。当然，因为共享内核，容器隔离性也没有虚拟机那么好 。

**Docker基本概念**  

Docker 包括三个基本概念:

- 镜像（Image）：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
- 容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中
  的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、
  删除、暂停等。
- 仓库（Repository）：仓库可看成一个代码控制中心，用来保存镜像。  

Docker 使用客户端-服务器 (C/S) 架构模式，使用远程API来管理和创建Docker容器。
Docker 容器通过 Docker 镜像来创建。容器与镜像的关系类似于面向对象编程中的对象与类。

| Docker | 面向对象 |
| ------ | -------- |
| 容器   | 对象     |
| 镜像   | 类       |

![image-20220529212818489](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220529212818489.png)  

## 2. docker安装和测试  

**准备工作**  

Docker CE 支持以下版本的 Ubuntu 操作系统：
Ubuntu Focal 20.04 (LTS)
Ubuntu Bionic 18.04 (LTS)
Ubuntu Xenial 16.04 (LTS)  

### 环境

> Linux系统版本：Ubuntu 22.04 Server x64
> Docker版本：Community 20.10.15

**卸载旧版本**  

旧版本的 Docker 称为 docker 或者 docker-engine ，使用以下命令卸载旧版本：  

> sudo apt-get remove docker docker-engine docker.io containerd runc  

### 脚本自动安装

> curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun

**使用 apt安装**  

> #首先更新源，安装必要的依赖软件
>
> sudo apt update
> sudo apt install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
>
> #导入源仓库的 GPG key
>
> curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
>
> #添加 Docker APT 软件源
>
> sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
>
> #安装 Docker 最新版本
>
> sudo apt update
> sudo apt install docker-ce docker-ce-cli containerd.io

如果需要安装指定版本，请参考：https://docs.docker.com/engine/install/ubuntu/ To install a specific version of Docker Engine 

查看一下docker版本 

> zhen@zhen:/home/wz/mynginx/composetest$ docker -v 
> Docker version 20.10.16, build aa7e414

**卸载docker**

如果需要卸载，不需要就别卸载。

1. 卸载 Docker Engine, CLI,和Containerd 包

   > sudo apt-get purge docker-ce docker-ce-cli containerd.io  

2. 删除残留文件:  

   > sudo rm -rf /var/lib/docker  

**启动 Docker CE**  

> 开机启动: sudo systemctl enable docker
> 启动:  docker：sudo systemctl start docker  

**建立 docker 用户组**  

默认情况下， docker 命令会使用 Unix socket 与 Docker 引擎通讯。而只有 root 用户和 docker 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 root用户。因此，更好地做法是将需要使用 docker 的用户加入 docker 用户组。建立 docker 组 :

> sudo groupadd docker  

**将当前用户加入 docker 组：**  

> sudo usermod -aG docker $USER  

退出当前终端并重新登录，进行如下测试。  

> zhen@zhen:/home/wz/mynginx/composetest$ docker run hello-world
>
> Hello from Docker!
> This message shows that your installation appears to be working correctly.
>
> To generate this message, Docker took the following steps:
>  1. The Docker client contacted the Docker daemon.
>  2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
>     (amd64)
>  3. The Docker daemon created a new container from that image which runs the
>     executable that produces the output you are currently reading.
>  4. The Docker daemon streamed that output to the Docker client, which sent it
>     to your terminal.
>
> To try something more ambitious, you can run an Ubuntu container with:
>  $ docker run -it ubuntu bash
>
> Share images, automate workflows, and more with a free Docker ID:
>  https://hub.docker.com/
>
> For more examples and ideas, visit:
>  https://docs.docker.com/get-started/

**若能正常输出以上信息，则说明安装成功。**  

## 3. 操作docker镜像  

Docker 运行容器前需要本地存在对应的镜像，如果本地不存在该镜像，Docker 会从镜像仓库下载该镜
像。
本章将介绍更多关于镜像的内容，包括：
从仓库获取镜像；
管理本地主机上的镜像；
介绍镜像实现的基本原理。  

### 3.1获取镜像

[Explore Docker's Container Image Repository | Docker Hub](https://hub.docker.com/search?q=&type=image)

上有大量的高质量的镜像可以用，这里我们就说一下怎么获取这些镜像。从 Docker 镜像仓库获取镜像的命令是 docker pull 。其命令格式为：

> docker pull [选项] [Docker Registry 地址[:端口号]/]仓库名[:标签]  

具体的选项可以通过 docker pull --help 命令看到，这里我们说一下镜像名称的格式。

- Docker 镜像仓库地址：地址的格式一般是 <域名/IP>[:端口号] 。默认地址是 Docker Hub。
- 仓库名：如之前所说，这里的仓库名是两段式名称，即 <用户名>/<软件名> 。对于 Docker Hub，
  如果不给出用户名，则默认为 library ，也就是官方镜像。    

> zhen@zhen:/home/wz$ docker pull redis:6.0.8
> 6.0.8: Pulling from library/redis
> bb79b6b2107f: Pull complete 
> 1ed3521a5dcb: Pull complete 
> 5999b99cee8f: Pull complete 
> 3f806f5245c9: Pull complete 
> f8a4497572b2: Pull complete 
> eafe3b6b8d06: Pull complete 
> Digest: sha256:21db12e5ab3cc343e9376d655e8eabbdbe5516801373e95a8a9e66010c5b8819
> Status: Downloaded newer image for redis:6.0.8
> docker.io/library/redis:6.0.8

上面的命令中没有给出 Docker 镜像仓库地址，因此将会从 Docker Hub 获取镜像。而镜像名称是redis:6.0.8，因此将会获取官方镜像 library/ redis仓库中标签为 6.0.8 的镜像。从下载过程中可以看到我们之前提及的分层存储的概念，镜像是由多层存储所构成。下载也是一层层的去下载，并非单一文件。下载过程中给出了每一层的 ID 的前 12 位。并且下载结束后，给出该镜像完整的 sha256 的摘要，以确保下载一致性。在使用上面命令的时候，你可能会发现，你所看到的层 ID 以及 sha256 的摘要和这里的不一样。这是因为官方镜像是一直在维护的，有任何新的 bug，或者版本更新，都会进行修复再以原来的标签发布，这样可以确保任何使用这个标签的用户可以获得更安全、更稳定的镜像。  

### 3.2 运行

有了镜像后，我们就能够以这个镜像为基础启动并运行一个容器。以上面的 redis:6.08 为例，如果我们打算启动里面的 bash 并且进行交互式操作的话，可以执行下面的命令。  

> zhen@zhen:/home/wz$ docker run -it --rm redis:6.0.8 bash
> root@9cc6017c54e2:/data# 

docker run 就是运行容器的命令，我们这里简要的说明一下上面用到的参数。  

- -it ：这是两个参数，一个是 -i ：交互式操作，一个是 -t 终端。我们这里打算进入 bash 执行
  一些命令并查看返回结果，因此我们需要交互式终端。
- --rm ：这个参数是说容器退出后随之将其删除。默认情况下，为了排障需求，退出的容器并不会立
  即删除，除非手动 docker rm 。我们这里只是随便执行个命令，看看结果，不需要排障和保留结
  果，因此使用 --rm 可以避免浪费空间。
- redis:6.0.8 ：这是指用 redis:6.0.8 镜像为基础来启动容器。
- bash ：放在镜像名后的是命令，这里我们希望有个交互式 Shell，因此用的是 bash 。  

进入容器后，我们可以在 Shell 下操作，执行任何所需的命令。这里，我们执行了 '**redis-server**'  ，这是 Linux 常用的查看当前系统版本的命令，从返回的结果可以看到容器内是 安装了redis6.0.8 系统  

> root@81faed055ffe:/data# redis-server
> 8:C 05 Jun 2022 01:42:53.703 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
> 8:C 05 Jun 2022 01:42:53.703 # Redis version=6.0.8, bits=64, commit=00000000, modified=0, pid=8, just started
> 8:C 05 Jun 2022 01:42:53.703 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
>                 _._                                                  
>            _.-``__ ''-._                                             
>       _.-``    `.  `_.  ''-._           Redis 6.0.8 (00000000/0) 64 bit
>   .-`` .-```.  ```\/    _.,_ ''-._                                   
>  (    '      ,       .-`  | `,    )     Running in standalone mode
>  |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
>  |    `-._   `._    /     _.-'    |     PID: 8
>   `-._    `-._  `-./  _.-'    _.-'                                   
>  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
>  |    `-._`-._        _.-'_.-'    |           http://redis.io        
>   `-._    `-._`-.__.-'_.-'    _.-'                                   
>  |`-._`-._    `-.__.-'    _.-'_.-'|                                  
>  |    `-._`-._        _.-'_.-'    |                                  
>   `-._    `-._`-.__.-'_.-'    _.-'                                   
>       `-._    `-.__.-'    _.-'                                       
>           `-._        _.-'                                           
>               `-.__.-'                                               
>
> 8:M 05 Jun 2022 01:42:53.704 # Server initialized
> 8:M 05 Jun 2022 01:42:53.704 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
> 8:M 05 Jun 2022 01:42:53.704 * Ready to accept connections

最后我们通过 `exit`退出了这个容器。  

### 3.3 查看本地已有的镜像

使用 docker image ls查看本地已有的镜像。  

>zhen@zhen:/home/wz$ docker  image ls
>REPOSITORY               TAG          IMAGE ID       CREATED         SIZE
>composetest_web          latest       31d233a3adc0   6 days ago      60.6MB
>127.0.0.1:5000/mynginx   latest       7489ae3bb3a0   6 days ago      142MB
>mynginx                  latest       7489ae3bb3a0   6 days ago      142MB
>nginx                    v3           7489ae3bb3a0   6 days ago      142MB
>nginx                    latest       0e901e68141f   7 days ago      142MB
>registry                 latest       773dbf02e42e   9 days ago      24.1MB
>redis                    alpine       c3ea2db12504   10 days ago     28.4MB
>python                   3.7-alpine   7642396105af   10 days ago     45.5MB
>hello-world              latest       feb5d9fea6a5   8 months ago    13.3kB
>redis                    6.0.8        16ecd2772934   19 months ago   104MB

果要删除本地的镜像，可以使用 docker image rm 命令，其格式为:

> docker image rm [选项] <镜像1> [<镜像2> ...]  

其中， <镜像> 可以是 镜像短 ID 、 镜像长 ID 、 镜像名 或者 镜像摘要 。  

我们可以用镜像的完整 ID，也称为 长 ID ，来删除镜像。使用脚本的时候可能会用长 ID，但是人工输入就太累了，所以更多的时候是用 短 ID 来删除镜像。 docker image ls 默认列出的就已经是短 ID了，一般取前3个字符以上，只要足够区分于别的镜像就可以了。  

如果观察上面这几个命令的运行输出信息的话，你会注意到删除行为分为两类，一类是 Untagged ，另一类是 Deleted 。镜像的唯一标识是其 ID 和摘要，而一个镜像可以有多个标签。因此当我们使用上面命令删除镜像的时候，实际上是在要求删除某个标签的镜像。所以首先需要做的是将满足我们要求的所有镜像标签都取消，这就是我们看到的 Untagged 的信息。因为一个镜像可以对应多个标签，因此当我们删除了所指定的标签后，可能还有别的标签指向了这个镜像，如果是这种情况，那么 Delete 行为就不会发生。所以并非所有的 docker rmi 都会产生删除镜像的行为，有可能仅仅是取消了某个标签而已。

当该镜像所有的标签都被取消了，该镜像很可能会失去了存在的意义，因此会触发删除行为。镜像是多层存储结构，因此在删除的时候也是从上层向基础层方向依次进行判断删除。镜像的多层结构让镜像复用变动非常容易，因此很有可能某个其它镜像正依赖于当前镜像的某一层。这种情况，依旧不会触发删除该层的行为。直到没有任何层依赖当前层时，才会真实的删除当前层。这就是为什么，有时候会奇怪，为什么明明没有别的标签指向这个镜像，但是它还是存在的原因，也是为什么有时候会发现所删除的层数和自己 docker pull 看到的层数不一样的源。除了镜像依赖以外，还需要注意的是容器对镜像的依赖。如果有用这个镜像启动的容器存在（即使容器没有运行），那么同样不可以删除这个镜像。之前讲过，容器是以镜像为基础，再加一层容器存储层，组成这样的多层存储结构去运行的。因此该镜像如果被这个容器所依赖的，那么删除必然会导致故障。如果这些容器是不需要的，应该先将它们删除，然后再来删除镜像。    



像其它可以承接多个实体的命令一样，可以使用 docker image ls -q 来配合使用 docker image rm ，这样可以成批的删除希望删除的镜像。我们在“镜像列表”章节介绍过很多过滤镜像列表的方式都可以拿过来使用。
比如，我们需要删除所有仓库名为 redis 的镜像：  

> docker image rm $(docker image ls -q redis)  

或者删除所有在 mongo:3.2 之前的镜像：  

> docker image rm $(docker image ls -q -f before=mongo:3.2)  

### 3.4使用 Dockerfile 定制镜像  

镜像的定制实际上就是定制每一层所添加的配置、文件。如果我们可以把每一层修改、安装、构建、操作的命令都写入一个脚本，用这个脚本来构建、定制镜像，那么之前提及的无法重复的问题、镜像构建透明性的问题、体积的问题就都会解决。这个脚本就是 Dockerfile。Dockerfile 是一个文本文件，其内包含了一条条的指令(Instruction)，每一条指令构建一层，因此每一条指令的内容，就是描述该层应当如何构建。  

还以之前定制 nginx 镜像为例，这次我们使用 Dockerfile 来定制。在一个空白目录中，建立一个文本文件，并命名为 Dockerfile ：  

>  mkdir mynginx
>
> cd mynginx
> touch Dockerfile  

其内容为：  

> FROM nginx
> RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html  

这个 Dockerfile 很简单，一共就两行。涉及到了两条指令， FROM 和 RUN 。然后build镜像

> docker build -t mynginx .  

使用`docker image ls` 查看新的镜像

**FROM 指定基础镜像**   

所谓定制镜像，那一定是以一个镜像为基础，在其上进行定制。就像我们之前运行了一个 nginx 镜像的容器，再进行修改一样，基础镜像是必须指定的。而 FROM 就是指定基础镜像，因此一个Dockerfile 中 FROM 是必备的指令，并且必须是第一条指令。  在[Docker Hub Container Image Library | App Containerization](https://hub.docker.com/) 上有非常多的高质量的官方镜像，有可以直接拿来使用的服务类的镜像。

除了选择现有镜像为基础镜像外，Docker 还存在一个特殊的镜像，名为 scratch 。这个镜像是虚拟的概念，并不实际存在，它表示一个空白的镜像。  

> FROM scratch
> ...  

如果你以 scratch 为基础镜像的话，意味着你不以任何镜像为基础，接下来所写的指令将作为镜像第一层开始存在。  

**RUN 执行命令**  

RUN 指令是用来执行命令行命令的。由于命令行的强大能力， RUN 指令在定制镜像时是最常用的指令
之一。其格式有两种：

- shell 格式： RUN <命令> ，就像直接在命令行中输入的命令一样。刚才写的 Dockerfile 中的 RUN
  指令就是这种格式。  

  > RUN echo '<h1>Hello, Docker!</h1>' > /usr/share/nginx/html/index.html  

- exec 格式： RUN ["可执行文件", "参数1", "参数2"] ，这更像是函数调用中的格式。  

既然 RUN 就像 Shell 脚本一样可以执行命令，那么我们是否就可以像 Shell 脚本一样把每个命令对应一
个 RUN 呢？比如这样：  

> FROM debian:jessie
> RUN apt-get update
> RUN apt-get install -y gcc libc6-dev make
> RUN wget -O redis.tar.gz "http://download.redis.io/releases/redis-3.2.5.tar.gz"
> RUN mkdir -p /usr/src/redis
> RUN tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1
> RUN make -C /usr/src/redis
> RUN make -C /usr/src/redis install  

之前说过，Dockerfile 中每一个指令都会建立一层， RUN 也不例外。每一个 RUN 的行为，就和刚才我们手工建立镜像的过程一样：新建立一层，在其上执行这些命令，执行结束后， commit 这一层的修改，构成新的镜像。而上面的这种写法，创建了 7 层镜像。这是完全没有意义的，而且很多运行时不需要的东西，都被装进了镜像里，比如编译环境、更新的软件包等等。结果就是产生非常臃肿、非常多层的镜像，不仅仅增加了构建部署的时间，也很容易出错。这是很多初学 Docker 的人常犯的一个错误。Union FS 是有最大层数限制的，比如 AUFS，曾经是最大不得超过 42 层，现在是不得超过 127 层。上面的 Dockerfile 正确的写法应该是这样:

> FROM debian:jessie
> RUN buildDeps='gcc libc6-dev make' \
> && apt-get update \
> && apt-get install -y $buildDeps \
> && wget -O redis.tar.gz "http://download.redis.io/releases/redis-
> 3.2.5.tar.gz" \
> && mkdir -p /usr/src/redis \
> && tar -xzf redis.tar.gz -C /usr/src/redis --strip-components=1 \
> && make -C /usr/src/redis \
> && make -C /usr/src/redis install \
> && rm -rf /var/lib/apt/lists/* \
> && rm redis.tar.gz \  
>
> && rm -r /usr/src/redis \
> && apt-get purge -y --auto-remove $buildDeps 

首先，之前所有的命令只有一个目的，就是编译、安装 redis 可执行文件。因此没有必要建立很多层，这只是一层的事情。因此，这里没有使用很多个 RUN 对一一对应不同的命令，而是仅仅使用一个 RUN指令，并使用 && 将各个所需命令串联起来。将之前的 7 层，简化为了 1 层。在撰写 Dockerfile 的时候，要经常提醒自己，这并不是在写 Shell 脚本，而是在定义每一层该如何构建。并且，这里为了格式化还进行了换行。Dockerfile 支持 Shell 类的行尾添加 \ 的命令换行方式，以及行首 # 进行注释的格式。良好的格式，比如换行、缩进、注释等，会让维护、排障更为容易，这是一个比较好的习惯。此外，还可以看到这一组命令的最后添加了清理工作的命令，删除了为了编译构建所需要的软件，清理了所有下载、展开的文件，并且还清理了 apt 缓存文件。这是很重要的一步，我们之前说过，镜像是多层存储，每一层的东西并不会在下一层被删除，会一直跟随着镜像。因此镜像构建时，一定要确保每一层只添加真正需要添加的东西，任何无关的东西都应该清理掉。   

**构建镜像**  

在 Dockerfile 文件所在目录执行：

> docker build -t nginx:v3 .  

这里我们使用了docker build命令来进行镜像的构建:

>  docker build [选项] <上下文路径/URL/->  

**镜像构建上下文（Context）**  

如果注意，会看到 docker build 命令最后有一个 . 。 . 表示当前目录，而 Dockerfile 就在当前目录，因此不少人以为这个路径是在指定 Dockerfile 所在路径，这么理解其实是不准确的。如果对应上面的命令格式，你可能会发现，这是在指定上下文路径。那么什么是上下文呢？  

首先我们要理解 docker build 的工作原理。Docker 在运行时分为 Docker 引擎（也就是服务端守护进程）和客户端工具。Docker 的引擎提供了一组 REST API，被称为 Docker Remote API，而如docker 命令这样的客户端工具，则是通过这组 API 与 Docker 引擎交互，从而完成各种功能。因此，虽然表面上我们好像是在本机执行各种 docker 功能，但实际上，一切都是使用的远程调用形式在服务端（Docker 引擎）完成。也因为这种 C/S 设计，让我们操作远程服务器的 Docker 引擎变得轻而易举。当我们进行镜像构建的时候，并非所有定制都会通过 RUN 指令完成，经常会需要将一些本地文件复制进镜像，比如通过 COPY 指令、 ADD 指令等。而 docker build 命令构建镜像，其实并非在本地构建，而是在服务端，也就是 Docker 引擎中构建的。那么在这种客户端/服务端的架构中，如何才能让服务端获得本地文件呢？这就引入了上下文的概念。当构建的时候，用户会指定构建镜像上下文的路径， docker build 命令得知这个路径后，会将路径下的所有内容打包，然后上传给 Docker 引擎。这样 Docker 引擎收到这个上下文包后，展开就会获得构建镜像所需的一切文件。  

如果在 Dockerfile 中这么写：  

> COPY ./package.json /app/  

这并不是要复制执行 docker build 命令所在的目录下的 package.json ，也不是复制 Dockerfile所在目录下的 package.json ，而是复制 上下文（context） 目录下的 package.json 。  

现在就可以理解刚才的命令 docker build -t nginx:v3 . 中的这个 . ，实际上是在指定上下文的目录， docker build 命令会将该目录下的内容打包交给 Docker 引擎以帮助构建镜像。

理解构建上下文对于镜像构建是很重要的，避免犯一些不应该的错误。比如有些初学者在发现 COPY/opt/xxxx /app 不工作后，于是干脆将 Dockerfile 放到了硬盘根目录去构建，结果发现 docker build 执行后，在发送一个几十 GB 的东西，极为缓慢而且很容易构建失败。那是因为这种做法是在让docker build 打包整个硬盘，这显然是使用错误。一般来说，应该会将 Dockerfile 置于一个空目录下，或者项目根目录下。如果该目录下没有所需文件，那么应该把所需文件复制一份过来。如果目录下有些东西确实不希望构建时传给 Docker 引擎，那么可以用 .gitignore 一样的语法写一个 .dockerignore ，该文件是用于剔除不需要作为上下文传递给 Docker 引擎的。那么为什么会有人误以为 . 是指定 Dockerfile 所在目录呢？这是因为在默认情况下，如果不额外指定 Dockerfile 的话，会将上下文目录下的名为 Dockerfile 的文件作为 Dockerfile。这只是默认行为，实际上 Dockerfile 的文件名并不要求必须为 Dockerfile ，而且并不要求必须位于上下文目录中，比如可以用 -f ../Dockerfile.php 参数指定某个文件作为 Dockerfile 。当然，一般大家习惯性的会使用默认的文件名 Dockerfile ，以及会将其置于镜像构建上下文目录中。

**其它 docker build 的用法**      

 直接用 Git repo 进行构建
或许你已经注意到了， docker build 还支持从 URL 构建，比如可以直接从 Git repo 中构建：  

>  docker build https://github.com/twang2218/gitlab-ce-zh.git#:8.14  

用给定的 tar 压缩包构建  

> docker build http://server/context.tar.gz  

如果所给出的 URL 不是个 Git repo，而是个 tar 压缩包，那么 Docker 引擎会下载这个包，并自动解压缩，以其作为上下文，开始构建。  

从标准输入中读取 Dockerfile 进行构建  

> docker build - < Dockerfile  

或

> cat Dockerfile | docker build -  

如果标准输入传入的是文本文件，则将其视为 Dockerfile ，并开始构建。这种形式由于直接从标准输入中读取 Dockerfile 的内容，它没有上下文，因此不可以像其他方法那样可以将本地文件 COPY 进镜像之类的事情。  

![image-20220605114748121](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220605114748121.png)





## 4. 操作Docker容器

容器是 Docker 三大核心概念之一。如果说Docker镜像是类，则Docker容器就是实例化后的对象，同一个镜像可以启动多个容器。简单的说，容器是独立运行的一个或一组应用，以及它们的运行态环境。对应的，虚拟机可以理解为模拟运行的一整套操作系统（提供了运行态环境和其他系统环境）和跑在上面的应用。    

#### 1. 启动容器

启动容器有两种方式，一种是基于镜像新建一个容器并启动，另外一个是将在终止状态（ stopped ）的容器重新启动。

新建容器:  所需要的命令主要为 docker run 。

> sudo docker run -t -i ubuntu:16.04 /bin/bash     

其中， -t 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上， -i 则让容器的标准输入保持打开。
在交互模式下，用户可以通过所创建的终端来输入命令。

当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：

- 检查本地是否存在指定的镜像，不存在就从公有仓库下载
- 利用镜像创建并启动一个容器
- 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
- 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
- 从地址池配置一个 ip 地址给容器
- 执行用户指定的应用程序
- 执行完毕后容器被终止

启动已终止容器:

可以利用 docker container start 命令，直接将一个已经终止的容器启动运行。
容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。可以在伪终端中利用 ps 或 top 来查看进程信息。    

#### 2. 后台运行

更多的时候，需要让 Docker 在后台运行而不是直接把执行命令的结果输出在当前宿主机下。此时，可以通过添加 -d 参数来实现。  

容器是否会长久运行，是和 docker run 指定的命令有关，和 -d 参数无关。
使用 -d 参数启动后会返回一个唯一的 id （CONTAINER ID），也可以通过 docker container ls 命
令来查看容器信息。  

要获取容器的输出信息，可以通过 docker container logs 命令。  

#### 3. 终止容器

可以使用 docker container stop 来终止一个运行中的容器。
此外，当 Docker 容器中指定的应用终结时，容器也自动终止。
例如对于上一章节中只启动了一个终端的容器，用户通过 exit 命令或 Ctrl+d 来退出终端时，所创建
的容器立刻终止。
注意：容器不退出，返回宿主机 则使用Ctrl+P+Q  

终止状态的容器可以用 docker container ls -a 命令看到。  

处于终止状态的容器，可以通过 docker container start 命令来重新启动。
此外， docker container restart 命令会将一个运行态的容器终止，然后再重新启动它 。

#### 4. 进入容器

在使用 -d 参数时，容器启动后会进入后台。
某些时候需要进入容器进行操作，包括使用 docker attach 命令或 docker exec 命令，推荐大家使
用 docker exec 命令，原因会在下面说明 :

docker exec 后边可以跟多个参数，这里主要说明 -i -t 参数。
只用 -i 参数时，由于没有分配伪终端，界面没有我们熟悉的 Linux 命令提示符，但命令执行结果仍然
可以返回。
当 -i -t 参数一起使用时，则可以看到我们熟悉的 Linux 命令提示符。  

#### 5 . 导出和导入容器

导出容器

如果要导出本地某个容器，可以使用 docker export 命令。  

> docker container ls -a  
>
> docker export 7691a814370e > ubuntu.tar

这样将导出容器快照到本地文件。  

导入容器

可以使用 docker import 从容器快照文件中再导入为镜像，例如  

> cat ubuntu.tar | docker import - test/ubuntu:v1.0  

此外，也可以通过指定 URL 或者某个目录来导入，例如  

> docker import http://example.com/exampleimage.tgz example/imagerepo  

用户既可以使用 _ _docker load_ 来导入镜像存储文件到本地镜像库，也可以使用 _docker import_ 来导入一个容器快照到本地镜像库。这两者的区别在于容器快照文件将丢弃所有的历史记录和元数据信息（即仅保存容器当时的快照状态），而镜像存储文件将保存完整记录，体积也要大。此外，从容器快照文件导入时可以重新指定标签等元数据信息。



## 5. Docker Compose 

Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用。  

Compose 项目是 Docker 官方的开源项目，负责实现对 Docker 容器集群的快速编排。从功能上看，跟OpenStack 中的 Heat 十分类似。其代码目前在 https://github.com/docker/compose 上开源。Compose 定位是 「定义和运行多个 Docker 容器的应用（Defining and running multi-container Docker applications）」，其前身是开源项目 Fig

我们知道使用一个 **Dockerfile** 模板文件，可以让用户很方便的定义一个单独的应用容器。然而，在日常工作中，经常会碰到需要多个容器相互配合来完成某项任务的情况。例如要实现一个 Web 项目，除了 Web 服务容器本身，往往还需要再加上后端的数据库服务容器，甚至还包括负载均衡容器等。  

Compose 恰好满足了这样的需求。它允许用户通过一个单独的 docker-compose.yml 模板文件
（YAML 格式）来定义一组相关联的应用容器为一个项目（project）。
Compose 中有两个重要的概念：

- 服务 ( service )：一个应用的容器，实际上可以包括若干运行相同镜像的容器实例。
- 项目 ( project )：由一组关联的应用容器组成的一个完整业务单元，在 docker-compose.yml 文件中定义

Compose 的默认管理对象是项目，通过子命令对项目中的一组容器进行便捷地生命周期管理。
Compose 项目由 Python 编写，实现上调用了 Docker 服务提供的 API 来对容器进行管理。因此，只要
所操作的平台支持 Docker API，就可以在其上利用 Compose 来进行编排管理。  

### 1. 安装和卸载

Compose 支持 Linux、macOS、Windows 10 三大平台。
Compose 可以通过 Python 的包管理工具 pip 进行安装，也可以直接下载编译好的二进制文件使用，甚至能够直接在 Docker 容器中运行。前两种方式是传统方式，适合本地环境下安装使用；最后一种方式则不破坏系统环境，更适合云计算场景。这里我们只介绍二进制安装方式，PIP安装的方式比较麻烦。Docker for Mac 、 Docker for Windows 自带 docker-compose 二进制文件，安装 Docker 之后可以直接使用。

>  zhen@zhen:/home/wz$ docker-compose -version
> docker-compose version 1.25.0, build unknown

在 Linux 上的也安装十分简单，从 官方 https://github.com/docker/compose/releases 处直接下载编译好的二进制文件即可。例如，在 Linux 64 位系统上直接下载对应的二进制包 

>  \# 先把docker-compose文件dump到当前目录  
>
> wget https://github.com/docker/compose/releases/download/1.27.4/dockercompose-Linux-x86_64
>
> #然后拷贝到/usr/bin/
>
> sudo cp -arf docker-compose-Linux-x86_64 /usr/bin/docker-compose
> sudo chmod +x /usr/bin/docker-compose 

卸载:

> sudo rm /usr/bin/docker-compose  

### 2. 使用

**准备**

创建一个测试目录：  

> mkdir composetest
> cd composetest

在测试目录中创建一个名为 app.py 的文件，并复制粘贴以下内容：composetest/app.py 文件代码  

```python
import time
import redis
from flask import Flask
app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)
def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)
@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)

```

在此示例中，redis 是应用程序网络上的 redis 容器的主机名，该主机使用的端口为 6379。
在 composetest 目录中创建另一个名为 requirements.txt 的文件，内容如下：  

```txt
flask
redis
```

**创建 Dockerfile 文件**

在 composetest 目录中，创建一个名为的文件 Dockerfile，内容如下：  

```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP app.py
ENV FLASK_RUN_HOST 0.0.0.0
#RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
COPY . .
CMD ["flask", "run"]
```

 Dockerfile 内容解释：

- FROM python:3.7-alpine: 从 Python 3.7 映像开始构建镜像。
- WORKDIR /code: 将工作目录设置为 /code。  
- ENV FLASK_APP app.py
  ENV FLASK_RUN_HOST 0.0.0.0
  设置 flask 命令使用的环境变量。RUN apk add --no-cache gcc musl-dev linux-headers: 安装 gcc，以便诸如 MarkupSafe 和SQLAlchemy 之类的 Python 包可以编译加速。这里先注释掉，下载太费时间了。  

- COPY requirements.txt requirements.txt
- RUN pip install -r requirements.txt
  复制 requirements.txt 并安装 Python 依赖项。
  COPY . .: 将 . 项目中的当前目录复制到 . 镜像中的工作目录。
  CMD ["flask", "run"]: 容器提供默认的执行命令为：flask run。  

**创建 docker-compose.yml**  

在测试目录中创建一个名为 docker-compose.yml 的文件，然后粘贴以下内容：  

```yaml
# yaml 配置
version: '3'
services:
  web:
    build: .
    ports:
     - "5000:5000"
  redis:
    image: "redis:alpine"

```

该 Compose 文件定义了两个服务：web 和 redis。

- web：该 web 服务使用从 Dockerfile 当前目录中构建的镜像。然后，它将容器和主机绑定到暴露
  的端口 5000。此示例服务使用 Flask Web 服务器的默认端口 5000 。
- redis：该 redis 服务使用 Docker Hub 的公共 Redis 映像。 

**使用 Compose 命令构建和运行您的应用**   

在测试目录中，执行以下命令来启动应用程序：  

> docker-compose up  

如果你想在后台执行该服务可以加上 -d 参数：docker-compose up -d  







# Unbuntu18.04 配置DPDK

## 1. VMware 添加网卡

网络适配器配置成桥接模式，为DPDK准备的，网络适配器3配置成NAT模式为ssh准备的。

![image-20220611225358402](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220611225358402.png)

**修改网卡的配置信息:**

在虚拟机关机状态下，修改虚拟机的.vmx文件， 修改ethernet0.virtualDev 从e1000到vmxnet3， 因为 vmware 的 vmxnet3 支持多队
列网卡  

> ethernet0.virtualDev = "vmxnet3" 

重启虚拟机， 查看网卡, 成功被被配置为vmxnet3，

> zhw@zhw:~/zhw/dpdk-stable-19.08.2$ ethtool -i eth0 
> driver: vmxnet3
> version: 1.5.0.0-k-NAPI
> firmware-version: 
> expansion-rom-version: 
> bus-info: 0000:03:00.0
> supports-statistics: yes
> supports-test: no
> supports-eeprom-access: no
> supports-register-dump: yes
> supports-priv-flags: no

**再看下eth1的网卡**

> zhw@zhw:~/zhw/dpdk-stable-19.08.2$ ethtool -i eth1
> driver: e1000
> version: 5.13.0-48-generic
> firmware-version: 
> expansion-rom-version: 
> bus-info: 0000:02:06.0
> supports-statistics: yes
> supports-test: yes
> supports-eeprom-access: yes
> supports-register-dump: yes
> supports-priv-flags: no

**查看系统是否支持多队列网卡 :**

执行： cat /proc/interrupts  

![image-20220611230847732](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220611230847732.png)

因为我这里配置4个CPU, 所以会有4个队列。对应的中断号分别是56、57、58、59。这里虚拟机有多少个CPU就会有多少个队列，如果你有8个CPU则设置亲缘性时要设置8个。

**设置中断号的亲缘性**：(选择性操作，不操作也可以)

root下：

```txt
echo 1 > /proc/irq/56/smp_affinity
echo 2 > /proc/irq/57/smp_affinity
echo 4 > /proc/irq/58/smp_affinity
echo 8 > /proc/irq/59/smp_affinity
```

 上面操作的目的是将4个网卡队列各自绑定到一个CPU。

## 2. 修改 ubuntu 系统的启动参数(设置巨页)

sudo vim /etc/default/grub

![image-20220611231400501](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220611231400501.png)

如果是虚拟机：

GRUB_CMDLINE_LINUX改成

> GRUB_CMDLINE_LINUX="find_preseed=/preseed.cfg noprompt net.ifnames=0 biosdevname=0 default_hugepagesz=2M hugepagesz=2M hugepages=1024 isolcpus=0-2"

如果是物理机：

>  default_hugepages=1G hugepagesz=1G hugepages=20 isolcpus=0-7

sudo update-grub

重启虚拟机



## 3. 编译 DPDK  

下载 dpdk : https://core.dpdk.org/download/  

![image-20220611231627953](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220611231627953.png)



**安装依赖包:**

> apt-get install numactl
>
> apt-get install libnuma-dev
>
> apt-get install net-tools



可以通过 usertools/dpdk-setup.sh ， 64 位系统选择 39  编译。

![image-20220611231711560](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220611231711560.png)

编译完会多出 x86_64-native-linux-gcc 的文件夹 。

但是编译的过程种会有一些错误:

1. /root/dpdk/dpdk-stable-19.08.2/x86_64-native-linuxapp-gcc/build/kernel/linux/kni/kni_net.c:737:20: error: initialization of ‘void (*)(struct net_device , unsigned int)’ from incompatible pointer type ‘void ()(struct net_device *)’ [-Werror=incompatible-pointer-types]
   .ndo_tx_timeout = kni_net_tx_timeout,

   **解决方法：**
   cd /root/dpdk/dpdk-stable-19.08.2/x86_64-native-linuxapp-gcc/build/kernel/linux/kni
   vi kni_net.c
   按esc，；set number 737行 然后按i进入插入模式，将报错部分注释掉
   ![image-20220611235502594](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220611235502594.png)

2. /root/dpdk/dpdk-stable-19.08.2/x86_64-native-linuxapp-gcc/build/kernel/linux/kni/kni_net.c:594:1: error: ‘kni_net_tx_timeout’ defined but not used [-Werror=unused-function]
   kni_net_tx_timeout(struct net_device *dev)
   ![img](https://img-blog.csdnimg.cn/adffd83987ae4cf395615430c2314bca.png)

   **解决方法：**
   cd /root/dpdk/dpdk-stable-19.08.2/x86_64-native-linuxapp-gcc/build/kernel/linux/kni
   Vi Makefile
   找到Makefile，去掉其中-Werror （按esc进入编辑模式, 按“/”再输入关键词Werror找到这行），重新编译。
   ![在这里插入图片描述](https://img-blog.csdnimg.cn/6e69e10ce3724807a51eae447022621c.png)

​		编译完成，结果如下（第一次比较慢，大概10分钟）：

![image-20220611235715848](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220611235715848.png)

编译完会多出 x86_64-native-linux-gcc 的文件夹 

**设置 DPDK 的环境变量**  

> export RTE_SDK=/home/dpdk (自己的dpdk目录)
> export RTE_TARGET=x86_64-native-linux-gcc  



**执行 testpmd**  

执行

>  usertools/dpdk-setup.sh  

![image-20220611235945710](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220611235945710.png)

1. 选择 43 插入 IGB_UIO 模块， 选择网卡为 vmxnet3 会加载此模块
2. 选择 44 插入 VFIO 模块，选择网卡为 e1000 会加载此模块
3. 选择 49 绑定 igb_uio 模块(我设置失败了)， 也可以退出，通过命令来执行(强烈推荐)。
   \# ifconfig eth0 down
   \# usertools/dpdk-devbind.py --bind=igb_uio eth0  

​				可能会报错: /usr/bin/env: ‘python’: No such file or directory

​				sudo ln -s /usr/bin/python3 /usr/bin/python

4. 选择 53 运行 testpmd  

![image-20220612000146410](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220612000146410.png)

​				有的会产生错误:

​				![image-20220612000312703](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220612000312703.png)

​				第三步没有执行成功。 

5. testpmd> show port info 0

   ![image-20220612000437676](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220612000437676.png)

​					表示成功了。



**编译helloword**

进入 example/helloworld ,
可以直接 make,
也可以通过 gcc 命令编译

> gcc -o helloword main.c -I /usr/local/include/dpdk/ -ldpdk -lpthread -lnuma -ldl   

执行helloworld, 注意一定要用sudo 执行，否则会错误。

> sudo ./helloworld -l 0-3 -n 4

![image-20220612000718861](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220612000718861.png)

可以看到在4个核上跑了。





# Jemalloc

除了 jemalloc 之外，业界还有一些著名的内存分配器实现，例如 ptmalloc 和 tcmalloc。对这三个内存分配器进行对比：

- ptmalloc 是基于 glibc 实现的内存分配器，它是一个标准实现，所以兼容性较好。pt 表示 per thread 的意思。当然 ptmalloc 确实在多线程的性能优化上下了很多功夫。由于过于考虑性能问题，多线程之间内存无法实现共享，只能每个线程都独立使用各自的内存，所以在内存开销上是有很大浪费的

- tcmalloc 出身于 Google，全称是 thread-caching malloc，所以 tcmalloc 最大的特点是带有线程缓存，tcmalloc 非常出名，目前在 Chrome、Safari 等知名产品中都有所应有。tcmalloc 为每个线程分配了一个局部缓存，对于小对象的分配，可以直接由线程局部缓存来完成，对于大对象的分配场景，tcmalloc 尝试采用自旋锁来减少多线程的锁竞争问题。
- jemalloc 借鉴了 tcmalloc 优秀的设计思路，所以在架构设计方面两者有很多相似之处，同样都包含 thread cache 的特性。但是 jemalloc 在设计上比 ptmalloc 和 tcmalloc 都要复杂，jemalloc 将内存分配粒度划分为 Small、Large、Huge 三个分类，并记录了很多 meta 数据，所以在空间占用上要略多于 tcmalloc，不过在大内存分配的场景，jemalloc 的内存碎片要少于 tcmalloc。jemalloc 内部采用红黑树管理内存块和分页，Huge 对象通过红黑树查找索引数据可以控制在指数级时间。

## 伙伴算法

采用了分离适配的设计思想，将物理内存按照 2 的次幂进行划分，内存分配时也是按照 2 的次幂大小进行按需分配，例如 4KB、 8KB、16KB 等。假设我们请求分配的内存大小为 10KB，那么会按照 16KB 分配。

伙伴算法把内存划分为 11 组不同的 2 次幂大小的内存块集合，每组内存块集合都用双向链表连接。链表中每个节点的内存块大小分别为 1、2、4、8、16、32、64、128、256、512 和 1024 个连续的 Page，例如第一组链表的节点为 2^0 个连续 Page，第二组链表的节点为 2^1 个连续 Page，以此类推。
![Drawing 5.png](https://s0.lgstatic.com/i/image/M00/71/62/Ciqc1F--HzqAdUBdAANa3t7uXSk503.png)

首先需要找到存储 2^4 连续 Page 所对应的链表，即数组下标为 4；
查找 2^4 链表中是否有空闲的内存块，如果有则分配成功；
如果 2^4 链表不存在空闲的内存块，则继续沿数组向上查找，即定位到数组下标为 5 的链表，链表中每个节点存储 2^5 的连续 Page；
如果 2^5 链表中存在空闲的内存块，则取出该内存块并将它分割为 2 个 2^4 大小的内存块，其中一块分配给进程使用，剩余的一块链接到 2^4 链表中。



当进程使用完内存归还时，需要检查其伙伴块的内存是否释放，所谓伙伴块是不仅大小相同，而且两个块的地址是连续的，其中低地址的内存块起始地址必须为 2 的整数次幂。如果伙伴块是空闲的，那么就会将两个内存块合并成更大的块，然后重复执行上述伙伴块的检查机制。直至伙伴块是非空闲状态，那么就会将该内存块按照实际大小归还到对应的链表中。频繁的合并会造成 CPU 浪费，所以并不是每次释放都会触发合并操作，当链表中的内存块个数小于某个阈值时，并不会触发合并操作。

伙伴算法有效地减少了外部碎片，但是有可能会造成非常严重的内部碎片，最严重的情况会带来 50% 的内存碎片。



## Slab算法

Slab 算法在伙伴算法的基础上，对小内存的场景专门做了优化，采用了内存池的方案，解决内部碎片问题。

Linux 内核使用的就是 Slab 算法，因为内核需要频繁地分配小内存，所以 Slab 算法提供了一种高速缓存机制，使用缓存存储内核对象，当内核需要分配内存时，基本上可以通过缓存中获取。此外 Slab 算法还可以支持通用对象的初始化操作，避免对象重复初始化的开销。下图是 Slab 算法的结构图，Slab 算法实现起来非常复杂，本文只做一个简单的了解。

![image.png](https://s0.lgstatic.com/i/image/M00/71/62/Ciqc1F--H2KAIoZ_AAcYUn319Hc822.png)

在 Slab 算法中维护着大小不同的 Slab 集合，在最顶层是 cache_chain，cache_chain 中维护着一组 kmem_cache 引用，kmem_cache 负责管理一块固定大小的对象池。通常会提前分配一块内存，然后将这块内存划分为大小相同的 slot，不会对内存块再进行合并，同时使用位图 bitmap 记录每个 slot 的使用情况。

kmem_cache 中包含三个 Slab 链表：完全分配使用 slab_full、部分分配使用 slab_partial和完全空闲 slabs_empty，这三个链表负责内存的分配和释放。每个链表中维护的 Slab 都是一个或多个连续 Page，每个 Slab 被分配多个对象进行存储。Slab 算法是基于对象进行内存管理的，它把相同类型的对象分为一类。当分配内存时，从 Slab 链表中划分相应的内存单元；当释放内存时，Slab 算法并不会丢弃已经分配的对象，而是将它保存在缓存中，当下次再为对象分配内存时，直接会使用最近释放的内存块。

单个 Slab 可以在不同的链表之间移动，例如当一个 Slab 被分配完，就会从 slab_partial 移动到 slabs_full，当一个 Slab 



## jemalloc架构设计

![image (1).png](https://s0.lgstatic.com/i/image/M00/71/62/Ciqc1F--H3KAEYJFAAp4aFcW83A719.png)

jemalloc 有以下几个核心概念：

- arena 是 jemalloc 最重要的部分，内存由一定数量的 arenas 负责管理。每个用户线程都会被绑定到一个 arena 上，线程采用 round-robin 轮询的方式选择可用的 arena 进行内存分配，为了减少线程之间的锁竞争，默认每个 CPU 会分配 4 个 arena。
- bin 用于管理不同档位的内存单元，每个 bin 管理的内存大小是按分类依次递增。因为 jemalloc 中小内存的分配是基于 Slab 算法完成的，所以会产生不同类别的内存块。
- chunk 是负责管理用户内存块的数据结构，chunk 以 Page 为单位管理内存，默认大小是 4M，即 1024 个连续 Page。每个 chunk 可被用于多次小内存的申请，但是在大内存分配的场景下只能分配一次。
- run 实际上是 chunk 中的一块内存区域，每个 bin 管理相同类型的 run，最终通过操作 run 完成内存分配。run 结构具体的大小由不同的 bin 决定，例如 8 字节的 bin 对应的 run 只有一个 Page，可以从中选取 8 字节的块进行分配。
- region 是每个 run 中的对应的若干个小内存块，每个 run 会将划分为若干个等长的 region，每次内存分配也是按照 region 进行分发。
  tcache 是每个线程私有的缓存，用于 small 和 large 场景下的内存分配，每个 tcahe 会对应一个 arena，tcache 本身也会有一个 bin 数组，称为tbin。与 arena 中 bin 不同的是，它不会有 run 的概念。tcache 每次从 arena 申请一批内存，在分配内存时首先在 tcache 查找，从而避免锁竞争，如果分配失败才会通过 run 执行内存分配。



以上核心概念的关系如下：

- 内存是由一定数量的 arenas 负责管理，线程均匀分布在 arenas 当中；
- 每个 arena 都包含一个 bin 数组，每个 bin 管理不同档位的内存块；
- 每个 arena 被划分为若干个 chunks，每个 chunk 又包含若干个 runs，每个 run 由连续的 Page 组成，run 才是实际分配内存的操作对象；
- 每个 run 会被划分为一定数量的 regions，在小内存的分配场景，region 相当于用户内存；
- 每个 tcache 对应 一个 arena，tcache 中包含多种类型的 bin。

以 Samll、Large 和 Huge 三种场景分析jemalloc的整体内存分配和释放流程

- Small：如果请求分配内存的大小小于 arena 中的最小的 bin，那么优先从线程中对应的 tcache 中进行分配。首先确定查找对应的 tbin 中是否存在缓存的内存块，如果存在则分配成功，否则找到 tbin 对应的 arena，从 arena 中对应的 bin 中分配 region 保存在 tbin 的 avail 数组中，最终从 availl 数组中选取一个地址进行内存分配，当内存释放时也会将被回收的内存块进行缓存。
- Large：如果请求分配内存的大小大于 arena 中的最小的 bin，但是不大于 tcache 中能够缓存的最大块，依然会通过 tcache 进行分配，但是不同的是此时会分配 chunk 以及所对应的 run，从 chunk 中找到相应的内存空间进行分配。内存释放时也跟 samll 场景类似，会把释放的内存块缓存在 tacache 的 tbin 中。此外还有一种情况，当请求分配内存的大小大于tcache 中能够缓存的最大块，但是不大于 chunk 的大小，那么将不会采用 tcache 机制，直接在 chunk 中进行内存分配。
- Huge：如果请求分配内存的大小大于 chunk 的大小，那么直接通过 mmap 进行分配，调用 munmap 进行回收。





JeMalloc将内存分成多个相同大小的chunk，数据存储在chunks中；每个chunk分为多个run，run负责请求、分配相应大小的内存并记录空闲和使用的regions的大小。

**Arena**
	Arena是JeMalloc的核心分配管理区域，对于多核系统，会默认分配4x逻辑CPU的Arena，线程采取轮询的方式来选择相应的Arena来进行内存分配。
每个arena内都会包含对应的管理信息，记录arena的分配情况。arena都有专属的chunks, 每个chunk的头部都记录chunk的分配信息。在使用某一个chunk的时候，会把chunk分割成多个run，并记录到bin中。不同size class的run属于不同的bin，bin内部使用红黑树来维护空闲的run，run内部使用bitmap来记录分配状态。
JeMalloc使用Buddy allocation 和 Slab allocation 组合作为内存分配算法，使用Buddy allocation将Chunk划分为不同大小的 run，使用 Slab allocation 将run划分为固定大小的 region，大部分内存分配直接查找对应的 run，从中分配空闲的 region，释放则标记region为空闲。

**Chunk**

Chunk是JeMalloc进行内存分配的单位，默认大小4MB。Chunk以Page（默认为4KB)为单位进行管理，每个Chunk的前6个Page用于存储后面其它Page的状态，比如是否待分配还是已经分配；而后面其它Page则用于进行实际的分配。

**Bin**

JeMalloc 中 small size classes 使用 slab 算法分配，会有多种不同大小的run，相同大小的run由bin 进行管理。
run是分配的执行者, 而分配的调度者是bin，bin负责记录当前arena中某一个size class范围内所有non-full run的使用情况。当有分配请求时，arena查找相应size class的bin，找出可用于分配的run，再由run分配region。由于只有small region分配需要run，因此bin也只对应small size class。在arena中， 不同bin管理不同size大小的run，在任意时刻， bin中会针对当前size保存一个run用于内存分配。

**Run**

Run是chunk的一块内存区域，大小是Page的整数倍，由bin进行管理，比如8字节的bin对应的run就只有1个page，可以从里面选取一个8字节的块进行分配。
small classes 从 run 中使用 slab 算法分配，每个 run 对应一块连续的内存，大小为 page size 倍数，划分为相同大小的 region，分配时从run 中分配一个空闲 region，释放时标记region为空闲，重复使用。run中采用bitmap记录分配区域的状态，bitmap能够快速计算出第一块空闲区域，且能很好的保证已分配区域的紧凑型。

**TCache**

TCache是线程的私有缓存空间，在分配内存时首先从tcache中分配，避免加锁；当TCache没有空闲空间时才会进入一般的分配流程。
每个TCache内部有一个arena，arena内部包含tbin数组来缓存不同大小的内存块，但没有run。



## JeMalloc使用指南

### JeMalloc库简介

JeMalloc提供了静态库libjemalloc.a和动态库libjemalloc.so，默认安装在/usr/local/lib目录。

### JeMalloc动态方式

通过-ljemalloc将JeMalloc链接到应用程序。通过LD_PRELOAD预载入JeMalloc库可以不用重新编译应用程序即可使用JeMalloc。
LD_PRELOAD=“/usr/lib/libjemalloc.so”

### JeMalloc静态方式

在编译选项的最后加入/usr/local/lib/libjemalloc.a链接静态库。

### JeMalloc生效

```c
JEMALLOC_EXPORT void (*__free_hook)(void *ptr) = je_free;
JEMALLOC_EXPORT void *(*__malloc_hook)(size_t size) = je_malloc;
JEMALLOC_EXPORT void *(*__realloc_hook)(void *ptr, size_t size) = je_realloc;
```



### 代码测试

```c
#include <stdio.h>
#include <stdlib.h>
#include <malloc.h>

int func_long_name_a();
int func_long_name_b();
int func_long_name_c();

int func_long_name_a(){
    printf("func_long_name_a called\n");
    func_long_name_b();
    return 0;
}

int func_long_name_b(){
    printf("func_long_name_b called\n");
    func_long_name_c();
    return 0;
}

int func_long_name_c(){
    printf("func_long_name_c called\n");
    int sizeArr[] = {1, 4095, 4096, 8192, 8193, 4*1024*1024, 10*1024*1024};
    int i;
    for(i = 0; i < 7; ++i){
        void * p = malloc(sizeArr[i]);
        free(p);
    }

    return 0;
}

int main(int argc, char ** argv)
{
    printf("main called\n");
    func_long_name_a();
    func_long_name_c();
    printf("main exit\n");
    return 0;
}
```

```c
#include <stdlib.h>
#include <jemalloc/jemalloc.h>

void
do_something(size_t i) {
        // Leak some memory.
        malloc(i * 100);
}

int
main(int argc, char **argv) {
        for (size_t i = 0; i < 1000; i++) {
                do_something(i);
        }

        // Dump allocator statistics to stderr.
        malloc_stats_print(NULL, NULL, NULL);

        return 0;
}

```

```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <time.h>
 
#define MAX_OBJECT_NUMBER       (1024)
#define MAX_MEMORY_SIZE         (1024*10)
 
struct BufferUnit{
   int   size;
   char* data;
};
 
struct BufferUnit   buffer_units[MAX_OBJECT_NUMBER];
 
void MallocBuffer(int buffer_size) {
 
for(int i=0; i<MAX_OBJECT_NUMBER; ++i)  {
    if (NULL != buffer_units[i].data)   continue;
 
    buffer_units[i].data = (char*)malloc(buffer_size);
    if (NULL == buffer_units[i].data)  continue;
 
    memset(buffer_units[i].data, 0x01, buffer_size);
    buffer_units[i].size = buffer_size;
    }
}
 
void FreeHalfBuffer(bool left_half_flag) {
    int half_index = MAX_OBJECT_NUMBER / 2;
    int min_index = 0;
    int max_index = MAX_OBJECT_NUMBER-1;
    if  (left_half_flag)
        max_index =  half_index;
    else
        min_index = half_index;
 
    for(int i=min_index; i<=max_index; ++i) {
        if (NULL == buffer_units[i].data) continue;
 
        free(buffer_units[i].data);
        buffer_units[i].data =  NULL;
        buffer_units[i].size = 0;
    }
}
 
int main() {
    memset(&buffer_units, 0x00, sizeof(buffer_units));
    int decrease_buffer_size = MAX_MEMORY_SIZE;
    bool left_half_flag   =   false;
    time_t  start_time = time(0);
    while(1)  {
        MallocBuffer(decrease_buffer_size);
        FreeHalfBuffer(left_half_flag);
        left_half_flag = !left_half_flag;
        --decrease_buffer_size;
        if (0 == decrease_buffer_size) break;
    }
    FreeHalfBuffer(left_half_flag);
    time_t end_time = time(0);
    long elapsed_time = difftime(end_time, start_time);
 
    printf("Used %ld seconds. \n", elapsed_time);
    return 1;
}

```







# tcmalloc

## bazel 安装

使用二进制文件安装

[Releases · bazelbuild/bazel · GitHub](https://github.com/bazelbuild/bazel/releases)

下载如下: 放到 /home/bin 下

![image-20220724233229849](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20220724233229849.png)

安装环境:

> ```
> sudo apt install g++ unzip zip
> ```

如果要编译java， 则需要安装:

> ```
> # Ubuntu 16.04 (LTS) uses OpenJDK 8 by default:
> sudo apt-get install openjdk-8-jdk
> 
> # Ubuntu 18.04 (LTS) uses OpenJDK 11 by default:
> sudo apt-get install openjdk-11-jdk
> ```

Run:

> ```
> chmod +x bazel-<version>-installer-linux-x86_64.sh
> ./bazel-<version>-installer-linux-x86_64.sh --user
> ```

设置环境变量:

> ```
> export PATH="$PATH:$HOME/bin"
> ```



使用bazel编译c++ 工程： [Build Tutorial - C++ - Bazel main](https://docs.bazel.build/versions/main/tutorial/cpp.html)



## 下载编译tcmalloc 

[GitHub - google/tcmalloc](https://github.com/google/tcmalloc)

使用bazel 编译:

> ```
> $ cd tcmalloc
> $ bazel test //tcmalloc/...
> INFO: Analyzed 112 targets (12 packages loaded, 606 targets configured).
> ...
> INFO: Build completed successfully, 827 total actions
> ```

显示编译成功

运行Helloworld

编译tcmalloc/testing:hello_main

> ```
> tcmalloc$ bazel build tcmalloc/testing:hello_main
> Extracting Bazel installation...
> Starting local Bazel server and connecting to it...
> INFO: Analyzed target //tcmalloc/testing:hello_main (31 packages loaded ...
> ...
> INFO: Build completed successfully, 102 total actions
> PASSED in 0.1s
> tcmalloc$
> ```

运行

> ```
> tcmalloc$ bazel run tcmalloc/testing:hello_main
> ...
> INFO: Found 1 target...
> ...
> INFO: Build completed successfully, 1 total action
> Current heap size = 73728 bytes
> hello world!
> new'd 1073741824 bytes at 0x14ea40000000
> Current heap size = 1073816576 bytes
> malloc'd 1073741824 bytes at 0x14eac0000000
> Current heap size = 2147558400 bytes
> $
> ```



## 学习blog

[TCMalloc解密 – Wallen's Blog (wallenwang.com)](https://wallenwang.com/2018/11/tcmalloc/)





# Cuda 学习书籍

CUDA C编程权威指南

CUDA并行程序设计 GPU编程指南

CUDA高性能并行计算 Duane Storti

CUDA并行程序射击：GPU编程 Shane Cook









# Libevent

安装和编译

```
tar -zxvf libevent-2.0.22-stable.tar.gz
cd libevent-2.0.22-stable/
./configure
make
sudo make install

注意

如果在libevent安装目录make之后会生成一个.libs/， 里面如果没有libevent_openssl.so说明系统没有安装openssl库。但是如果安装了，依然没有这个文件生成，可能需要制定openssl路径

ln -s  /usr/local/ssl/include/openssl    /usr/include/openssl
```



**测试**

服务端:

```
./hello-world
```

客户端:

```
netcat 192.168.2.105 9995
```



## 1 evBuffer

libevent 的 evbuffer 实现了为向后面添加数据和从前面移除数据而优化的字节队列。evbuffer 用于处理缓冲网络 IO 的“缓冲”部分。它不提供调度 IO 或者当 IO 就绪时触发 IO 的 功能:这是 bufferevent 的工作。函数都在 event2/buffer.h中声明。

**创建evbuffer**

```c
struct evbuffer *evbuffer_new(void);
void evbuffer_free(struct evbuffer *buf);
```

**向evbuffer中添加数据**

```c++
int evbuffer_add(struct evbuffer *buf, const void *data, size_t datlen);
//这个函数添加 data 处的 datalen 字节到 buf 的末尾,
//成功时返回0,失败时返回-1。

int evbuffer_add_printf(struct evbuffer *buf, const char *fmt, ...)
int evbuffer_add_vprintf(struct evbuffer *buf, const char *fmt, va_list ap);
//这些函数添加格式化的数据到 buf 末尾。
//格式参数和其他参数的处理分别与 C 库函数 printf 和 vprintf 相同。函数返回添加的字节数。

int evbuffer_expand(struct evbuffer *buf, size_t datlen);
//这个函数修改缓冲区的最后一块,或者添加一个新的块,
//使得缓冲区足以容纳 datlen 字节, 而不需要更多的内存分配。
```

**evbuffer数据移动**

```c
int evbuffer_add_buffer(struct evbuffer *dst, struct evbuffer *src);
//evbuffer_add_buffer()将 src 中的所有数据移动到 dst 末尾,成功时返回0,失败时返回-1。
int evbuffer_remove_buffer(struct evbuffer *src, 
                    struct evbuffer *dst,
                    size_t datlen);
//evbuffer_remove_buffer()函数从 src 中移动 datlen 字节到 dst 末尾,尽量少进行复制。如果字节数小于 datlen,所有字节被移动。函数返回移动的字节数。
```



## 2. evconnlistener

evconnlistener 机制提供了监听和接受 TCP 连接的方法。所有函数和类型都在 event2/listener.h 中声明。

**创建和释放evconnlistener**

```c++
struct evconnlistener *evconnlistener_new(struct event_base *base, evconnlistener_cb cb, void *ptr, unsigned flags, int backlog, evutil_socket_t fd);
//evconnlistener_new()函数假定已经将套接字绑定到要监听的端口,然后通过 fd 传入这个套接字。
struct evconnlistener *evconnlistener_new_bind(struct event_base *base, evconnlistener_cb cb, void *ptr, unsigned flags, int backlog,
    const struct sockaddr *sa, int socklen);
//两个 evconnlistener_new*()函数都分配和返回一个新的连接监听器对象。连接监听器使 用 event_base 来得知什么时候在给定的监听套接字上有新的 TCP 连接。新连接到达时,监听 器调用你给出的回调函数。
//两个函数中,base参数都是监听器用于监听连接的 event_base。cb是收到新连接时要调 用的回调函数;
//如果 cb 为 NULL,则监听器是禁用的,直到设置了回调函数为止。
	// typedef void (*evconnlistener_cb)(struct evconnlistener *listener, evutil_socket_t sock, struct sockaddr *addr, int len, void              *ptr);
	// 接收到新连接会调用提供的回调函数 。 ptr是调用evconnlistener_new() 时用户提供的指针。
//ptr 指针将传递给回调函数。
//flags 参数控制回调函数的行为
	// LEV_OPT_LEAVE_SOCKETS_BLOCKING     默认情况下,连接监听器接收新套接字后,会将其设置为非阻塞的
	// LEV_OPT_CLOSE_ON_FREE              如果设置了这个选项,释放连接监听器会关闭底层套接字。
	// LEV_OPT_CLOSE_ON_EXEC              如果设置了这个选项,连接监听器会为底层套接字设置 close-on-exec 标志。
	// LEV_OPT_REUSEABLE                  重用端口
	// LEV_OPT_THREADSAFE                 为监听器分配锁,这样就可以在多个线程中安全地使用了。
//如果要 libevent 分配和绑定套接字,可以调用 evconnlistener_new_bind() ,传输要绑定到的地址和地址长度。

void evconnlistener_free(struct evconnlistener *lev);
//要释放连接监听器,调用 evconnlistener_free()。
```



**启用和禁用evconnlistener**

```c++
int evconnlistener_disable(struct evconnlistener *lev);
int evconnlistener_enable(struct evconnlistener *lev);
```

**设置回调函数**

```c
void evconnlistener_set_cb(struct evconnlistener *lev, evconnlistener_cb cb, void *arg);
```

**获取socket和event_base**

```c
evutil_socket_t evconnlistener_get_fd(struct evconnlistener *lev);
struct event_base *evconnlistener_get_base(struct evconnlistener *lev);
```



## 3. 内存管理回调设置

默认情况下，libevent 使用C 库的内存管理函数在堆上分配内存。通过提供malloc、realloc和free 的替代函数，可以让libevent 使用其他的内存管理器。

希望libevent 使用一个更高效的分配器时；或者希望libevent 使用一个工具分配器，以便检查内存泄漏时，可能需要这样做。

```c
void event_set_mem_functions(void *(*malloc_fn)(size_t sz),
                             void *(*realloc_fn)(void *ptr, size_t sz),
                             void (*free_fn)(void *ptr));
```

可以在禁止event_set_mem_functions 函数的配置下编译libevent 。这时候使用event_set_mem_functions 将不会编译或者链接。



## 4. event_base

***event_base_new()\***函数分配并且返回一个新的具有默认设置的 event_base。函数会检测环境变量,返回一个到 event_base 的指针。如果发生错误,则返回 NULL。选择各种方法时,函数会选择 OS 支持的最快方法。

```c
struct event_base *event_base_new(void);
```

event_config 是一个容纳 event_base 配置信息的不透明结构体。需要 event_base 时,将 event_config 传递给**event_base_new_with_config ()。**

```c
struct event_config *event_config_new(void);
struct event_base *event_base_new_with_config(const struct event_config *cfg);
void event_config_free(struct event_config *cfg);
```



```c
int event_config_avoid_method(struct event_config *cfg, const char *method);
enum event_method_feature {
    EV_FEATURE_ET = 0x01,
    EV_FEATURE_O1 = 0x02,
    EV_FEATURE_FDS = 0x04,
};
int event_config_require_features(struct event_config *cfg,
                                  enum event_method_feature feature);
enum event_base_config_flag {
    EVENT_BASE_FLAG_NOLOCK = 0x01,
    EVENT_BASE_FLAG_IGNORE_ENV = 0x02,
    EVENT_BASE_FLAG_STARTUP_IOCP = 0x04,
    EVENT_BASE_FLAG_NO_CACHE_TIME = 0x08,
    EVENT_BASE_FLAG_EPOLL_USE_CHANGELIST = 0x10,
    EVENT_BASE_FLAG_PRECISE_TIMER = 0x20
};
int event_config_set_flag(struct event_config *cfg,
    enum event_base_config_flag flag);
```

调用 event_config_avoid_method ()可以通过名字让 libevent 避免使用特定的可用后端 。 调用 event_config_require_feature ()让 libevent 不使用不能提供所有指定特征的后端。 调用 event_config_set_flag()让 libevent 在创建 event_base 时设置一个或者多个将在下面介绍的运行时标志。

**event_config_require_features()可识别的特征值有:**

- EV_FEATURE_ET:要求支持边沿触发的后端
- EV_FEATURE_O1:要求添加、删除单个事件,或者确定哪个事件激活的操作是 O(1)复杂度的后端
- EV_FEATURE_FDS:要求支持任意文件描述符,而不仅仅是套接字的后端

**event_config_set_flag()可识别的选项值有:**

- EVENT_BASE_FLAG_NOLOCK :不要为 event_base 分配锁。设置这个选项可以 为 event_base 节省一点用于锁定和解锁的时间,但是让在多个线程中访问 event_base 成为不安全的。
- EVENT*BASE_FLAG_IGNORE_ENV :选择使用的后端时,不要检测 EVENT** 环境 变量。使用这个标志需要三思:这会让用户更难调试你的程序与 libevent 的交互。
- EVENT_BASE_FLAG_STARTUP_IOCP:仅用于 Windows,让 libevent 在启动时就 启用任何必需的 IOCP 分发逻辑,而不是按需启用。
- EVENT_BASE_FLAG_NO_CACHE_TIME :不是在事件循环每次准备执行超时回调时 检测当前时间,而是在每次超时回调后进行检测。注意:这会消耗更多的 CPU时间。
- EVENT_BASE_FLAG_EPOLL_USE_CHANGELIST :告诉 libevent ,如果决定使 用 epoll 后端,可以安全地使用更快的基于 changelist 的后端。epoll-changelist 后端可以 在后端的分发函数调用之间,同样的 fd 多次修改其状态的情况下,避免不必要的系统 调用。但是如果传递任何使用 dup()或者其变体克隆的 fd 给 libevent,epoll-changelist 后端会触发一个内核 bug,导致不正确的结果。在不使用 epoll 后端的情况下,这个标 志是没有效果的。也可以通过设置
- EVENT_EPOLL_USE_CHANGELIST 环境变量来 打开 epoll-changelist 选项。



**有时候需要检查 event_base 支持哪些特征,或者当前使用哪种方法。**

```c
const char **event_get_supported_methods(void);

int i;
const char **methods = event_get_supported_methods();
printf("Starting Libevent %s.  Available methods are:\n",
    event_get_version());
for (i=0; methods[i] != NULL; ++i) {
    printf("    %s\n", methods[i]);
}
```

**释放event_base**

```c
void event_base_free(struct event_base *base);
```



**设置优先级**

```c
int event_base_priority_init(struct event_base *base, int n_priorities);
```

成功时这个函数返回 0,失败时返回 -1。base 是要修改的 event_base,n_priorities 是要支 持的优先级数目,这个数目至少是 1 。每个新的事件可用的优先级将从 0 (最高) 到 n_priorities-1(最低)。常量 EVENT_MAX_PRIORITIES 表示 n_priorities 的上限。

**调用fork()之后需要重新初始化**

```c
int event_reinit(struct event_base *base);
```

## 5. event_loop

一旦有了一个已经注册了某些事件的 event_base, 就需要让 libevent 等待事件并且通知事件的发生。

```c
#define EVLOOP_ONCE             0x01
#define EVLOOP_NONBLOCK         0x02
#define EVLOOP_NO_EXIT_ON_EMPTY 0x04
int event_base_loop(struct event_base *base, int flags);
```

默认情况下,event_base_loop()函数运行 event_base 直到其中没有已经注册的事件为止。执行循环的时候 ,函数重复地检查是否有任何已经注册的事件被触发 (比如说,读事件 的文件描述符已经就绪,可以读取了;或者超时事件的超时时间即将到达 )。如果有事件被触发,函数标记被触发的事件为 “激活的”,并且执行这些事件。

在 flags 参数中设置一个或者多个标志就可以改变 event_base_loop()的行为。如果设置了 EVLOOP_ONCE ,循环将等待某些事件成为激活的 ,执行激活的事件直到没有更多的事件可以执行,然会返回。如果设置了 EVLOOP_NONBLOCK,循环不会等待事件被触发: 循环将仅仅检测是否有事件已经就绪,可以立即触发,如果有,则执行事件的回调。

```c
int event_base_dispatch(struct event_base *base);
//event_base_dispatch ()等同于没有设置标志的 event_base_loop ( )。所以, event_base_dispatch ()将一直运行,直到没有已经注册的事件了,或者调用 了 event_base_loopbreak()或者 event_base_loopexit()为止。
```



**停止循环**

```c
int event_base_loopexit(struct event_base *base, const struct timeval *tv);
//让 event_base 在给定时间之后停止循环。如果 tv 参数为 NULL, event_base 会立即停止循环,没有延时。
//如果 event_base 当前正在执行任何激活事件的回调,则回调会继续运行,直到运行完所有激活事件的回调之才退出。
int event_base_loopbreak(struct event_base *base);
//让 event_base 立即退出循环。它与 event_base_loopexit (base,NULL)的不同在于,如果 event_base 当前正在执行激活事件的回调 ,它将在执行完当前正在处理的事件后立即退出。
```



## 6. event

libevent 的基本操作单元是事件。每个事件代表一组条件的集合,这些条件包括:

- 文件描述符已经就绪,可以读取或者写入
- 文件描述符变为就绪状态,可以读取或者写入(仅对于边沿触发 IO)
- 超时事件
- 发生某信号
- 用户触发事件
  所有事件具有相似的生命周期。调用 libevent 函数设置事件并且关联到 event_base 之后, 事件进入“已初始化(initialized)”状态。此时可以将事件添加到 event_base 中,这使之进入“未决(pending)”状态。在未决状态下,如果触发事件的条件发生(比如说,文件描述 符的状态改变,或者超时时间到达 ),则事件进入“激活(active)”状态,(用户提供的)事件回调函数将被执行。如果配置为“持久的(persistent)”,事件将保持为未决状态。否则, 执行完回调后,事件不再是未决的。删除操作可以让未决事件成为非未决(已初始化)的 ; 添加操作可以让非未决事件再次成为未决的。

**创建事件**

使用 event_new()接口创建事件。 事件处于**已初始化和非未决状态**。

```c
#define EV_TIMEOUT      0x01
//这个标志表示某超时时间流逝后事件成为激活的。构造事件的时候,EV_TIMEOUT 标志是 被忽略的:可以在添加事件的时候设置超时 ,也可以不设置。超时发生时,回调函数的 what 参数将带有这个标志。
#define EV_READ         0x02
//表示指定的文件描述符已经就绪,可以读取的时候,事件将成为激活的。
#define EV_WRITE        0x04
//表示指定的文件描述符已经就绪,可以写入的时候,事件将成为激活的。
#define EV_SIGNAL       0x08
//用于实现信号检测
#define EV_PERSIST      0x10
//表示事件是“持久的”
#define EV_ET           0x20
//表示如果底层的 event_base 后端支持边沿触发事件,则事件应该是边沿触发的。
typedef void (*event_callback_fn)(evutil_socket_t, short, void *);
struct event *event_new(struct event_base *base, evutil_socket_t fd, short what, event_callback_fn cb, void *arg);
void event_free(struct event *event);
```

**添加事件**

构造事件之后,在将其添加到 event_base 之前实际上是不能对其做任何操作的。使用event_add()将事件添加到 event_base。

**事件处于未决的状态。**

```c
int event_add(struct event *ev, const struct timeval *tv);
```

在非未决的事件上调用 event_add()将使其在配置的 event_base 中成为未决的。成功时 函数返回0,失败时返回-1。

如果 tv 为 NULL,添加的事件不会超时。否则, tv 以秒和微秒指定超时值。

如果对已经未决的事件调用 event_add(),事件将保持未决状态,并在指定的超时时间被重新调度。



**删除事件**

**事件处于非未决状态**

```c
int event_del(struct event *ev);
```



**事件的优先级**

多个事件同时触发时,libevent 没有定义各个回调的执行次序。可以使用优先级来定义某些事件比其他事件更重要。

在前一章讨论过,每个 event_base 有与之相关的一个或者多个优先级。在初始化事件之后, 但是在添加到 event_base 之前,可以为其设置优先级。

```c
int event_priority_set(struct event *event, int priority);
```

事件的优先级是一个在 0和 event_base 的优先级减去1之间的数值。成功时函数返回 0,失 败时返回-1。

多个不同优先级的事件同时成为激活的时候 ,低优先级的事件不会运行 。libevent 会执行高优先级的事件,然后重新检查各个事件。只有在没有高优先级的事件是激活的时候 ,低优先级的事件才会运行。

**检查事件的状态**

```c
int event_pending(const struct event *ev, short what, struct timeval *tv_out);
//确定给定的事件是否是未决的或者激活的。如果是,而且 what 参 数设置了 EV_READ、EV_WRITE、EV_SIGNAL 或者 EV_TIMEOUT 等标志,则函数会返回事件当前为之未决或者激活的所有标志 。如果提供了 tv_out 参数,并且 what 参数中设置了 EV_TIMEOUT 标志,而事件当前正因超时事件而未决或者激活,则 tv_out 会返回事件 的超时值。

#define event_get_signal(ev) /* ... */
evutil_socket_t event_get_fd(const struct event *ev);
//返回为事件配置的文件描述符或者信号值。
struct event_base *event_get_base(const struct event *ev);
//返回为事件配置的 event_base
short event_get_events(const struct event *ev);
//返回事件的标志(EV_READ、EV_WRITE 等)。
event_callback_fn event_get_callback(const struct event *ev);
void *event_get_callback_arg(const struct event *ev);
//返回事件的回调函数及其参数指针。
int event_get_priority(const struct event *ev);
//返回事件的优先级
void event_get_assignment(const struct event *event,
        struct event_base **base_out,
        evutil_socket_t *fd_out,
        short *events_out,
        event_callback_fn *callback_out,
        void **arg_out);
//复制所有为事件分配的字段到提供的指针中。任何为 NULL 的参数会被忽略。
```

**触发一次事件**

如果不需要多次添加一个事件,或者要在添加后立即删除事件,而事件又不需要是持久的 , 则可以使用 event_base_once()。

```c
int event_base_once(struct event_base *, evutil_socket_t, short, void (*)(evutil_socket_t, short, void *), void *, const struct timeval *);
```

**手动激活事件**

极少数情况下使用

```c
void event_active(struct event *ev, int what, short ncalls);
```

**事件之间的状态图**

![image-20221217213737272](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20221217213737272.png)



## 7. Bufferevent

很多时候,除了响应事件之外,应用还希望做一定的数据缓冲。比如说,写入数据的时候 ,通常的运行模式是:

- 决定要向连接写入一些数据,把数据放入到缓冲区中
- 等待连接可以写入
- 写入尽量多的数据
- 记住写入了多少数据,如果还有更多数据要写入,等待连接再次可以写入

这种缓冲 IO 模式很通用,libevent 为此提供了一种通用机制,即bufferevent。

bufferevent 由一个底层的传输端口(如套接字 ),一个读取缓冲区和一个写入缓冲区组成。与通常的事件在底层传输端口已经就绪,可以读取或者写入的时候执行回调不同的是,bufferevent 在读取或者写入了足够量的数据之后调用用户提供的回调。

- `基于套接字的 bufferevent`:使用 event_*接口作为后端,通过底层流式套接字发送或者接收数据的 bufferevent
- `异步 IO bufferevent`:使用 Windows IOCP 接口,通过底层流式套接字发送或者接收数据的 bufferevent(仅用于 Windows,试验中)
- `过滤型 bufferevent`:将数据传输到底层 bufferevent 对象之前,处理输入或者输出数据的 bufferevent:比如说,为了压缩或者转换数据。
- `成对的 bufferevent`:相互传输数据的两个 bufferevent。



**回调和水位**

每个 bufferevent 有两个数据相关的回调:一个读取回调和一个写入回调。默认情况下,从底层传输端口读取了任意量的数据之后会调用读取回调 ;

输出缓冲区中足够量的数据被清空到底层传输端口后写入回调会被调用。通过调整 bufferevent 的读取和写入 “水位 (watermarks )”可以覆盖这些函数的默认行为。

每个 bufferevent 有四个水位:

- `读取低水位` :读取操作使得输入缓冲区的数据量在此级别或者更高时 ,读取回调将被调用。默认值为 0,所以每个读取操作都会导致读取回调被调用。
- `读取高水位` :输入缓冲区中的数据量达到此级别后, bufferevent 将停止读取,直到输入缓冲区中足够量的数据被抽取 ,使得数据量低于此级别 。默认值是无限 ,所以永远不会因为输入缓冲区的大小而停止读取。
- `写入低水位` :写入操作使得输出缓冲区的数据量达到或者低于此级别时 ,写入回调将被调用。默认值是 0,所以只有输出缓冲区空的时候才会调用写入回调。
- `写入高水位` :bufferevent 没有直接使用这个水位。它在 bufferevent 用作另外一 个 bufferevent 的底层传输端口时有特殊意义。

```c
typedef void (*bufferevent_data_cb)(struct bufferevent *bev, void *ctx);
typedef void (*bufferevent_event_cb)(struct bufferevent *bev,
    short events, void *ctx);
void bufferevent_setcb(struct bufferevent *bufev,
    bufferevent_data_cb readcb, bufferevent_data_cb writecb,
    bufferevent_event_cb eventcb, void *cbarg);
void bufferevent_getcb(struct bufferevent *bufev,
    bufferevent_data_cb *readcb_ptr,
    bufferevent_data_cb *writecb_ptr,
    bufferevent_event_cb *eventcb_ptr,
    void **cbarg_ptr);
//修改 bufferevent 的一个或者多个回调 。readcb、writecb和eventcb函数将分别在已经读取足够的数据 、已经写入足够的数据 ,或者发生错误时被调用 。

//每个回调函数的第一个参数都是发生了事件的bufferevent ,最后一个参数都是调用bufferevent_setcb()时用户提供的 cbarg 参数:可以通过它向回调传递数据。事件回调 的 events 参数是一个表示事件标志的位掩码:请看前面的 “回调和水位”节。
```

```c
void bufferevent_enable(struct bufferevent *bufev, short events);
void bufferevent_disable(struct bufferevent *bufev, short events);
short bufferevent_get_enabled(struct bufferevent *bufev);
```

可以启用或者禁用 bufferevent 上的 EV_READ、EV_WRITE 或者 EV_READ | EV_WRITE 事件。没有启用读取或者写入事件时, bufferevent 将不会试图进行数据读取或者写入。

没有必要在输出缓冲区空时禁用写入事件: bufferevent 将自动停止写入,然后在有数据等 待写入时重新开始。

类似地,没有必要在输入缓冲区高于高水位时禁用读取事件 :bufferevent 将自动停止读取, 然后在有空间用于读取时重新开始读取。

默认情况下,新创建的 bufferevent 的写入是启用的,但是读取没有启用。可以调用 bufferevent_get_enabled()确定 bufferevent 上当前启用的事件。



```c
void bufferevent_setwatermark(struct bufferevent *bufev, short events,
    size_t lowmark, size_t highmark);
```

bufferevent_setwatermark()函数调整单个 bufferevent 的读取水位、写入水位,或者同时调 整二者。(如果 events 参数设置了 EV_READ,调整读取水位。如果 events 设置了 EV_WRITE 标志,调整写入水位)

对于高水位,0表示“无限”。

**延迟回调**

默认情况下,bufferevent 的回调在相应的条件发生时立即被执行 。(evbuffer 的回调也是这样的,随后会介绍)在依赖关系复杂的情况下 ,这种立即调用会制造麻烦 。比如说,假如某个回调在 evbuffer A 空的时候向其中移入数据 ,而另一个回调在 evbuffer A 满的时候从中取出数据。这些调用都是在栈上发生的,在依赖关系足够复杂的时候,有栈溢出的风险。

要解决此问题,可以请求 bufferevent(或者 evbuffer)延迟其回调。条件满足时,延迟回调不会立即调用,而是在 event_loop()调用中被排队,然后在通常的事件回调之后执行 。

**bufferevent选项标志**

创建 bufferevent 时可以使用一个或者多个标志修改其行为。可识别的标志有:

- BEV_OPT_CLOSE_ON_FREE :释放 bufferevent 时关闭底层传输端口。这将关闭底层套接字,释放底层 bufferevent 等。
- BEV_OPT_THREADSAFE :自动为 bufferevent 分配锁,这样就可以安全地在多个线程中使用 bufferevent。
- BEV_OPT_DEFER_CALLBACKS :设置这个标志时, bufferevent 延迟所有回调,如上所述。
- BEV_OPT_UNLOCK_CALLBACKS :默认情况下,如果设置 bufferevent 为线程安全 的,则 bufferevent 会在调用用户提供的回调时进行锁定。设置这个选项会让 libevent 在执行回调的时候不进行锁定。

**使用bufferevent**

基于套接字的 bufferevent 是最简单的,它使用 libevent 的底层事件机制来检测底层网络套 接字是否已经就绪,可以进行读写操作,并且使用底层网络调用(如 readv 、 writev 、 WSASend、WSARecv)来发送和接收数据。

```c
struct bufferevent *bufferevent_socket_new(
    struct event_base *base,
    evutil_socket_t fd,
    enum bufferevent_options options);
```

base 是 event_base,options 是表示 bufferevent 选项(BEV_OPT_CLOSE_ON_FREE 等) 的位掩码, fd是一个可选的表示套接字的文件描述符。如果想以后设置文件描述符,可以设置fd为-1。成功时函数返回一个 bufferevent,失败则返回 NULL。

```c
int bufferevent_socket_connect(struct bufferevent *bev, struct sockaddr *address, int addrlen);
//在bufferevent上启动链接
//address 和 addrlen 参数跟标准调用 connect()的参数相同。如果还没有为 bufferevent 设置套接字,调用函数将为其分配一个新的流套接字,并且设置为非阻塞的。
//如果已经为 bufferevent 设置套接字,调用bufferevent_socket_connect() 将告知 libevent 套接字还未连接,直到连接成功之前不应该对其进行读取或者写入操作。
//连接完成之前可以向输出缓冲区添加数据。
```

**释放bufferevent**

```c
void bufferevent_free(struct bufferevent *bev);
```

如果设置了 BEV_OPT_CLOSE_ON_FREE 标志,并且 bufferevent 有一个套接字或者底层 bufferevent 作为其传输端口,则释放 bufferevent 将关闭这个传输端口。



**操作bufferevent中的数据**

**通过bufferevent 得到evbuffer**

```c
struct evbuffer *bufferevent_get_input(struct bufferevent *bufev);
struct evbuffer *bufferevent_get_output(struct bufferevent *bufev);
```

它们分别返回输入和输出缓冲区。如果写入操作因为数据量太少而停止(或者读取操作因为太多数据而停止 ),则向输出缓冲 区添加数据(或者从输入缓冲区移除数据)将自动重启操作。

**向bufferevent的输出缓冲区添加数据**

```c
int bufferevent_write(struct bufferevent *bufev,
    const void *data, size_t size);
int bufferevent_write_buffer(struct bufferevent *bufev,
    struct evbuffer *buf);
//这些函数向 bufferevent 的输出缓冲区添加数据。 bufferevent_write()将内存中从 data 处开 始的 size 字节数据添加到输出缓冲区的末尾 。bufferevent_write_buffer()移除 buf 的所有内 容,将其放置到输出缓冲区的末尾。
```

**向bufferevent的输入缓冲区移除数据**

```c
size_t bufferevent_read(struct bufferevent *bufev, void *data, size_t size);
int bufferevent_read_buffer(struct bufferevent *bufev,
    struct evbuffer *buf);
//这些函数从 bufferevent 的输入缓冲区移除数据。bufferevent_read()至多从输入缓冲区移除 size 字节的数据,将其存储到内存中 data 处。函数返回实际移除的字节数。 bufferevent_read_buffer()函数抽空输入缓冲区的所有内容,将其放置到 buf 中, 对于 bufferevent_read(),data 处的内存块必须有足够的空间容纳 size 字节数据。
```

**清空缓冲区**

```c
int bufferevent_flush(struct bufferevent *bufev,
    short iotype, enum bufferevent_flush_mode state);
```

清空 bufferevent 要求 bufferevent 强制从底层传输端口读取或者写入尽可能多的数据 ,而忽略其他可能保持数据不被写入的限制条件 。函数的细节功能依赖于 bufferevent 的具体类型。

# Muduo <一>

## 1. 编译和安装

git: [GitHub - chenshuo/muduo: Event-driven network library for multi-threaded Linux server in C++11](https://github.com/chenshuo/muduo/tree/master)

git clone https://github.com/chenshuo/muduo.git

安装依赖库:

> sudo apt install g++ cmake make libboost-dev 
>
> 可安装三个非必须的依赖库curl, c-ares DNS, google protobuf
>
> sudo apt install  libcur14-openssl-dev libc-ares-dev
>
> sudo apt install protobuf-compiler libprotobuf-dev

编译:

> cd muduo
>
> ./build.sh -j2  
>
> 编译muduo库和它的自带的例子，生成的可执行文件和静态库文件分别位于../build/debug/{bin, lib}
>
> ./build.sh install 
>
> 将muduo头文件和库文件安装到 ../build/debug-install/{include, lib}, 
>
> release 版本:
>
> BUILD_TYPE=release ./build.sh -j2  
>
> BUILD_TYPE=release ./build.sh install 

**如何在自己的程序中使用muduo**

muduo是静态链接的c++程序库， 使用muduo的时候，只需要设置好头文件的路径和库文件路径并链接相应的静态库文件(0lmuduo_net -lmuduo_base)即可.

muduo里面也提供了指导如何把自己的项目包含进自己的工程。[Tutorial of Muduo network library](https://github.com/chenshuo/muduo-tutorial)



## 2  muduo 的架构和概念

[muduo](https://github.com/chenshuo/muduo/tree/cpp11) 是陈硕开发的tcp网络编程库， 是支持非阻塞IO + one event loop peer thread, 不支持阻塞IO.  支持线程安全， 支持多核多线程， 只能用于linux, 不支持跨平台， 作为一款tcp网络库，还是很优秀的，里面有很多值得学习的地方， 这里我将分若干个章节剖析里面的源代码。

muduo 中类的职责和概念划分的非常清晰，在《Linux 多线程服务器端编程》一书的 6.3.1 章节有详细的介绍。实际上目前很多网络库的接口设计也都受到了 muduo 的影响，例如 360 的 evpp 等。

而 muduo 的整体风格受到 Netty 的影响，整个架构依照 Reactor 模式，基本与如下图所示相符：

![image-20230406203115289](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230406203115289.png)

所谓 Reactor 模式，是指有一个循环的过程，不断监听对应事件是否触发，事件触发时调用对应的 callback 进行处理。这里的事件在 muduo 中包括 Socket 可读写事件、定时器事件。负责事件循环的部分在 muduo 被命名为 `EventLoop`， 主要用于相应计时器和IO事件。muduo采用基于对象而非面向对象的设计风格，事件回调大多以boost::function + std::bind 表达， 用户在使用时不需要继承其中的class. 网络库的核心位于muduo/net 和muduo/net/poller

> .
> ├── **Acceptor.cc**                                              接收器，用于服务端接收连接
> ├── Acceptor.h                           
> ├── boilerplate.cc
> ├── boilerplate.h
> ├── **Buffer.cc**                                                  缓冲区，非阻塞IO必备 
> ├── Buffer.h
> ├── Callbacks.h
> ├── **Channel.cc**                                             用于每个socket连接的事件分发
> ├── Channel.h
> ├── CMakeLists.txt
> ├── **Connector.cc**                                          连接器， 用于客户端发起连接
> ├── Connector.h
> ├── Endian.h                                                 网络字节序与本机字节序的转换
> ├── **EventLoop.cc**                                        事件分发器
> ├── EventLoop.h
> ├── **EventLoopThread.cc**                            新建一个专门用于Eventloop的线程
> ├── EventLoopThread.h
> ├── **EventLoopThreadPool.cc**                    muduo默认的多线程IO模型
> ├── EventLoopThreadPool.h
> ├── **InetAddress.cc**                                   IP 地址的简单封装
> ├── InetAddress.h
> ├── **poller**                                                    IO multiplexing 的实现
> │   ├── DefaultPoller.cc
> │   ├── EPollPoller.cc
> │   ├── EPollPoller.h
> │   ├── PollPoller.cc
> │   └── PollPoller.h
> ├── **Poller.cc**                                                IO multiplexing 的基类接口
> ├── Poller.h
> ├── **Socket.cc**                                          封装sockets描述符，负责关闭连接
> ├── Socket.h
> ├── **SocketsOps.cc**                              封装底层的socket的API
> ├── SocketsOps.h
> ├── **TcpClient.cc**                                   TCP客户端
> ├── TcpClient.h
> ├── **TcpConnection.cc**                     负责IO连接的事件
> ├── TcpConnection.h
> ├── **TcpServer.cc**                              TCP服务端
> ├── TcpServer.h
> ├── Timer.cc                                     定时器
> ├── Timer.h
> ├── TimerId.h
> ├── TimerQueue.cc
> ├── TimerQueue.h
> └── ZlibStream.h



对于使用muduo的头文件，只需要掌握5个关键类: Buffer, EventLoop, TcpConnection, TcpClient, TcpServer。 这篇只做大体简单介绍各个模块的作用，每个模块我会单独来讲。

**公开接口:**

- **Buffer** ： 数据的读写需要通过ibuffer进行， 用户代码不需要调用read/write, 只需要处理收到的数据和准备好要发送的数据。
- **InetAddress：**封装IPv4/IPv6地址, 它不能解析域名，只认IP地址。 因为直接用`gethostbyname`解析域名会阻塞IO线程。
- **EventLoop：** 事件循环，每个线程只能有一个EventLoop实体，它负责IO和定时器事件的分派。它用eventfd来异步唤醒， 用TimeQueue作为计时器管理， 用Poller作为IO multiplexing.
- **EventLoopThread** 启动一个线程，在其中运行EventLoop::loop()
- **Tcpconnection** 是封装了TCP的连接， 它不能发起连接
- **TCPClient** 用于编写网络客户端，能发起连接，并且由重试功能
- **TCPServer** 用于编写网络服务器，接收客户的连接。

在这些类中，`Tcpconnection`的生命期依靠`shared_ptr` 管理(由用户和库共同控制), Buffer的生命期由TcpConnection控制。 其余类的生命期是由用户控制。 

内部接口:

-  **Channel**是 负责注册和响应IO事件，注意它不拥有fd。 它是Acceptor, Connector, EventLoop, TimeQueue, TcpConnection的成员，生命期由后者控制。
- **Socket** 是一个RAII handle, 控制一个fd, 并在析构的时候关闭fd, 它是Acceptor, TcpConnection的成员，生命期由后者控制。
- **SocketOps**封装各种Sockets系统调用。
- **Poller**是PollPoller和EPollPoller的基类，它是EventLoop的成员，生命期由后者控制。
- **Connector** 用于发起连接
- **Acceptor** 用于接收连接
- **TimeQueue** 用timerfd实现定时， 是EventLoop的成员
- **EventLoopThreadPool**用于创建IO线程池， 用于把TcpConnection分派到某个EventLoop的线程上。它是TcpServer的成员，生命期由后者控制。

![image-20230406225449143](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230406225449143.png)	

 



## 3 TCP的网络编程论

muduo里面实现是基于事件的非阻塞网络编程流式， 把原来的"**主动调用recv来接收数据， 主动调用accept来接收连接，主动调用send来发送数据**"的思路换成 " **注册一个接收/发送数据的回调， 网络库收到数据/接收到数据会调用我， 直接把数据给我，供我消费。需要发送数据的时候，只管往连接中写，网络库会负责无阻塞地发送**"， 同时整个过程中应该避免阻塞。

muduo里面认为：TCP最主要是处理桑三个半事件。

1. 连接的建立
2. 连接的断开：包括主动断开和被动断开
3. 消息到达，文件描述符可读。
4. 消息发送完毕。这个算半个事件。这里的"发送完毕"是指将数据写入操作系统的缓冲区，将由TCP协议栈负责数据的发送和重传，不代表对方已经收到数据。

我们接下来分析下 muduo 是怎么处理和实现这三个半事件的。

我们来看一个最简单的示例， 以一个echo的回显。

```c++
#include "examples/simple/echo/echo.h"

#include "muduo/base/Logging.h"
#include "muduo/net/EventLoop.h"

#include <unistd.h>

// using namespace muduo;
// using namespace muduo::net;

int main()
{
  LOG_INFO << "pid = " << getpid();
  muduo::net::EventLoop loop;
  muduo::net::InetAddress listenAddr(2007);
  EchoServer server(&loop, listenAddr);
  server.start();
  loop.loop();
}
```

echo-server 的代码量非常简洁。一个典型的 muduo 的 TcpServer 工作流程如下：

1. 建立一个事件循环器 EventLoop
2. 建立对应的业务服务器 TcpServer
3. 设置 TcpServer 的 Callback
4. 启动 server
5. 开启事件循环

### 连接的建立

在我们单纯使用 linux 的 API，编写一个简单的 Tcp 服务器时，建立一个新的连接通常需要四步：

> 步骤 1. socket() // 调用 socket 函数建立监听 socket
> 步骤 2. bind() // 绑定地址和端口
> 步骤 3. listen() // 开始监听端口
> 步骤 4. accept() // 返回新建立连接的 fd

**<一> EventLoop的构造**

```c++
EventLoop::EventLoop()
  : looping_(false),
    quit_(false),
    eventHandling_(false),
    callingPendingFunctors_(false),
    iteration_(0),
    threadId_(CurrentThread::tid()),
    poller_(Poller::newDefaultPoller(this)),
    timerQueue_(new TimerQueue(this)),
    wakeupFd_(createEventfd()),
    wakeupChannel_(new Channel(this, wakeupFd_)),
    currentActiveChannel_(NULL)
{
  LOG_DEBUG << "EventLoop created " << this << " in thread " << threadId_;
  if (t_loopInThisThread)
  {
    LOG_FATAL << "Another EventLoop " << t_loopInThisThread
              << " exists in this thread " << threadId_;
  }
  else
  {
    t_loopInThisThread = this;
  }
  wakeupChannel_->setReadCallback(
      std::bind(&EventLoop::handleRead, this));
  // we are always reading the wakeupfd
  wakeupChannel_->enableReading();
}
```

1.  首先构造一个EventLoop的对象， 在EventLoop里面， 我们new 一个poller_的对象， newDefaultPoller是一个静态函数，通过环境变量来选择是Epoll还是Poll。

```c++
Poller* Poller::newDefaultPoller(EventLoop* loop)
{
  if (::getenv("MUDUO_USE_POLL"))
  {
    return new PollPoller(loop);
  }
  else
  {
    return new EPollPoller(loop);
  }
}
```

​	2.  其次，new一个timerQueue_的对象.

```c++
TimerQueue::TimerQueue(EventLoop* loop)
  : loop_(loop),
    timerfd_(createTimerfd()),
    timerfdChannel_(loop, timerfd_),
    timers_(),
    callingExpiredTimers_(false)
{
  timerfdChannel_.setReadCallback(
      std::bind(&TimerQueue::handleRead, this));
  // we are always reading the timerfd, we disarm it with timerfd_settime.
  timerfdChannel_.enableReading();
}
```

​	2.1 创建一个用于定时器事件的fd, 使用`timerfd_create(CLOCK_MONOTONIC, TFD_NONBLOCK | TFD_CLOEXEC);`, 包含头文件` #include <sys/timerfd.h>`， 返回fd给timerfd_。 

```c++
int createTimerfd()
{
  int timerfd = ::timerfd_create(CLOCK_MONOTONIC,
                                 TFD_NONBLOCK | TFD_CLOEXEC);
  if (timerfd < 0)
  {
    LOG_SYSFATAL << "Failed in timerfd_create";
  }
  return timerfd;
}
```

​	2.2 把定时器事件的fd传给`channel`,  由`channel` 作管理

```c++
Channel::Channel(EventLoop* loop, int fd__)
  : loop_(loop),
    fd_(fd__),
    events_(0),
    revents_(0),
    index_(-1),
    logHup_(true),
    tied_(false),
    eventHandling_(false),
    addedToLoop_(false)
{
}
```

​	2.3 同时在TimerQueue里面绑定读IO的callback,  并且enableReading(),  在这里我们还没有调用等待事件的函数。再继续往下。

```c++
  timerfdChannel_.setReadCallback(
      std::bind(&TimerQueue::handleRead, this));
  // we are always reading the timerfd, we disarm it with timerfd_settime.
  timerfdChannel_.enableReading();

//....
void TimerQueue::handleRead()
{
  loop_->assertInLoopThread();
  Timestamp now(Timestamp::now());
  readTimerfd(timerfd_, now);

  std::vector<Entry> expired = getExpired(now);

  callingExpiredTimers_ = true;
  cancelingTimers_.clear();
  // safe to callback outside critical section
  for (const Entry& it : expired)
  {
    it.second->run();
  }
  callingExpiredTimers_ = false;

  reset(expired, now);
}
```

3. 创建一个`eventfd` 用于唤醒， 并将其可读事件注册到 EventLoop 中，和timerfd的流程一样。至此， EventLoop的构造工作就完成了。

```c++
int createEventfd()
{
  int evtfd = ::eventfd(0, EFD_NONBLOCK | EFD_CLOEXEC);
  if (evtfd < 0)
  {
    LOG_SYSERR << "Failed in eventfd";
    abort();
  }
  return evtfd;
}
```

**<二> InetAddress **

这一步很简单，把主机字节序的ip port转成网络字节序的ip port, 这里还支持ipv6, 默认为ipv4。

```c++
InetAddress::InetAddress(uint16_t portArg, bool loopbackOnly, bool ipv6)
{
  static_assert(offsetof(InetAddress, addr6_) == 0, "addr6_ offset 0");
  static_assert(offsetof(InetAddress, addr_) == 0, "addr_ offset 0");
  if (ipv6)
  {
    memZero(&addr6_, sizeof addr6_);
    addr6_.sin6_family = AF_INET6;
    in6_addr ip = loopbackOnly ? in6addr_loopback : in6addr_any;
    addr6_.sin6_addr = ip;
    addr6_.sin6_port = sockets::hostToNetwork16(portArg);
  }
  else
  {
    memZero(&addr_, sizeof addr_);
    addr_.sin_family = AF_INET;
    in_addr_t ip = loopbackOnly ? kInaddrLoopback : kInaddrAny;
    addr_.sin_addr.s_addr = sockets::hostToNetwork32(ip);
    addr_.sin_port = sockets::hostToNetwork16(portArg);
  }
}
```

这里有个小技巧， 结构体addr6_ 和 addr_是union类型的，使用可以稍微节省点内存。

```c++
  union
  {
    struct sockaddr_in addr_;
    struct sockaddr_in6 addr6_;
  };
```

使用`offsetof` 来判断结构成员的偏移

```
offsetof(InetAddress, addr6_)
offsetof(InetAddress, addr_)
```



**<三> TcpServer的构造**

首先在 TcpServer 对象构建时，TcpServer 的属性 acceptor 同时也被建立。在 Acceptor 的构造函数中分别调用了 socket 函数和 bind 函数完成了 **步骤 1**和**步骤 2**。即，当 `TcpServer server(&loop, listenAddr)` 执行结束时，监听 socket 已经建立好，并已绑定到对应地址和端口了。并设置connect的时候回调和发送message时候的回调。

```c++
EchoServer::EchoServer(muduo::net::EventLoop* loop,
                       const muduo::net::InetAddress& listenAddr)
  : server_(loop, listenAddr, "EchoServer")
{
  server_.setConnectionCallback(
      std::bind(&EchoServer::onConnection, this, _1));
  server_.setMessageCallback(
      std::bind(&EchoServer::onMessage, this, _1, _2, _3));
}
```

```c++
TcpServer::TcpServer(EventLoop* loop,
                     const InetAddress& listenAddr,
                     const string& nameArg,
                     Option option)
  : loop_(CHECK_NOTNULL(loop)),
    ipPort_(listenAddr.toIpPort()),
    name_(nameArg),
    acceptor_(new Acceptor(loop, listenAddr, option == kReusePort)),
    threadPool_(new EventLoopThreadPool(loop, name_)),
    connectionCallback_(defaultConnectionCallback),
    messageCallback_(defaultMessageCallback),
    nextConnId_(1)
{
  acceptor_->setNewConnectionCallback(
      std::bind(&TcpServer::newConnection, this, _1, _2));
}
```

这里我们new一个Accept, 主要把ip 传进去。

```c++
Acceptor::Acceptor(EventLoop* loop, const InetAddress& listenAddr, bool reuseport)
  : loop_(loop),
    acceptSocket_(sockets::createNonblockingOrDie(listenAddr.family())),
    acceptChannel_(loop, acceptSocket_.fd()),
    listening_(false),
    idleFd_(::open("/dev/null", O_RDONLY | O_CLOEXEC))
{
  assert(idleFd_ >= 0);
  acceptSocket_.setReuseAddr(true);
  acceptSocket_.setReusePort(reuseport);
  acceptSocket_.bindAddress(listenAddr);
  acceptChannel_.setReadCallback(
      std::bind(&Acceptor::handleRead, this));
}
```

调用`sockets::createNonblockingOrDie(listenAddr.family())` 创建socket, 再把fd交给acceptChannel_ 管理， 同时设置socket的状态， 并绑定ip port. 

```c++
int sockets::createNonblockingOrDie(sa_family_t family)
{
#if VALGRIND
  int sockfd = ::socket(family, SOCK_STREAM, IPPROTO_TCP);
  if (sockfd < 0)
  {
    LOG_SYSFATAL << "sockets::createNonblockingOrDie";
  }

  setNonBlockAndCloseOnExec(sockfd);
#else
  int sockfd = ::socket(family, SOCK_STREAM | SOCK_NONBLOCK | SOCK_CLOEXEC, IPPROTO_TCP);
  if (sockfd < 0)
  {
    LOG_SYSFATAL << "sockets::createNonblockingOrDie";
  }
#endif
  return sockfd;
}
```

```c++
void Socket::setReuseAddr(bool on)
{
  int optval = on ? 1 : 0;
  ::setsockopt(sockfd_, SOL_SOCKET, SO_REUSEADDR,
               &optval, static_cast<socklen_t>(sizeof optval));
  // FIXME CHECK
}

void Socket::setReusePort(bool on)
{
#ifdef SO_REUSEPORT
  int optval = on ? 1 : 0;
  int ret = ::setsockopt(sockfd_, SOL_SOCKET, SO_REUSEPORT,
                         &optval, static_cast<socklen_t>(sizeof optval));
  if (ret < 0 && on)
  {
    LOG_SYSERR << "SO_REUSEPORT failed.";
  }
#else
  if (on)
  {
    LOG_ERROR << "SO_REUSEPORT is not supported.";
  }
#endif
}

void sockets::bindOrDie(int sockfd, const struct sockaddr* addr)
{
  int ret = ::bind(sockfd, addr, static_cast<socklen_t>(sizeof(struct sockaddr_in6)));
  if (ret < 0)
  {
    LOG_SYSFATAL << "sockets::bindOrDie";
  }
}
```

而当执行 `server.start()` 时，主要做了两个工作：

1. 在监听 socket 上启动 listen 函数，也就是 **步骤 3**；
2. 将监听 socket 的可读事件注册到 EventLoop 中。

```c++
void TcpServer::start()
{
  if (started_.getAndSet(1) == 0)
  {
    threadPool_->start(threadInitCallback_);

    assert(!acceptor_->listening());
    loop_->runInLoop(
        std::bind(&Acceptor::listen, get_pointer(acceptor_)));
  }
}
```

```c++
void Acceptor::listen()
{
  loop_->assertInLoopThread();
  listening_ = true;
  acceptSocket_.listen();
  acceptChannel_.enableReading();
}
```



此时，程序已完成对socket的监听，但还不够，因为此时程序的主角 `EventLoop` 尚未启动。当调用 `loop.loop()` 时，程序开始循环监听该 socket 的可读事件。

```c++
void EventLoop::loop()
{
  assert(!looping_);
  assertInLoopThread();
  looping_ = true;
  quit_ = false;  // FIXME: what if someone calls quit() before loop() ?
  LOG_TRACE << "EventLoop " << this << " start looping";

  while (!quit_)
  {
    activeChannels_.clear();
    pollReturnTime_ = poller_->poll(kPollTimeMs, &activeChannels_);
    ++iteration_;
    if (Logger::logLevel() <= Logger::TRACE)
    {
      printActiveChannels();
    }
    // TODO sort channel by priority
    eventHandling_ = true;
    for (Channel* channel : activeChannels_)
    {
      currentActiveChannel_ = channel;
      currentActiveChannel_->handleEvent(pollReturnTime_);
    }
    currentActiveChannel_ = NULL;
    eventHandling_ = false;
    doPendingFunctors();
  }

  LOG_TRACE << "EventLoop " << this << " stop looping";
  looping_ = false;
}
```

我们可以看到， loop会一直再循环里面从epoll/poll 读取`activeChannels_` ， 对于那些已经active的事件， 我们调用handleEvent(), 再handleEventWithGuard调用各种已经注册事件的callback。

```c++
void Channel::handleEvent(Timestamp receiveTime)
{
  std::shared_ptr<void> guard;
  if (tied_)
  {
    guard = tie_.lock();
    if (guard)
    {
      handleEventWithGuard(receiveTime);
    }
  }
  else
  {
    handleEventWithGuard(receiveTime);
  }
}

void Channel::handleEventWithGuard(Timestamp receiveTime)
{
  eventHandling_ = true;
  LOG_TRACE << reventsToString();
  if ((revents_ & POLLHUP) && !(revents_ & POLLIN))
  {
    if (logHup_)
    {
      LOG_WARN << "fd = " << fd_ << " Channel::handle_event() POLLHUP";
    }
    if (closeCallback_) closeCallback_();
  }

  if (revents_ & POLLNVAL)
  {
    LOG_WARN << "fd = " << fd_ << " Channel::handle_event() POLLNVAL";
  }

  if (revents_ & (POLLERR | POLLNVAL))
  {
    if (errorCallback_) errorCallback_();
  }
  if (revents_ & (POLLIN | POLLPRI | POLLRDHUP))
  {
    if (readCallback_) readCallback_(receiveTime);
  }
  if (revents_ & POLLOUT)
  {
    if (writeCallback_) writeCallback_();
  }
  eventHandling_ = false;
}
```

当新连接请求建立时，可读事件触发，该事件的 callback 实际上就是 Acceptor::handleRead() 方法。

在 Acceptor::handleRead() 方法中，做了三件事：

1. 调用了 accept 函数，完成了 **步骤 4**，实现了连接的建立。得到一个已连接 socket 的 fd。
2. 创建 TcpConnection 对象。
3. 将已连接 socket 的可读事件注册到 EventLoop 中。

再handleRead里面会调用我们再TcpServer设置的回调函数。

`acceptor_->setNewConnectionCallback(std::bind(&TcpServer::newConnection, this, _1, _2));` 调用 `TcpServer::newConnection`

```c++
void Acceptor::handleRead()
{
  loop_->assertInLoopThread();
  InetAddress peerAddr;
  //FIXME loop until no more
  int connfd = acceptSocket_.accept(&peerAddr);
  if (connfd >= 0)
  {
    // string hostport = peerAddr.toIpPort();
    // LOG_TRACE << "Accepts of " << hostport;
    if (newConnectionCallback_)
    {
      newConnectionCallback_(connfd, peerAddr);
    }
    else
    {
      sockets::close(connfd);
    }
  }
  else
  {
    LOG_SYSERR << "in Acceptor::handleRead";
    // Read the section named "The special problem of
    // accept()ing when you can't" in libev's doc.
    // By Marc Lehmann, author of libev.
    if (errno == EMFILE)
    {
      ::close(idleFd_);
      idleFd_ = ::accept(acceptSocket_.fd(), NULL, NULL);
      ::close(idleFd_);
      idleFd_ = ::open("/dev/null", O_RDONLY | O_CLOEXEC);
    }
  }
}
```



```c++
void TcpServer::newConnection(int sockfd, const InetAddress& peerAddr)
{
  loop_->assertInLoopThread();
  EventLoop* ioLoop = threadPool_->getNextLoop();
  char buf[64];
  snprintf(buf, sizeof buf, "-%s#%d", ipPort_.c_str(), nextConnId_);
  ++nextConnId_;
  string connName = name_ + buf;

  LOG_INFO << "TcpServer::newConnection [" << name_
           << "] - new connection [" << connName
           << "] from " << peerAddr.toIpPort();
  InetAddress localAddr(sockets::getLocalAddr(sockfd));
  // FIXME poll with zero timeout to double confirm the new connection
  // FIXME use make_shared if necessary
  TcpConnectionPtr conn(new TcpConnection(ioLoop,
                                          connName,
                                          sockfd,
                                          localAddr,
                                          peerAddr));
  connections_[connName] = conn;
  conn->setConnectionCallback(connectionCallback_);
  conn->setMessageCallback(messageCallback_);
  conn->setWriteCompleteCallback(writeCompleteCallback_);
  conn->setCloseCallback(
      std::bind(&TcpServer::removeConnection, this, _1)); // FIXME: unsafe
  ioLoop->runInLoop(std::bind(&TcpConnection::connectEstablished, conn));
}
```

最后，会调用TcpConnection::connectEstablished， 调用connectionCallback_(), 也就是我们再TcpServer上设置的回调。

```c++
void TcpConnection::connectEstablished()
{
  loop_->assertInLoopThread();
  assert(state_ == kConnecting);
  setState(kConnected);
  channel_->tie(shared_from_this());
  channel_->enableReading();

  connectionCallback_(shared_from_this());
}
//connectionCallback_ 调用的是
 server_.setConnectionCallback(std::bind(&EchoServer::onConnection, this, _1));
```

这里还有一个需要注意的点，创建的 TcpConnnection 对象是个 shared_ptr，该对象会被保存在 TcpServer 的 connections 中。这样才能保证引用计数大于 0，对象不被释放。

至此，一个新的连接已完全建立好，该连接的socket可读事件也已注册到 EventLoop 中了。



###  消息的读取

在新连接建立的时候，会将新连接的 socket 的可读事件注册到 EventLoop 中。假如客户端发送消息，导致已连接 socket 的可读事件触发，该事件对应的 callback 同样也会在 EventLoop::loop() 中被调用。 这个和上面的步骤是一样的。

再TcpServer里面通过调用`setMessageCallback` 和 `setConnectionCallback`。

```c++
  void setConnectionCallback(const ConnectionCallback& cb)
  { connectionCallback_ = cb; }

  /// Set message callback.
  /// Not thread safe.
  void setMessageCallback(const MessageCallback& cb)
  { messageCallback_ = cb; }
```



上一节我们说到，在`Acceptor::handleRead()`方法中，我们在第二步创建了TcpConnection的对象，在TcpConnection中， 我们设置了一系类，read, write, close, error的call back函数。这些事件的触发都在EventLoop::loop() 里面， 当有可读事件触发时，我们调用了`TcpConnection::handleRead()` 函数。

```c++
TcpConnection::TcpConnection(EventLoop* loop,
                             const string& nameArg,
                             int sockfd,
                             const InetAddress& localAddr,
                             const InetAddress& peerAddr)
  : loop_(CHECK_NOTNULL(loop)),
    name_(nameArg),
    state_(kConnecting),
    reading_(true),
    socket_(new Socket(sockfd)),
    channel_(new Channel(loop, sockfd)),
    localAddr_(localAddr),
    peerAddr_(peerAddr),
    highWaterMark_(64*1024*1024)
{
  channel_->setReadCallback(
      std::bind(&TcpConnection::handleRead, this, _1));
  channel_->setWriteCallback(
      std::bind(&TcpConnection::handleWrite, this));
  channel_->setCloseCallback(
      std::bind(&TcpConnection::handleClose, this));
  channel_->setErrorCallback(
      std::bind(&TcpConnection::handleError, this));
  LOG_DEBUG << "TcpConnection::ctor[" <<  name_ << "] at " << this
            << " fd=" << sockfd;
  socket_->setKeepAlive(true);
}
```


在 TcpConnection::handleRead 方法中，主要做了两件事：

1. 从 socket 中读取数据，并将其放入 inputbuffer 中
2. 调用 messageCallback，执行业务逻辑。

messageCallback 是在建立新连接时，将 `TcpServer::messageCallback` 方法 bind 到了 `TcpConnection::messageCallback` 的方法。

`TcpServer::messageCallback` 就是业务逻辑的主要实现函数。通常情况下，我们可以在里面实现消息的编解码、消息的分发等工作，这里就不再深入探讨了。

```c++
void TcpConnection::handleRead(Timestamp receiveTime)
{
  loop_->assertInLoopThread();
  int savedErrno = 0;
  ssize_t n = inputBuffer_.readFd(channel_->fd(), &savedErrno);
  if (n > 0)
  {
    messageCallback_(shared_from_this(), &inputBuffer_, receiveTime);
  }
  else if (n == 0)
  {
    handleClose();      // 断开处理
  }
  else
  {
    errno = savedErrno;
    LOG_SYSERR << "TcpConnection::handleRead";
    handleError();
  }
}
```

在我们上面给出的示例代码中，echo-server 的 messageCallback 非常简单，就是直接将得到的数据，重新 send 回去。在实际的业务处理中，一般都会调用 TcpConnection::send() 方法，给客户端回复消息。

这里需要注意的是，在 messageCallback 中，用户会有可能会把任务抛给自定义的 Worker 线程池处理。
但是这个在 Worker 线程池中任务，**切忌直接对 Buffer 的操作**。因为 Buffer 并不是线程安全的。

我们需要记住一个准则:

> **所有对 IO 和 buffer 的读写，都应该在 IO 线程中完成。**

一般情况下，先在交给 Worker 线程池之前，应该现在 IO 线程中把 Buffer 进行切分解包等动作。将解包后的消息交由线程池处理，避免多个线程操作同一个资源

### 消息的发送

用户通过调用 TcpConnection::send() 向客户端回复消息。由于 muduo 中使用了 OutputBuffer，因此消息的发送过程比较复杂。

首先需要注意的是线程安全问题, 上文说到对于消息的读写必须都在 EventLoop 的同一个线程 (通常称为 IO 线程) 中进行：
因此，TcpConnection::send 必须要保证线程安全性，它是这么做的：

```c++
void TcpConnection::send(Buffer* buf)
{
  if (state_ == kConnected)
  {
    if (loop_->isInLoopThread())
    {
      sendInLoop(buf->peek(), buf->readableBytes());
      buf->retrieveAll();
    }
    else
    {
      void (TcpConnection::*fp)(const StringPiece& message) = &TcpConnection::sendInLoop;
      loop_->runInLoop(
          std::bind(fp, this,     // FIXME
                    buf->retrieveAllAsString()));
                    //std::forward<string>(message)));
    }
  }
}
```

检测 send 的时候，是否在当前 IO 线程，如果是的话，直接进行写相关操作 `sendInLoop`。
如果不在一个线程的话，需要将该任务抛给 IO 线程执行 `runInloop`, 以保证 write 动作是在 IO 线程中执行的。我们后面会讲解 `runInloop` 的具体实现。

在 sendInloop 中，做了下面几件事：

1. 假如 OutputBuffer 为空，则直接向 socket 写数据
2. 如果向 socket 写数据没有写完，则统计剩余的字节个数，并进行下一步。没有写完可能是因为此时 socket 的 TCP 缓冲区已满了。
3. 如果此时 OutputBuffer 中的旧数据的个数和未写完字节个数之和大于 highWaterMark，则将 highWaterMarkCallback 放入待执行队列中
4. **将对应 socket 的可写事件注册到 EventLoop 中**

注意：直到发送消息的时候，muduo 才会把 socket 的可写事件注册到了 EventLoop 中。在此之前只注册了可读事件。

连接 socket 的可写事件对应的 callback 是 TcpConnection::handleWrite()
当某个 socket 的可写事件触发时，TcpConnection::handleWrite 会做两个工作：

1. 尽可能将数据从 OutputBuffer 中向 socket 中 write 数据
2. 如果 OutputBuffer 没有剩余的，则 **将该 socket 的可写事件移除**，并调用 writeCompleteCallback

```c++
void TcpConnection::handleWrite()
{
  loop_->assertInLoopThread();
  if (channel_->isWriting())
  {
    ssize_t n = sockets::write(channel_->fd(),
                               outputBuffer_.peek(),
                               outputBuffer_.readableBytes());
    if (n > 0)
    {
      outputBuffer_.retrieve(n);
      if (outputBuffer_.readableBytes() == 0)
      {
        channel_->disableWriting();
        if (writeCompleteCallback_)
        {
          loop_->queueInLoop(std::bind(writeCompleteCallback_, shared_from_this()));
        }
        if (state_ == kDisconnecting)
        {
          shutdownInLoop();
        }
      }
    }
    else
    {
      LOG_SYSERR << "TcpConnection::handleWrite";
      // if (state_ == kDisconnecting)
      // {
      //   shutdownInLoop();
      // }
    }
  }
  else
  {
    LOG_TRACE << "Connection fd = " << channel_->fd()
              << " is down, no more writing";
  }
}
```



### 为什么要移除可写事件

因为当 OutputBuffer 中没数据时，我们不需要向 socket 中写入数据。但是此时 socket 一直是处于可写状态的， 这将会导致 TcpConnection::handleWrite() 一直被触发， 因为muduo的Eventloop 采用的是epoll level trigger。然而这个触发毫无意义，因为并没有什么可以写的。

所以 muduo 的处理方式是，当 OutputBuffer 还有数据时，socket 可写事件是注册状态。当 OutputBuffer 为空时，则将 socket 的可写事件移除。

此外，highWaterMarkCallback 和 writeCompleteCallback 一般配合使用，起到限流的作用。

### 为什么采用level trigger,  不是edge tigger

- 为了和传统的poll 兼容， 因为在文件描述符数目较少，活动文件描述符比较高时， epool 不见得比poll 高效。
- level tigger 编程更容易， 以往 select/poll 的经验都可以继续用， 不可能发生漏掉事件的bug
- 读写的时候不必出现EAGAIN, 可以节省系统调用次数，降低延迟

### 连接的断开

我们看下 muduo 对于连接的断开是怎么处理的。
连接的断开分为被动断开和主动断开。主动断开和被动断开的处理方式基本一致，因此本文只讲下被动断开的部分。

被动断开即客户端断开了连接，server 端需要感知到这个断开的过程，然后进行的相关的处理。

其中感知远程断开这一步是在 Tcp 连接的可读事件处理函数 `handleRead` 中进行的：当对 socket 进行 read 操作时，返回值为 0，则说明此时连接已断开。

接下来会做四件事情：

1. 将该 TCP 连接对应的事件从 EventLoop 移除
2. 调用用户的 ConnectionCallback
3. 将对应的 TcpConnection 对象从 Server 移除。
4. close 对应的 fd。此步骤是在析构函数中自动触发的，当 TcpConnection 对象被移除后，引用计数为 0，对象析构时会调用 close。

```c++
void TcpConnection::handleClose()
{
  loop_->assertInLoopThread();
  LOG_TRACE << "fd = " << channel_->fd() << " state = " << stateToString();
  assert(state_ == kConnected || state_ == kDisconnecting);
  // we don't close fd, leave it to dtor, so we can find leaks easily.
  setState(kDisconnected);
  channel_->disableAll();

  TcpConnectionPtr guardThis(shared_from_this());
  connectionCallback_(guardThis);
  // must be the last line
  closeCallback_(guardThis);
}
```



### runInLoop 的实现

在讲解消息的发送过程时候，我们讲到为了保证对 buffer 和 socket 的写动作是在 IO 线程中进行，使用了一个 `runInLoop` 函数，将该写任务抛给了 IO 线程处理。我们接下来看下 `runInLoop` 的实现：

```c++
void EventLoop::runInLoop(Functor cb)
{
  if (isInLoopThread())
  {
    cb();
  }
  else
  {
    queueInLoop(std::move(cb));
  }
}

void EventLoop::queueInLoop(Functor cb)
{
  {
  MutexLockGuard lock(mutex_);
  pendingFunctors_.push_back(std::move(cb));
  }

  if (!isInLoopThread() || callingPendingFunctors_)
  {
    wakeup();
  }
}
```

这里可以看到，做了一层判断。如果调用时是此 EventLoop 的运行线程，则直接执行此函数。否则调用 `queueInLoop` 函数。

这里有两个动作：

1. 加锁，然后将该函数放到该 EventLoop 的 pendingFunctors_队列中。
2. 判断是否要唤醒 EventLoop，如果是则调用 wakeup() 唤醒该 EventLoop。

这里有几个问题：

- 为什么要唤醒 EventLoop？
- wakeup 是怎么实现的?
- pendingFunctors_是如何被消费的?

 

参考：

[万字长文梳理Muduo库核心代码及优秀编程细节思想剖析 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/495016351)

[muduo 源码剖析 | 编程沉思录 (cyhone.com)](https://www.cyhone.com/articles/analysis-of-muduo/)

《Linux多线程服务端编程：使用muduo C++网络库》



# Muduo <二>  异步log的实现

# 1. logging

日志(logging) 有两个意思:

- 诊断日志: 即是我们日常debug 使用的文本文件记录trace。常用的log 有log4j, logback, log4cpp, ezlogger等常用的日志库。
- 交易日志: 即是数据库的write-ahead log， 文件系统的journaling 等， 用于记录状态的变更，通过回访日志可以逐步恢复每一次修改之后的状态。

这篇文章我们主要介绍下muduo的log的实现，我觉得挺有意思的。muduo的日志库是C++ stream 风格的， 不需要保持格式字符串和参数类型的一致性， 可以随用随写，而且是类型安全的。我们需要的日志功能主要有下面几个需求:

1.  日志消息有多种级别(level), 如TRACE, DEBUG, INFO, WARN, FATAL 等。
2. 调整日志级别不需要重新编译，也不需要重启进程， 只要调用setloglevel() 就能即时6生效。
3. 往本地存储写文件时，考虑日志文件的滚动。文件大小(例如每写满1GB就换下一个文件)和时间(例如每天零点新建一个日志文件，不论前一个文件有没有写满)

**注意:** 往文件中写日志的一个常见的问题时，万一程序崩溃，那么最后若干条日志就丢失了，因为日志库不能每条消息都flush硬盘，更不能每条日志都open/close文件，这样性能开销太大。 muduo采用两个办法来应对这一点: 

1. 定期(默认3s)将缓冲区内的日志消息flush到硬盘；
2. 每条内存的日志消息都带有cookie, 其值为某个函数的地址，这样通过core dump 文件中查找cookie就能找到尚未来得及写入磁盘的消息。

日志消息的格式有几个要点:

1. 尽量每条日志占一行。
2. 时间戳精确到微秒。 muduo的每条消息时通过gettimeofday(2) 获得当前时间， 这个函数不是系统调用， 不会有什么性能损失。
3. 打印线程的id。
4. 打印日志的级别
5. 打印源文件名和行号。

# 2. 性能需求

编写linux 服务端程序的时候，我们需要一个高效的日志库。

- 每秒写几千上万条日志的时候没有明显的i性能损失
- 能应对一个进程产生大量的日志数据的场景， 例如 1GM/min
- 不阻塞正常的执行流程
- 在多线程中，不造成race condition.

为了满足这样的性能指标， muduo日志库的实现有几个优化措施。

1. 时间戳字符串中的日期和时间两部分时缓存的， 一秒之内的多条日志只需重新格式化微秒部分。
2. 日志消息的前4个字段时定长的，避免在运行期求字符串的长度。
3. 线程id时格式化为字符串。
4. 每行日志消息的源文件名部分采用了编译器计算来获得basename， 避免运行期strchr开销。

# 3. muduo的日志库分析

muduo的日志库采用的是双缓冲(double buffering)的技术. 准备两块buffer: A和B， 前端负责往buffer A填充数据， 后端负责将buffer B的数据写入文件。

当buffer A写满之后，交换A和B， 让后端将buffer A的数据写入文件，而前端则往buffer B填入新的日志消息，如此往复。用两个buffer的好处是在新建日志消息的时候不必等待磁盘文件操作， 也避免每条新日志消息都触发后端日志线程。换言之，前端将多条日志消息拼成一个大的buffer传送给后端，相当于批处理，减少了线程唤醒的频度，降低开销。muduo异步日志的性能开销大约是前端每写一条日志消息耗时1.0us ~ 1.6us.

在介绍`AsyncLogging` 文件之前，我们先看下`LogFile` 和 `Logger` ， `LogStream` ，`FixBuffer` 的实现。

## LogFile

`logfile` 比较简单，主要是rollfile的实现比较有意思。 我们先看下头文件。

```c++
class LogFile : noncopyable  // 不可被复制
{
 public:
  LogFile(const string& basename,     // basename 文件名
          off_t rollSize,             // rollSize 设置roll的大小的阈值，大于这个size的大小就开始roll. 
          bool threadSafe = true,     // 是否为线程安全， 为mutex 服务
          int flushInterval = 3,      // 几秒一次flush
          int checkEveryN = 1024);    // 检查roll 或者 flush的 次数
  ~LogFile();
  void append(const char* logline, int len);   // 给外部调用的只有这三个函数
  void flush();
  bool rollFile();
 private:
  void append_unlocked(const char* logline, int len);
  static string getLogFileName(const string& basename, time_t* now);
  const string basename_;
  const off_t rollSize_;
  const int flushInterval_;
  const int checkEveryN_;
  int count_;
  std::unique_ptr<MutexLock> mutex_;
  time_t startOfPeriod_;
  time_t lastRoll_;
  time_t lastFlush_;
  std::unique_ptr<FileUtil::AppendFile> file_; // 文件操作

  const static int kRollPerSeconds_ = 60*60*24;
};
```

 cpp文件,  构造函数比较简单。

```c++
LogFile::LogFile(const string& basename,
                 off_t rollSize,
                 bool threadSafe,
                 int flushInterval,
                 int checkEveryN)
  : basename_(basename),
    rollSize_(rollSize),
    flushInterval_(flushInterval),
    checkEveryN_(checkEveryN),
    count_(0),
    mutex_(threadSafe ? new MutexLock : NULL),
    startOfPeriod_(0),    // 记录每一次roll的时间， roll 完会更新
    lastRoll_(0),         // 记录上一次roll的时间
    lastFlush_(0)         // 记录上一次的flush/roll的时间
{
  assert(basename.find('/') == string::npos);
  rollFile();
}
```

我们看下append的实现，append 需要输入append的内容和长度， 如果是线程的安全的，加锁去append。

```c++
void LogFile::append(const char* logline, int len)
{
  if (mutex_)
  {
    MutexLockGuard lock(*mutex_);
    append_unlocked(logline, len);
  }
  else
  {
    append_unlocked(logline, len);
  }
}
```

这里调用了`append_unlocked`, 我们看下这个函数的实现。我们调用文件操作append写入len字节的logline， `writtenBytes` 会返回`append`的大小，如果 `writtenBytes` 大于设置的`rollSize_`， 我们就开始'`roolsize` 的操作， 否则就判断count的次数， 我们默认的是1024次。如果上一次`lastFlush_` 超过了3s， 我们就写到文件中。 

```c++
void LogFile::append_unlocked(const char* logline, int len)
{
  file_->append(logline, len);

  if (file_->writtenBytes() > rollSize_)
  {
    rollFile();
  }
  else
  {
    ++count_;
    if (count_ >= checkEveryN_)
    {
      count_ = 0;
      time_t now = ::time(NULL);
      time_t thisPeriod_ = now / kRollPerSeconds_ * kRollPerSeconds_;
      if (thisPeriod_ != startOfPeriod_)
      {
        rollFile();
      }
      else if (now - lastFlush_ > flushInterval_)
      {
        lastFlush_ = now;
        file_->flush();
      }
    }
  }
}
```

我们 看下真正`rollsize`的实现,  我们创建文件名+时间+主机名+ pid.log的文件名， 返回文件名和时间，如果此刻大于我们的上一次的roll的时间， 我们就新建一个文件。

```c++
bool LogFile::rollFile()
{
  time_t now = 0;
  string filename = getLogFileName(basename_, &now);
  time_t start = now / kRollPerSeconds_ * kRollPerSeconds_;

  if (now > lastRoll_)
  {
    lastRoll_ = now;
    lastFlush_ = now;
    startOfPeriod_ = start;
    file_.reset(new FileUtil::AppendFile(filename));
    return true;
  }
  return false;
}
```

```c++
string LogFile::getLogFileName(const string& basename, time_t* now)
{
  string filename;
  filename.reserve(basename.size() + 64);
  filename = basename;

  char timebuf[32];
  struct tm tm;
  *now = time(NULL);
  gmtime_r(now, &tm); // FIXME: localtime_r ?
  strftime(timebuf, sizeof timebuf, ".%Y%m%d-%H%M%S.", &tm);
  filename += timebuf;

  filename += ProcessInfo::hostname();

  char pidbuf[32];
  snprintf(pidbuf, sizeof pidbuf, ".%d", ProcessInfo::pid());
  filename += pidbuf;

  filename += ".log";

  return filename;
}
```



## Logger 

我们看到的loglevel 有六个级别，用枚举类型来定义的， `setLogLevel` 是`static` 类型的， 所以在多线程的写多个文件， 也会全局生效。

```c++
class Logger
{
 public:
  enum LogLevel
  {
    TRACE,
    DEBUG,
    INFO,
    WARN,
    ERROR,
    FATAL,
    NUM_LOG_LEVELS,
  };

  // compile time calculation of basename of source file
  class SourceFile
  {
   public:
  Logger(SourceFile file, int line);
  Logger(SourceFile file, int line, LogLevel level);
  Logger(SourceFile file, int line, LogLevel level, const char* func);
  Logger(SourceFile file, int line, bool toAbort);
  ~Logger();
  LogStream& stream() { return impl_.stream_; }
  static LogLevel logLevel();
  static void setLogLevel(LogLevel level);
  typedef void (*OutputFunc)(const char* msg, int len);
  typedef void (*FlushFunc)();
  static void setOutput(OutputFunc);
  static void setFlush(FlushFunc);
  static void setTimeZone(const TimeZone& tz);
      
  private:
    class Impl
    {
     public:
      typedef Logger::LogLevel LogLevel;
      Impl(LogLevel level, int old_errno, const SourceFile& file, int line);
      void formatTime();
      void finish();

      Timestamp time_;
      LogStream stream_;
      LogLevel level_;
      int line_;
      SourceFile basename_;
    };
 Impl impl_;
};
    
extern Logger::LogLevel g_logLevel;
inline Logger::LogLevel Logger::logLevel()
{
  return g_logLevel;
}
    
```

muduo使用预定义的方式定义了八种log 的级别， 这样我们调用`LOG_TRACE` 就会调用到`logstream` 流里面。

```c++
#define LOG_TRACE if (muduo::Logger::logLevel() <= muduo::Logger::TRACE) \
  muduo::Logger(__FILE__, __LINE__, muduo::Logger::TRACE, __func__).stream()
#define LOG_DEBUG if (muduo::Logger::logLevel() <= muduo::Logger::DEBUG) \
  muduo::Logger(__FILE__, __LINE__, muduo::Logger::DEBUG, __func__).stream()
#define LOG_INFO if (muduo::Logger::logLevel() <= muduo::Logger::INFO) \
  muduo::Logger(__FILE__, __LINE__).stream()
#define LOG_WARN muduo::Logger(__FILE__, __LINE__, muduo::Logger::WARN).stream()
#define LOG_ERROR muduo::Logger(__FILE__, __LINE__, muduo::Logger::ERROR).stream()
#define LOG_FATAL muduo::Logger(__FILE__, __LINE__, muduo::Logger::FATAL).stream()
#define LOG_SYSERR muduo::Logger(__FILE__, __LINE__, false).stream()
#define LOG_SYSFATAL muduo::Logger(__FILE__, __LINE__, true).stream()
```

我们再看下`setLogLevel()` ， 就是把level 设置给全局的loglevel, 比较简单。

```c++
void Logger::setLogLevel(Logger::LogLevel level)
{
  g_logLevel = level;
}
```



## LogStream

在logstream 类里面定义了两个类， 一个是FixedBuffer, 设置我们在stream 定义的缓存的大小，定义了小缓存为kSmallBuffer = 4k,  最大的缓存为4MB. 

`FixedBuffer` 的定义了一个`data_[size]`的数组和一个指向当前写入的字节的指针， 作用很简单， 就是预先开辟出一块内存，然后往里面写入(append) len 个字节， cur 会指向当前已写入的字节的多少。

```c++
const int kSmallBuffer = 4000;
const int kLargeBuffer = 4000*1000;

template<int SIZE>
class FixedBuffer : noncopyable
{
 public:
  FixedBuffer()
    : cur_(data_)
  {
    setCookie(cookieStart);
  }

  ~FixedBuffer()
  {
    setCookie(cookieEnd);
  }
  void append(const char* /*restrict*/ buf, size_t len)
  {
    // FIXME: append partially
    if (implicit_cast<size_t>(avail()) > len)
    {
      memcpy(cur_, buf, len);
      cur_ += len;
    }
  }
  const char* data() const { return data_; }
  int length() const { return static_cast<int>(cur_ - data_); }
  // write to data_ directly
  char* current() { return cur_; }
  int avail() const { return static_cast<int>(end() - cur_); }
  void add(size_t len) { cur_ += len; }

  void reset() { cur_ = data_; }
  void bzero() { memZero(data_, sizeof data_); }

  // for used by GDB
  const char* debugString();
  void setCookie(void (*cookie)()) { cookie_ = cookie; }
  // for used by unit test
  string toString() const { return string(data_, length()); }
  StringPiece toStringPiece() const { return StringPiece(data_, length()); }

 private:
  const char* end() const { return data_ + sizeof data_; }
  // Must be outline function for cookies.
  static void cookieStart();
  static void cookieEnd();

  void (*cookie_)();
  char data_[SIZE];
  char* cur_;
};
```

![image-20230416210559996](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230416210559996.png)



这里最重要的是logstream的类。定义了Buffer的大小为4k， 并且重载了若干个`operator<<`, 这样我们写log文件的时候可以使用`LOGTRACE<<` 来写入， 这里也很简单， 就是把字符串，数字等写入缓存buffer内。

```c++
class LogStream : noncopyable
{
  typedef LogStream self;
 public:
  typedef detail::FixedBuffer<detail::kSmallBuffer> Buffer;

  self& operator<<(bool v)
  {
    buffer_.append(v ? "1" : "0", 1);
    return *this;
  }

  self& operator<<(short);
  self& operator<<(unsigned short);
  self& operator<<(int);
  self& operator<<(unsigned int);
  self& operator<<(long);
  self& operator<<(unsigned long);
  self& operator<<(long long);
  self& operator<<(unsigned long long);

  self& operator<<(const void*);

  self& operator<<(float v)
  {
    *this << static_cast<double>(v);
    return *this;
  }
  self& operator<<(double);
  // self& operator<<(long double);

  self& operator<<(char v)
  {
    buffer_.append(&v, 1);
    return *this;
  }

  self& operator<<(const char* str)
  {
    if (str)
    {
      buffer_.append(str, strlen(str));
    }
    else
    {
      buffer_.append("(null)", 6);
    }
    return *this;
  }

  self& operator<<(const unsigned char* str)
  {
    return operator<<(reinterpret_cast<const char*>(str));
  }

  self& operator<<(const string& v)
  {
    buffer_.append(v.c_str(), v.size());
    return *this;
  }

  self& operator<<(const StringPiece& v)
  {
    buffer_.append(v.data(), v.size());
    return *this;
  }

  self& operator<<(const Buffer& v)
  {
    *this << v.toStringPiece();
    return *this;
  }

  void append(const char* data, int len) { buffer_.append(data, len); }
  const Buffer& buffer() const { return buffer_; }
  void resetBuffer() { buffer_.reset(); }
 private:
  void staticCheck();
  template<typename T>
  void formatInteger(T);
  Buffer buffer_;
  static const int kMaxNumericSize = 48;
};
```



接下来就是我们最重要的异步log的实现。

## AsyncLogging

在AsyncLogging 中， 采用了四个缓冲区，这样可以进一步减少或者避免等待。

```c++
class AsyncLogging : noncopyable
{
 public:

  AsyncLogging(const string& basename,
               off_t rollSize,
               int flushInterval = 3);

  ~AsyncLogging()
  {
    if (running_)
    {
      stop();
    }
  }

  void append(const char* logline, int len);

  void start()
  {
    running_ = true;
    thread_.start();
    latch_.wait();
  }

  void stop() NO_THREAD_SAFETY_ANALYSIS
  {
    running_ = false;
    cond_.notify();
    thread_.join();
  }

 private:

  void threadFunc();

  typedef muduo::detail::FixedBuffer<muduo::detail::kLargeBuffer> Buffer; // 4MB的缓冲
  typedef std::vector<std::unique_ptr<Buffer>> BufferVector; // 缓冲数组
  typedef BufferVector::value_type BufferPtr;  // 缓冲的指针

  const int flushInterval_;
  std::atomic<bool> running_;
  const string basename_;
  const off_t rollSize_;
  muduo::Thread thread_;
  muduo::CountDownLatch latch_;
  muduo::MutexLock mutex_;                     // 线程安全
  muduo::Condition cond_ GUARDED_BY(mutex_);
  BufferPtr currentBuffer_ GUARDED_BY(mutex_);   // 当前缓冲
  BufferPtr nextBuffer_ GUARDED_BY(mutex_);      // 预备缓冲
  BufferVector buffers_ GUARDED_BY(mutex_);     //待写入文件的已填满的缓冲
};
```

先来看发送方的代码。

```c++
AsyncLogging::AsyncLogging(const string& basename,
                           off_t rollSize,
                           int flushInterval)
  : flushInterval_(flushInterval),
    running_(false),
    basename_(basename),
    rollSize_(rollSize),
    thread_(std::bind(&AsyncLogging::threadFunc, this), "Logging"),
    latch_(1),
    mutex_(),
    cond_(mutex_),
    currentBuffer_(new Buffer),
    nextBuffer_(new Buffer),
    buffers_()
{
  currentBuffer_->bzero();
  nextBuffer_->bzero();
  buffers_.reserve(16);
}

void AsyncLogging::append(const char* logline, int len)
{
  muduo::MutexLockGuard lock(mutex_);
  if (currentBuffer_->avail() > len)   // 缓存没满， 写入缓冲中
  {
    currentBuffer_->append(logline, len);
  }
  else                               // 缓存满了
  {
    buffers_.push_back(std::move(currentBuffer_));   // 放入待写入的缓存中

    if (nextBuffer_)                         // 准备好另一块缓冲
    {
      currentBuffer_ = std::move(nextBuffer_);  // 把当前缓存的指针指向空buffer中
    }
    else
    {
      currentBuffer_.reset(new Buffer); // Rarely happens
    }
    currentBuffer_->append(logline, len);    // 当前缓冲继续append, 并通知
    cond_.notify();
  }
}
```

前端生成一条日志消息的时候会调用append()。 在这个函数中，如果当前缓冲(currentBuffer_)剩余空间足够大，则会直接把消息追加在到缓存中。否则说明当前缓冲区已满/不够，就把它送入buffers， 并把另一块备用的buffer设置为当前的缓冲区，然后追加消息并通知后端写入日志数据。如果写入速度太快，一下子把两块缓存都用完了，那么只好分配新的缓存作为当前缓存。

再看下接收方实现。

```c++
void AsyncLogging::threadFunc()
{
  assert(running_ == true);
  latch_.countDown();
  LogFile output(basename_, rollSize_, false); // LogfIle是真正写入的
  BufferPtr newBuffer1(new Buffer);      // 准备好第一块buffer
  BufferPtr newBuffer2(new Buffer);      // 准备好第二块buffer
  newBuffer1->bzero();
  newBuffer2->bzero();
  BufferVector buffersToWrite;
  buffersToWrite.reserve(16);
  while (running_)
  {
    assert(newBuffer1 && newBuffer1->length() == 0);
    assert(newBuffer2 && newBuffer2->length() == 0);
    assert(buffersToWrite.empty());

    {
      muduo::MutexLockGuard lock(mutex_);
      if (buffers_.empty())  // unusual usage!
      {
        cond_.waitForSeconds(flushInterval_);   // 等待条件触发
      }
      buffers_.push_back(std::move(currentBuffer_));  // 条件满足时， 先将当前缓冲区移入buffers
      currentBuffer_ = std::move(newBuffer1);        // 并立刻将空闲的newBuffer1 作为新的缓冲
      buffersToWrite.swap(buffers_);                 // 将空的buffersToWrite 填充
      if (!nextBuffer_)
      {
        nextBuffer_ = std::move(newBuffer2);    // 如果nextBuffer为空， 将buffer2 作为新的nextBuffer, 这样前端始终有一个buffer可供调配
      }
    }

    assert(!buffersToWrite.empty());

    if (buffersToWrite.size() > 25)  // 如果长度超过25， 丢弃一部分
    {
      char buf[256];
      snprintf(buf, sizeof buf, "Dropped log messages at %s, %zd larger buffers\n",
               Timestamp::now().toFormattedString().c_str(),
               buffersToWrite.size()-2);
      fputs(buf, stderr);
      output.append(buf, static_cast<int>(strlen(buf)));
      buffersToWrite.erase(buffersToWrite.begin()+2, buffersToWrite.end());
    }

    for (const auto& buffer : buffersToWrite)   // 遍历数组，真正写入内存中
    {
      // FIXME: use unbuffered stdio FILE ? or use ::writev ?
      output.append(buffer->data(), buffer->length());
    }

    if (buffersToWrite.size() > 2)
    {
      // drop non-bzero-ed buffers, avoid trashing
      buffersToWrite.resize(2);         // buffersToWrite的大小设置为2
    }

    if (!newBuffer1)    // 将buffersToWrite内的buffer重新填充为newBuffer1 
    {
      assert(!buffersToWrite.empty());
      newBuffer1 = std::move(buffersToWrite.back());
      buffersToWrite.pop_back();
      newBuffer1->reset();
    }

    if (!newBuffer2) // 将buffersToWrite内的buffer重新填充为newBuffer2
    {
      assert(!buffersToWrite.empty());
      newBuffer2 = std::move(buffersToWrite.back());
      buffersToWrite.pop_back();
      newBuffer2->reset();
    }

    buffersToWrite.clear();
    output.flush();
  }
  output.flush();
}
```

我们看下运行的图示:

一开始我们先分配好四个缓存， A, B ,C ，D. 初始时都是空的。

1. **缓冲A， B都没满， 但是超时。**写入频度不高， 后端3s 超市后将" 当前缓冲currentBuffer" 写入文件。在2.9s的时候， currBuffer使用了80%， 在第三秒的时候后端线程先把curr送到buffers, 再把newBuffer1设置为curr。 随后3s+, 交换buffers 和buffersToWrite， 后端开始将buffer A写入文件，写完再把new1再填上，供下次cond返回。

![image-20230416232620108](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230416232620108.png)



2. **缓冲A满了，B未满。**在3s超时之前已经写满了当前缓冲，于是唤醒后端线程开始写入文件。在第1.5s的时候， currBuffer使用了80%， 第1.8s， curr写满，将当前缓冲送入buffers_， 并将nextbuffer_ 设置为当前缓冲，然后开始写入。当后端线程唤醒之后(1.8s+)， 先将curr送入buffers_, 再将new1, new2设置为当前缓冲。

![image-20230416233005823](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230416233005823.png)



3. **缓冲AB都满。**前端在短时间内写入大量的消息，用完了两个缓冲，并重新分配了一块新的缓冲。在第1.8s的时候， A已经写满， B也接近写满。并且已经notify() 后端线程，但是由于种种原因， 后端线程并没有立刻开始工作， 直到1.9s的之后， B也写满了， 前端线程重新分配了缓冲E， 到了1.8s+， 后端线程开始写入，将CD 两块缓冲交给前端，并开始将ABE写入文件。完成之后， 用AB重新填充那两块缓冲，释放缓冲E。

![image-20230416233522121](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230416233522121.png)



4. **文件写入速度慢，导致前端耗尽了两个缓冲。** 前1.8s+ 和前面的第二种相同， 前端写满了一个缓冲，唤醒后后端线程开始写入文件。之后，后端花了较长时间才将数据写完。这期间前端又用完了两个缓冲(CD)，并分配了新的缓冲(E), 这期间前端的notify已经丢失。当后端写完之后，发现buffers_ 不为空， 立刻进入下一循环。替换前端的两个缓冲，并开始一次将CDE写入。如果buffersToWrite的大小超过了两个， 将重新把buffersToWrite的大小设置为2个， 再将new1, new2填充缓冲。

![image-20230416234324019](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230416234324019.png)



Muduo <三>  





































# 设计模式

设计模式是我们软件架构开发中不可缺失的一部分，通过学习设计模式，我们可以更好理解的代码的结构和层次。

## 设计原则

设计原则是早于设计方法出现的，所以的设计原则都要依赖于设计方法。这里主要有八个设计原则。

- **依赖倒置**

  - 高层模块不应该依赖低层模块，两者都应该依赖抽象  
  - 抽象不应该依赖具体实现，具体实现应该依赖于抽象  

  这个怎么理解呢？ 第一点是上层的模块，例如直接给用户使用的API， 不能直接暴露底层的实现给用户， 要通过中间的一层抽象接口。

![image-20230403234456184](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230403234456184.png)

例如: 自动驾驶系统公司是高层，汽车生产厂商为低层，它们不应该互相依赖，一方变动另一方也会跟着变动；而应该抽象一个自动驾驶行业标准，高层和低层都依赖它；这样以来就解耦了两方的变动；自动驾驶系统、汽车生产厂商都是具体实现，它们应该都依赖自动驾驶行业标准（抽象）；  

- **开放封闭**

  - 一个类应该对扩展（组合和继承）开放，对修改关闭；

  类中的接口应该对需要组合/继承的类暴露接口，对需要常变的封装到private里面。

- **面向接口**  

  - 不将变量类型声明为某个特定的具体类，而是声明为某个接口；
  - 客户程序无需获知对象的具体类型，只需要知道对象所具有的接口；
  - 减少系统中各部分的依赖关系，从而实现“高内聚、松耦合”的类型设计方案  

- **封装变化点**  

  - 将稳定点和变化点分离，扩展修改变化点；让稳定点和变化点的实现层次分离；  

- **单一职责**  

  - 一个类应该仅有一个引起它变化的原因  

- **里氏替换**  

  - 子类型必须能够替换掉它的父类型；主要出现在子类覆盖父类实现，原来使用父类型的程序可能出现错误；覆盖了父类方法却没有实现父类方法的职责；

-  **接口隔离**  

  -  不应该强迫客户依赖于它们不用的方法；
  - 一般用于处理一个类拥有比较多的接口，而这些接口涉及到很多职责;
  - 客户端不应该依赖它不需要的接口。一个类对另一个类的依赖应该建立在最小的接口上  

- **组合优于继承**  

  - 继承耦合度高，组合耦合度低 

## 1. 模板方法

定义：定义一个操作中的算法的骨架 ，而将一些步骤延迟到子类中。Template Method使得子类可以不改变一个算法的结构即可重定义该算法的某些特定步骤。

要点：

- 最常用的设计模式，子类可以复写父类子流程，使父类的骨架流程丰富；
- 反向控制流程的典型应用；
- 父类 protected 保护子类需要复写的子流程；这样子类的子流程只能父类来调用；  

结构图：

![image-20230405110539617](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230405110539617.png)

**我们可以看到，父类是一个抽象类，父类的接口由子类来复写实现。**

- **我们先来看下没有用模板方法写的代码。**

  我们顶一个了类zoo，里面的实现由show0, show1, show2......， 如果想添加一个新的show的，则需要在类zoo里面添加，这样就破坏了 类zoo的封装， 加入类zoo我们不想暴露给用户去修改呢？

```c++
#include <iostream>
using namespace std;
class ZooShow {
public:
    ZooShow(int type = 1) : _type(type) {}

public:
    void Show() {
        if (Show0())
            PlayGame();
        Show1();
        Show2();
        Show3();
    }

private:
    void PlayGame() {
        cout << "after Show0, then play game" << endl;
    }

    bool Show0() {
        if (_type == 1) {
            // 
            return true;
        } else if (_type == 2 ) {
            //  ...
        } else if (_type == 3) {

        }
        cout << _type << " show0" << endl;
        return true;
    }

    void Show1() {
        if (_type == 1) {
            cout << _type << " Show1" << endl;
        } else if (_type == 2) {
            cout << _type << " Show1" << endl;
        } else if (_type == 3) {

        }
    }

    void Show2() {
        if (_type == 20) {
            
        }
        cout << "base Show2" << endl;
    }

    void Show3() {
        if (_type == 1) {
            cout << _type << " Show1" << endl;
        } else if (_type == 2) {
            cout << _type << " Show1" << endl;
        }
    }
private:
    int _type;
};


int main () {
    ZooShow *zs = new ZooShow(1);
 
    bzs->Show();
    return 0;
}
```



- **使用模板方法**

  我们把父类的接口固定，然后定义若干个子类去继承父类的接口。在使用时只需定义父类的指针，指向要实现的子类的对象即可。

```c++
#include <iostream>
using namespace std;
// 开闭
class ZooShow {
public:
    void Show() {
        // 如果子表演流程没有超时的话，进行一个中场游戏环节；如果超时，直接进入下一个子表演流程
        if (Show0())
            PlayGame();
        Show1();
        Show2();
        Show3();
    }
    
private:
    void PlayGame() {
        cout << "after Show0, then play game" << endl;
    }
    bool expired;
    // 对其他用户关闭，但是子类开放的
protected:
    virtual bool Show0() {
        cout << "show0" << endl;
        if (! expired) {
            return true;
        }
        return false;
    }
    virtual void Show2() {
        cout << "show2" << endl;
    }
    virtual void Show1() {

    }
    virtual void Show3() {

    }
};

// 框架
// 模板方法模式
class ZooShowEx10 : public ZooShow {
protected:
    virtual void Show0() {
        if (! expired) {
            return true;
        }
        return false;
    }
}

class ZooShowEx1 : public ZooShow {
protected:
    virtual bool Show0() {
        cout << "ZooShowEx1 show0" << endl;
        if (! expired) { // 里氏替换
            return true;
        }
        return false;
    }
    virtual void Show2(){
        cout << "show3" << endl;
    }
};

class ZooShowEx2 : public ZooShow {
protected:
    virtual void Show1(){
        cout << "show1" << endl;
    }
    virtual void Show2(){
        cout << "show3" << endl;
    }
};

class ZooShowEx3 : public ZooShow {
protected:
    virtual void Show1(){
        cout << "show1" << endl;
    }
    virtual void Show3(){
        cout << "show3" << endl;
    }
    virtual void Show4() {
        //
    }
};
/*
*/
int main () {
    ZooShow *zs = new ZooShowEx10; // 晚绑定
    // ZooShow *zs1 = new ZooShowEx1;
    // ZooShow *zs2 = new ZooShowEx2;
    zs->Show();
    return 0;
}
```



## 2. 观察者模式 

定义: 定义对象间的一种一对多（变化）的依赖关系，以便当一个对象(Subject)的状态发生改变时，所有依赖于它的对象都得到通知并自动更新。  

要点: 

- 观察者模式使得我们可以独立地改变目标与观察者，从而使二者之间的关系松耦合;
- 观察者自己决定是否订阅通知，目标对象并不关注谁订阅了;
- 观察者不要依赖通知顺序，目标对象也不知道通知顺序;
- 常用在基于事件的ui框架中，也是 MVC 的组成部分；
- 常用在分布式系统中、actor框架中；  

结构图：

![image-20230405113324563](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230405113324563.png)



首先我们有一个抽象的父类，然后几个具体实现的子类， 一旦我们observer由任何的变化，我们就需要对每个子类的进行通知。

- 没有使用观察者模式

可以看到，我们定义n多个子类去接收CalcTemperature的改变。

```c++

class DisplayA {
public:
    void Show(float temperature);
};

class DisplayB {
public:
    void Show(float temperature);
};

class DisplayC {
public:
    void Show(float temperature);
}

class WeatherData {
};

class DataCenter {
public:
    void TempNotify() {
        DisplayA *da = new DisplayA;
        DisplayB *db = new DisplayB;
        DisplayC *dc = new DisplayC;
        // DisplayD *dd = new DisplayD;
        float temper = this->CalcTemperature();
        da->Show(temper);
        db->Show(temper);
        dc->Show(temper);
        dc->Show(temper);
    }
private:
    float CalcTemperature() {
        WeatherData * data = GetWeatherData();
        // ...
        float temper/* = */;
        return temper;
    }
    WeatherData * GetWeatherData(); // 不同的方式
};

int main() {
    DataCenter *center = new DataCenter;
    center->TempNotify();
    return 0;
}
```



- 观察者模式

```c++
#include <list>
#include <algorithm>
using namespace std;
//
class IDisplay {
public:
    virtual void Show(float temperature) = 0;
    virtual ~IDisplay() {}
};

class DisplayA : public IDisplay {
public:
    virtual void Show(float temperature) {
        cout << "DisplayA Show" << endl;
    }
private:
    void jianyi();
};

class DisplayB : public IDisplay{
public:
    virtual void Show(float temperature) {
        cout << "DisplayB Show" << endl;
    }
};

class DisplayC : public IDisplay{
public:
    virtual void Show(float temperature) {
        cout << "DisplayC Show" << endl;
    }
};

class DisplayD : public IDisplay{
public:
    virtual void Show(float temperature) {
        cout << "DisplayC Show" << endl;
    }
};

class WeatherData {
};

// 应对稳定点，抽象
// 应对变化点，扩展（继承和组合）
class DataCenter {
public:
    void Attach(IDisplay * ob) {
        //
    }
    void Detach(IDisplay * ob) {
        //
    }
    void Notify() {
        float temper = CalcTemperature();
        for (auto iter : obs) {
            iter.Show(temper);
        }
    }

// 接口隔离
private:
    WeatherData * GetWeatherData();

    float CalcTemperature() {
        WeatherData * data = GetWeatherData();
        // ...
        float temper/* = */;
        return temper;
    }
    std::list<IDisplay*> obs;
};

int main() {
    // 单例模式
    DataCenter *center = new DataCenter;
    // ... 某个模块
    IDisplay *da = new DisplayA();
    center->Attach(da);

    // ...
    IDisplay *db = new DisplayB();
    center->Attach(db);
    
    IDisplay *dc = new DisplayC();
    center->Attach(dc);

    center->Notify();
    
    //-----
    center->Detach(db);
    center->Notify();


    //....
    center->Attach(dd);

    center->Notify();
    return 0;
}
```



## 3. 策略模式  

定义: 定义一系列算法，把它们一个个封装起来，并且使它们可互相替换。该模式使得算法可独立于使用它的客户程序而变化。  

要点: 

- 策略模式提供了一系列可重用的算法，从而可以使得类型在运行时方便地根据需要在各个算法之间进行切换；  
- 策略模式消除了条件判断语句；也就是在解耦合；  

结构图:

![image-20230405114314169](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230405114314169.png)

策略模式的本质是用来消除多余的if else的。

- 没有使用策略模式

  ```c++
  enum VacationEnum {
  	VAC_Spring,
      VAC_QiXi,
  	VAC_Wuyi,
  	VAC_GuoQing,
      VAC_ShengDan,
  };
  
  class Promotion {
      VacationEnum vac;
  public:
      double CalcPromotion(){
          if (vac == VAC_Spring {
              // 春节
          }
          else if (vac == VAC_QiXi) {
              // 七夕
          }
          else if (vac == VAC_Wuyi) {
              // 五一
          }
  		else if (vac == VAC_GuoQing) {
  			// 国庆
  		}
          else if (vac == VAC_ShengDan) {
  
          }
       }
      
  };
  ```

- 使用策略模式

```c++
class Context {

};

// 稳定点：抽象去解决它
// 变化点：扩展（继承和组合）去解决它
class ProStategy {
public:
    virtual double CalcPro(const Context &ctx) = 0;
    virtual ~ProStategy(); 
};
// cpp
class VAC_Spring : public ProStategy {
public:
    virtual double CalcPro(const Context &ctx){}
};
// cpp
class VAC_QiXi : public ProStategy {
public:
    virtual double CalcPro(const Context &ctx){}
};
class VAC_QiXi1  : public VAC_QiXi {
public:
    virtual double CalcPro(const Context &ctx){}
};
// cpp
class VAC_Wuyi : public ProStategy {
public:
    virtual double CalcPro(const Context &ctx){}
};
// cpp
class VAC_GuoQing : public ProStategy {
public:
    virtual double CalcPro(const Context &ctx){}
};

class VAC_Shengdan : public ProStategy {
public:
    virtual double CalcPro(const Context &ctx){}
};

class Promotion {
public:
    Promotion(ProStategy *sss) : s(sss){}
    ~Promotion(){}
    double CalcPromotion(const Context &ctx){
        return s->CalcPro(ctx);
    }
private:
    ProStategy *s;
};

int main () {
    Context ctx;
    ProStategy *s = new VAC_QiXi1();
    Promotion *p = new Promotion(s);
    p->CalcPromotion(ctx);
    return 0;
}
```



## 4. 工厂模式

**定义:**  定义一个用于创建对象的接口，让子类决定实例化哪一个类。Factory Method使得一个类的实例化延迟到子类。

主要是为了创建同类对象的接口， 同类对象**只有一个**相同的职责。

看一个例子, 没有用工厂模式, 我们需要使用大量的if, else 来判断什么类型的时间。

![image-20230423103315209](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230423103315209.png)

```c++
#include <string>
// 实现导出数据的接口, 导出数据的格式包含 xml，json，文本格式txt 后面可能扩展excel格式csv
class IExport {
public:
    virtual bool Export(const std::string &data) = 0;
    virtual ~IExport(){}
};

class ExportXml : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class ExportJson : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};
// csv
class ExportTxt : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};
int main() {
    std::string choose/* = */;
    if (choose == "txt") {
        /***/
        IExport *e = new ExportTxt();
        /***/
        e->Export("hello world");
    } else if (choose == "json") {
        /***/
        IExport *e = new ExportJson();
        /***/
        e->Export("hello world");
    } else if (choose == "xml") {
        IExport *e = new ExportXml();
        e->Export("hello world");
    } else if (choose == "csv") {
        IExport *e = new ExportXml();
        e->Export("hello world");
    }
}

```

使用工厂模式：用`IExportFactory`把接口创建出来， 真正实现延迟到子类实现。

```c++
#include <string>
// 实现导出数据的接口, 导出数据的格式包含 xml，json，文本格式txt 后面可能扩展excel格式csv
class IExport {
public:
    virtual bool Export(const std::string &data) = 0;
    virtual ~IExport(){}
};

class ExportXml : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class ExportJson : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class ExportTxt : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class ExportCSV : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class IExportFactory {
public:
    IExportFactory() {
        _export = nullptr;
    }
    virtual ~IExportFactory() {
        if (_export) {
            delete _export;
            _export = nullptr;
        }
    }
    bool Export(const std::string &data) {
        if (_export == nullptr) {
            _export = NewExport();
        }
        return _export->Export(data);
    }
protected:
    virtual IExport * NewExport(/* ... */) = 0;
private:
    IExport* _export;
};

class ExportXmlFactory : public IExportFactory {
protected:
    virtual IExport * NewExport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IExport * temp = new ExportXml();
        // 可能之后有什么操作
        return temp;
    }
};
class ExportJsonFactory : public IExportFactory {
protected:
    virtual IExport * NewExport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IExport * temp = new ExportJson;
        // 可能之后有什么操作
        return temp;
    }
};
class ExportTxtFactory : public IExportFactory {
protected:
    IExport * NewExport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IExport * temp = new ExportTxt;
        // 可能之后有什么操作
        return temp;
    }
};

class ExportCSVFactory : public IExportFactory {
protected:
    virtual IExport * NewExport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IExport * temp = new ExportCSV;
        // 可能之后有什么操作
        return temp;
    }
};

int main () {
    IExportFactory *factory = new ExportCSVFactory();
    factory->Export("hello world");
    return 0;
}
```

1） 如果为每一个具体的 ConcreteProduct 类的实例化提供一个函数体， 那么我们可能不得不在系统中添加了一个方法来处理这个新建的 ConcreteProduct，这样 Factory 的接口永远就不肯能封闭（ Close）。 当然我们可以通过创建一个 Factory 的子类来通过多态实现这一点，但是这也是以新建一个类作为代价的。  

2）在实现中我们可以通过参数化工厂方法，即给 FactoryMethod（ ）传递一个参数用以 决定是创建具体哪一个具体的 Product。当
然也可以通过模板化避免 1）中的子类创建子类，其方法就是将具体 Product 类作为模板参数，实现起来也很简单。可以看出， Factory 模式对于对象的创建给予开发人员提供了很好的实现策略，但是Factory 模式仅仅局限于一类类（就是说 Product 是一类，有一个共同的基类），如果我们要为不同类的类提供一个对象创建的接口，那就要用 AbstractFactory 了。  



## 5. 抽象工厂

**定义:**  提供一个接口, 让接口负责创建一系列的"相关或者相互依赖的对象"， 无需指定他们具体的类。

主要是为了创建同类对象的接口，和工厂模式最主要的区分是，同类对象具有**很多相同的职责**。

![image-20230423103258779](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230423103258779.png)

```c++
#include <string>
// 实现导出数据的接口, 导出数据的格式包含 xml，json，文本格式txt 后面可能扩展excel格式csv
class IExport {
public:
    virtual bool Export(const std::string &data) = 0;
    virtual ~IExport(){}
};

class ExportXml : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class ExportJson : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class ExportTxt : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class ExportCSV : public IExport {
public:
    virtual bool Export(const std::string &data) {
        return true;
    }
};

class IImport {
public:
    virtual bool Import(const std::string &data) = 0;
    virtual ~IImport(){}
};

class ImportXml : public IImport {
public:
    virtual bool Import(const std::string &data) {
        return true;
    }
};

class ImportJson : public IImport {
public:
    virtual bool Import(const std::string &data) {
        return true;
    }
};

class ImportTxt : public IImport {
public:
    virtual bool Import(const std::string &data) {
        return true;
    }
};

class ImportCSV : public IImport {
public:
    virtual bool Import(const std::string &data) {
        // ....
        return true;
    }
};

class IDataApiFactory {
public:
    IDataApiFactory() {
        _export = nullptr;
        _import = nullptr;
    }
    virtual ~IDataApiFactory() {
        if (_export) {
            delete _export;
            _export = nullptr;
        }
        if (_import) {
            delete _import;
            _import = nullptr;
        }
    }
    bool Export(const std::string &data) {
        if (_export == nullptr) {
            _export = NewExport();
        }
        return _export->Export(data);
    }
    bool Import(const std::string &data) {
        if (_import == nullptr) {
            _import = NewImport();
        }
        return _import->Import(data);
    }
protected:
    virtual IExport * NewExport(/* ... */) = 0;
    virtual IImport * NewImport(/* ... */) = 0;
private:
    IExport *_export;
    IImport *_import;
};

class XmlApiFactory : public IDataApiFactory {
protected:
    virtual IExport * NewExport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IExport * temp = new ExportXml;
        // 可能之后有什么操作
        return temp;
    }
    virtual IImport * NewImport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IImport * temp = new ImportXml;
        // 可能之后有什么操作
        return temp;
    }
};

class JsonApiFactory : public IDataApiFactory {
protected:
    virtual IExport * NewExport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IExport * temp = new ExportJson;
        // 可能之后有什么操作
        return temp;
    }
    virtual IImport * NewImport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IImport * temp = new ImportJson;
        // 可能之后有什么操作
        return temp;
    }
};
class TxtApiFactory : public IDataApiFactory {
protected:
    virtual IExport * NewExport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IExport * temp = new ExportTxt;
        // 可能之后有什么操作
        return temp;
    }
    virtual IImport * NewImport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IImport * temp = new ImportTxt;
        // 可能之后有什么操作
        return temp;
    }
};

class CSVApiFactory : public IDataApiFactory {
protected:
    virtual IExport * NewExport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IExport * temp = new ExportCSV;
        // 可能之后有什么操作
        return temp;
    }
    virtual IImport * NewImport(/* ... */) {
        // 可能有其它操作，或者许多参数
        IImport * temp = new ImportCSV;
        // 可能之后有什么操作
        return temp;
    }
};

// 相关性  依赖性    工作当中
int main () {
    IDataApiFactory *factory = new CSVApiFactory();
    factory->Import("hello world");
    factory->Export("hello world");
    return 0;
}
```

AbstractFactory 模式和 Factory 模式的区别是初学（使用）设计模式时候的一个容易引起困惑的地方。 实际上， AbstractFactory 模式是为创建一组（ 有多类） 相关或依赖的对象提供创建接口， 而 Factory 模式正如我在相应的文档中分析的是为一类对象提供创建接口或延
迟对象的创建到子类中实现。并且可以看到， AbstractFactory 模式通常都是使用 Factory 模式实现（ ConcreteFactory1）。  

## 6. 责任链模式

**定义:**   使多个对象都有机会处理请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递请求，直到有一个对象处理它为止。 这个思路很容易理解，一条链上有若干个请求，每个请求都有可能被每一个节点处理，处理完可能就不需要后面的来处理了。

  ![image-20230423104152894](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230423104152894.png)

Chain of Responsibility 模式中 ConcreteHandler 将自己的后继对象（向下传递消息的对象）记录在自己的后继表中，当一个请求到来时， ConcreteHandler 会先检查看自己有没有匹配的处理程序， 如果有就自己处理， 否则传递给它的后继。 当然这里示例程序中为了简化，ConcreteHandler 只是简单的检查看自己有没有后继，有的话将请求传递给后继进行处理，没有的话就自己处理。  

```c++
#include <iostream>
using namespace std;
class Handle
{
public:
	virtual ~Handle();
	virtual void HandleRequest() = 0;
	void SetSuccessor(Handle* succ) { _succ = succ; }
	Handle* GetSuccessor() { return _succ; }
protected:
	Handle() {	_succ = NULL; }
	Handle(Handle* succ) { this->_succ = succ; }
private:
	Handle* _succ;
};
class ConcreteHandleA:public Handle
{
public:
	ConcreteHandleA();
	~ConcreteHandleA();
	ConcreteHandleA(Handle* succ) : Handle(succ) {}
	void HandleRequest(){
		if (this->GetSuccessor() != 0) {
			cout<<"ConcreteHandleA request to next ....."<<endl;
			this->GetSuccessor()->HandleRequest();
		} else {
			cout<<"ConcreteHandleA handle"<<endl;
		}
	}
protected:
private:
};

class ConcreteHandleB:public Handle
{
public:
	ConcreteHandleB();
	~ConcreteHandleB();
	ConcreteHandleB(Handle* succ):: Handle(succ) {}
	void HandleRequest() {
		if (this->GetSuccessor() != 0) {
			cout<<"ConcreteHandleA request to next ....."<<endl;
			this->GetSuccessor()->HandleRequest();
		} else {
			cout<<"ConcreteHandleA handle"<<endl;
		}
	}
protected:
private:
};

int main() {
	Handle* h1 = new ConcreteHandleA();
	Handle* h2 = new ConcreteHandleB();
	h1->SetSuccessor(h2);
	h1->HandleRequest();
	return 0;
}
```

Chain of Responsibility 模式的示例代码实现很简单，这里就其测试结果给出说明：ConcreteHandleA 的对象和 h1 拥有一个后继 ConcreteHandleB 的对象 h2,当一个请求到来时候， h1 检查看自己有后继，于是 h1 直接将请求传递给其后继 h2 进行处理， h2 因为没有后继，当请求到来时候，就只有自己提供响应了。  

Chain of Responsibility 模式的最大的一个有点就是给系统降低了耦合性， 请求的发送者完全不必知道该请求会被哪个应答对象处理，极大地降低了系统的耦合性。  



## 7 装饰器模式

在 OO 设计和开发过程， 可能会经常遇到以下的情况： 我们需要为一个已经定义好的类添加新的职责（操作）， 通常的情况我们会给定义一个新类继承自定义好的类，这样会带来一个问题（ 将在本模式的讨论中给出）。通过继承的方式解决这样的情况还带来了系统的复
杂性，因为继承的深度会变得很深。而 Decorator 提供了一种给类增加职责的方法，不是通过继承实现的，而是通过组合。

![image-20230423110230354](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230423110230354.png)

在 结 构 图 中 ， ConcreteComponent 和 Decorator 需 要 有 同 样 的 接 口 ， 因 此ConcreteComponent 和 Decorator 有着一个共同的父类。这里有人会问，让 Decorator 直接维护一个指向 ConcreteComponent 引用（指针） 不就可以达到同样的效果， 答案是肯定并且是否定的。 肯定的是你可以通过这种方式实现， 否定的是你不要用这种方式实现， 因为通过这种方式你就只能为这个特定的 ConcreteComponent 提供修饰操作了，当有了一个新的ConcreteComponent 你又要去新建 一 个 Decorator 来 实 现 。 但是通过结构图中的ConcreteComponent 和 Decorator 有一个公共基类， 就可以利用 OO 中多态的思想来实现只要是 Component 型别的对象都可以提供修饰操作的类，这种情况下你就算新建了100个 。Component 型别的类 ConcreteComponent，也都可以由 Decorator 一个类搞定。这也正是Decorator 模式的关键和威力所在了。当然如果你只用给 Component 型别类添加一种修饰， 则 Decorator 这个基类就不是很必
要了。  

没有用装饰器

```c++
// 普通员工有销售奖金，累计奖金，部门经理除此之外还有团队奖金；后面可能会添加环比增长奖金，同时可能产生不同的奖金组合；
// 销售奖金 = 当月销售额 * 4%
// 累计奖金 = 总的回款额 * 0.2%
// 部门奖金 = 团队销售额 * 1%
// 环比奖金 = (当月销售额-上月销售额) * 1%
// 销售后面的参数可能会调整
class Context {
public:
    bool isMgr;
    // User user;
    // double groupsale;
};

class Bonus {
public:
    double CalcBonus(Context &ctx) {
        double bonus = 0.0;
        bonus += CalcMonthBonus(ctx);
        bonus += CalcSumBonus(ctx);
        if (ctx.isMgr) {
            bonus += CalcGroupBonus(ctx);
        }
        return bonus;
    }
private:
    double CalcMonthBonus(Context &ctx) {
        double bonus/* = */;
        return bonus;
    }
    double CalcSumBonus(Context &ctx) {
        double bonus/* = */;
        return bonus;
    }
    double CalcGroupBonus(Context &ctx) {
        double bonus/* = */;
        return bonus;
    }
};

int main() {
    Context ctx;
    // 设置 ctx
    Bonus *bonus = new Bonus;
    bonus->CalcBonus(ctx);
}
```



```c++
#include <iostream>
// 普通员工有销售奖金，累计奖金，部门经理除此之外还有团队奖金；后面可能会添加环比增长奖金，同时可能产生不同的奖金组合；
// 销售奖金 = 当月销售额 * 4%
// 累计奖金 = 总的回款额 * 0.2%
// 部门奖金 = 团队销售额 * 1%
// 环比奖金 = (当月销售额-上月销售额) * 1%
// 销售后面的参数可能会调整
using namespace std;
class Context {
public:
    bool isMgr;
    // User user;
    // double groupsale;
};

class CalcBonus {    
public:
    CalcBonus(CalcBonus * c = nullptr) : cc(c) {}
    virtual double Calc(Context &ctx) {
        return 0.0; // 基本工资
    }
    virtual ~CalcBonus() {}

protected:
    CalcBonus* cc;
};

class CalcMonthBonus : public CalcBonus {
public:
    CalcMonthBonus(CalcBonus * c) : CalcBonus(c) {}
    virtual double Calc(Context &ctx) {
        double mbonus /*= 计算流程忽略*/; 
        return mbonus + cc->Calc(ctx);
    }
};

class CalcSumBonus : public CalcBonus {
public:
    CalcSumBonus(CalcBonus * c) : CalcBonus(c) {}
    virtual double Calc(Context &ctx) {
        double sbonus /*= 计算流程忽略*/; 
        return sbonus + cc->Calc(ctx);
    }
};

class CalcGroupBonus : public CalcBonus {
public:
    CalcGroupBonus(CalcBonus * c) : CalcBonus(c) {}
    virtual double Calc(Context &ctx) {
        double gbnonus /*= 计算流程忽略*/; 
        return gbnonus + cc->Calc(ctx);
    }
};

class CalcCycleBonus : public CalcBonus {
public:
    CalcCycleBonus(CalcBonus * c) : CalcBonus(c) {}
    virtual double Calc(Context &ctx) {
        double gbnonus /*= 计算流程忽略*/; 
        return gbnonus + cc->Calc(ctx);
    }
};

int main() {
    // 1. 普通员工
    Context ctx1;
    CalcBonus *base = new CalcBonus();
    CalcBonus *cb1 = new CalcMonthBonus(base);
    CalcBonus *cb2 = new CalcSumBonus(cb1);


    cb2->Calc(ctx1);
    // 2. 部门经理
    Context ctx2;
    CalcBonus *cb3 = new CalcGroupBonus(cb1);
    cb3->Calc(ctx2);
}
```





# IO_uring

io_uring 是 kernel natvie aio 的一种，它是 Linux Kernel 5.1 版本加入一个特性。通过设计 io_uring 这套全新的 aysnc IO 系统调用接口，让应用程序可以获得更高的性能，更好的兼容性。

io_uring 围绕高效进行设计，其设计了一对共享的 ring buffer 用于应用和内核之间的通信，通过该设计实现了如下的三个好处：

（1）避免在提交和完成事件中存在内存拷贝；

（2）避免了 libaio 中在提交和完成任务的时候系统调用过程；

（3）该队列采用了无锁的访问模式，通过内存屏障减少了竞争

在共享的 ring buffer 设计中，针对提交队列（SQ），应用是 IO 提交的生产者（producer），内核是消费者（consumer）；反过来，针对完成队列（CQ），内核是完成事件的生产者，应用是消费者。

另外，io_uring 还存在如下的优势：

（1）提交和完成不需要经过系统调用，而且减少了对用户态线程的阻塞；该部分的支持主要通过共享的 ring buffer 和设置 polling 模式来实现。

（2）支持 Block 层的 polling 模式

（3）支持 buffered IO，充分利用缓存，减少数据碰盘产生的系统延迟；

![image-20230619225538224](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230619225538224.png)

## 1. 数据结构

**Completion Queue**  

- `user_data` :  此字段从初始请求提交开始，可以包含应用程序识别所述请求所需的任何信息。一个常见的用例是让它成为原始请求的指针。内核不会触及这个字段，它只是从提交到完成事件直接进行。
- `res` : res 保存请求的结果。把它想象成回归来自系统调用的值。对于正常的读/写操作，这将类似于 read（2） 或写（2）.对于成功的操作，它将包含传输的字节数。如果发生故障，它将包含负错误值。例如，如果发生 I/O 错误，res 将包含 -EIO。
- ` flag`:  可以携带与此操作相关的元数据。截至目前，此字段未使用

```c++
struct io_uring_cqe {
__u64 user_data;
__s32 res;
__u32 flags;
};
```

**Submission Queue**  

```c
struct io_uring_sqe {
    __u8 opcode;
    __u8 flags;
    __u16 ioprio;
    __s32 fd;
    __u64 off;
    __u64 addr;
    __u32 len;
    union {
        __kernel_rwf_t rw_flags;
        __u32 fsync_flags;
        __u16 poll_events;
        __u32 sync_range_flags;
        __u32 msg_flags;
    };
	__u64 user_data;
    union {
        __u16 buf_index;
        __u64 __pad2[3];
    };
};
```

- `opcode` : 描述此特定请求的操作代码（或简称操作代码）。一个这样的操作码是IORING_OP_READV，这是一个向量读取。
- `flag` :  TODO
- `ioprio` :  是此请求的优先事项。为正常读/写，这遵循 ioprio_set（2） 系统调用概述的定义。
- `fd` :  文件描述符
- `off` :  保存操作应发生的偏移量
- `addr` :  包含操作应执行 IO 的地址（如果opcode 描述传输数据的操作）。如果操作是某种形式的向量读/写， 这将是一个指向结构体 iovec 数组的指针， 例如 preadv（2） 使用。对于非矢量 IO 传输，addr 必须直接包含地址.
- `len` : 和addr 一起使用。
- `union` : 针对特定操作符的集合
- `user_date` : 只是简单第拷贝用户的数据
- `buf_index` :  TODO





# 内存对齐的函数

## 1. 函数

```c++
void *memalign(size_t alignment, size_t size);
int posix_memalign(void **memptr, size_t alignment, size_t size);
void *aligned_alloc(size_t alignment, size_t size);
void *aligned_malloc(size_t alignment, size_t size);
```

## 2. 属性

```c++
double x[100] __attribute__((aligned(64)))
```





# OpenMP

## 1. 简介

OpenMP是一种用于线程之间共享内存的并行方式，需要编译器支持，当前流行的编译器几乎都支持OpenMP. 但是使用OpenMP将应用程序性能提升到极致，仍然是一种挑战，这种挑战是来自于允许线程竟态条件存在的弱内存模型(relaxed memory module). 所谓的relexed, 指的主内存中的变量值不会被立即更改，因为更改会会导致操作成本过高。

- 弱内存模型: 所以处理器的主内存或缓存中的变量值不会立即被更新
- 竟态条件（race condition): 对于相同的运算却得到多个不同的结果，运算所得到的结果却决于参与运算线程的时间安排。

因为OpenMP具有弱内存模型，所以一个线程的内存视图需要OpenMP barrier 或flush 操作才能鱼其他线程通信，防止竟态条件。OpenMP适用于单个节点上的内存，不适用分布式内存的多个节点。

下表中列出常见的OpenMP概念。属于和指令。

| OpenMP 主体                   | OpenMP pragma                                                | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 并行区域                      | #pragma omp parallel                                         | 将再该指令后面的区域内生成线程                               |
| work sharing 循环             | #pargma omp for                                              | 再线程之间的平均分配工作。调度语句包括静态, 动态，引到和自动 |
| 将并行区域鱼work sharing 结合 | #pragma omp parallel for                                     |                                                              |
| 约减                          | #pargma omp parallel for reduction（+：sum）, (min:xmin), or (max:xmax) |                                                              |
| 同步                          | #pargma omp barrier                                          | 运行多个线程是，此调用创建一个停止点，以便再进行下一步动作之前，所有线程都可以的重新组合 |
| 串行部分                      | #pargma omp masked                                           | 防止多个线程执行同意代码。当在一个并行区域中有一个函数，并且只允许这个函数再一个线程上执行时，可以使用这个指令 |
| 锁                            | #pargma omp critical or atomic                               | 在特殊情况下使用，用于高级实现                               |
| 并行区域                      | \#pragma omp for nowait                                      | **nowait 子句用于消除隐式的 barrier**                        |



**安装:**

进入官网[OpenMP](https://www.open-mpi.org/software/ompi/v4.1/)，下载稳定版本.

解压后进入文件夹，运行`./configure --prefix=/home/jame/Public/openmp`

编译运行

> make -j4
> sudo make install

路径配置

> sudo vim ~/.bashrc

添加

> export PATH="$PATH:/home/jame/Public/openmp/bin"
> export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/home/jame/Public/openmp/lib"

运行:

> mpirun -np 4 hello_c

CMakeLists配置

```c++
set(CMAKE_CXX_FLAGS "-Wall -fopenmp")

find_package(OpenMP REQUIRED)
if(OpenMP_FOUND)
    message(STATUS "########## Found openmp ##########") 
    set(CMAKE_C_FLAGS ${CMAKE_C_FLAGS} ${OPENMP_C_FLAGS})
    set(CMAKE_CXX_FLAGS ${CMAKE_CXX_FLAGS} ${OPENMP_CXX_FLAGS})
else()
    message(FATAL_ERROR "Openmp not found!")
endif()

```

## 2. OpenMP的简单程序实例

控制并行区域的线程数量

- Default: 通常是节点的做大线程数，但也会根据编译器以及是否存在MPI级别而有所不同。
- 环境变量: 通过 `OMP_NUM_THREADS` 环境变量来设置具体大小: 例如: `export OMP_NUM_THREADS = 16 `
- 函数调用: 调用函数`omp_set_threads` 例如: `omp_set_threads(16)`
- pargma: 例如: `#pargma omp parallel num_threads(16)`

```c
#include <stdio.h>
#include <omp.h>

int main(int argc, char *argv[]){
   int nthreads, thread_id;
   nthreads = omp_get_num_threads();
   thread_id = omp_get_thread_num();
   printf("Goodbye slow serial world and Hello OpenMP!\n");
   printf("  I have %d thread(s) and my thread id is %d\n",nthreads,thread_id);
}
```

使用gcc 

> gcc -o -fopenmp HelloOpenMP HelloOpenMP.c

或者使用cmake

```cmake
cmake_minimum_required (VERSION 3.0)
project (HelloOpenMP)

set (CMAKE_C_STANDARD 99)

# Set vectorization flags for a few compilers
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -g -O3")
if ("${CMAKE_C_COMPILER_ID}" STREQUAL "Clang") # using Clang
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstrict-aliasing -Rpass-analysis=loop-vectorize")

elseif ("${CMAKE_C_COMPILER_ID}" STREQUAL "GNU") # using GCC
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -fstrict-aliasing -fopt-info-loop")

elseif ("${CMAKE_C_COMPILER_ID}" STREQUAL "Intel") # using Intel C
   set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ansi-alias -qopt-report-phase=loop")

elseif (CMAKE_C_COMPILER_ID MATCHES "MSVC")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")

elseif (CMAKE_C_COMPILER_ID MATCHES "XL")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")

elseif (CMAKE_C_COMPILER_ID MATCHES "Cray")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}")

endif()

find_package(OpenMP)

# Adds build target of stream_triad with source code files
add_executable(HelloOpenMP HelloOpenMP.c)
set_target_properties(HelloOpenMP PROPERTIES COMPILE_FLAGS ${OpenMP_C_FLAGS})
set_target_properties(HelloOpenMP PROPERTIES LINK_FLAGS "${OpenMP_C_FLAGS}")

add_executable(HelloOpenMP_fix1 HelloOpenMP_fix1.c)
set_target_properties(HelloOpenMP_fix1 PROPERTIES COMPILE_FLAGS ${OpenMP_C_FLAGS})
set_target_properties(HelloOpenMP_fix1 PROPERTIES LINK_FLAGS "${OpenMP_C_FLAGS}")

add_executable(HelloOpenMP_fix2 HelloOpenMP_fix2.c)
set_target_properties(HelloOpenMP_fix2 PROPERTIES COMPILE_FLAGS ${OpenMP_C_FLAGS})
set_target_properties(HelloOpenMP_fix2 PROPERTIES LINK_FLAGS "${OpenMP_C_FLAGS}")

add_executable(HelloOpenMP_fix3 HelloOpenMP_fix3.c)
set_target_properties(HelloOpenMP_fix3 PROPERTIES COMPILE_FLAGS ${OpenMP_C_FLAGS})
set_target_properties(HelloOpenMP_fix3 PROPERTIES LINK_FLAGS "${OpenMP_C_FLAGS}")

add_executable(HelloOpenMP_fix4 HelloOpenMP_fix4.c)
set_target_properties(HelloOpenMP_fix4 PROPERTIES COMPILE_FLAGS ${OpenMP_C_FLAGS})
set_target_properties(HelloOpenMP_fix4 PROPERTIES LINK_FLAGS "${OpenMP_C_FLAGS}")

add_executable(HelloOpenMP_fix5 HelloOpenMP_fix5.c)
set_target_properties(HelloOpenMP_fix5 PROPERTIES COMPILE_FLAGS ${OpenMP_C_FLAGS})
set_target_properties(HelloOpenMP_fix5 PROPERTIES LINK_FLAGS "${OpenMP_C_FLAGS}")

# Cleanup
add_custom_target(distclean COMMAND rm -rf CMakeCache.txt CMakeFiles
                  Makefile cmake_install.cmake stream_triad.dSYM ipo_out.optrpt)

```



输出:

> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 1 thread(s) and my thread id is 0

这个不是我们想要的，更改代码:

```c
#include <stdio.h>
#include <omp.h>

int main(int argc, char *argv[]){
   int nthreads, thread_id;
   #pragma omp parallel
   {
      nthreads = omp_get_num_threads();
      thread_id = omp_get_thread_num();
      printf("Goodbye slow serial world and Hello OpenMP!\n");
      printf("  I have %d thread(s) and my thread id is %d\n",nthreads,thread_id);
   }
}
```



输出

> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 1                                                                                                                 
>
> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 1                                                                                                                 
>
> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 1                                                                                                                 
>
> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 1 

正如看到的，所有的thread id  都是1， 因为这两个是共享变量。

我们把nthreads， 和 thread_id 改成私有变量，定义到循环里面。

输出:

> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 1                                                                                                                 
>
> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 2                                                                                                                 
>
> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 0                                                                                                                 
>
> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 3

打印输出是随机的，这取决于每个处理器的写入顺序以及他们被刷新到标准输出的方式。

假如我们不想把每个线程输出都打印出来，只打印一次；

```c
#include <stdio.h>
#include <omp.h>

int main(int argc, char *argv[]){
   #pragma omp parallel
   {
      int nthreads = omp_get_num_threads();
      int thread_id = omp_get_thread_num();
      #pragma omp single
      {
         printf("Goodbye slow serial world and Hello OpenMP!\n");
         printf("  I have %d thread(s) and my thread id is %d\n",nthreads,thread_id);
      }
   }
}
```

输出:

> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 2  

只有一个线程可以输出，而且是随机的。假如我们想输出第一个线程的结果，我们把`single` 改成`master`. 

输出:

> Goodbye slow serial world and Hello OpenMP!                                                                                                                
>
>   I have 4 thread(s) and my thread id is 0  

我们来进一步优化，使用更少的pargma。 

```c
#include <stdio.h>
#include <omp.h>

int main(int argc, char *argv[]){
   printf("Goodbye slow serial world and Hello OpenMP!\n");
   #pragma omp parallel
   if (omp_get_thread_num() == 0) {
      printf("  I have %d thread(s) and my thread id is %d\n",
             omp_get_num_threads(), omp_get_thread_num());
   }
}
```

在这个例子中，我们学到了

- 在并行区域之外定义的变量，默认情况下这个变量将在并行区域中共享
- 使用masked子句比single的限制更加严格，因为他需要线程0来执行代码块。masked子句在结尾也没有隐含的barrier.
- 不同线程之间存在竟态条件。

## 3. 高级OpenMP

在OpenMP的usercase 中， 主要有三种情况。

- 循环级OpenMP
- 高级OpenMP
- MPI+OpenMP

### 1. 循环级OpenMP

应用场景如下

- 只需要适度或者小幅度提升并行度。
- 有充足的内存可用资源
- 大量的计算需求只集中在若干个for循环或do循环中



### 2. 高级OpenMP

与循环级相比， 高级可以提供更高的性能。标准的OpenMP自下而上运行，并在循环级实现应用程序的性能，高级OpenMP是从整个系统的角度进行设计，采用自上而下的方法处理系统的内存，系统核心以及相关硬件，在这个过程中，我们只改变了用法，最终实现的效果减少了很多线程启动的成本，以及阻碍循环级OpenMP伸缩能力的同步成本。



### 3. OpenMP + MPI

OpenMP 也可以用来增加分布式内存的并行性，可以在节点内进行，或者更好地是可以对共享内存进行快速访问的处理器集合内进行，通常称为非统一内存访问(NUMA). 

高级OpenMP的核心策略是通过最小化fork/join开销和内存延迟来改进标准的循环级并行性，减少等待时间，与标准的openMP不同的是，高级OpenMP的线程只生成一次，当不需要的时候进入休眠状态，线程的边界是手动指定的，并且同步被降至最低。



## 4. openmp的检测工具

- valgrind: 与Openmp联合使用，检测线程中未初始化的内存或越界访问。
- Call graph: cachegrind 工程代码的调用图以及概要文件
- Allinea/ARM Map: 获得线程启动和barrier的总体成本的高级分析器
- Intel Inspector: 用于检测线程的竟态条件。

## Example 

OpenMp实现递归

```c
double PairwiseSumByTask(double* restrict var, long ncells)
{
   double sum;
   #pragma omp parallel
   {
      #pragma omp master
      {
         sum = PairwiseSumBySubtask(var, 0, ncells);
      }
   }
   return(sum);
}

double PairwiseSumBySubtask(double* restrict var, long nstart, long nend)
{
   long nsize = nend - nstart;
   long nmid = nsize/2;
   double x,y;
   if (nsize == 1){
      return(var[nstart]);
   }

   #pragma omp task shared(x) mergeable final(nsize > 10)   // 启动两个子任务
   x = PairwiseSumBySubtask(var, nstart, nstart + nmid);
   #pragma omp task shared(y) mergeable final(nsize > 10)
   y = PairwiseSumBySubtask(var, nend - nmid, nend);
   #pragma omp taskwait      //等待任务完成

   return(x+y);
}
```



# MPI

## 1. 简介

MPI(Message Passing Interface) 重要性在于，它允许程序员访问额外的计算节点，从而通过向仿真程序中添加更多节点来处理规模越来越大的问题。消息传递是指从一个进程迅速发送到另一个进程的能力。MPI是一种基于库的语言，它不需要特殊的编译器或操作系统的调整， 所有的MPI程序都有一个基本的结构和流程。在程序开始的时候调用MPI_Init, 在程序退出的时候调用MPI_Finalize. 需要包含头文件` "mpi.h" `

**安装**

下载:

- Download `mpich-3.4.1.tar.gz` at `https://www.mpich.org/downloads/`
- 版本会更新，地址应该不会变

解压后进入目录

> ```bash
> tar -xzvf mpich-3.4.1.tar.gz
> cd mpich-3.4.1
> ```

配置

```bash
./configure --prefix=/home/Desktop/HPC/mpich-3.4.1/mpich-install 2>&1 
```

报错:

- - 根据提示加上 `--with-device=ch4:ofi` 即可
  - 加上后再次报错`No Fortran compiler found. If you don't need to build any Fortran  programs, you can disable Fortran support using --disable-fortran. If  you do want to build Fortran programs, you need to install a Fortran  compiler such as gfortran or ifort before you can proceed.`
  - 这是因为没有安装Fortran compiler
  - 根据提示加上 `--disable-fortran` 即可

- ```bash
  ./configure --disable-fortran  --with-device=ch4:ofi  --prefix=/home/Desktop/HPC/mpich-3.4.1/mpich-install 2>&1
  ```

编译:

```bash
make            #等待一段漫长的时间
make install    #权限不够加 sudo
```

添加环境变量

```bash
sudo gedit ~/.bashrc
```

打开`.bashrc` 文件后在末尾添加

```bash
export MPI_ROOT=/home/Desktop/HPC/mpich-3.4.1/mpich-install #这一步对应你自己的安装地址
export PATH=$MPI_ROOT/bin:$PATH
export MANPATH=$MPI_ROOT/man:$MANPATH
```

```bash
source ~/.bashrc
```

- which mpicc 查看位置信息
- mpichversion 查看版本信息，出现版本号说明安装成功



## 2. MPI 函数调用和命令

### 基本的函数调用

```c
ret = MPI_Init(&argc, &argv);
ret = MPI_Finalize();
```



在通信的组中，将所需的进程数量和进程rank 称为通信器。 MPI的一个主要功能是启动远程进程，并将这些进程绑定起来，以便在进程之间进行消息传递。默认的通信器是 `MPI_COMM_WORLD` , 它是由MPI_Init 在每个并行作业开始时设置的。

- 进程: 是一种独立的计算单元，拥有部分内存的所有权并控制用户空间的资源。
- rank: 是一种唯一的， 可移植的标识符，用于区分进程集中的各个进程，通常这个值是从0到进程数减1的整数。

```c
ret = MPI_Comm_rank(MPI_COMM_WORLD, &rank);
ret = MPI_Comm_size(MPI_COMM_WORLD, &nprocs)
```



**现在常用的启动命令是:** 

- mpirun -n <nprocs>
- mpiexec -n <nprocs>
- rprun
- srun

n 是进程数

**Example**

```c
#include <mpi.h>     // mpi的函数和变量的包含文件
#include <stdio.h>
int main(int argc, char **argv)
{
   MPI_Init(&argc, &argv);   // 程序启动后初始化

   int rank, nprocs;
   MPI_Comm_rank(MPI_COMM_WORLD, &rank);   //获取进程的rank
   MPI_Comm_size(MPI_COMM_WORLD, &nprocs); //获取由mpirun 命令确定的程序中的rank number

   printf("Rank %d of %d\n", rank, nprocs);

   MPI_Finalize();    // 完成MPI来同步rank, 然后退出
   return 0;
}
```

```cmake
cmake_minimum_required(VERSION 2.8)
project(MinWorkExampleMPI)

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O3")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fstrict-aliasing -march=native -mtune=native")

# Require MPI for this project:
find_package(MPI REQUIRED)

add_executable(MinWorkExampleMPI MinWorkExampleMPI.c)

set_target_properties(MinWorkExampleMPI PROPERTIES INCLUDE_DIRECTORIES "${MPI_C_INCLUDE_PATH}")
set_target_properties(MinWorkExampleMPI PROPERTIES COMPILE_FLAGS "${MPI_C_COMPILE_FLAGS}")
set_target_properties(MinWorkExampleMPI PROPERTIES LINK_FLAGS "${MPI_C_LINK_FLAGS}")
target_link_libraries(MinWorkExampleMPI PRIVATE "${MPI_C_LIBRARIES}")

# Add a test:
enable_testing()
add_test(SimpleTest ${MPIEXEC} ${MPIEXEC_NUMPROC_FLAG} ${MPIEXEC_MAX_NUMPROCS} ${MPIEXEC_PREFLAGS}
         ${CMAKE_CURRENT_BINARY_DIR}/MinWorkExampleMPI ${MPIEXEC_POSTFLAGS})

# Cleanup
add_custom_target(distclean COMMAND rm -rf CMakeCache.txt CMakeFiles
                  Makefile cmake_install.cmake CTestTestfile.cmake Testing)
```

run:

> cmake .
>
> make 
>
> make test

程序结束之后，执行清理动作

> make clean
>
> make distclean

**基本的发送和接收函数**

```c
MPI_Send(void* data, int count, MPI_Datatype datatype, int dest, int tag, MPI_COMM comm);
MPI_Recv(void *data, int count, MPI_Datatype datatype, int sourece, int tag, MPI_COMM comm, MPI_Statue *status);
```

为了避免阻塞，可调用MPI_Sendrecv， 让MPI 库保证通信的正确性

```c
MPI_Sendrecv(void *send_data, int count, MPI_Datatype datatype, int dest, int tag, void *recv_data, int count, MPI_Datatype datatype, int sourece, int tag, MPI_COMM comm, MPI_Statue *status);
```

异步调用MPI_Isend， MPI_Irecv

```c
   MPI_Request requests[2] = {MPI_REQUEST_NULL, MPI_REQUEST_NULL}; // 定义请求数组

   MPI_Irecv(xrecv, count, MPI_DOUBLE, partner_rank, tag, comm, &requests[0]); 
   MPI_Isend(xsend, count, MPI_DOUBLE, partner_rank, tag, comm, &requests[1]);
   MPI_Waitall(2, requests, MPI_STATUSES_IGNORE); // 调用waitall 等待发送和接收完成
# MPI_Waitall(int count, MPI_Request requests[], MPI_Status statuses[])
```

在使用异步函数开始通信的时候，必须使用

```c
MPI_Request_free() 或
int MPI_Wait(MPI_Request *request, MPI_Status *status)   或
int MPI_Test(MPI_Request *request, int *flag, MPI_Status *status)
来释放句柄， 避免内存泄漏

```



## 3. MPI聚合通信

 ### MPI_Barrier

最简单的聚合通信调用使用 MPI_Barrier, 它主要用于同步MPI通信器中的所有进程。

例如:

```c
#include <mpi.h>
#include <unistd.h>
#include <stdio.h>
int main(int argc, char *argv[])
{
   double start_time, main_time;

   MPI_Init(&argc, &argv);
   int rank;
   MPI_Comm_rank(MPI_COMM_WORLD, &rank);

   // barrier synchronizes all the processes so they start at about the same time
   MPI_Barrier(MPI_COMM_WORLD);
   // get the starting value of the timer using the MPI_Wtime routine
   start_time = MPI_Wtime();

   sleep(30); // represents work being done

   // synchronize the processes, which has the effect that we get the longest time taken
   MPI_Barrier(MPI_COMM_WORLD);
   // get the timer value and subtract off the starting value to get the elapsed time
   main_time = MPI_Wtime() - start_time;
   if (rank == 0) printf("Time for work is %lf seconds\n", main_time);

   MPI_Finalize();
   return 0;
}
```

在启动计时器之前及停止之前插入barrier, 这将迫使所有进程上的计时器几乎在同一时间启动。通过在计时器停止之前插入barrier， 可得到所有进程的最长运行时间。但是在实际使用中，不建议使用，会导致应用程序严重性能下降。

### MPI_Bcast

MPI_Bcast是广播的作用， 主要用途之一是将从输入文件中读取的值发送给所有其他进程，如果每个进程都试图打开一个文件，并且当进程数量很多，那么完成打开文件可能需要几分钟的时间。

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <mpi.h>
int main(int argc, char *argv[])
{
   int rank, input_size;
   char *input_string, *line;
   FILE *fin;

   MPI_Init(&argc, &argv);
   MPI_Comm_rank(MPI_COMM_WORLD, &rank);

   if (rank == 0){
      fin = fopen("../file.in", "r");
      fseek(fin, 0, SEEK_END);             // 指针指向文件末尾
      input_size = ftell(fin);            // 计算文件大小
      fseek(fin, 0, SEEK_SET);            // 设置指针指向文件头
      input_string = (char *)malloc((input_size+1)*sizeof(char));  // 分配大小
      fread(input_string, 1, input_size, fin);
      input_string[input_size] = '\0';
   }

   MPI_Bcast(&input_size, 1, MPI_INT, 0, MPI_COMM_WORLD);  //广播size
   if (rank != 0) input_string = (char *)malloc((input_size+1)*sizeof(char));  // 其他进程分配空间
   MPI_Bcast(input_string, input_size, MPI_CHAR, 0, MPI_COMM_WORLD);

   if (rank == 0) fclose(fin);

   line = strtok(input_string,"\n");
   while (line != NULL){
      printf("%d:input string is %s\n",rank,line);
      line = strtok(NULL,"\n");
   }
   free(input_string);

   MPI_Finalize();
   return 0;
}

```

广播更大的数据块比单独广播多个较小的数据块会取得更好的效果。因此，我们将广播整个文件，首先对文件的大小进行广播，以便每个进程可分配一个合适的输入缓冲区，然后对数据进行广播。

### MPI_Reduce

MPI_Reduce 调用接收一个普通数组或多维数组作为参数，并将这些值合并为一个标量结果， 在约减过程中可执行许多其他的操作，例如:

```c
MPI_MAX (数组中的最大值)
MPI_MIN (数组中的最小值)
MPI_SUM (数组的总和)
MPI_MINLOC(最小值的索引)
MPI_MAXLOC(最大值的索引)
    
MPI_Reduce(&main_time, &max_time, 1, MPI_DOUBLE, MPI_MAX, 0, MPI_COMM_WORLD);
MPI_Reduce(&main_time, &min_time, 1, MPI_DOUBLE, MPI_MIN, 0, MPI_COMM_WORLD);
MPI_Reduce(&main_time, &avg_time, 1, MPI_DOUBLE, MPI_SUM, 0, MPI_COMM_WORLD);
```

约减结果我们将存储在本例的进程 rank0 中(MPI_Reduce中的第6个参数)，但如果希望所有进程都带有这个值，我们将使用`MPI_Allreduce`



### MPI_Gather

收集操作可以描述为整理操作，其中来自于所有处理器的数据将汇集在一起并堆叠到单个数组中。

```c
#include <stdio.h>
#include <time.h>
#include <unistd.h>
#include <mpi.h>
#include "timer.h"
int main(int argc, char *argv[])
{
   int rank, nprocs;
   double total_time;
   struct timespec tstart_time;

   MPI_Init(&argc, &argv);
   MPI_Comm_rank(MPI_COMM_WORLD, &rank);
   MPI_Comm_size(MPI_COMM_WORLD, &nprocs);

   cpu_timer_start(&tstart_time);
   sleep(30); // represents work being done
   total_time += cpu_timer_stop(tstart_time);

   double times[nprocs];    // 收集所有进程的数据
   MPI_Gather(&total_time, 1, MPI_DOUBLE, times, 1, MPI_DOUBLE, 0, MPI_COMM_WORLD);
   if (rank == 0) {    // 在主线程上打印
      for (int i=0; i<nprocs; i++){
         printf("%d:Time for work on rank %d is %lf seconds\n", i, i, times[i]);
      }
   }

   MPI_Finalize();
   return 0;
}
```



### MPI_Scatter / MPI_Scatterv

scatter操作时gather操作的逆向操作。在这个操作中，数据将一个进程发送到通信组中的所有其他进程。scatter操作最常见的用途是在并行策略中将数据组分发给其他进程，由MPI_Scatter 和 MPI_Scatterv来实现。

```c
#include <stdio.h>
#include <stdlib.h>
#include <mpi.h>
int main(int argc, char *argv[])
{
   int rank, nprocs, ncells = 100000;

   MPI_Init(&argc, &argv);
   MPI_Comm comm = MPI_COMM_WORLD;
   MPI_Comm_rank(comm, &rank);
   MPI_Comm_size(comm, &nprocs);

   // Compute the size of the array on every process
   long ibegin = ncells *(rank  )/nprocs;
   long iend   = ncells *(rank+1)/nprocs;
   int  nsize  = (int)(iend-ibegin); // MPI routines can only handle ints

   double *a_global, *a_test;
   if (rank == 0) {
      // Setup data on master process
      a_global = (double *)malloc(ncells*sizeof(double));
      // initializing data and arrays
      for (int i=0; i<ncells; i++) {
         a_global[i] = (double)i;
      }
   }

   // Get the sizes and offsets into global arrays for communication
   int nsizes[nprocs], offsets[nprocs];
   MPI_Allgather(&nsize, 1, MPI_INT, nsizes, 1, MPI_INT, comm);
   offsets[0] = 0;
   for (int i = 1; i<nprocs; i++){
      offsets[i] = offsets[i-1] + nsizes[i-1];
   }

   // Distribute the data onto the other processors
   double *a = (double *)malloc(nsize*sizeof(double));
   MPI_Scatterv(a_global, nsizes, offsets, MPI_DOUBLE, a, nsize, MPI_DOUBLE, 0, comm);

   // Now compute
   for (int i=0; i<nsize; i++){
      a[i] += 1.0;
   }

   if (rank == 0) {
      // Return array data to master process, perhaps for output
      a_test = (double *)malloc(ncells*sizeof(double));
   }

   MPI_Gatherv(a, nsize, MPI_DOUBLE, a_test, nsizes, offsets, MPI_DOUBLE, 0, comm);

   if (rank == 0){
      int ierror = 0;
      for (int i=0; i<ncells; i++){
         if (a_test[i] != a_global[i] + 1.0) {
            printf("Error: index %d a_test %lf a_global %lf\n",
                   i,a_test[i],a_global[i]);
            ierror++;
         }
      }
      printf("Report: Correct results %d errors %d\n",ncells-ierror,ierror);
   }

   free(a);
   if (rank == 0) {
      free(a_global);
      free(a_test);
   }

   MPI_Finalize();
   return 0;
}
```



## 4. 高级MPI

### 使用自定义的MPI数据类型

MPI有一组丰富的函数，可以从基本的MPI类型创建新的自定义MPI数据类型。下面是一些MPI数据类型创建函数的列表:

- **MPI_Type_contiguous**: 将一个连续的数据块转换为一种类型
- **MPI_Type_vector**: 通过跨步数据块创建一个类型
- **MPI_Type_create_subarray**: 创建较大数组的矩形子集
- **MPI_Type_indexed/MPI_Type_create_hindexed**: 创建通过一组块长度和位移描述的不规则索引。
- **MPI_Type_create_struct**: 创建一种数据类型，以可移植方式将数据项封装在结构中。在这种方式中，考虑到了编译器的填充。

描述数据类型并将转换为新数据类型后，必须在使用它之前进行初始化。需要使用下面额外的例程来提交和释放这些类型。类型必须在使用之前提交，并且释放它，从而避免内存泄漏。

- MPI_Type_Commit: 使用所需的内存分配或其他设置，对新的自定义类型进行初始化。
- MPI_Type_Free: 释放数据类型创建中用到的所有内存资源或数据结构。



```c
MPI_Datatype horiz_type, vert_type, depth_type;
int array_sizes[] = {ksize+2*nhalo, jsize+2*nhalo, isize+2*nhalo};

int subarray_starts[] = {0, 0, 0};
int hsubarray_sizes[] = {ksize+2*nhalo, jsize+2*nhalo, nhalo};
MPI_Type_create_subarray(3, array_sizes, hsubarray_sizes, subarray_starts, MPI_ORDER_C, MPI_DOUBLE, &horiz_type);

MPI_Type_commit(&horiz_type);
MPI_Type_commit(&vert_type);
MPI_Type_commit(&depth_type);

MPI_Type_free(&horiz_type);
MPI_Type_free(&vert_type);
MPI_Type_free(&depth_type);
```



### MPI 中的笛卡尔拓扑

- MPI_Dims_create: 计算数组中的有效的值。
- MPI_Cart_create: 接受生成的dims数组和一个periodic输入数组。
- MPI_Cart_coords: 获取进程网格布局
- MPI_Cart_shift： 完成对邻域的获取，第二个参数指定方向，第三个参数指定该方向上的进程位移或数量，输出结果是相邻处理器的rank。





```c
   int dims[3] = {nprocz, nprocy, nprocx}; // needs to be initialized
   int periods[3]={0,0,0};
   int coords[3];
   MPI_Dims_create(nprocs, 3, dims);
   MPI_Comm cart_comm;
   MPI_Cart_create(MPI_COMM_WORLD, 3, dims, periods, 0, &cart_comm);
   MPI_Cart_coords(cart_comm, rank, 3, coords);
   int xcoord = coords[2];
   int ycoord = coords[1];
   int zcoord = coords[0];
   int nleft, nrght, nbot, ntop, nfrnt, nback;
   MPI_Cart_shift(cart_comm, 2, 1, &nleft, &nrght);
   MPI_Cart_shift(cart_comm, 1, 1, &nbot,  &ntop);
   MPI_Cart_shift(cart_comm, 0, 1, &nfrnt, &nback);
```



## 5 MPI和OpenMP混合

MPI和OpenMP的第一步是把MPI_Init 替换成MPI_Init_thread

```c
MPI_Init_thread(&argc, &argv, int thread_model, int *thread_model_provided)
```

MPI 标准定义了四种线程模型。这些模型通过MPI调用提供了不同级别的线程安全，按照线程安全性的递增顺序，如下:

- MPI_THREAD_SINGLE: 执行一个线程
- MPI_THREAD_FUNNELED: 多线程， 但只有主线程进行MPI调用
- MPI_THREAD_SERIALIZED : 多线程，但每次只有一个线程进行MPI 调用
- MPI_THREAD_MULTIPLE: 多线程， 并且多个线程对MPI进行调用

许多应用程序在主循环级别进行通信，OpenMP线程被用于关键的计算循环，MPI_THREAD_FUNNELED 是一个好的选择。

```c
   int provided;
   MPI_Init_thread(&argc, &argv, MPI_THREAD_FUNNELED, &provided);

   int rank, nprocs;
   MPI_Comm_rank(MPI_COMM_WORLD, &rank);
   MPI_Comm_size(MPI_COMM_WORLD, &nprocs);
   if (rank == 0) {
      #pragma omp parallel
      #pragma omp master
         printf("requesting MPI_THREAD_FUNNELED with %d threads\n",
                omp_get_num_threads());
      if (provided != MPI_THREAD_FUNNELED){
         printf("Error: MPI_THREAD_FUNNELED not available. Aborting ...\n");
         MPI_Finalize();
         exit(0);
      }
   }
```







# CUDA

 ## 1. Hello, world

- CMake（3.10 以上），只需在 LANGUAGES 后面加上 CUDA 即可启用。

- 然后在 add_executable 里直接加你的 .cu 文件，和 .cpp 一样。

```cmake
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Release)

project(hellocuda LANGUAGES CXX CUDA)

add_executable(main main.cu)
```

```c
#include<stdio.h>
__global__ void hello_world(void)
{
	printf("GPU: Hello world!\n");
}
int main(int argc, char** argv)
{
	printf("CPU: Hello world!\n");
	hello_world << <1, 10 >> > ();
	cudaDeviceReset(); //if no this line ,it can not output hello world from gpu
	return 0;
}
```



定义函数 kernel，前面加上 `__global__ `修饰符，即可让他在 GPU 上执行。不过调用 kernel 时，不能直接 kernel()，而是要用 kernel<<<1, 1>>>() 这样的三重尖括号语法。•运行以后，就会在 GPU 上执行 printf 了。这里的 kernel 函数在 GPU 上执行，称为核函数，用 `__global__ `修饰的就是核函数. 

![image-20230702152635380](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702152635380.png)



然而如果直接编译运行刚刚那段代码，是不会打印出 Hello, world! 的。这是因为 GPU 和 CPU 之间的通信，为了高效，是**异步**的。也就是 CPU 调用 kernel<<<1, 1>>>() 后，并不会立即在 GPU 上执行完毕，再返回。实际上只是把 kernel 这个任务推送到 GPU 的执行队列上，然后立即返回，并不会等待执行完毕。因此可以调用 cudaDeviceSynchronize()，让 CPU 陷入等待，等 GPU 完成队列的所有任务后再返回。从而能够在 main 退出前等到 kernel 在 GPU 上执行完。

- `__global__ `用于定义核函数，他在 GPU 上执行，从 CPU 端通过三重尖括号语法调用，可以有参数，不可以有返回值。
- 而` __device__` 则用于定义设备函数，他在 GPU 上执行，但是从 GPU 上调用的，而且不需要三重尖括号，和普通函数用起来一样，可以有参数，有返回值。
- `__device__ `将函数定义在 GPU 上，而 `__host__ `则相反，将函数定义在 CPU 上
- 通过 `__host__ __device__` 这样的双重修饰符，可以把函数同时定义在 CPU 和 GPU 上，这样 CPU 和 GPU 都可以调用
- host 可以调用 global；global 可以调用 device；device 可以调用 device。

**声明为内联函数**

inline 在现代 C++ 中的效果是声明一个函数为 weak 符号，和性能优化意义上的内联无关。优化意义上的内联指把函数体直接放到调用者那里去。

因此 CUDA 编译器提供了一个“私货”关键字：`__inline__ `来声明一个函数为内联。不论是 CPU 函数还是 GPU 都可以使用，只要你用的 CUDA 编译器。GCC 编译器相应的私货则是 `__attribute__((“inline”))`

```c
#include <cstdio>
#include <cuda_runtime.h>

__device__ __inline__ void say_hello() {
    printf("Hello, world!\n");
}

__global__ void kernel() {
    say_hello();
}

int main() {
    kernel<<<1, 1>>>();
    cudaDeviceSynchronize();
    return 0;
}

```

注意声明为 `__inline__ `不一定就保证内联了，如果函数太大编译器可能会放弃内联化。因此 CUDA 还提供 __forceinline__ 这个关键字来强制一个函数为内联。GCC 也有相应的 `__attribute__((“always_inline”))`。

此外，还有 `__noinline__` 来禁止内联优化。

**让** **constexpr** **函数自动变成** **CPU** **和** **GPU** **都可以调用**

这样相当于把 constexpr 函数自动变成修饰 __host__ __device__，从而两边都可以调用, 因为 constexpr 通常都是一些可以内联的函数，数学计算表达式之类的，一个个加上太累了，所以产生了这个需求, 不过必须指定 --expt-relaxed-constexpr 这个选项才能用这个特性，我们可以用 CMake 的生成器表达式来实现只对 .cu 文件开启此选项（不然给到 gcc 就出错了）。当然，constexpr 里没办法调用 printf，也不能用 __syncthreads 之类的 GPU 特有的函数，因此也不能完全替代 `__host__ `和 `__device__`

```cmake
add_executable(main main.cu)
target_compile_options(main PUBLIC $<$<COMPILE_LANGUAGE:CUDA>:--expt-relaxed-constexpr>)
```



**通过** **#ifdef** **指令针对** **CPU** **和** **GPU** **生成不同的代码**

CUDA 编译器具有多段编译的特点。一段代码他会先送到 CPU 上的编译器（通常是系统自带的编译器比如 gcc 和 msvc）生成 CPU 部分的指令码。然后送到真正的 GPU 编译器生成 GPU 指令码。最后再链接成同一个文件，看起来好像只编译了一次一样，实际上你的代码会被预处理很多次。他在 GPU 编译模式下会定义 `__CUDA_ARCH__ `这个宏，利用 #ifdef 判断该宏是否定义，就可以判断当前是否处于 GPU 模式，从而实现一个函数针对 GPU 和 CPU 生成两份源码级不同的代码。

```c
#include <cstdio>
#include <cuda_runtime.h>

__host__ __device__ void say_hello() {
#ifdef __CUDA_ARCH__
    printf("Hello, world from GPU!\n");
#else
    printf("Hello, world from CPU!\n");
#endif
}

__global__ void kernel() {
    say_hello();
}

int main() {
    kernel<<<1, 1>>>();
    cudaDeviceSynchronize();
    say_hello();
    return 0;
}
```

其实 `__CUDA_ARCH__` 是一个整数，表示当前编译所针对的 GPU 的架构版本号是多少。这里是 520 表示版本号是 5.2.0，最后一位始终是 0 不用管，我们通常简称他的版本号为 52 就行了。这个版本号是编译时指定的版本，不是运行时检测到的版本。编译器默认就是最老的 52，能兼容所有 GTX900 以上显卡。

**通过** **CMake** **设置架构版本号**

可以用 CMAKE_CUDA_ARCHITECTURES 这个变量，设置要针对哪个架构生成 GPU 指令码, 如果不指定，编译器默认的版本号是 52，他是针对 GTX900 系列显卡的。不过英伟达的架构版本都是向前兼容的，即版本号为 75 的 RTX2080 也可以运行版本号为 52 的指令码，虽然不够优化，但是至少能用。也就是要求：编译期指定的版本 ≤ 运行时显卡的版本。可以指定多个版本号，之间用分号分割。运行时可以自动选择最适合当前显卡的版本号，通常用于打包发布的时候。不过这样会导致 GPU 编译器重复编译很多遍，每次针对不同的架构，所以编译会变得非常慢，生成的可执行文件也会变大。

```cmake
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Release)
set(CMAKE_CUDA_ARCHITECTURES 52;70;75;86)

project(hellocuda LANGUAGES CXX CUDA)

add_executable(main main.cu)
```



## 2. 线程与板块

**三重尖括号里的数字代表什么意思？**

第一个参数指的block的数量， 第二个参数指的是线程的数量。 <<<block, thread>>>

**获取线程编号**

可以通过 threadIdx.x 获取当前线程的编号，我们打印一下试试看。这是 CUDA 中的特殊变量之一，只有在核函数里才可以访问。

可以看到线程编号从0开始计数，打印出了0，1，2。这也是我们指定了线程数量为 3 的缘故。

```c
#include <cstdio>
#include <cuda_runtime.h>

__global__ void kernel() {
    printf("Hello, world!\n");
}

int main() {
    kernel<<<1, 3>>>();
    cudaDeviceSynchronize();
    return 0;
}
```

还可以用 blockDim.x 获取当前线程数量，也就是我们在尖括号里指定的 3。

**线程之上：板块**

CUDA 中还有一个比线程更大的概念，那就是板块（block），一个板块可以有多个线程组成。这就是为什么刚刚获取线程数量的变量用的是 blockDim，实际上 blockDim 的含义是每个板块有多少个线程。•要指定板块的数量，只需调节三重尖括号里第一个参数即可。总之：<<<板块数量，每个板块中的线程数量>>>

**获取板块编号和数量**

板块的编号可以用 blockIdx.x 获取。板块的总数可以用 gridDim.x 获取。板块之间是高度并行的，不保证执行的先后顺序。线程之间也是，这里线程打印顺序没乱，不过是碰巧小于32而已。

**注意不要混淆**

- 当前线程在板块中的编号：threadIdx   

- 当前板块中的线程数量：blockDim    

- 当前板块的编号：blockIdx

- 总的板块数量：gridDim

- 线程(thread)：并行的最小单位

- 板块(block)：包含若干个线程

- 网格(grid)：指整个任务，包含若干个板块

- 从属关系：线程＜板块＜网格

- 调用语法：<<<gridDim, blockDim>>>
- 如需总的线程数量：blockDim * gridDim
- 如需总的线程编号：blockDim * blockIdx + threadIdx

![image-20230702170246892](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702170246892.png)



CUDA 也支持三维的板块和线程区间。

只要在三重尖括号内指定的参数改成 dim3 类型即可。dim3 的构造函数就是接受三个无符号整数（unsigned int）非常简单。

dim3(x, y, z)  这样在核函数里就可以通过 threadIdx.y 获取 y 方向的线程编号，以此类推

需要二维的话，只需要把 dim3 最后一位（z方向）的值设为 1 即可。这样就只有 xy 方向有大小，就相当于二维了，不会有性能损失。实际上一维的 <<<m, n>>> 不过是 <<<dim3(m, 1, 1), dim3(n, 1, 1)>>> 的简写而已。

![image-20230702171256127](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702171256127.png)



**分离** **__device__** **函数的声明和定义：出错**

默认情况下 GPU 函数必须定义在同一个文件里。如果你试图分离声明和定义，调用另一个文件里的 `__device__ `或 `__global__` 函数，就会出错。

**分离** **__device__** **函数的声明和定义：解决**

开启 CMAKE_CUDA_SEPARABLE_COMPILATION 选项（设为 ON），即可启用分离声明和定义的支持。

不过我还是建议把要相互调用的 __device__ 函数放在同一个文件，这样方便编译器自动内联优化

```cmake
set(CMAKE_CUDA_SEPARABLE_COMPILATION ON)
```



从 Kelper 架构开始，__global__ 里可以调用另一个 __global__，也就是说核函数可以调用另一个核函数，且其三重尖括号里的板块数和线程数可以动态指定，无需先传回到 CPU 再进行调用，这是 CUDA 特有的能力。

```c
#include <cstdio>
#include <cuda_runtime.h>

__global__ void another() {
    printf("another: Thread %d of %d\n", threadIdx.x, blockDim.x);
}

__global__ void kernel() {
    printf("kernel: Thread %d of %d\n", threadIdx.x, blockDim.x);
    int numthreads = threadIdx.x * threadIdx.x + 1;
    another<<<1, numthreads>>>();
    printf("kernel: called another with %d threads\n", numthreads);
}

int main() {
    kernel<<<1, 3>>>();
    cudaDeviceSynchronize();
    return 0;
}

```



## 3. 内存管理

**如何从核函数里返回数据**

我们试着把 kernel 的返回类型声明为 int，试图从 GPU 返回数据到 CPU。

但发现这样做会在编译期出错，为什么？

刚刚说了 kernel 的调用是**异步**的，返回的时候，并不会实际让 GPU 把核函数执行完毕，必须 cudaDeviceSynchronize() 等待他执行完毕（和线程的 join 很像）。所以，不可能从 kernel 里通过返回值获取 GPU 数据，因为 kernel 返回时核函数并没有真正在 GPU 上执行。所以核函数返回类型必须是 void。

他们出错时，并不会直接终止程序，也不会抛出 C++ 的异常，而是返回一个错误代码，告诉你出的具体什么错误，这是出于通用性考虑。

这个错误代码的类型是 cudaError_t，其实就是个 enum 类型，相当于 int。可以通过 cudaGetErrorName 获取该 enum 的具体名字。这里显示错误号为 77，具体名字是 cudaErrorIllegalAddress。意思是我们访问了非法的地址，和 CPU 上的 Segmentation Fault 差不多。

```c
#include <cstdio>
#include <cuda_runtime.h>

__global__ void kernel(int *pret) {
    *pret = 42;
}

int main() {
    int ret = 0;
    kernel<<<1, 1>>>(&ret);
    cudaError_t err = cudaDeviceSynchronize();
    printf("error code: %d\n", err);
    printf("error name: %s\n", cudaGetErrorName(err));
    printf("%d\n", ret);
    return 0;
}
```

其实 CUDA toolkit 安装时，会默认附带一系列案例代码，这些案例中提供了一些非常有用的头文件和工具类，比如这个文件：

/opt/cuda/samples/common/inc/helper_cuda.h

把他和 helper_string.h 一起拷到头文件目录里，然后改一下 CMakeLists.txt 让他包含这个头文件目录。

他定义了 checkCudaErrors 这个宏，使用时只需：

`checkCudaErrors(cudaDeviceSynchronize())`

即可自动帮你检查错误代码并打印在终端，然后退出。还会报告出错所在的行号，函数名等，很方便。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"

__global__ void kernel(int *pret) {
    *pret = 42;
}

int main() {
    int ret = 0;
    kernel<<<1, 1>>>(&ret);
    checkCudaErrors(cudaDeviceSynchronize());
    return 0;
}
```

```cmake
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Release)

project(hellocuda LANGUAGES CXX CUDA)

add_executable(main main.cu)
target_include_directories(main PUBLIC ../../include)
```



原来，GPU 和 CPU 各自使用着独立的内存。CPU 的内存称为主机内存(host)。GPU 使用的内存称为设备内存(device)，他是显卡上板载的，速度更快，又称显存。

而不论栈还是 malloc 分配的都是 CPU 上的内存，所以自然是无法被 GPU 访问到。

因此可以用用 cudaMalloc 分配 GPU 上的显存，这样就不出错了，结束时 cudaFree 释放。

cudaMemcpy 会自动进行同步操作，即和 cudaDeviceSynchronize() 等价！因此前面的 cudaDeviceSynchronize() 实际上可以删掉了。

![image-20230702172227908](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702172227908.png)

**统一内存地址技术（Unified Memory）**

还有一种在比较新的显卡上支持的特性，那就是统一内存(managed)，只需把 cudaMalloc 换成 cudaMallocManaged 即可，释放时也是通过 cudaFree。这样分配出来的地址，不论在 CPU 还是 GPU 上都是一模一样的，都可以访问。而且拷贝也会自动按需进行（当从 CPU 访问时），无需手动调用 cudaMemcpy，大大方便了编程人员，特别是含有指针的一些数据结构。

- 主机内存(host)：malloc、free

- 设备内存(device)：cudaMalloc、cudaFree

- 统一内存(managed)：cudaMallocManaged、cudaFree

![image-20230702172328151](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702172328151.png)

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"

__global__ void kernel(int *pret) {
    *pret = 42;
}

int main() {
    int *pret;
    checkCudaErrors(cudaMalloc(&pret, sizeof(int)));
    kernel<<<1, 1>>>(pret);

    int ret;
    checkCudaErrors(cudaMemcpy(&ret, pret, sizeof(int), cudaMemcpyDeviceToHost));
    printf("result: %d\n", ret);

    cudaFree(pret);
    return 0;
}
```



## 4. 数组

**分配数组**

如 malloc 一样，可以用 cudaMalloc 配合 n * sizeof(int)，分配一个大小为 n 的整型数组。这样就会有 n 个连续的 int 数据排列在内存中，而 arr 则是指向其起始地址。然后把 arr 指针传入 kernel，即可在里面用 arr[i] 访问他的第 i 个元素。

然后因为我们用的统一内存(managed)，所以同步以后 CPU 也可以直接读取

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"

__global__ void kernel(int *arr, int n) {
    for (int i = 0; i < n; i++) {
        arr[i] = i;
    }
}

int main() {
    int n = 32;
    int *arr;
    checkCudaErrors(cudaMallocManaged(&arr, n * sizeof(int)));
    kernel<<<1, 1>>>(arr, n);
    checkCudaErrors(cudaDeviceSynchronize());
    for (int i = 0; i < n; i++) {
        printf("arr[%d]: %d\n", i, arr[i]);
    }
    cudaFree(arr);
    return 0;
}
```

**多个线程，并行地给数组赋值**

•刚刚的 for 循环是串行的，我们可以把线程数量调为 n，然后用 threadIdx.x 作为 i 索引。这样就实现了，每个线程负责给数组中一个元素的赋值。

```c
__global__ void kernel(int *arr, int n) {
    int i = threadIdx.x;
    arr[i] = i;
}
```

**小技巧：网格跨步循环（grid-stride loop**

无论调用者指定了多少个线程（blockDim），都能自动根据给定的 n 区间循环，不会越界，也不会漏掉几个元素。

这样一个 for 循环非常符合 CPU 上常见的 parallel for 的习惯，又能自动匹配不同的 blockDim，看起来非常方便。

```c
__global__ void kernel(int *arr, int n) {
    for (int i = threadIdx.x; i < n; i += blockDim.x) {
        arr[i] = i;
    }
}
```

**从线程到板块**

核函数内部，用之前说到的 blockDim.x * blockIdx.x + threadIdx.x 来获取线程在整个网格中编号。

外部调用者，则是根据不同的 n 决定板块的数量（gridDim），而每个板块具有的线程数量（blockDim）则是固定的 128。

因此，我们可以用 n / 128 作为 gridDim，这样总的线程数刚好的 n，实现了每个线程负责处理一个元素。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"

__global__ void kernel(int *arr, int n) {
    int i = blockDim.x * blockIdx.x + threadIdx.x;
    arr[i] = i;
}

int main() {
    int n = 65536;
    int *arr;
    checkCudaErrors(cudaMallocManaged(&arr, n * sizeof(int)));

    int nthreads = 128;
    int nblocks = n / nthreads;
    kernel<<<nblocks, nthreads>>>(arr, n);

    checkCudaErrors(cudaDeviceSynchronize());
    for (int i = 0; i < n; i++) {
        printf("arr[%d]: %d\n", i, arr[i]);
    }

    cudaFree(arr);
    return 0;
}
```

但这样的话，n 只能是的 128 的整数倍，如果不是就会漏掉最后几个元素。

解决方法就是：采用向上取整的除法。

可是 C 语言好像没有向上整除的除法这个运算符？没关系，用这个式子即可：

(n + nthreads - 1) / nthreads

例如：(7 + 3) / 4 = 2，(8 + 3 / 4) = 2。

由于向上取整，这样会多出来一些线程，因此要在 kernel 内判断当前 i 是否超过了 n，如果超过就要提前退出，防止越界。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"

__global__ void kernel(int *arr, int n) {
    int i = blockDim.x * blockIdx.x + threadIdx.x;
    if (i > n) return;
    arr[i] = i;
}

int main() {
    int n = 65535;
    int *arr;
    checkCudaErrors(cudaMallocManaged(&arr, n * sizeof(int)));

    int nthreads = 128;
    int nblocks = (n + nthreads - 1) / nthreads;
    kernel<<<nblocks, nthreads>>>(arr, n);

    checkCudaErrors(cudaDeviceSynchronize());
    for (int i = 0; i < n; i++) {
        printf("arr[%d]: %d\n", i, arr[i]);
    }

    cudaFree(arr);
    return 0;
}
```

**网格跨步循环：应用于线程和板块一起上的情况**

网格跨步循环实际上本来是这样，利用扁平化的线程数量和线程编号实现动态大小。

同样，无论调用者指定每个板块多少线程（blockDim），总共多少板块（gridDim）。都能自动根据给定的 n 区间循环，不会越界，也不会漏掉几个元素。

这样一个 for 循环非常符合 CPU 上常见的 parallel for 的习惯，又能自动匹配不同的 blockDim 和 gridDim，看起来非常方便。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"

__global__ void kernel(int *arr, int n) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n; i += blockDim.x * gridDim.x) {
        arr[i] = i;
    }
}

int main() {
    int n = 65536;
    int *arr;
    checkCudaErrors(cudaMallocManaged(&arr, n * sizeof(int)));

    kernel<<<32, 128>>>(arr, n);

    checkCudaErrors(cudaDeviceSynchronize());
    for (int i = 0; i < n; i++) {
        printf("arr[%d]: %d\n", i, arr[i]);
    }

    cudaFree(arr);
    return 0;
}
```



## 5. C++ 封装

你知道吗？std::vector 作为模板类，其实有两个模板参数：std::vector<T, AllocatorT>

那为什么我们平时只用了 std::vector<T> 呢？因为第二个参数默认是 std::allocator<T>。

也就是 std::vector<T> 等价于 std::vector<T, std::allocator<T>>。

std::allocator<T> 的功能是负责分配和释放内存，初始化 T 对象等等。

他具有如下几个成员函数：

T *allocate(size_t n)            // 分配长度为n，类型为T的数组，返回其起始地址

void deallocate(T *p, size_t n)    // 释放长度为n，起始地址为p，类型为T的数组



**抽象的** **std::allocator** **接口**

vector 会调用 std::allocator<T> 的 allocate/deallocate 成员函数，他又会去调用标准库的 malloc/free 分配和释放内存空间（即他分配是的 CPU 内存）。

我们可以自己定义一个和 std::allocator<T> 一样具有 allocate/deallocate 成员函数的类，这样就可以“骗过”vector，让他不是在 CPU 内存中分配，而是在 CUDA 的统一内存(managed)上分配。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <vector>

template <class T>
struct CudaAllocator {
    using value_type = T;

    T *allocate(size_t size) {
        T *ptr = nullptr;
        checkCudaErrors(cudaMallocManaged(&ptr, size * sizeof(T)));
        return ptr;
    }

    void deallocate(T *ptr, size_t size = 0) {
        checkCudaErrors(cudaFree(ptr));
    }
};

__global__ void kernel(int *arr, int n) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n; i += blockDim.x * gridDim.x) {
        arr[i] = i;
    }
}

int main() {
    int n = 65536;
    std::vector<int, CudaAllocator<int>> arr(n);

    kernel<<<32, 128>>>(arr.data(), n);

    checkCudaErrors(cudaDeviceSynchronize());
    for (int i = 0; i < n; i++) {
        printf("arr[%d]: %d\n", i, arr[i]);
    }

    return 0;
}
```

**进一步：避免初始化为0**

vector 在初始化的时候（或是之后 resize 的时候）会调用所有元素的无参构造函数，对 int 类型来说就是零初始化。然而这个初始化会是在 CPU 上做的，因此我们需要禁用他。

可以通过给 allocator 添加 construct 成员函数，来魔改 vector 对元素的构造。默认情况下他可以有任意多个参数，而如果没有参数则说明是无参构造函数。

因此我们只需要判断是不是有参数，然后是不是传统的 C 语言类型（plain-old-data），如果是，则跳过其无参构造，从而避免在 CPU 上低效的零初始化。

```c
template <class T>
struct CudaAllocator {
    using value_type = T;

    T *allocate(size_t size) {
        T *ptr = nullptr;
        checkCudaErrors(cudaMallocManaged(&ptr, size * sizeof(T)));
        return ptr;
    }

    void deallocate(T *ptr, size_t size = 0) {
        checkCudaErrors(cudaFree(ptr));
    }

    template <class ...Args>
    void construct(T *p, Args &&...args) {
        if constexpr (!(sizeof...(Args) == 0 && std::is_pod_v<T>))
            ::new((void *)p) T(std::forward<Args>(args)...);
    }
};
```

**进一步：核函数可以是一个模板函数**

刚刚说过 CUDA 的优势在于对 C++ 的完全支持。所以 `__global__ `修饰的核函数自然也是可以为模板函数的。

调用模板时一样可以用自动参数类型推导，如有手动指定的模板参数（单尖括号）请放在三重尖括号的前面。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <vector>

template <class T>
struct CudaAllocator {
    using value_type = T;

    T *allocate(size_t size) {
        T *ptr = nullptr;
        checkCudaErrors(cudaMallocManaged(&ptr, size * sizeof(T)));
        return ptr;
    }

    void deallocate(T *ptr, size_t size = 0) {
        checkCudaErrors(cudaFree(ptr));
    }

    template <class ...Args>
    void construct(T *p, Args &&...args) {
        if constexpr (!(sizeof...(Args) == 0 && std::is_pod_v<T>))
            ::new((void *)p) T(std::forward<Args>(args)...);
    }
};

template <int N, class T>
__global__ void kernel(T *arr) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < N; i += blockDim.x * gridDim.x) {
        arr[i] = i;
    }
}

int main() {
    constexpr int n = 65536;
    std::vector<int, CudaAllocator<int>> arr(n);

    kernel<n><<<32, 128>>>(arr.data());

    checkCudaErrors(cudaDeviceSynchronize());
    for (int i = 0; i < n; i++) {
        printf("arr[%d]: %d\n", i, arr[i]);
    }

    return 0;
}
```

**进一步：核函数可以接受函子（functor），实现函数式编程**

不过要注意三点：

1.这里的 Func 不可以是 Func const &，那样会变成一个指向 CPU 内存地址的指针，从而出错。所以 CPU 向 GPU 的传参必须按值传。

2.做参数的这个函数必须是一个有着成员函数 operator() 的类型，即 functor 类。而不能是独立的函数，否则报错。

3.这个函数必须标记为 `__device__`，即 GPU 上的函数，否则会变成 CPU 上的函数。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <vector>

template <class T>
struct CudaAllocator {
    using value_type = T;

    T *allocate(size_t size) {
        T *ptr = nullptr;
        checkCudaErrors(cudaMallocManaged(&ptr, size * sizeof(T)));
        return ptr;
    }

    void deallocate(T *ptr, size_t size = 0) {
        checkCudaErrors(cudaFree(ptr));
    }

    template <class ...Args>
    void construct(T *p, Args &&...args) {
        if constexpr (!(sizeof...(Args) == 0 && std::is_pod_v<T>))
            ::new((void *)p) T(std::forward<Args>(args)...);
    }
};

template <class Func>
__global__ void parallel_for(int n, Func func) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n; i += blockDim.x * gridDim.x) {
        func(i);
    }
}

struct MyFunctor {
    __device__ void operator()(int i) const {
        printf("number %d\n", i);
    }
};

int main() {
    int n = 65536;

    parallel_for<<<32, 128>>>(n, MyFunctor{});

    checkCudaErrors(cudaDeviceSynchronize());

    return 0;
}
```

**进一步：函子可以是** **lambda** **表达式**

可以直接写 lambda 表达式，不过必须在 [] 后，() 前，插入 __device__ 修饰符。

而且需要开启 --extended-lambda 开关。

为了只对 .cu 文件开启这个开关，可以用 CMake 的生成器表达式，限制 flag 只对 CUDA 源码生效，这样可以混合其他 .cpp 文件也不会发生 gcc 报错的情况了。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <vector>

template <class T>
struct CudaAllocator {
    using value_type = T;

    T *allocate(size_t size) {
        T *ptr = nullptr;
        checkCudaErrors(cudaMallocManaged(&ptr, size * sizeof(T)));
        return ptr;
    }

    void deallocate(T *ptr, size_t size = 0) {
        checkCudaErrors(cudaFree(ptr));
    }

    template <class ...Args>
    void construct(T *p, Args &&...args) {
        if constexpr (!(sizeof...(Args) == 0 && std::is_pod_v<T>))
            ::new((void *)p) T(std::forward<Args>(args)...);
    }
};

template <class Func>
__global__ void parallel_for(int n, Func func) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n; i += blockDim.x * gridDim.x) {
        func(i);
    }
}

int main() {
    int n = 65536;

    parallel_for<<<32, 128>>>(n, [] __device__ (int i) {
        printf("number %d\n", i);
    });

    checkCudaErrors(cudaDeviceSynchronize());

    return 0;
}

```

```cmake
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Release)

project(hellocuda LANGUAGES CXX CUDA)

add_executable(main main.cu)
target_include_directories(main PUBLIC ../../include)
target_compile_options(main PUBLIC $<$<COMPILE_LANGUAGE:CUDA>:--extended-lambda>)
```

**如何捕获外部变量？**

如果试图用 [&] 捕获变量是会出错的，毕竟这时候捕获到的是堆栈（CPU内存）上的变量 arr 本身，而不是 arr 所指向的内存地址（GPU内存）。你可能会想，是不是可以用 [=] 按值捕获，这样捕获到的就是指针了吧？错了，不要忘了我们第二课说过，vector 的拷贝是深拷贝（绝大多数C++类都是深拷贝，除了智能指针和原始指针）。这样只会把 vector 整个地拷贝到 GPU 上！而不是浅拷贝其起始地址指针。

正确的做法是先获取 arr.data() 的值到 arr_data 变量，然后用 [=] 按值捕获 arr_data，函数体里面也通过 arr_data 来访问 arr。

为什么这样？因为 data() 返回一个起始地址的原始指针，而原始指针是浅拷贝的，所以可以拷贝到 GPU 上，让他访问。这样和之前作为核函数参数是一样的，不过是作为 Func 结构体统一传入了。或者在 [] 里这样直接写自定义捕获的表达式也是可以的，这样就可以用同一变量名.

```c
    parallel_for<<<32, 128>>>(n, [arr = arr.data()] __device__ (int i) {
        arr[i] = i;
    });
```



## 6. thrust 库

**替换成 CUDA 官方提供的 thrust::universal_vector**

•虽然自己实现 CudaAllocator 很有趣，也帮助我们理解了底层原理。但是既然 CUDA 官方已经提供了 thrust 库，那就用他们的好啦。

•universal_vector 会在统一内存上分配，因此不论 GPU 还是 CPU 都可以直接访问到。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <thrust/universal_vector.h>

template <class Func>
__global__ void parallel_for(int n, Func func) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n; i += blockDim.x * gridDim.x) {
        func(i);
    }
}

int main() {
    int n = 65536;
    float a = 3.14f;
    thrust::universal_vector<float> x(n);
    thrust::universal_vector<float> y(n);

    for (int i = 0; i < n; i++) {
        x[i] = std::rand() * (1.f / RAND_MAX);
        y[i] = std::rand() * (1.f / RAND_MAX);
    }

    parallel_for<<<n / 512, 128>>>(n, [a, x = x.data(), y = y.data()] __device__ (int i) {
        x[i] = a * x[i] + y[i];
    });
    checkCudaErrors(cudaDeviceSynchronize());

    for (int i = 0; i < n; i++) {
        printf("x[%d] = %f\n", i, x[i]);
    }

    return 0;
}

```

```cmake
cmake_minimum_required(VERSION 3.10)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Release)

project(hellocuda LANGUAGES CXX CUDA)

add_executable(main main.cu)
target_include_directories(main PUBLIC ../../include)
target_compile_options(main PUBLIC $<$<COMPILE_LANGUAGE:CUDA>:--extended-lambda>)
target_compile_options(main PUBLIC $<$<COMPILE_LANGUAGE:CUDA>:--expt-relaxed-constexpr>)

```

**换成分离的** **device_vector** **和** **host_vector**

而 device_vector 则是在 GPU 上分配内存，host_vector 在 CPU 上分配内存。

可以通过 = 运算符在 device_vector 和 host_vector 之间拷贝数据，他会自动帮你调用 cudaMemcpy，非常智能。

比如这里的 x_dev = x_host 会把 CPU 内存中的 x_host 拷贝到 GPU 的 x_dev 上

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <thrust/device_vector.h>
#include <thrust/host_vector.h>

template <class Func>
__global__ void parallel_for(int n, Func func) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n; i += blockDim.x * gridDim.x) {
        func(i);
    }
}

int main() {
    int n = 65536;
    float a = 3.14f;
    thrust::host_vector<float> x_host(n);
    thrust::host_vector<float> y_host(n);

    for (int i = 0; i < n; i++) {
        x_host[i] = std::rand() * (1.f / RAND_MAX);
        y_host[i] = std::rand() * (1.f / RAND_MAX);
    }

    thrust::device_vector<float> x_dev = x_host;
    thrust::device_vector<float> y_dev = x_host;

    parallel_for<<<n / 512, 128>>>(n, [a, x_dev = x_dev.data(), y_dev = y_dev.data()] __device__ (int i) {
        x_dev[i] = a * x_dev[i] + y_dev[i];
    });

    x_host = x_dev;

    for (int i = 0; i < n; i++) {
        printf("x[%d] = %f\n", i, x_host[i]);
    }

    return 0;
}
```

**模板函数：thrust::generate**

thrust 提供了很多类似于标准库里的模板函数，比如 thrust::generate(b, e, f) 对标 std::generate，用于批量调用 f() 生成一系列（通常是随机）数，写入到 [b, e) 区间。

其前两个参数是 device_vector 或 host_vector 的迭代器，可以通过成员函数 begin() 和 end() 得到。第三个参数可以是任意函数，这里用了 lambda 表达式。

```c
int main() {
    int n = 65536;
    float a = 3.14f;
    thrust::host_vector<float> x_host(n);
    thrust::host_vector<float> y_host(n);

    auto float_rand = [] {
        return std::rand() * (1.f / RAND_MAX);
    };
    thrust::generate(x_host.begin(), x_host.end(), float_rand);
    thrust::generate(y_host.begin(), y_host.end(), float_rand);

    thrust::device_vector<float> x_dev = x_host;
    thrust::device_vector<float> y_dev = x_host;

    parallel_for<<<n / 512, 128>>>(n, [a, x_dev = x_dev.data(), y_dev = y_dev.data()] __device__ (int i) {
        x_dev[i] = a * x_dev[i] + y_dev[i];
    });

    x_host = x_dev;

    for (int i = 0; i < n; i++) {
        printf("x[%d] = %f\n", i, x_host[i]);
    }

    return 0;
}
```

**模板函数：thrust::or_each**

同理，还有 thrust::for_each(b, e, f) 对标 std::for_each。他会把 [b, e) 区间的每个元素 x 调用一遍 f(x)。这里的 x 实际上是一个引用。如果 b 和 e 是常值迭代器则是个常引用，可以用 cbegin()，cend() 获取常值迭代器。

当然还有 thrust::reduce，thrust::sort，thrust::find_if，thrust::count_if，thrust::reverse，thrust::inclusive_scan 等

**thrust** **模板函数的特点：根据容器类型，自动决定在** **CPU** **还是** **GPU** **执行**

for_each 可以用于 device_vector 也可用于 host_vector。当用于 host_vector 时则函数是在 CPU 上执行的，用于 device_vector 时则是在 GPU 上执行的。

这就是为什么我们用于 x_host 那个 for_each 的 lambda 没有修饰，而用于 x_dev 的那个 lambda 需要修饰` __device__`。

```c
    int n = 65536;
    float a = 3.14f;
    thrust::host_vector<float> x_host(n);
    thrust::host_vector<float> y_host(n);

    thrust::for_each(x_host.begin(), x_host.end(), [] (float &x) {
        x = std::rand() * (1.f / RAND_MAX);
    });
    thrust::for_each(y_host.begin(), y_host.end(), [] (float &y) {
        y = std::rand() * (1.f / RAND_MAX);
    });

    thrust::device_vector<float> x_dev = x_host;
    thrust::device_vector<float> y_dev = x_host;

    thrust::for_each(x_dev.begin(), x_dev.end(), [] __device__ (float &x) {
        x += 100.f;
    });

    thrust::for_each(x_dev.cbegin(), x_dev.cend(), [] __device__ (float const &x) {
        printf("%f\n", x);
    });
```



## 7. 原子操作

**经典案例：数组求和**

如何并行地对数组进行求和操作？

首先让我们试着用串行的思路来解题。

因为 `__global__ `函数不能返回值，只能通过指针。因此我们先分配一个大小为 1 的 sum 数组，其中 sum[0] 用来返回数组的和。这样我们同步之后就可以通过 sum[0] 看到求和的结果了。

可是算出来的结果却明显不对，为什么？

这是因为 GPU 上的线程是并行执行的，然而 sum[0] += arr[i] 这个操作，实际上被拆分成四步：

读取 sum[0] 到寄存器A, 读取 arr[i] 到寄存器B, 让寄存器A的值加上寄存器B的值, 写回寄存器A到 sum[0]

这样有什么问题呢？

•假如有两个线程分别在 i=0 和 i=1，同时执行：

•线程0：读取 sum[0] 到寄存器A（A=0）

•线程1：读取 sum[0] 到寄存器A（A=0）

•线程0：读取 arr[0] 到寄存器B（B=arr[0]）

•线程1：读取 arr[1] 到寄存器B（B=arr[1]）

•线程0：让寄存器A加上寄存器B（A=arr[0]）

•线程1：让寄存器A加上寄存器B（A=arr[1]）

•线程0：写回寄存器A到 sum[0]（sum[0]=arr[0]）

•线程1：写回寄存器A到 sum[0]（sum[0]=arr[1]）

•这样一来最后 sum[0] 的值是 arr[1]。而不是我们期望的 arr[0] + arr[1]，即算出来的总和变少了！

**解决：使用原子操作**

•所以，熟悉 CPU 上并行编程的同学们可能就明白了，要用 atomic 对吧！

•原子操作的功能就是保证读取/加法/写回三个操作，不会有另一个线程来打扰。

•CUDA 也提供了这种函数，即 atomicAdd。效果和 += 一样，不过是原子的。他的第一个参数是个指针，指向要修改的地址。第二个参数是要增加多少。也就是说：

•atomicAdd(dst, src) 和 *dst += src 差不多。

```c
__global__ void parallel_sum(int *sum, int const *arr, int n) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n; i += blockDim.x * gridDim.x) {
        atomicAdd(&sum[0], arr[i]);
    }
}
```



**atomicAdd：会返回旧值**

•old = atomicAdd(dst, src) 其实相当于：

•old = *dst; *dst += src;

•利用这一点可以实现往一个全局的数组 res 里追加数据的效果（push_back），其中 sum 起到了记录当前数组大小的作用。

•因为返回的旧值就相当于在数组里“分配”到了一个位置一样，不会被别人占据。

**其他原子操作**

•atomicAdd(dst, src)：*dst += src

•atomicSub(dst, src)：*dst -= src

•atomicOr(dst, src)：*dst |= src

•atomicAnd(dst, src)：*dst &= src

•atomicXor(dst, src)：*dst ^= src

•atomicMax(dst, src)：*dst = std::max(*dst, src)

•atomicMin(dst, src)：*dst = std::min(*dst, src)

当然，他们也都会返回旧值（如果需要的话）

```c
__global__ void parallel_sum(int *sum, int const *arr, int n) {
    int local_sum = 0;
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n; i += blockDim.x * gridDim.x) {
        local_sum += arr[i];
    }
    atomicAdd(&sum[0], local_sum);
}
```

**atomicExch：原子地写入并读取旧值**

•除了刚刚那几个带有运算的原子操作，也有这种单纯是写入而没有读出的。

•old = atomicExch(dst, src) 相当于：

•old = *dst; *dst = src;

•注：Exch是exchange的简写，对标的是std::atomic的exchange函数。

**atomicCAS：原子地判断是否相等，相等则写入，并读取旧值**

•old = atomicCAS(dst, cmp, src) 相当于：

•old = *dst;

•if (old == cmp)

• *dst = src;

•为什么需要这么复杂的一个原子指令？

•atomicCAS 的作用在于他可以用来实现任意 CUDA 没有提供的原子读-修改-写回指令。比如这里我们通过 atomicCAS 实现了整数 atomicAdd 同样的效果。

```c
__device__ __inline__ int my_atomic_add(int *dst, int src) {
    int old = *dst, expect;
    do {
        expect = old;
        old = atomicCAS(dst, expect, expect + src);
    } while (expect != old);
    return old;
}
```



## 8. 板块与共享内存

**到底为什么需要区分出板块的概念？**

•之前说到实际的线程数量就是板块数量(gridDim)乘以每板块线程数量(blockDim)。

•那么为什么中间要插一个板块呢？感觉很不直观，不如直接说线程数量不就好了？

•这还得从 GPU 的硬件架构说起。

![image-20230702191348825](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702191348825.png)

**SM（Streaming Multiprocessors）与板块（block）**

- GPU 是由多个流式多处理器（SM）组成的。每个 SM 可以处理一个或多个板块。
- SM 又由多个流式单处理器（SP）组成。每个 SP 可以处理一个或多个线程。
- 每个 SM 都有自己的一块共享内存（shared memory），他的性质类似于 CPU 中的缓存——和主存相比很小，但是很快，用于缓冲临时数据。还有点特殊的性质，我们稍后会讲。
- 通常板块数量总是大于 SM 的数量，这时英伟达驱动就会在多个 SM 之间调度你提交的各个板块。正如操作系统在多个 CPU 核心之间调度线程那样……
- 不过有一点不同，GPU 不会像 CPU 那样做时间片轮换——板块一旦被调度到了一个 SM 上，就会一直执行，直到他执行完退出，这样的好处是不存在保存和切换上下文（寄存器，共享内存等）的开销，毕竟 GPU 的数据量比较大，禁不起这样切换来切换去……
- 一个 SM 可同时运行多个板块，这时多个板块共用同一块共享内存（每块分到的就少了）。
- 而板块内部的每个线程，则是被进一步调度到 SM 上的每个 SP。

**无原子的解决方案：sum变成数组**

•刚刚的数组求和例子，其实可以不需要原子操作。

•首先，声明 sum 为比原数组小 1024 倍的数组。

•然后在 GPU 上启动 n / 1024 个线程，每个负责原数组中 1024 个数的求和，然后写入到 sum 的对应元素中去。

•因为每个线程都写入了不同的地址，所以不存在任何冲突，也不需要原子操作了。

•然后求出的大小为 n / 1024 的数组，已经足够小了，可以直接在 CPU 上完成最终的求和。也就是 GPU 先把数据尺寸缩减 1024 倍到 CPU 可以接受的范围内，然后让 CPU 完成的思路。

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <vector>
#include "CudaAllocator.h"
#include "ticktock.h"

__global__ void parallel_sum(int *sum, int const *arr, int n) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n / 1024; i += blockDim.x * gridDim.x) {
        int local_sum = 0;
        for (int j = i * 1024; j < i * 1024 + 1024; j++) {
            local_sum += arr[j];
        }
        sum[i] = local_sum;
    }
}

int main() {
    int n = 1<<24;
    std::vector<int, CudaAllocator<int>> arr(n);
    std::vector<int, CudaAllocator<int>> sum(n / 1024);

    for (int i = 0; i < n; i++) {
        arr[i] = std::rand() % 4;
    }

    TICK(parallel_sum);
    parallel_sum<<<n / 1024 / 128, 128>>>(sum.data(), arr.data(), n);
    checkCudaErrors(cudaDeviceSynchronize());

    int final_sum = 0;
    for (int i = 0; i < n / 1024; i++) {
        final_sum += sum[i];
    }
    TOCK(parallel_sum);

    printf("result: %d\n", final_sum);

    return 0;
}
```

**先读取到线程局部数组，然后分步缩减**

•刚刚我们直接用了一个 for 循环迭代所有1024个元素，实际上内部仍然是一个串行的过程，数据是强烈依赖的（local_sum += arr[j] 可以体现出，下一时刻的 local_sum 依赖于上一时刻的 local_sum）。

要消除这种依赖，可以通过右边这样的逐步缩减，这样每个 for 循环内部都是没有数据依赖，从而是可以并行的（对 CPU 而言是 SIMD 和指令级并行，虽然 GPU 没有，但为了引出共享内存的概念我才这样改）

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <vector>
#include "CudaAllocator.h"
#include "ticktock.h"

__global__ void parallel_sum(int *sum, int const *arr, int n) {
    for (int i = blockDim.x * blockIdx.x + threadIdx.x;
         i < n / 1024; i += blockDim.x * gridDim.x) {
        int local_sum[1024];
        for (int j = 0; j < 1024; j++) {
            local_sum[j] = arr[i * 1024 + j];
        }
        for (int j = 0; j < 512; j++) {
            local_sum[j] += local_sum[j + 512];
        }
        for (int j = 0; j < 256; j++) {
            local_sum[j] += local_sum[j + 256];
        }
        for (int j = 0; j < 128; j++) {
            local_sum[j] += local_sum[j + 128];
        }
        for (int j = 0; j < 64; j++) {
            local_sum[j] += local_sum[j + 64];
        }
        for (int j = 0; j < 32; j++) {
            local_sum[j] += local_sum[j + 32];
        }
        for (int j = 0; j < 16; j++) {
            local_sum[j] += local_sum[j + 16];
        }
        for (int j = 0; j < 8; j++) {
            local_sum[j] += local_sum[j + 8];
        }
        for (int j = 0; j < 4; j++) {
            local_sum[j] += local_sum[j + 4];
        }
        for (int j = 0; j < 2; j++) {
            local_sum[j] += local_sum[j + 2];
        }
        for (int j = 0; j < 1; j++) {
            local_sum[j] += local_sum[j + 1];
        }
        sum[i] = local_sum[0];
    }
}

int main() {
    int n = 1<<24;
    std::vector<int, CudaAllocator<int>> arr(n);
    std::vector<int, CudaAllocator<int>> sum(n / 1024);

    for (int i = 0; i < n; i++) {
        arr[i] = std::rand() % 4;
    }

    TICK(parallel_sum);
    parallel_sum<<<n / 1024 / 128, 128>>>(sum.data(), arr.data(), n);
    checkCudaErrors(cudaDeviceSynchronize());

    int final_sum = 0;
    for (int i = 0; i < n / 1024; i++) {
        final_sum += sum[i];
    }
    TOCK(parallel_sum);

    printf("result: %d\n", final_sum);

    return 0;
}

```

![image-20230702191923294](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702191923294.png)

**板块的共享内存（shared memory）**

•刚刚已经实现了无数据依赖可以并行的 for，那么如何把他真正变成并行的呢？这就是板块的作用了，我们可以把刚刚的线程升级为板块，刚刚的 for 升级为线程，然后把刚刚 local_sum 这个线程局部数组升级为板块局部数组。那么如何才能实现**板块局部数组**呢？

•同一个板块中的每个线程，都共享着一块存储空间，他就是共享内存。在 CUDA 的语法中，共享内存可以通过定义一个修饰了 __shared__ 的变量来创建。因此我们可以把刚刚的 local_sum 声明为 `__shared__ `就可以让他从每个线程有一个，升级为每个板块有一个了。

•然后把刚刚的 j 换成板块编号，i 换成线程编号就好啦。

```c
__global__ void parallel_sum(int *sum, int const *arr, int n) {
    __shared__ int local_sum[1024];
    int j = threadIdx.x;
    int i = blockIdx.x;
    local_sum[j] = arr[i * 1024 + j];
    if (j < 512) {
        local_sum[j] += local_sum[j + 512];
    }
    if (j < 256) {
        local_sum[j] += local_sum[j + 256];
    }
    if (j < 128) {
        local_sum[j] += local_sum[j + 128];
    }
    if (j < 64) {
        local_sum[j] += local_sum[j + 64];
    }
    if (j < 32) {
        local_sum[j] += local_sum[j + 32];
    }
    if (j < 16) {
        local_sum[j] += local_sum[j + 16];
    }
    if (j < 8) {
        local_sum[j] += local_sum[j + 8];
    }
    if (j < 4) {
        local_sum[j] += local_sum[j + 4];
    }
    if (j < 2) {
        local_sum[j] += local_sum[j + 2];
    }
    if (j == 0) {
        sum[i] = local_sum[0] + local_sum[1];
    }
}
```

•但是刚刚算出来的结果好像不对了？

•这是因为 SM 执行一个板块中的线程时，并不是全部同时执行的。而是一会儿执行这个线程，一会儿执行那个线程。有可能一个线程已经执行到 if (j < 32) 了，而另一个线程还没执行完 if (j < 64)，从而出错。可是为什么 GPU 要这样设计？

•因为其中某个线程有可能因为在等待内存数据的抵达，这时大可以切换到另一个线程继续执行计算任务，等这个线程陷入内存等待时，原来那个线程说不定就好了呢？（记得上节课说过内存延迟是阻碍 CPU 性能提升的一大瓶颈，GPU 也是如此。CPU 解决方案是超线程技术，一个物理核提供两个逻辑核，当一个逻辑核陷入内存等待时切换到另一个逻辑核上执行，避免空转。GPU 的解决方法就是单个 SM 执行很多个线程，然后在遇到内存等待时，就自动切换到另一个线程）

**板块内线程的同步**

•因此，我们可以给每个 if 分支后面加上 __syncthreads() 指令。

•他的功能是，强制同步当前板块内的所有线程。也就是让所有线程都运行到 __syncthreads() 所在位置以后，才能继续执行下去。

•这样就能保证之前其他线程的 local_sum 都已经写入成功了。

![image-20230702192212782](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702192212782.png)

```c
__global__ void parallel_sum(int *sum, int const *arr, int n) {
    __shared__ int local_sum[1024];
    int j = threadIdx.x;
    int i = blockIdx.x;
    local_sum[j] = arr[i * 1024 + j];
    __syncthreads();
    if (j < 512) {
        local_sum[j] += local_sum[j + 512];
    }
    __syncthreads();
    if (j < 256) {
        local_sum[j] += local_sum[j + 256];
    }
    __syncthreads();
    if (j < 128) {
        local_sum[j] += local_sum[j + 128];
    }
    __syncthreads();
    if (j < 64) {
        local_sum[j] += local_sum[j + 64];
    }
    __syncthreads();
    if (j < 32) {
        local_sum[j] += local_sum[j + 32];
    }
    __syncthreads();
    if (j < 16) {
        local_sum[j] += local_sum[j + 16];
    }
    __syncthreads();
    if (j < 8) {
        local_sum[j] += local_sum[j + 8];
    }
    __syncthreads();
    if (j < 4) {
        local_sum[j] += local_sum[j + 4];
    }
    __syncthreads();
    if (j < 2) {
        local_sum[j] += local_sum[j + 2];
    }
    __syncthreads();
    if (j == 0) {
        sum[i] = local_sum[0] + local_sum[1];
    }
}
```

**线程组（warp）：32个线程为一组**

•其实，SM 对线程的调度是按照 32 个线程为一组来调度的。也就是说，0-31号线程为一组，32-63号线程为一组，以此类推。

•因此 SM 的调度无论如何都是对一整个线程组（warp）进行的，不可能出现一个组里只有单独一个线程被调走，要么 32 个线程一起调走。

•所以其实 j < 32 之后，就不需要 __syncthreads() 了。因为此时所有访问 local_sum 的线程都在一个组里嘛！反正都是一起调度走，不需要同步。

•结果却错了

•其实是编译器自作聪明优化了我们对 local_sum 的访问，导致结果不对的。解决：把 local_sum 数组声明为 volatile 禁止编译器优化.

```c
__global__ void parallel_sum(int *sum, int const *arr, int n) {
    __shared__ volatile int local_sum[1024];
    int j = threadIdx.x;
    int i = blockIdx.x;
    local_sum[j] = arr[i * 1024 + j];
    __syncthreads();
    if (j < 512) {
        local_sum[j] += local_sum[j + 512];
    }
    __syncthreads();
    if (j < 256) {
        local_sum[j] += local_sum[j + 256];
    }
    __syncthreads();
    if (j < 128) {
        local_sum[j] += local_sum[j + 128];
    }
    __syncthreads();
    if (j < 64) {
        local_sum[j] += local_sum[j + 64];
    }
    __syncthreads();
    if (j < 32) {
        local_sum[j] += local_sum[j + 32];
    }
    if (j < 16) {
        local_sum[j] += local_sum[j + 16];
    }
    if (j < 8) {
        local_sum[j] += local_sum[j + 8];
    }
    if (j < 4) {
        local_sum[j] += local_sum[j + 4];
    }
    if (j < 2) {
        local_sum[j] += local_sum[j + 2];
    }
    if (j == 0) {
        sum[i] = local_sum[0] + local_sum[1];
    }
}
```

**什么是线程组分歧（warp divergence）**

•GPU 线程组（warp）中 32 个线程实际是绑在一起执行的，就像 CPU 的 SIMD 那样。因此如果出现分支（if）语句时，如果 32 个 cond 中有的为真有的为假，则会导致两个分支都被执行！不过在 cond 为假的那几个线程在真分支会避免修改寄存器和访存，产生副作用。而为了避免会产生额外的开销。因此建议 GPU 上的 if 尽可能 32 个线程都处于同一个分支，要么全部真要么全部假，否则实际消耗了两倍时间！

![image-20230702192655655](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702192655655.png)

**避免线程组产生分歧**

•解决：我们加 if 的初衷是为了节省不必要的运算用的，然而对于 j < 32 以下那几个并没有节省运算（因为分支是按 32 个线程一组的），反而增加了分歧需要避免副作用的开销。因此可以把 j < 32 以下的那几个赋值合并为一个，这样反而快。

```c
__global__ void parallel_sum(int *sum, int const *arr, int n) {
    __shared__ volatile int local_sum[1024];
    int j = threadIdx.x;
    int i = blockIdx.x;
    local_sum[j] = arr[i * 1024 + j];
    __syncthreads();
    if (j < 512) {
        local_sum[j] += local_sum[j + 512];
    }
    __syncthreads();
    if (j < 256) {
        local_sum[j] += local_sum[j + 256];
    }
    __syncthreads();
    if (j < 128) {
        local_sum[j] += local_sum[j + 128];
    }
    __syncthreads();
    if (j < 64) {
        local_sum[j] += local_sum[j + 64];
    }
    __syncthreads();
    if (j < 32) {
        local_sum[j] += local_sum[j + 32];
        local_sum[j] += local_sum[j + 16];
        local_sum[j] += local_sum[j + 8];
        local_sum[j] += local_sum[j + 4];
        local_sum[j] += local_sum[j + 2];
        if (j == 0) {
            sum[i] = local_sum[0] + local_sum[1];
        }
    }
}

```

**使用网格跨步循环一次读取多个** **arr** **元素**

•可见共享内存中做求和开销还是有点大，之后那么多次共享内存的访问，前面却只有一次全局内存 arr 的访问，是不是太少了。

•因此可以通过网格跨步循环增加每个线程访问 arr 的次数，从而超过共享内存部分的时间。

•当然也别忘了在 main 中增加 gridDim 的大小。

```c
__global__ void parallel_sum(int *sum, int const *arr, int n) {
    __shared__ volatile int local_sum[1024];
    int j = threadIdx.x;
    int i = blockIdx.x;
    int temp_sum = 0;
    for (int t = i * 1024 + j; t < n; t += 1024 * gridDim.x) {
        temp_sum += arr[t];
    }
    local_sum[j] = temp_sum;
    __syncthreads();
    if (j < 512) {
        local_sum[j] += local_sum[j + 512];
    }
    __syncthreads();
    if (j < 256) {
        local_sum[j] += local_sum[j + 256];
    }
    __syncthreads();
    if (j < 128) {
        local_sum[j] += local_sum[j + 128];
    }
    __syncthreads();
    if (j < 64) {
        local_sum[j] += local_sum[j + 64];
    }
    __syncthreads();
    if (j < 32) {
        local_sum[j] += local_sum[j + 32];
        local_sum[j] += local_sum[j + 16];
        local_sum[j] += local_sum[j + 8];
        local_sum[j] += local_sum[j + 4];
        local_sum[j] += local_sum[j + 2];
        if (j == 0) {
            sum[i] = local_sum[0] + local_sum[1];
        }
    }
}

int main() {
    int n = 1<<24;
    std::vector<int, CudaAllocator<int>> arr(n);
    std::vector<int, CudaAllocator<int>> sum(n / 4096);

    for (int i = 0; i < n; i++) {
        arr[i] = std::rand() % 4;
    }

    TICK(parallel_sum);
    parallel_sum<<<n / 4096, 1024>>>(sum.data(), arr.data(), n);
    checkCudaErrors(cudaDeviceSynchronize());

    int final_sum = 0;
    for (int i = 0; i < n / 4096; i++) {
        final_sum += sum[i];
    }
    TOCK(parallel_sum);

    printf("result: %d\n", final_sum);

    return 0;
}
```

**通过函数模板封装一下**

```c
#include <cstdio>
#include <cuda_runtime.h>
#include "helper_cuda.h"
#include <vector>
#include "CudaAllocator.h"
#include "ticktock.h"

template <int blockSize, class T>
__global__ void parallel_sum_kernel(T *sum, T const *arr, int n) {
    __shared__ volatile int local_sum[blockSize];
    int j = threadIdx.x;
    int i = blockIdx.x;
    T temp_sum = 0;
    for (int t = i * blockSize + j; t < n; t += blockSize * gridDim.x) {
        temp_sum += arr[t];
    }
    local_sum[j] = temp_sum;
    __syncthreads();
    if constexpr (blockSize >= 1024) {
        if (j < 512)
            local_sum[j] += local_sum[j + 512];
        __syncthreads();
    }
    if constexpr (blockSize >= 512) {
        if (j < 256)
            local_sum[j] += local_sum[j + 256];
        __syncthreads();
    }
    if constexpr (blockSize >= 256) {
        if (j < 128)
            local_sum[j] += local_sum[j + 128];
        __syncthreads();
    }
    if constexpr (blockSize >= 128) {
        if (j < 64)
            local_sum[j] += local_sum[j + 64];
        __syncthreads();
    }
    if (j < 32) {
        if constexpr (blockSize >= 64)
            local_sum[j] += local_sum[j + 32];
        if constexpr (blockSize >= 32)
            local_sum[j] += local_sum[j + 16];
        if constexpr (blockSize >= 16)
            local_sum[j] += local_sum[j + 8];
        if constexpr (blockSize >= 8)
            local_sum[j] += local_sum[j + 4];
        if constexpr (blockSize >= 4)
            local_sum[j] += local_sum[j + 2];
        if (j == 0) {
            sum[i] = local_sum[0] + local_sum[1];
        }
    }
}

template <int reduceScale = 4096, int blockSize = 256, class T>
int parallel_sum(T const *arr, int n) {
    std::vector<int, CudaAllocator<int>> sum(n / reduceScale);
    parallel_sum_kernel<blockSize><<<n / reduceScale, blockSize>>>(sum.data(), arr, n);
    checkCudaErrors(cudaDeviceSynchronize());
    T final_sum = 0;
    for (int i = 0; i < n / reduceScale; i++) {
        final_sum += sum[i];
    }
    return final_sum;
}

int main() {
    int n = 1<<24;
    std::vector<int, CudaAllocator<int>> arr(n);
    std::vector<int, CudaAllocator<int>> sum(n / 4096);

    for (int i = 0; i < n; i++) {
        arr[i] = std::rand() % 4;
    }

    TICK(parallel_sum);
    int final_sum = parallel_sum(arr.data(), n);
    TOCK(parallel_sum);

    printf("result: %d\n", final_sum);

    return 0;
}

```

**进一步，当数组非常大，缩减后的数组可以继续递归地用** **GPU** **求和**

```c
template <int reduceScale = 4096, int blockSize = 256, int cutoffSize = reduceScale * 2, class T>
int parallel_sum(T const *arr, int n) {
    if (n > cutoffSize) {
        std::vector<int, CudaAllocator<int>> sum(n / reduceScale);
        parallel_sum_kernel<blockSize><<<n / reduceScale, blockSize>>>(sum.data(), arr, n);
        return parallel_sum(sum.data(), n / reduceScale);
    } else {
        checkCudaErrors(cudaDeviceSynchronize());
        T final_sum = 0;
        for (int i = 0; i < n; i++) {
            final_sum += arr[i];
        }
        return final_sum;
    }
}
```

•刚刚说到虽然用了 atomicAdd 按理说是非常低效的，然而却没有低效，这是因为编译器自动优化成刚刚用 BLS 的数组求和了！可以看到他优化后的效率和我们的 BLS 相仿，甚至还要快一些！

•结论：刚刚我们深入研究了如何 BLS 做数组求和，只是出于学习原理的目的。实际做求和时，直接写 atomicAdd 即可。反正编译器会自动帮我们优化成 BLS，而且他优化得比我们手写的更好……

•然后 atomicMax 求数组最大值，也同理。

## 9. 共享内存进阶

**GPU** **的内存模型**

![image-20230702193319191](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702193319191.png)

![image-20230702193331521](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702193331521.png)

**全局内存：在** **main()** **中通过** **cudaMalloc** **分配的内存**

![image-20230702193350751](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702193350751.png)

**共享内存：每个****板块都有一个，通过** **__shared__** **声明**

![image-20230702193409979](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702193409979.png)

**寄存器：存储着每个线程的局部变量**

![image-20230702193425919](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702193425919.png)

**板块中线程数量过多：寄存器打翻（register spill）**

•GPU 线程的寄存器，实际上也是一块比较小而块的内存，称之为寄存器仓库（register file）。板块内的所有的线程共用一个寄存器仓库。

•当板块中的线程数量（blockDim）过多时，就会导致每个线程能够分配到的寄存器数量急剧缩小。而如果你的程序恰好用到了非常多的寄存器，那就没办法全部装在高效的寄存器仓库里，而是要把一部分“打翻”到一级缓存中，这时对这些寄存器读写的速度就和一级缓存一样，相对而言低效了。若一级缓存还装不下，那会打翻到所有 SM 共用的二级缓存。

•此外，如果在线程局部分配一个数组，并通过动态下标访问（例如遍历 BVH 时用到的模拟栈），那无论如何都是会打翻到一级缓存的，因为寄存器不能动态寻址。

•对于 Fermi 架构来说，每个线程最多可以有 63 个寄存器（每个有 4 字节）。

**板块中的线程数量过少：延迟隐藏（latency hiding）失效**

•我们说过，每个 SM 一次只能执行板块中的一个线程组（warp），也就是32个线程。

•而当线程组陷入内存等待时，可以切换到另一个线程，继续计算，这样一个 warp 的内存延迟就被另一个 warp 的计算延迟给隐藏起来了。因此，如果线程数量太少的话，就无法通过在多个 warp 之间调度来隐藏内存等待的延迟，从而低效。

•此外，最好让板块中的线程数量（blockDim）为32的整数倍，否则假如是 33 个线程的话，那还是需要启动两个 warp，其中第二个 warp 只有一个线程是有效的，非常浪费。

•结论：对于使用寄存器较少、访存为主的核函数（例如矢量加法），使用大 blockDim 为宜。反之（例如光线追踪）使用小 blockDim，但也不宜太小。

**共享内存：什么是区块（bank）**

•GPU 的共享内存，实际上是 32 块内存条通过并联组成的（有点类似 CPU 的双通道内存条）。

•每个 bank 都可以独立地访问，他们每个时钟周期都可以读取一个 int。

•然后，他们把地址空间分为 32 分，第 i 根内存条，负责 addr % 32 == i 的那几个 int 的存储。这样交错存储，可以保证随机访问时，访存能够尽量分摊到 32 个区块，这样速度就提升了 32 倍。

•比如：__shared__ int arr[1024];

•那么 arr[0] 是存储在 bank 0，arr[1] 是 bank 1……arr[32] 又是 bank 0，arr[33] 又是 bank 1。

![image-20230702193700778](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702193700778.png)

**区块冲突（bank conflict）**

•然而这种设计有一个问题，那就是如果多个线程同时访问到了同一个 bank 的话，就需要排队。比如右图，线程0-3同时访问了 bank 0，但是同一个 bank 是需要排队，也就是串行访问的，所以线程0-3实际上没有真正并行起来，慢了4倍！

•那可能你觉得好像没什么问题，反正一般不会让两个线程访问 __shared__ 数组的同一个元素嘛！但是别忘了，刚刚说了 bank 是按照 addr % 32 来划分的，也就是说 arr[0] 和 arr[32] 是同属于 bank 0 的，如果两个线程同时访问了 arr[0] 和 arr[32] 就会出现 bank conflict 导致必须排队影响性能！

![image-20230702193749908](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702193749908.png)

![image-20230702193756651](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20230702193756651.png)

**GPU** **优化手法总结**

•线程组分歧（wrap divergence）：尽量保证 32 个线程都进同样的分支，否则两个分支都会执行。

•延迟隐藏（latency hiding）：需要有足够的 blockDim 供 SM 在陷入内存等待时调度到其他线程组。

•寄存器打翻（register spill）：如果核函数用到很多局部变量（寄存器），则 blockDim 不宜太大。

•共享内存（shared memory）：全局内存比较低效，如果需要多次使用，可以先读到共享内存。

•跨步访问（coalesced acccess）：建议先顺序读到共享内存，让高带宽的共享内存来承受跨步。

•区块冲突（bank conflict）：同一个 warp 中多个线程访问共享内存中模 32 相等的地址会比较低效，可以把数组故意搞成不对齐的 33 跨步来避免。

•顺便一提，英伟达的 warp 大小是 32，而 AMD 的显卡则是 64，其他概念如共享内存基本类似。







# GDB 常用命令

| 命令名称    | 命令缩写 | 说明                                                         |
| ----------- | -------- | ------------------------------------------------------------ |
| run         | r        | 运行程序                                                     |
| continue    | c        | 让暂停的程序继续运行                                         |
| break       | b        | 添加断点                                                     |
| tbreak      | tb       | 添加临时断点                                                 |
| backtrace   | bt       | 查看当前线程的调用堆栈                                       |
| frame       | f        | 切换到当前调用线程的指定堆栈                                 |
| info        | info     | 查看断点、线程等信息                                         |
| enable      | enable   | 启动某个断点                                                 |
| disable     | disable  | 禁用某个断点                                                 |
| delete      | del      | 删除断点                                                     |
| list        | l        | 显示源码                                                     |
| print       | p        | 打印或修改变量、寄存器的值                                   |
| ptype       | ptype    | 查看变量的类型                                               |
| thread      | thread   | 切换到指定的线程                                             |
| next        | n        | 运行到下一行                                                 |
| step        | s        | 如果有调用函数，则进入函数调用的内部，相当于step into        |
| until       | u        | 运行到指定的行停下来                                         |
| finish      | fi       | 结束当前调用函数，到上一层函数调用处                         |
| return      | return   | 结束当前调用函数并返回指定的值，到上一层函数调用处           |
| jump        | j        | 将当前程序的执行流跳转到指定的行或地址                       |
| disassemble | dis      | 查看汇编代码                                                 |
| set args    |          | 设置程序启动你那个命令行参数                                 |
| show args   |          | 查看设置的命令行参数                                         |
| watch       | watch    | 监视某个变量或内存地址的值是否发生了变化                     |
| display     | display  | 监视的变量或者内存地址，在程序中断后自动输出监控的变量或内存地址 |
| dir         | dir      | 重定向源码文件的位置                                         |

# 无锁消息队列

## 原子操作

对于gcc、g++编译器来讲，它们提供了一组API来做原子操作  

```c
type __sync_fetch_and_add (type *ptr, type value, ...)
type __sync_fetch_and_sub (type *ptr, type value, ...)
type __sync_fetch_and_or (type *ptr, type value, ...)
type __sync_fetch_and_and (type *ptr, type value, ...)
type __sync_fetch_and_xor (type *ptr, type value, ...)
type __sync_fetch_and_nand (type *ptr, type value, ...)
bool __sync_bool_compare_and_swap (type *ptr, type oldval type newval, ...)
type __sync_val_compare_and_swap (type *ptr, type oldval type newval, ...)
type __sync_lock_test_and_set (type *ptr, type value, ...)
void __sync_lock_release (type *ptr, ...)
```

详细文档见：https://gcc.gnu.org/onlinedocs/gcc-4.1.1/gcc/Atomic-Builtins.html#Atomic-Builtins
对于c++11来讲，也有一组atomic的接口：详细文档见： https://en.cppreference.com/w/cpp/atomic  

但是这些原子操作都是怎么实现的呢？
**X86的架构**
Intel X86指令集提供了指令前缀lock用于锁定前端串行总线FSB，保证了指令执行时不会收到其他处理器
的干扰。比如：  

```c
static int lxx_atomic_add(int *ptr, int increment)
{
    int old_value = *ptr;
    __asm__ volatile("lock; xadd %0, %1 \n\t"
                    : "=r"(old_value), "=m"(*ptr)
                    : "0"(increment), "m"(*ptr)
                    : "cc", "memory");
    return *ptr;
}
```

## 为什么需要无锁队列

锁引起的问题：

- Cache损坏(Cache trashing)
- 在同步机制上的争抢队列
- 动态内存分配  

### Cache损坏(Cache trashing)  

在保存和恢复上下文的过程中还隐藏了额外的开销：Cache中的数据会失效,因为它缓存的是将被换出任务的数据,这些数据对于新换进的任务是没用的。处理器的运行速度比主存快N倍,所以大量的处理器时间被浪费在处理器与主存的数据传输上。这就是在处理器和主存之间引入Cache的原因。Cache是一种速度更快但容量更小的内存(也更加昂贵),当处理器要访问主存中的数据时,这些数据首先被拷贝到Cache中，因为这些数据在不久的将来可能又会被处理器访问。Cache misses对性能有非常大的影响,因为处理器访问Cache中的数据将比直接访问主存快得多。线程被频繁抢占产生的Cache损坏将导致应用程序性能下降。  

### 在同步机制上的争抢队列 

阻塞不是微不足道的操作。它导致操作系统暂停当前的任务或使其进入睡眠状态(等待，不占用任何的处理器)。直到资源(例如互斥锁)可用，被阻塞的任务才可以解除阻塞状态(唤醒)。在一个负载较重的应用程序中使用这样的阻塞队列来在线程之间传递消息会导致严重的争用问题。也就是说，任务将大量的时间(睡眠，等待，唤醒)浪费在获得保护队列数据的互斥锁，而不是处理队列中的数据上。非阻塞机制大展伸手的机会到了。任务之间不争抢任何资源，在队列中预定一个位置，然后在这个位置上插入或提取数据。这中机制使用了一种被称之为CAS(比较和交换)的特殊操作，这个特殊操作是一种特殊的指令，它可以原子的完成以下操作:它需要3个操作数m，A，B，其中m是一个内存地址，操作将m指向的内存中的内容与A比较，如果相等则将B写入到m指向的内存中并返回true，如果不相等则什么也不做返回false。

### 动态内存分配  

在多线程系统中,需要仔细的考虑动态内存分配。当一个任务从堆中分配内存时，标准的内存分配机制会阻塞所有与这个任务共享地址空间的其它任务(进程中的所有线程)。这样做的原因是让处理更简单，且它工作得很好。两个线程不会被分配到一块相同的地址的内存，因为它们没办法同时执行分配请求。显然
线程频繁分配内存会导致应用程序性能下降(必须注意,向标准队列或map插入数据的时候都会导致堆上的动态内存分配)  

![image-20231012224129757](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231012224129757.png)

## 无锁队列的实现

![image-20231012224218810](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231012224218810.png)

单写单读的场景。



### 源码分析-yqueue_t  

yqueue_t是比如ypipe_t更底层的类  

**类接口和变量**

```c
// T is the type of the object in the queue.队列中元素的类型
// N is granularity(粒度) of the queue，简单来说就是yqueue_t一个结点可以装载N个T类
型的元素
template <typename T, int N> class yqueue_t
{ 
public:
    inline yqueue_t ();// Create the queue.
    inline ~yqueue_t ();// Destroy the queue.
    inline T &front ();// Returns reference to the front element of the queue. If the queue is empty, behaviour is undefined.
    inline T &back ();// Returns reference to the back element of the queue.If the queue is empty, behaviour is undefined.
    inline void push ();// Adds an element to the back end of the queue.
    inline void pop ();// Removes an element from the front of the queue.
    inline void unpush ()// Removes element from the back end of the queue。
回滚时使用
private:
    // Individual memory chunk to hold N elements.
    struct chunk_t
    {
        T values [N];  // //每个chunk_t可以容纳N个T类型的元素，以后就以一个chunk_t为单位申请内存
        chunk_t *prev;
        chunk_t *next;
    };
    chunk_t *begin_chunk;
    int begin_pos;
    chunk_t *back_chunk;
    int back_pos;
    chunk_t *end_chunk;
    int end_pos;
    atomic_ptr_t<chunk_t> spare_chunk; //空闲块（我把所有元素都已经出队的块称为空闲块），读写线程的共享变量
};
```

yqueue_t的实现，每次批量分配一批元素，减少内存的分配和释放（解决不断动态内存分配的问题）。yqueue_t内部由一个一个chunk组成，每个chunk保存N个元素。  

![image-20231012225542788](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231012225542788.png)

当队列空间不足时每次分配一个chunk_t，每个chunk_t能存储N个元素。在数据出队列后，队列有多余空间的时候，回收的chunk也不是马上释放，而是根据局部性原理先回收到spare_chunk里面，当再次需要分配chunk_t的时候从spare_chunk中获取。 

 ![image-20231012225625836](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231012225625836.png)

yqueue_t内部有三个chunk_t类型指针以及对应的索引位置：  

- begin_chunk/begin_pos：begin_chunk用于指向队列头的chunk，begin_pos用于指向队列第一个元素在当前chunk中的位置。
- back_chunk/back_pos：back_chunk用于指向队列尾的chunk，back_pos用于指向队列最后一个元素在当前chunk的位置。
- end_chunk/end_pos：由于chunk是批量分配的，所以end_chunk用于指向分配的最后一个chunk位置。  

这里特别需要注意区分back_chunk/back_pos和end_chunk/end_pos的作用：

- back_chunk/back_pos：对应的是元素存储位置；
- end_chunk/end_pos：决定是否要分配chunk或者回收chunk  

![image-20231012225915179](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231012225915179.png)

上图中：
有三块chunk，分别由begin_chunk、back_chunk、end_chunk组成。

- begin_pos指向begin_chunk中的第n个元素。
- back_pos指向back_chunk的最后一个元素  
- 由于back_pos已经指向了back_chunk的最后一个元素，所以end_pos就指向了end_chunk的第一个元素。  

另外还有一个spare_chunk指针，用于保存释放的chunk指针，当需要再次分配chunk的时候，会首先查看这里，从这里分配chunk。这里使用了原子的cas操作来完成，利用了操作系统的局部性原理。  

```c
// Create the queue.
inline yqueue_t()
{
    begin_chunk = (chunk_t *)malloc(sizeof(chunk_t)); // 预先分配chunk
    alloc_assert(begin_chunk);
    begin_pos = 0;
    back_chunk = NULL; //back_chunk总是指向队列中最后一个元素所在的chunk，现在还没有元素，所以初始为空
    back_pos = 0;
    end_chunk = begin_chunk; //end_chunk总是指向链表的最后一个chunk
    end_pos = 0;
}
```

![image-20231012231847015](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231012231847015.png)

end_chunk总是指向最后分配的chunk，刚分配出来的chunk，end_pos也总是为0。back_chunk需要chunk有元素插入的时候才指向对应的chunk。  

```c
// Returns reference to the front element of the queue.
// If the queue is empty, behaviour is undefined.
inline T &front() // 返回的是引用，是个左值，调用者可以通过其修改容器的值
{
	return begin_chunk->values[begin_pos]; // 队列首个chunk对应的的begin_pos
} /
/ Returns reference to the back element of the queue.
// If the queue is empty, behaviour is undefined.
inline T &back() // 返回的是引用，是个左值，调用者可以通过其修改容器的值
{
	return back_chunk->values[back_pos];
}
```

这里的front()或者back()函数，需要注意的返回的是左值引用，我们可以修改其值。

对于先进后出队列而言：

- begin_chunk->values[begin_pos]代表队列头可读元素， 读取队列头元素即是读取begin_pos位置的元素；
- back_chunk->values[back_pos]代表队列尾可写元素，写入元素时则是更新back_pos位置的元素，要确保元素真正生效，还需要调用push函数更新back_pos的位置，避免下次更新的时候又是更新当前back_pos位置对应的元素。  

更新下一个元素写入位置，如果end_pos超过chunk的索引位置(==N)则申请一个chunk（先尝试从spare_chunk获取，如果为空再申请分配全新的chunk）  

```c
// Adds an element to the back end of the queue.
inline void push()
{
	back_chunk = end_chunk;
	back_pos = end_pos; // 更新可写的位置，根据end_pos取更新
	if (++end_pos != N) //end_pos!=N表明这个chunk还没有满
		return;
    chunk_t *sc = spare_chunk.xchg(NULL); // 为什么设置为NULL？ 因为如果把之前值
    取出来了则没有spare chunk了，所以设置为NULL
    if (sc) // 如果有spare chunk则继续复用它
    {
        end_chunk->next = sc;
        sc->prev = end_chunk;
    } else // 没有则重新分配
    {
        end_chunk->next = (chunk_t *)malloc(sizeof(chunk_t)); // 分配一个chunk
        alloc_assert(end_chunk->next);
        end_chunk->next->prev = end_chunk;
    } 
    end_chunk = end_chunk->next;
    end_pos = 0;
}
```

这里分为两种情况：

- 第一种情况：++end_pos != N，说明当前chunk还有空余的位置可以继续插入新元素；
- 第二种情况：++end_pos == N，说明该chunk只有[N-1]的索引位置可以写入元素了，需要再分配一个chunk空间。
  - 需要新分配chunk时，先尝试从spare_chunk获取，如果获取到则直接使用，如果
    spare_chunk为NULL则需要重新分配chunk。
  - 最终都是要更新end_chunk和end_pos。  

![image-20231012232323025](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231012232323025.png)

第一种情况 ++end_pos != N，此时 back_pos和end_pos相差一个位置，即是 (back_pos +1)%N ==end_pos。  

![image-20231012232414018](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231012232414018.png)

第二种情况：++end_pos == N，说明该chunk只有N-1的位置可以写入元素了，需要再分配一个chunk空间。  

这里主要更新下一次读取的位置，并检测是否需要释放chunk（先保存到spare_chunk，然后检测spare_chunk返回值是否为空，如果返回值不为空说明之前有保存chunk，但我们只能保存一个chunk，所以把之前的chunk释放掉）  

```c
// Removes an element from the front end of the queue.
inline void pop()
{
    if (++begin_pos == N) // 删除满一个chunk才回收chunk
    {
        chunk_t *o = begin_chunk;
        begin_chunk = begin_chunk->next; // 更新begin_chunk位置
        begin_chunk->prev = NULL;
        begin_pos = 0;
        // 'o' has been more recently used than spare_chunk,
        // so for cache reasons we'll get rid of the spare and
        // use 'o' as the spare.
        chunk_t *cs = spare_chunk.xchg(o); //由于局部性原理，总是保存最新的空闲块而释放先前的空闲快
        free(cs);
    }
}
```

整个chunk的元素都被取出队列才去回收chunk，而且是把最后回收的chunk保存到spare_chunk，然后释放之前保存的chunk。
这里有两个点需要注意：

1. pop掉的元素，其销毁工作交给调用者完成，即是pop前调用者需要通过front()接口读取并进行销毁（比如动态分配的对象）。
2. 空闲块的保存，要求是原子操作。因为闲块是读写线程的共享变量，因为在push中也使用了spare_chunk。  



### 源码分析-ypipe_t  

ypipe_t在yqueue_t的基础上构建一个单写单读的无锁队列  

最核心的点：
w: 用来控制是否需要唤醒读端，当读端没有数据可以读取的时候，将c变量设置为NULL
f: 用来控制写入位置，当该f被更新到c的时候读端才能看到写入的数据
r: 用来控制可读位置，特别重点注意，这个r不是读位置的索引，而是读位置==r的时候说明已经队列为空了。  

```c
template <typename T, int N>
class ypipe_t
{ 
public:
    // Initialises the pipe.
    inline ypipe_t();
    // The destructor doesn't have to be virtual. It is mad virtual
    // just to keep ICC and code checking tools from complaining.
    inline virtual ~ypipe_t();
    // Write an item to the pipe. Don't flush it yet. If incomplete is
    // set to true the item is assumed to be continued by items
    // subsequently written to the pipe. Incomplete items are neverflushed down
    the stream.
    // 写入数据，incomplete参数表示写入是否还没完成，在没完成的时候不会修改flush指针，即这部分数据不会让读线程看到。
	inline void write(const T &value_, bool incomplete_);
    // Pop an incomplete item from the pipe. Returns true is such
	// item exists, false otherwise.
    inline bool unwrite(T *value_);
    // Flush all the completed items into the pipe. Returns false if
    // the reader thread is sleeping. In that case, caller is obliged to
    // wake the reader up before using the pipe again.
    // 刷新所有已经完成的数据到管道，返回false意味着读线程在休眠，在这种情况下调用者需要唤醒读线程。
	inline bool flush();
    // Check whether item is available for reading.
    // 这里面有两个点，一个是检查是否有数据可读，一个是预取
	inline bool check_read();
    // Reads an item from the pipe. Returns false if there is no value.
    // available.
	inline bool read(T *value_);
    // Applies the function fn to the first elemenent in the pipe
    // and returns the value returned by the fn.
    // The pipe mustn't be empty or the function crashes.
    inline bool probe(bool (*fn)(T &));
protected:
    // Allocation-efficient queue to store pipe items.
    // Front of the queue points to the first prefetched item, back of
    // the pipe points to last un-flushed item. Front is used only by
    // reader thread, while back is used only by writer thread.
    yqueue_t<T, N> queue;
    // Points to the first un-flushed item. This variable is used
    // exclusively by writer thread.
    T *w;//指向第一个未刷新的元素,只被写线程使用
    // Points to the first un-prefetched item. This variable is used
    // exclusively by reader thread.
    T *r;//指向第一个还没预提取的元素，只被读线程使用
    // Points to the first item to be flushed in the future.
    T *f;//指向下一轮要被刷新的一批元素中的第一个
    // The single point of contention between writer and reader thread.
    // Points past the last flushed item. If it is NULL,
    // reader is asleep. This pointer should be always accessed using
    // atomic operations.
    atomic_ptr_t<T> c;//读写线程共享的指针，指向每一轮刷新的起点（看代码的时候会详细说）。当c为空时，表示读线程睡眠（只会在读线程中被设置为空）
    // Disable copying of ypipe object.
    ypipe_t(const ypipe_t &);
    const ypipe_t &operator=(const ypipe_t &);
};
```

主要接口：  

- void write (const T &value, bool incomplete)：写入数据，incomplete参数表示写入是否还没完成，在没完成的时候不会修改flush指针，即这部分数据不会让读线程看到 。
- bool flush ()：刷新所有已经完成的数据到管道，返回false意味着读线程在休眠，在这种情况下调用者需要唤醒读线程。  
- bool read (T *value_):读数据，将读出的数据写入value指针中，返回false意味着没有数据可读 。

![image-20231016230921926](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016230921926.png)

**初始化:**

```c
inline ypipe_t()
{
    // Insert terminator element into the queue.
    queue.push();//yqueue_t的尾指针加1，开始back_chunk为空，现在back_chunk指向第一个chunk_t块的第一个位置
    // Let all the pointers to point to the terminator.
    // (unless pipe is dead, in which case c is set to NULL).
    r = w = f = &queue.back();//就是让r、w、f、c四个指针都指向这个end迭代器
    c.set(&queue.back()); // 保存[0]索引的位置
}
```

**write函数**

```c
// 写入数据，incomplete参数表示写入是否还没完成，在没完成的时候不会修改flush指针，即这部分数据不会让读线程看到。
inline void write(const T &value_, bool incomplete_)
{
    // Place the value to the queue, add new terminator element.
    queue.back() = value_;
    queue.push(); // 更新下一次写的位置
    // Move the "flush up to here" poiter.
    if (!incomplete_) // 如果f不更新，flush的时候 read也是没有数据
    	f = &queue.back(); // 记录要刷新的位置
}
```

![image-20231016231123998](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016231123998.png)

**flush函数**

主要是将w更新到f的位置，说明已经写到的位置。cas函数，原子操作，线程安全，把私有成员ptr指针与参数cmp_指针比较：如果相等，就把ptr设置为参数val_的值，返回ptr设置之前的值；如果不相等直接返回ptr值。  

```c
    // 刷新所有已经完成的数据到管道，返回false意味着读线程在休眠，在这种情况下调用者需要唤醒读线程。
    // 批量刷新的机制， 写入批量后唤醒读线程；
    inline bool flush()
    {
        //  If there are no un-flushed items, do nothing.
        if (w == f) // 不需要刷新，即是还没有新元素加入
            return true;

        //  Try to set 'c' to 'f'.
        // read时如果没有数据可以读取则c的值会被置为NULL
        if (c.cas(w, f) != w) // 尝试将c设置为f，即是准备更新w的位置
        {
            //  Compare-and-swap was unseccessful because 'c' is NULL.
            //  This means that the reader is asleep. Therefore we don't
            //  care about thread-safeness and update c in non-atomic
            //  manner. We'll return false to let the caller know
            //  that reader is sleeping.
            c.set(f); // 更新w的位置
            w = f;
            return false; //线程看到flush返回false之后会发送一个消息给读线程，这需要写业务去做处理
        }
        else  // 读端还有数据可读取
        {
            //  Reader is alive. Nothing special to do now. Just move
            //  the 'first un-flushed item' pointer to 'f'.
            w = f;             // 只需要更新w的位置
            return true;
        }
    }
```

![image-20231016231606668](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016231606668.png)

**read函数**

```c
    //  Check whether item is available for reading.
    // 这里面有两个点，一个是检查是否有数据可读，一个是预取
    inline bool check_read()
    {
        //  Was the value prefetched already? If so, return.
        if (&queue.front() != r && r) //判断是否在前几次调用read函数时已经预取数据了return true;
            return true;

        //  There's no prefetched value, so let us prefetch more values.
        //  Prefetching is to simply retrieve the
        //  pointer from c in atomic fashion. If there are no
        //  items to prefetch, set c to NULL (using compare-and-swap).
        // 两种情况
        // 1. 如果c值和queue.front()相等， 返回c值并将c值置为NULL，此时没有数据可读
        // 2. 如果c值和queue.front()不等， 返回c值，此时可能有数据度的去
        r = c.cas(&queue.front(), NULL); //尝试预取数据

        //  If there are no elements prefetched, exit.
        //  During pipe's lifetime r should never be NULL, however,
        //  it can happen during pipe shutdown when items are being deallocated.
        if (&queue.front() == r || !r) //判断是否成功预取数据
            return false;

        //  There was at least one value prefetched.
        return true;
    }

    //  Reads an item from the pipe. Returns false if there is no value.
    //  available.
    inline bool read(T *value_)
    {
        //  Try to prefetch a value.
        if (!check_read())
            return false;

        //  There was at least one value prefetched.
        //  Return it to the caller.
        *value_ = queue.front();
        queue.pop();
        return true;
    }
```

如果：

- 指针r指向的是队头元素（r==&queue.front()） （最核心的一点r不是我们read的位置索引，而是用来识别我们可以读取到哪个位置就不能再读）

- 或者r没有指向任何元素（NULL）  

则说明队列中并没有可读的数据，这个时候check_read尝试去预取数据。所谓的预取就是令 r=c (cas函数就是返回c本身的值，看上面关于cas的实现)， 而c在write中被指向f（见上图），这时从queue.front()到f这个位置的数据都被预取出来了，然后每次调用read都能取出一段。值得注意的是，当c==&queue.front()时，代表数据被取完了，这时把c指向NULL，接着读线程会睡眠，这也是给写线程检查读线程是否睡眠的标志。  

继续上面写入AB数据的场景，第一次调用read时，会先check_read，把指针r指向指针c的位置（所谓的预取），这时r,c,w,f的关系如下：  

![image-20231016232123953](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016232123953.png)

在 “7.read(&ret),函数返回false,ret没有获取到值”的时候，front()和r相等。  

- 如果此时在r = c.cas(&queue.front(), NULL); 执行时没有flush的操作。则说明没有数据可以读取，最终返回false；  
- 如果在r = c.cas(&queue.front(), NULL); 之前写入方write新数据后并调用了flush，则r被更新，最终返回true。  

而_c指针，则是读写线程都可以操作，因此需要使用原子的CAS操作来修改，它的可能值有以下几种：  

- NULL：读线程设置，此时意味着已经没有数据可读，读线程在休眠。  
- 非零：写线程设置，这里又区分两种情况：  
  - 旧值为_w的情况下，cas(_w,_f)操作修改为_f，意味着如果原先的值为_w，则原子性的修改为_f，表示有更多已被刷新的数据可读。  
  - 在旧值为NULL的情况下，此时读线程休眠，因此可以安全的设置为当前_f指针的位置。  

![image-20231016232448929](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016232448929.png)



## 基于循环循环数组的无锁队列  

- ArrayLockFreeQueue数据结构，可以理解为一个环形数组；
- 多线程写入时候，m_maximumReadIndex、m_writeIndex索引如何更新
- 在更新m_maximumReadIndex的时候为什么要让出cpu；
- 多线程读取的时候，m_readIndex如何更新。
- 可读位置是由m_maximumReadIndex控制，而不是m_writeIndex去控制的。
  - m_maximumReadIndex的更新由m_writeIndex。  

```c
#define ARRAY_LOCK_FREE_Q_DEFAULT_SIZE 65535 // 2^16

template <typename ELEM_T, QUEUE_INT Q_SIZE = ARRAY_LOCK_FREE_Q_DEFAULT_SIZE>
class ArrayLockFreeQueue
{
public:

	ArrayLockFreeQueue();
	virtual ~ArrayLockFreeQueue();

	QUEUE_INT size();

	bool enqueue(const ELEM_T &a_data);

	bool dequeue(ELEM_T &a_data);

    bool try_dequeue(ELEM_T &a_data);

private:

	ELEM_T m_thequeue[Q_SIZE];

	volatile QUEUE_INT m_count;
	volatile QUEUE_INT m_writeIndex;

	volatile QUEUE_INT m_readIndex;

	volatile QUEUE_INT m_maximumReadIndex; //最后一个已经完成入列操作的元素在数组中的下标

	inline QUEUE_INT countToIndex(QUEUE_INT a_count);
};
```

三种不同下标：

- m_count; // 队列的元素个数
- m_writeIndex;//新元素入列时存放位置在数组中的下标
- m_readIndex;// 下一个出列的元素在数组中的下标
- m_maximumReadIndex; //最后一个已经完成入列操作的元素在数组中的下标。如果它的值跟m_writeIndex不一致，表明有写请求尚未完成。这意味着，有写请求成功申请了空间但数据还没完全写进队列。所以如果有线程要读取，必须要等到写线程将数据完全写入到队列之后。  

必须指明的是使用3种不同的下标都是必须的，因为队列允许任意数量的生产者和消费者围绕着它工作。已经存在一种基于循环数组的无锁队列，使得唯一的生产者和唯一的消费者可以良好的工作。它的实现相当简洁非常值得阅读  

该程序使用gcc内置的sync_bool_compare_and_swap，但重新做了宏定义封装。

```c
#define CAS(a_ptr, a_oldVal, a_newVal) sync_bool_compare_and_swap(a_ptr, a_oldVal, a_newVal)  
```

```c
template <typename ELEM_T, QUEUE_INT Q_SIZE>
ArrayLockFreeQueue<ELEM_T, Q_SIZE>::ArrayLockFreeQueue() :
	m_writeIndex(0),
	m_readIndex(0),
	m_maximumReadIndex(0)
{
	m_count = 0;
}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
ArrayLockFreeQueue<ELEM_T, Q_SIZE>::~ArrayLockFreeQueue()
{

}
template <typename ELEM_T, QUEUE_INT Q_SIZE>
inline QUEUE_INT ArrayLockFreeQueue<ELEM_T, Q_SIZE>::countToIndex(QUEUE_INT a_count)
{
	return (a_count % Q_SIZE);		// 取余的时候
}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
QUEUE_INT ArrayLockFreeQueue<ELEM_T, Q_SIZE>::size()
{
	QUEUE_INT currentWriteIndex = m_writeIndex;
	QUEUE_INT currentReadIndex = m_readIndex;

	if(currentWriteIndex>=currentReadIndex)
		return currentWriteIndex - currentReadIndex;
	else
		return Q_SIZE + currentWriteIndex - currentReadIndex;

}
```

入队列

```c
template <typename ELEM_T, QUEUE_INT Q_SIZE>
bool ArrayLockFreeQueue<ELEM_T, Q_SIZE>::enqueue(const ELEM_T &a_data)
{
	QUEUE_INT currentWriteIndex;		// 获取写指针的位置
	QUEUE_INT currentReadIndex;
	// 1. 获取可写入的位置
	do
	{
		currentWriteIndex = m_writeIndex;
		currentReadIndex = m_readIndex;
		if(countToIndex(currentWriteIndex + 1) ==
			countToIndex(currentReadIndex))
		{
			return false;	// 队列已经满了	
		}
		// 目的是为了获取一个能写入的位置
	} while(!CAS(&m_writeIndex, currentWriteIndex, (currentWriteIndex+1)));
	// 获取写入位置后 currentWriteIndex 是一个临时变量，保存我们写入的位置
	// We know now that this index is reserved for us. Use it to save the data
	m_thequeue[countToIndex(currentWriteIndex)] = a_data;  // 把数据更新到对应的位置

	// 2. 更新可读的位置，按着m_maximumReadIndex+1的操作
 	// update the maximum read index after saving the data. It wouldn't fail if there is only one thread 
	// inserting in the queue. It might fail if there are more than 1 producer threads because this
	// operation has to be done in the same order as the previous CAS
	while(!CAS(&m_maximumReadIndex, currentWriteIndex, (currentWriteIndex + 1)))
	{
		 // this is a good place to yield the thread in case there are more
		// software threads than hardware processors and you have more
		// than 1 producer thread
		// have a look at sched_yield (POSIX.1b)
		sched_yield();		// 当线程超过cpu核数的时候如果不让出cpu导致一直循环在此。
	}

	AtomicAdd(&m_count, 1);

	return true;

}
```

如果一个位置被标记为X，标识这个位置里存放了数据。空白表示位置是空的。对于下图的情况,队列中存放了两个元素。WriteIndex指示的位置是新元素将会被插入的位置。ReadIndex指向的位置中的元素将会在下一次pop操作中被弹出  

![image-20231016233330403](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233330403.png)

当生产者准备将数据插入到队列中,它首先通过增加WriteIndex的值来申请空间。MaximumReadIndex指向最后一个存放有效数据的位置(也就是实际的队列尾)。  

![image-20231016233404323](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233404323.png)

一旦空间的申请完成,生产者就可以将数据拷贝到刚刚申请到的位置中。完成之后增加MaximumReadIndex使得它与WriteIndex的一致。  

![image-20231016233428368](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233428368.png)

现在队列中有3个元素，接着又有一个生产者尝试向队列中插入元素。  

![image-20231016233446118](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233446118.png)

在第一个生产者完成数据拷贝之前，又有另外一个生产者申请了一个新的空间准备拷贝数据。现在有两个生产者同时向队列插入数据。  

![image-20231016233511814](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233511814.png)

现在生产者开始拷贝数据，在完成拷贝之后，对MaximumReadIndex的递增操作必须严格遵循一个顺序：第一个生产者线程首先递增MaximumReadIndex，接着才轮到第二个生产者。这个顺序必须被严格遵守的原因是，我们必须保证数据被完全拷贝到队列之后才允许消费者线程将其出列（while(!CAS(&m_maximumReadIndex, currentWriteIndex, (currentWriteIndex + 1))){sched_yield(); } 让出cpu的目的也是为了让排在最前面的生产者完成m_maximumReadIndex的更新）

![image-20231016233557543](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233557543.png)

第一个生产者完成了数据拷贝，并对MaximumReadIndex完成了递增，现在第二个生产者可以递增MaximumReadIndex了  

![image-20231016233620487](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233620487.png)

第二个生产者完成了对MaximumReadIndex的递增,现在队列中有5个元素 。

**出队列**

```c
template <typename ELEM_T, QUEUE_INT Q_SIZE>
bool ArrayLockFreeQueue<ELEM_T, Q_SIZE>::try_dequeue(ELEM_T &a_data)
{
    return dequeue(a_data);
}

template <typename ELEM_T, QUEUE_INT Q_SIZE>
bool ArrayLockFreeQueue<ELEM_T, Q_SIZE>::dequeue(ELEM_T &a_data)
{
	QUEUE_INT currentMaximumReadIndex;
	QUEUE_INT currentReadIndex;

	do
	{
		 // to ensure thread-safety when there is more than 1 producer thread
       	// a second index is defined (m_maximumReadIndex)
		currentReadIndex = m_readIndex;
		currentMaximumReadIndex = m_maximumReadIndex;

		if(countToIndex(currentReadIndex) ==
			countToIndex(currentMaximumReadIndex))		// 如果不为空，获取到读索引的位置
		{
			// the queue is empty or
			// a producer thread has allocate space in the queue but is 
			// waiting to commit the data into it
			return false;
		}
		// retrieve the data from the queue
		a_data = m_thequeue[countToIndex(currentReadIndex)]; // 从临时位置读取的

		// try to perfrom now the CAS operation on the read index. If we succeed
		// a_data already contains what m_readIndex pointed to before we 
		// increased it
		if(CAS(&m_readIndex, currentReadIndex, (currentReadIndex + 1)))
		{
			AtomicSub(&m_count, 1);	// 真正读取到了数据，元素-1
			return true;
		}
	} while(true);

	assert(0);
	 // Add this return statement to avoid compiler warnings
	return false;

}
```

WriteIndex指示的位置是新元素将会被插入的位置。ReadIndex指向的位置中的元素将会在下一次pop操作中被弹出  

![image-20231016233838818](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233838818.png)

消费者线程拷贝数组ReadIndex位置的元素，然后尝试用CAS操作将ReadIndex加1。如果操作成功消费者成功的将数据出列。因为CAS操作是原子的，所以只有唯一的线程可以在同一时刻更新ReadIndex的值。如果操作失败，读取新的ReadIndex值，以重复以上操作(copy数据，CAS)。  

![image-20231016233931136](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233931136.png)

现在又有一个消费者将元素出列，队列变成空。  

![image-20231016233948748](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016233948748.png)

现在有一个生产者正在向队列中添加元素。它已经成功的申请了空间，但尚未完成数据拷贝。任何其它企图从队列中移除元素的消费者都会发现队列非空(因为writeIndex不等于readIndex)。但它不能读取readIndex所指向位置中的数据，因为readIndex与MaximumReadIndex相等。消费者将会在do循环中不断的反复尝试，直到生产者完成数据拷贝增加MaximumReadIndex的值，或者队列变成空(这在多个消费者的场景下会发生)。  

![image-20231016234039993](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231016234039993.png)

## 参考文档

【基于数组的无锁队列(译)】https://zhuanlan.zhihu.com/p/33985732
【A Fast Lock-Free Queue for C++】 https://moodycamel.com/blog/2013/a-fast-lock-free-queuefor-c++.htm#benchmarks
【A Fast General Purpose Lock-Free Queue for C++】https://moodycamel.com/blog/2014/a-fastgeneral-purpose-lock-free-queue-for-c++.htm#benchmarks
[10] sched_yield documentation
[11] lock-free single producer - single consumer circular queue
《说说无锁(Lock-Free)编程那些事》https://www.yuque.com/docs/share/31e0227e-1925-472c-93c5-3384fe43fd2f?#  





# profiling - 将 pprof 与 gperftools 一起使用会导致 curl 错误

**如果您选择运行分析器的方法 1，使用环境变量 `gcc`默认情况下只是忽略您的链接，因为您没有使用该库中的任何符号。您需要将它包含在 `-Wl,--no-as-needed` 中像这样的标志:**

> g++ -g -O3 -Wl,--no-as-needed -lprofiler -Wl,--as-needed 01_substring_sort_a.C 01_substring_sort.C -o example



编译:

> CPUPROFILE=prof.data ./example01
>
> pprof --text ./example01 prof.data



## perf 工具出现下面错误

 zhw@zhw:~/zhen/The-Art-of-Writing-Efficient-Programs/Chapter02/build$ perf stat ./example01 
Error:
Access to performance monitoring and observability operations is limited.
Consider adjusting /proc/sys/kernel/perf_event_paranoid setting to open
access to performance monitoring and observability operations for processes
without CAP_PERFMON, CAP_SYS_PTRACE or CAP_SYS_ADMIN Linux capability.
More information can be found at 'Perf events and tool security' document:
https://www.kernel.org/doc/html/latest/admin-guide/perf-security.html
perf_event_paranoid setting is 4:
  -1: Allow use of (almost) all events by all users
      Ignore mlock limit after perf_event_mlock_kb without CAP_IPC_LOCK
>= 0: Disallow raw and ftrace function tracepoint access
>= 1: Disallow CPU event access
>= 2: Disallow kernel profiling
>To make the adjusted perf_event_paranoid setting permanent preserve it
>in /etc/sysctl.conf (e.g. kernel.perf_event_paranoid = <setting>)

### 解决办法:

> sudo sh -c "echo -1 > /proc/sys/kernel/perf_event_paranoid"
>
> 或者
>
> sudo sysctl -w kernel.perf_event_paranoid=-1

**使用perf tools**

> perf stat ./example02

>  Performance counter stats for './example02':
>
>           1,489.22 msec task-clock                       #    1.000 CPUs utilized             
>                  6      context-switches                 #    4.029 /sec                      
>                  0      cpu-migrations                   #    0.000 /sec                      
>             73,898      page-faults                      #   49.622 K/sec                     
>      6,962,300,987      cpu_core/cycles/                 #    4.675 G/sec                     
>      <not counted>      cpu_atom/cycles/                                                        (0.00%)
>     12,999,547,532      cpu_core/instructions/           #    8.729 G/sec                     
>      <not counted>      cpu_atom/instructions/                                                  (0.00%)
>      3,581,247,796      cpu_core/branches/               #    2.405 G/sec                     
>      <not counted>      cpu_atom/branches/                                                      (0.00%)
>          1,159,954      cpu_core/branch-misses/          #  778.901 K/sec                     
>      <not counted>      cpu_atom/branch-misses/                                                 (0.00%)
>     41,218,876,254      cpu_core/slots/                  #   27.678 G/sec                     
>     10,668,415,030      cpu_core/topdown-retiring/       #     25.9% Retiring                 
>        168,317,043      cpu_core/topdown-bad-spec/       #      0.4% Bad Speculation          
>      1,293,141,215      cpu_core/topdown-fe-bound/       #      3.1% Frontend Bound           
>     29,095,677,355      cpu_core/topdown-be-bound/       #     70.6% Backend Bound            
>        883,448,723      cpu_core/topdown-heavy-ops/      #      2.1% Heavy Operations          #     23.7% Light Operations         
>        161,642,651      cpu_core/topdown-br-mispredict/  #      0.4% Branch Mispredict         #      0.0% Machine Clears           
>        484,927,955      cpu_core/topdown-fetch-lat/      #      1.2% Fetch Latency             #      2.0% Fetch Bandwidth          
>     12,931,412,158      cpu_core/topdown-mem-bound/      #     31.4% Memory Bound              #     39.2% Core Bound               
>     
>        1.489706059 seconds time elapsed
>             
>        1.421514000 seconds user
>        0.068072000 seconds sys 

**生成report** 

> perf record -e branches,branch-misses ./example02
>
> perf report





## 安装gprof

> sudo apt-get -y install gperf



CmakeList.txt:

```c++
cmake_minimum_required(VERSION 3.15)

project(example LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS ON)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O3 -Wl,--no-as-needed")
add_executable(example01 01_substring_sort.C 01_substring_sort_a.C)
target_link_libraries(example01 profiler unwind)

add_executable(example03 03_substring_sort.C 03_substring_sort_a.C)
target_link_libraries(example03 profiler unwind)

add_executable(example04 04_substring_sort.C 04_substring_sort_a.C)
target_link_libraries(example04 profiler unwind)

add_executable(example05 05_compare_timer.C)
target_link_libraries(example04 profiler unwind)

add_executable(example05a 05a_compare_timer.C)
target_link_libraries(example04 profiler unwind)

add_executable(example06 06_compare_timer.C)
target_link_libraries(example06 profiler unwind)

add_executable(example07 07_compare_timer.C)
target_link_libraries(example07 profiler unwind)

add_executable(example10 10_compare_mbm.C)
target_link_libraries(example10 profiler unwind benchmark)

add_custom_target(distclean COMMAND rm -rf CMakeCache.txt CMakeFiles
                  Makefile cmake_install.cmake ShallowWater.dSYM ipo_out.optrpt)
```



## 安装 google benchmark

https://www.cnblogs.com/apocelipes/p/10348925.html







# ntyco 

https://github.com/wangbojing/NtyCo

## 协程的原理

### 问题：协程如何使用？ 与线程使用有何区别  

在做网络 IO 编程的时候， 有一个非常理想的情况， 就是每次 accept 返回的时候， 就为新来的客户端分配一个线程， 这样一个客户端对应一个线程。 就不会有多个线程共用一个 sockfd。 每请求每线程的方式， 并且代码逻辑非常易读。 但是这只是理想，线程创建代价，调度代价就呵呵了。先来看一下每请求每线程的代码如下：  

![image-20231126101347572](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231126101347572.png)

如果我们有协程， 我们就可以这样实现。 参考代码如下：  

![image-20231126101426650](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231126101426650.png)



**线程的 API 思维来使用协程， 函数调用的性能来测试协程**  

NtyCo 封装出来了若干接口，一类是协程本身的，二类是 posix 的异步封装协程 API： while  

1. 协程创建:

   当我们需要异步调用的时候，我们会创建一个协程。比如 accept 返回一个新的sockfd，创建一个客户端处理的子过程。 再比如需要监听多个端口的时候，创建一个 server的子过程，这样多个端口同时工作的，是符合微服务的架构的。  

   ```c++
   int nty_coroutine_create(nty_coroutine **new_co, proc_coroutine func, void *arg)
   ```

​	参数 1： nty_coroutine **new_co， 需要传入空的协程的对象， 这个对象是由内部创建的， 并且在函数返回的时候， 会返回一个内部创建的协程对象。
​	参数 2： proc_coroutine func， 协程的子过程。 当协程被调度的时候， 就会执行该函数。
​	参数 3： void *arg， 需要传入到新协程中的参数。协程不存在亲属关系， 都是一致的调度关系， 接受调度器的调度。 调用 create API
就会创建一个新协程，新协程就会加入到调度器的就绪队列中 。

2. 协程调度器的运行  

   ```c++
   void nty_schedule_run(void)
   ```

3. POSIX 异步封装 API：  

   ```c++
   int nty_socket(int domain, int type, int protocol)
   int nty_accept(int fd, struct sockaddr *addr, socklen_t *len)
   int nty_recv(int fd, void *buf, int length)
   int nty_send(int fd, const void *buf, int length)
   int nty_close(int fd)
   ```

   

**实现 IO 异步操作**

  大部分的朋友会关心 IO 异步操作如何实现，在 send 与 recv 调用的时候，如何实现异步操作的。  

```c++
while (1) {
    int nready = epoll_wait(epfd, events, EVENT_SIZE, -1);
    for (i = 0;i < nready;i ++) {
            int sockfd = events[i].data.fd;
            if (sockfd == listenfd) {
            int connfd = accept(listenfd, xxx, xxxx);
            setnonblock(connfd);
            ev.events = EPOLLIN | EPOLLET;
            ev.data.fd = connfd;
            epoll_ctl(epfd, EPOLL_CTL_ADD, connfd, &ev);
        } else {
            epoll_ctl(epfd, EPOLL_CTL_DEL, sockfd, NULL);
            recv(sockfd, buffer, length, 0);
            //parser_proto(buffer, length);
            send(sockfd, buffer, length, 0);
            epoll_ctl(epfd, EPOLL_CTL_ADD, sockfd, NULL);
        }
    }
}
```

在进行 IO 操作（recv， send）之前，先执行了 epoll_ctl 的 del 操作，将相应的 sockfd 从 epfd中删除掉， 在执行完 IO 操作（recv， send）再进行 epoll_ctl 的 add 的动作。 这段代码看起来似乎好像没有什么作用。如果是在多个上下文中， 这样的做法就很有意义了。 能够保证 sockfd 只在一个上下文中能够操作 IO 的。不会出现在多个上下文同时对一个 IO 进行操作的。 协程的 IO 异步操作正式是采用此模式进行的。  

![image-20231126105626095](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231126105626095.png)

在协程的上下文 IO 异步操作（nty_recv， nty_send） 函数，步骤如下：

1. 将 sockfd 添加到 epoll 管理中。

2. 进行上下文环境切换， 由协程上下文 yield 到调度器的上下文。

3. 调度器获取下一个协程上下文。 Resume 新的协程

   

   IO 异步操作的上下文切换的时序图如下：  

![image-20231126110143704](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231126110143704.png)



### 回调协程的子过程  

在 create 协程后， 何时回调子过程？ 何种方式回调子过程？首先来回顾一下 x86_64 寄存器的相关知识。 x86_64 的寄存器有 16 个 64 位寄存器，分别是： %rax, %rbx,%rcx, %esi, %edi, %rbp, %rsp, %r8, %r9, %r10, %r11, %r12, %r13, %r14, %r15。  \

%rax 作为函数返回值使用的。
%rsp 栈指针寄存器， 指向栈顶%rdi, %rsi, %rdx, %rcx, %r8, %r9 用作函数参数， 依次对应第 1 参数， 第 2 参数。。。
%rbx, %rbp, %r12, %r13, %r14, %r15 用作数据存储， 遵循调用者使用规则， 换句话说， 就是随便用。 调用子函数之前要备份它， 以防它被修改%r10, %r11 用作数据存储， 就是使用前要先保存原值  

以 NtyCo 的实现为例，来分析这个过程。 CPU 有一个非常重要的寄存器叫做 EIP，用来存储 CPU 运行下一条指令的地址。 我们可以把回调函数的地址存储到 EIP 中，将相应的参数存储到相应的参数寄存器中。 实现子过程调用的逻辑代码如下：  

```c++
static void _exec(void *lt) {
	nty_coroutine *co = (nty_coroutine*)lt;
	co->func(co->arg);
	co->status |= (BIT(NTY_COROUTINE_STATUS_EXITED) | BIT(NTY_COROUTINE_STATUS_FDEOF) | BIT(NTY_COROUTINE_STATUS_DETACH));
	nty_coroutine_yield(co);
}
void nty_coroutine_init(nty_coroutine *co) {
    //ctx 就是协程的上下文
    co->ctx.edi = (void*)co; //设置参数
    co->ctx.eip = (void*)_exec; //设置回调函数入口
    //当实现上下文切换的时候，就会执行入口函数_exec , _exec 调用子过程 func
}
```



### 协程的内部原语操作有哪些？分别如何实现的？  

协程的核心原语操作： create, resume, yield。协程的原语操作有create 怎么没有 exit？ 以 NtyCo 为例，协程一旦创建就不能有用户自己销毁，必须得以子过程执行结束，就会自动销毁协程的上下文数据。 以_exec 执行入口函数返回而销毁协程的上下文与相关信息。 co->func(co->arg) 是子过程，若用户需要长久运行协程，就必须要在 func 函数里面写入循环等操作。 所以 NtyCo 里面没有实现 exit 的原语操作。  

create： 创建一个协程。

1. 调度器是否存在， 不存在也创建。 调度器作为全局的单例。 将调度器的实例存储在线程的私有空间 pthread_setspecific。
2. 分配一个 coroutine 的内存空间，分别设置 coroutine 的数据项，栈空间，栈大小，初始状态，创建时间，子过程回调函数， 子
   过程的调用参数。
3. 将新分配协程添加到就绪队列 ready_queue 中  



```c++
pthread_key_t global_sched_key;
static pthread_once_t sched_key_once = PTHREAD_ONCE_INIT;

static void nty_coroutine_sched_key_destructor(void *data) {   // 释放数据
	free(data);
}
static void nty_coroutine_sched_key_creator(void) {
    //创建线程私有数据，线程内都可访问，线程结束时调用nty_coroutine_sched_key_destructor
	assert(pthread_key_create(&global_sched_key, nty_coroutine_sched_key_destructor) == 0);
	assert(pthread_setspecific(global_sched_key, NULL) == 0);
	
	return ;
}
// 读取线程的私有数据nty_schedule
static inline nty_schedule *nty_coroutine_get_sched(void) {
	return pthread_getspecific(global_sched_key);
}

int nty_schedule_create(int stack_size) {

	int sched_stack_size = stack_size ? stack_size : NTY_CO_MAX_STACKSIZE;

	nty_schedule *sched = (nty_schedule*)calloc(1, sizeof(nty_schedule));
	if (sched == NULL) {
		printf("Failed to initialize scheduler\n");
		return -1;
	}

	assert(pthread_setspecific(global_sched_key, sched) == 0); // 加到私有变量里面

	sched->poller_fd = nty_epoller_create(); //创建epoll
	if (sched->poller_fd == -1) {
		printf("Failed to initialize epoller\n");
		nty_schedule_free(sched);
		return -2;
	}

	nty_epoller_ev_register_trigger();  // 把poller_fd 加到epoll里面

	sched->stack_size = sched_stack_size;
	sched->page_size = getpagesize();

#ifdef _USE_UCONTEXT
	int ret = posix_memalign(&sched->stack, sched->page_size, sched->stack_size);
	assert(ret == 0);
#else
	sched->stack = NULL;
	bzero(&sched->ctx, sizeof(nty_cpu_ctx));
#endif

	sched->spawned_coroutines = 0;
	sched->default_timeout = 3000000u;

	RB_INIT(&sched->sleeping); // 初始化红黑树
	RB_INIT(&sched->waiting);

	sched->birth = nty_coroutine_usec_now();

	TAILQ_INIT(&sched->ready);
	TAILQ_INIT(&sched->defer);
	LIST_INIT(&sched->busy);

}

int nty_coroutine_create(nty_coroutine **new_co, proc_coroutine func, void *arg) {

	// 保证nty_coroutine_sched_key_creator 在线程中只初始化一次
    assert(pthread_once(&sched_key_once, nty_coroutine_sched_key_creator) == 0);
	nty_schedule *sched = nty_coroutine_get_sched();

	if (sched == NULL) {    //如果没有sched， 则创建
		nty_schedule_create(0); // 创建schedule 128k
		
		sched = nty_coroutine_get_sched();
		if (sched == NULL) {
			printf("Failed to create scheduler\n");
			return -1;
		}
	}

	nty_coroutine *co = calloc(1, sizeof(nty_coroutine));
	if (co == NULL) {
		printf("Failed to allocate memory for new coroutine\n");
		return -2;
	}

#ifdef _USE_UCONTEXT
	co->stack = NULL;
	co->stack_size = 0;
#else
	int ret = posix_memalign(&co->stack, getpagesize(), sched->stack_size);
	if (ret) {
		printf("Failed to allocate stack for new coroutine\n");
		free(co);
		return -3;
	}
	co->stack_size = sched->stack_size;
#endif
	co->sched = sched;
	co->status = BIT(NTY_COROUTINE_STATUS_NEW); //
	co->id = sched->spawned_coroutines ++;
	co->func = func;
#if CANCEL_FD_WAIT_UINT64
	co->fd = -1;
	co->events = 0;
#else
	co->fd_wait = -1;
#endif
	co->arg = arg;
	co->birth = nty_coroutine_usec_now();
	*new_co = co;

	TAILQ_INSERT_TAIL(&co->sched->ready, co, ready_next); //加入到ready tree里面

	return 0;
}
```



**yield： 让出 CPU。**  

```c++
void nty_coroutine_yield(nty_coroutine *co) {
	co->ops = 0;
#ifdef _USE_UCONTEXT
	if ((co->status & BIT(NTY_COROUTINE_STATUS_EXITED)) == 0) {
		_save_stack(co);
	}
	swapcontext(&co->ctx, &co->sched->ctx);
#else
	_switch(&co->sched->ctx, &co->ctx);
#endif
}
```

参数：当前运行的协程实例
调用后该函数不会立即返回， 而是切换到最近执行 resume 的上下文。 该函数返回是在执行 resume 的时候，会有调度器统一选择 resume 的，然后再次调用 yield 的。 resume 与 yield 是两个可逆过程的原子操作。  

**resume： 恢复协程的运行权**  

参数： 需要恢复运行的协程实例调用后该函数也不会立即返回，而是切换到运行协程实例的 yield 的位置。 返回是在等协程相应事务处理完成后， 主动 yield 会返回到 resume 的地方。  

### 协程的上下文如何切换？ 切换代码如何实现？  

首先来回顾一下 x86_64 寄存器的相关知识。 x86_64 的寄存器有 16 个 64 位寄存器，分别是： %rax, %rbx, %rcx, %esi, %edi, %rbp, %rsp, %r8, %r9, %r10, %r11, %r12,%r13, %r14, %r15。%rax 作为函数返回值使用的。%rsp 栈指针寄存器， 指向栈顶%rdi, %rsi, %rdx, %rcx, %r8, %r9 用作函数参数， 依次对应第 1 参数， 第 2 参数。。。%rbx, %rbp, %r12, %r13, %r14, %r15 用作数据存储， 遵循调用者使用规则， 换句话说， 就是随便用。 调用子函数之前要备份它， 以防它被修改%r10, %r11 用作数据存储， 就是使用前要先保存原值。
上下文切换， 就是将 CPU 的寄存器暂时保存， 再将即将运行的协程的上下文寄存器， 分别mov 到相对应的寄存器上。 此时上下文完成切换。 如下图所示：  

![image-20231126164518014](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231126164518014.png)

切换_switch 函数定义：  

```c++
int _switch(nty_cpu_ctx *new_ctx, nty_cpu_ctx *cur_ctx);
```

参数 1： 即将运行协程的上下文，寄存器列表
参数 2：正在运行协程的上下文， 寄存器列表
我们 nty_cpu_ctx 结构体的定义， 为了兼容 x86， 结构体项命令采用的是 x86 的寄存器名字命名  

```c++
typedef struct _nty_cpu_ctx {
    void *esp; //
    void *ebp;
    void *eip;
    void *edi;
    void *esi;
    void *ebx;
    void *r1;
    void *r2;
    void *r3;
    void *r4;
    void *r5;
} nty_cpu_ctx;
```

_switch 返回后， 执行即将运行协程的上下文。 是实现上下文的切换  

按照 x86_64 的寄存器定义， %rdi 保存第一个参数的值，即 new_ctx 的值， %rsi 保存第二
个参数的值， 即保存 cur_ctx 的值。 X86_64 每个寄存器是 64bit， 8byte。
Movq %rsp, 0(%rsi) 保存在栈指针到 cur_ctx 实例的 rsp 项
Movq %rbp, 8(%rsi)
Movq (%rsp), %rax #将栈顶地址里面的值存储到 rax 寄存器中。 Ret 后出栈， 执行栈顶
Movq %rbp, 8(%rsi) #后续的指令都是用来保存 CPU 的寄存器到 new_ctx 的每一项中
Movq 8(%rdi), %rbp #将 new_ctx 的值
Movq 16(%rdi), %rax #将指令指针 rip 的值存储到 rax 中
Movq %rax, (%rsp) # 将存储的 rip 值的 rax 寄存器赋值给栈指针的地址的值。
Ret # 出栈， 回到栈指针， 执行 rip 指向的指令。
上下文环境的切换完成  



新创建的协程， 创建完成后， 加入到就绪集合， 等待调度器的调度； 协程在运行完成后，进行 IO 操作，此时 IO 并未准备好，进入等待状态集合； IO 准备就绪，协程开始运行，后续进行 sleep 操作，此时进入到睡眠状态集合。  

**就绪(ready)， 睡眠(sleep)， 等待(wait)集合该采用如何数据结构来存储？**
就绪(ready)集合并不没有设置优先级的选型， 所有在协程优先级一致， 所以可以使用队列来存储就绪的协程， 简称为就绪队列（ready_queue）。

睡眠(sleep)集合需要按照睡眠时长进行排序，采用红黑树来存储， 简称睡眠树(sleep_tree)红黑树在工程实用为<key, value>, key 为睡眠时长，value 为对应的协程结点。

等待(wait)集合，其功能是在等待 IO 准备就绪，等待 IO 也是有时长的，所以等待(wait)集合采用红黑树的来存储，简称等待树(wait_tree)，此处借鉴 nginx 的设计。
数据结构如下图所示：  

![image-20231126170651812](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231126170651812.png)

Coroutine 就是协程的相应属性， status 表示协程的运行状态。 sleep 与wait 两颗红黑树， ready 使用的队列， 比如某协程调用 sleep 函数， 加入睡眠树(sleep_tree)， status |= S 即可。 比如某协程在等待树(wait_tree)中， 而 IO 准备就绪放入 ready 队列中， 只需要移出等待树(wait_tree)， 状 态更改 status &= ~W 即可。 有一个前提条件就是不管何种运行状态的协程，都在就绪队列中， 只是同时包含有其他的运行状态。  

每一协程都需要使用的而且可能会不同属性的，就是协程属性。每一协程都需要的而且数据一致的，就是调度器的属性。 比如栈大小的数值，每个协程都一样的后不做更改可以作为调度器的属性，如果每个协程大小不一致，则可以作为协程的属性。用来管理所有协程的属性， 作为调度器的属性。 比如 epoll 用来管理每一个协程对应的 IO，是需要作为调度器属性  。

定义一个协程结构体需要多少域，我们描述了每一个协程有自己的上下文环境，需要保存 CPU 的寄存器 ctx；需要有子过程的回调函数 func；需要有子过程回调函数的参数 arg；需要定义自己的栈空间stack； 需要有自己栈空间的大小 stack_size；需要定义协程的创建时间
birth；需要定义协程当前的运行状态 status；需要定当前运行状态的结点（ready_next, wait_node, sleep_node）； 需要定义协程 id； 需要定义调度器的全局对象 sched。
协程的核心结构体如下  

```c++
typedef struct _nty_coroutine {

	//private
	
#ifdef _USE_UCONTEXT
	ucontext_t ctx;
#else
	nty_cpu_ctx ctx;
#endif
	proc_coroutine func;
	void *arg;
	void *data;
	size_t stack_size;
	size_t last_stack_size;
	
	nty_coroutine_status status;
	nty_schedule *sched;

	uint64_t birth;
	uint64_t id;
#if CANCEL_FD_WAIT_UINT64
	int fd;
	unsigned short events;  //POLL_EVENT
#else
	int64_t fd_wait;
#endif
	char funcname[64];
	struct _nty_coroutine *co_join;

	void **co_exit_ptr;
	void *stack;
	void *ebp;
	uint32_t ops;
	uint64_t sleep_usecs;

	RB_ENTRY(_nty_coroutine) sleep_node;
	RB_ENTRY(_nty_coroutine) wait_node;

	LIST_ENTRY(_nty_coroutine) busy_next;

	TAILQ_ENTRY(_nty_coroutine) ready_next;
	TAILQ_ENTRY(_nty_coroutine) defer_next;
	TAILQ_ENTRY(_nty_coroutine) cond_next;

	TAILQ_ENTRY(_nty_coroutine) io_next;
	TAILQ_ENTRY(_nty_coroutine) compute_next;

	struct {
		void *buf;
		size_t nbytes;
		int fd;
		int ret;
		int err;
	} io;

	struct _nty_coroutine_compute_sched *compute_sched;
	int ready_fds;
	struct pollfd *pfds;
	nfds_t nfds;
} nty_coroutine;
```

**调度器的属性，需要有保存 CPU 的寄存器上下文 ctx，可以从协程运行状态yield 到调度器运行的。从协程到调度器用 yield，从调度器到协程用 resume以下为协程的定义。**  

![image-20231126212302867](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231126212302867.png)

```c++
typedef struct _nty_schedule {
	uint64_t birth;
#ifdef _USE_UCONTEXT
	ucontext_t ctx;
#else
	nty_cpu_ctx ctx;
#endif
	void *stack;
	size_t stack_size;
	int spawned_coroutines;
	uint64_t default_timeout;
	struct _nty_coroutine *curr_thread;
	int page_size;

	int poller_fd;
	int eventfd;
	struct epoll_event eventlist[NTY_CO_MAX_EVENTS];
	int nevents;

	int num_new_events;
	pthread_mutex_t defer_mutex;

	nty_coroutine_queue ready;
	nty_coroutine_queue defer;

	nty_coroutine_link busy;
	
	nty_coroutine_rbtree_sleep sleeping;
	nty_coroutine_rbtree_wait waiting;

	//private 

} nty_schedule;
```

### 问题：协程如何被调度？  

调度器的实现，有两种方案，一种是生产者消费者模式，另一种多状态运行。  

#### 生产者消费者模式  

![image-20231126212444264](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231126212444264.png)



```c++
while (1) {
    //遍历睡眠集合，将满足条件的加入到 ready
    nty_coroutine *expired = NULL;
    while ((expired = sleep_tree_expired(sched)) != ) {
    	TAILQ_ADD(&sched->ready, expired);
    }
    //遍历等待集合，将满足添加的加入到 ready
    nty_coroutine *wait = NULL;
    int nready = epoll_wait(sched->epfd, events, EVENT_MAX, 1);
    for (i = 0;i < nready;i ++) {
        wait = wait_tree_search(events[i].data.fd);
        TAILQ_ADD(&sched->ready, wait);
	}
    // 使用 resume 回复 ready 的协程运行权
    while (!TAILQ_EMPTY(&sched->ready)) {
        nty_coroutine *ready = TAILQ_POP(sched->ready);
        resume(ready);
	}
}
```



#### 多状态运行  

![联想截图_20231126215645](C:\Users\zhen\Pictures\cut\联想截图_20231126215645.png)

```c++
while (1) {
    //遍历睡眠集合，使用 resume 恢复 expired 的协程运行权
    nty_coroutine *expired = NULL;
    while ((expired = sleep_tree_expired(sched)) != ) {
        resume(expired);
    }
    //遍历等待集合，使用 resume 恢复 wait 的协程运行权
    nty_coroutine *wait = NULL;
    int nready = epoll_wait(sched->epfd, events, EVENT_MAX, 1);
    for (i = 0;i < nready;i ++) {
        wait = wait_tree_search(events[i].data.fd);
        resume(wait);
    }
    // 使用 resume 恢复 ready 的协程运行权
    while (!TAILQ_EMPTY(sched->ready)) {
        nty_coroutine *ready = TAILQ_POP(sched->ready);
        resume(ready);
}
```



#### NtyCo的hook技术

hook的技术的本质就是不改变系统函数名，再调用的时候先调用到自己的逻辑，然后再进行系统调用，达到可以修改的目的。

```c++
typedef int (*socket_t)(int domain, int type, int protocol);
socket_t socket_f = NULL;

typedef int(*connect_t)(int, const struct sockaddr *, socklen_t);
connect_t connect_f = NULL;

typedef ssize_t(*read_t)(int, void *, size_t);
read_t read_f = NULL;

typedef ssize_t(*recv_t)(int sockfd, void *buf, size_t len, int flags);
recv_t recv_f = NULL;

typedef ssize_t(*recvfrom_t)(int sockfd, void *buf, size_t len, int flags,
        struct sockaddr *src_addr, socklen_t *addrlen);
recvfrom_t recvfrom_f = NULL;

typedef ssize_t(*write_t)(int, const void *, size_t);
write_t write_f = NULL;

typedef ssize_t(*send_t)(int sockfd, const void *buf, size_t len, int flags);
send_t send_f = NULL;

typedef ssize_t(*sendto_t)(int sockfd, const void *buf, size_t len, int flags,
        const struct sockaddr *dest_addr, socklen_t addrlen);
sendto_t sendto_f = NULL;

typedef int(*accept_t)(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
accept_t accept_f = NULL ;

// new-syscall
typedef int(*close_t)(int);
close_t close_f = NULL;

int init_hook(void) {

	socket_f = (socket_t)dlsym(RTLD_NEXT, "socket");
	
	read_f = (read_t)dlsym(RTLD_NEXT, "read");
	recv_f = (recv_t)dlsym(RTLD_NEXT, "recv");
	recvfrom_f = (recvfrom_t)dlsym(RTLD_NEXT, "recvfrom");

	write_f = (write_t)dlsym(RTLD_NEXT, "write");
	send_f = (send_t)dlsym(RTLD_NEXT, "send");
    sendto_f = (sendto_t)dlsym(RTLD_NEXT, "sendto");

	accept_f = (accept_t)dlsym(RTLD_NEXT, "accept");
	close_f = (close_t)dlsym(RTLD_NEXT, "close");
	connect_f = (connect_t)dlsym(RTLD_NEXT, "connect");

}
```

**重新定义socket函数**

```c++
int socket(int domain, int type, int protocol) {

	if (!socket_f) init_hook();
	int fd = socket_f(domain, type, protocol);
	if (fd == -1) {
		printf("Failed to create a new socket\n");
		return -1;
	}
	int ret = fcntl(fd, F_SETFL, O_NONBLOCK);
	if (ret == -1) {
		close(ret);
		return -1;
	}
	int reuse = 1;
	setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, (char *)&reuse, sizeof(reuse));
	
	return fd;
}
```

我们的逻辑函数

```c++
int nty_socket(int domain, int type, int protocol) {

	int fd = socket(domain, type, protocol);  //socket 会进入到我们定义的函数中
	if (fd == -1) {
		printf("Failed to create a new socket\n");
		return -1;
	}
	int ret = fcntl(fd, F_SETFL, O_NONBLOCK);
	if (ret == -1) {
		close(ret);
		return -1;
	}
	int reuse = 1;
	setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, (char *)&reuse, sizeof(reuse));
	
	return fd;
}

```



```c++
ssize_t read(int fd, void *buf, size_t count) {
	if (!read_f) init_hook();
	struct pollfd fds;
	fds.fd = fd;
	fds.events = POLLIN | POLLERR | POLLHUP;
	nty_poll_inner(&fds, 1, 1);
	int ret = read_f(fd, buf, count);
	if (ret < 0) {
		//if (errno == EAGAIN) return ret;
		if (errno == ECONNRESET) return -1;
		//printf("recv error : %d, ret : %d\n", errno, ret);
		
	}
	return ret;
}

ssize_t recv(int fd, void *buf, size_t len, int flags) {

	if (!recv_f) init_hook();

	struct pollfd fds;
	fds.fd = fd;
	fds.events = POLLIN | POLLERR | POLLHUP;

	nty_poll_inner(&fds, 1, 1);

	int ret = recv_f(fd, buf, len, flags);
	if (ret < 0) {
		//if (errno == EAGAIN) return ret;
		if (errno == ECONNRESET) return -1;
		//printf("recv error : %d, ret : %d\n", errno, ret);
		
	}
	return ret;
}

ssize_t recvfrom(int fd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen) {

	if (!recvfrom_f) init_hook();

	struct pollfd fds;
	fds.fd = fd;
	fds.events = POLLIN | POLLERR | POLLHUP;

	nty_poll_inner(&fds, 1, 1);

	int ret = recvfrom_f(fd, buf, len, flags, src_addr, addrlen);
	if (ret < 0) {
		if (errno == EAGAIN) return ret;
		if (errno == ECONNRESET) return 0;
		
		printf("recv error : %d, ret : %d\n", errno, ret);
		assert(0);
	}
	return ret;

}

ssize_t write(int fd, const void *buf, size_t count) {

	if (!write_f) init_hook();

	int sent = 0;

	int ret = write_f(fd, ((char*)buf)+sent, count-sent);
	if (ret == 0) return ret;
	if (ret > 0) sent += ret;

	while (sent < count) {
		struct pollfd fds;
		fds.fd = fd;
		fds.events = POLLOUT | POLLERR | POLLHUP;

		nty_poll_inner(&fds, 1, 1);
		ret = write_f(fd, ((char*)buf)+sent, count-sent);
		//printf("send --> len : %d\n", ret);
		if (ret <= 0) {			
			break;
		}
		sent += ret;
	}

	if (ret <= 0 && sent == 0) return ret;
	
	return sent;
}

ssize_t send(int fd, const void *buf, size_t len, int flags) {

	if (!send_f) init_hook();

	int sent = 0;

	int ret = send_f(fd, ((char*)buf)+sent, len-sent, flags);
	if (ret == 0) return ret;
	if (ret > 0) sent += ret;

	while (sent < len) {
		struct pollfd fds;
		fds.fd = fd;
		fds.events = POLLOUT | POLLERR | POLLHUP;

		nty_poll_inner(&fds, 1, 1);
		ret = send_f(fd, ((char*)buf)+sent, len-sent, flags);
		//printf("send --> len : %d\n", ret);
		if (ret <= 0) {			
			break;
		}
		sent += ret;
	}

	if (ret <= 0 && sent == 0) return ret;
	
	return sent;
}

ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
        const struct sockaddr *dest_addr, socklen_t addrlen) {

	if (!sendto_f) init_hook();

	struct pollfd fds;
	fds.fd = sockfd;
	fds.events = POLLOUT | POLLERR | POLLHUP;

	nty_poll_inner(&fds, 1, 1);

	int ret = sendto_f(sockfd, buf, len, flags, dest_addr, addrlen);
	if (ret < 0) {
		if (errno == EAGAIN) return ret;
		if (errno == ECONNRESET) return 0;
		
		printf("recv error : %d, ret : %d\n", errno, ret);
		assert(0);
	}
	return ret;

}

int accept(int fd, struct sockaddr *addr, socklen_t *len) {

	if (!accept_f) init_hook();

	int sockfd = -1;
	int timeout = 1;
	nty_coroutine *co = nty_coroutine_get_sched()->curr_thread;
	
	while (1) {
		struct pollfd fds;
		fds.fd = fd;
		fds.events = POLLIN | POLLERR | POLLHUP;
		nty_poll_inner(&fds, 1, timeout);

		sockfd = accept_f(fd, addr, len);
		if (sockfd < 0) {
			if (errno == EAGAIN) {
				continue;
			} else if (errno == ECONNABORTED) {
				printf("accept : ECONNABORTED\n");
				
			} else if (errno == EMFILE || errno == ENFILE) {
				printf("accept : EMFILE || ENFILE\n");
			}
			return -1;
		} else {
			break;
		}
	}

	int ret = fcntl(sockfd, F_SETFL, O_NONBLOCK);
	if (ret == -1) {
		close(sockfd);
		return -1;
	}
	int reuse = 1;
	setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, (char *)&reuse, sizeof(reuse));
	
	return sockfd;
}

int close(int fd) {

	if (!close_f) init_hook();

	return close_f(fd);
}



int connect(int fd, const struct sockaddr *addr, socklen_t addrlen) {

	if (!connect_f) init_hook();

	int ret = 0;

	while (1) {

		struct pollfd fds;
		fds.fd = fd;
		fds.events = POLLOUT | POLLERR | POLLHUP;
		nty_poll_inner(&fds, 1, 1);

		ret = connect_f(fd, addr, addrlen);
		if (ret == 0) break;

		if (ret == -1 && (errno == EAGAIN ||
			errno == EWOULDBLOCK || 
			errno == EINPROGRESS)) {
			continue;
		} else {
			break;
		}
	}

	return ret;
}

```



```c++
int nty_accept(int fd, struct sockaddr *addr, socklen_t *len) {
	int sockfd = -1;
	int timeout = 1;
	nty_coroutine *co = nty_coroutine_get_sched()->curr_thread;
	
	while (1) {
		struct pollfd fds;
		fds.fd = fd;
		fds.events = POLLIN | POLLERR | POLLHUP;
		nty_poll_inner(&fds, 1, timeout);

		sockfd = accept(fd, addr, len);
		if (sockfd < 0) {
			if (errno == EAGAIN) {
				continue;
			} else if (errno == ECONNABORTED) {
				printf("accept : ECONNABORTED\n");
				
			} else if (errno == EMFILE || errno == ENFILE) {
				printf("accept : EMFILE || ENFILE\n");
			}
			return -1;
		} else {
			break;
		}
	}

	int ret = fcntl(sockfd, F_SETFL, O_NONBLOCK);
	if (ret == -1) {
		close(sockfd);
		return -1;
	}
	int reuse = 1;
	setsockopt(fd, SOL_SOCKET, SO_REUSEADDR, (char *)&reuse, sizeof(reuse));
	
	return sockfd;
}


int nty_connect(int fd, struct sockaddr *name, socklen_t namelen) {

	int ret = 0;

	while (1) {

		struct pollfd fds;
		fds.fd = fd;
		fds.events = POLLOUT | POLLERR | POLLHUP;
		nty_poll_inner(&fds, 1, 1);

		ret = connect(fd, name, namelen);
		if (ret == 0) break;

		if (ret == -1 && (errno == EAGAIN ||
			errno == EWOULDBLOCK || 
			errno == EINPROGRESS)) {
			continue;
		} else {
			break;
		}
	}

	return ret;
}

//recv 
// add epoll first
//
ssize_t nty_recv(int fd, void *buf, size_t len, int flags) {
	
	struct pollfd fds;
	fds.fd = fd;
	fds.events = POLLIN | POLLERR | POLLHUP;

	nty_poll_inner(&fds, 1, 1);

	int ret = recv(fd, buf, len, flags);
	if (ret < 0) {
		//if (errno == EAGAIN) return ret;
		if (errno == ECONNRESET) return -1;
		//printf("recv error : %d, ret : %d\n", errno, ret);
		
	}
	return ret;
}


ssize_t nty_send(int fd, const void *buf, size_t len, int flags) {
	
	int sent = 0;

	int ret = send(fd, ((char*)buf)+sent, len-sent, flags);
	if (ret == 0) return ret;
	if (ret > 0) sent += ret;

	while (sent < len) {
		struct pollfd fds;
		fds.fd = fd;
		fds.events = POLLOUT | POLLERR | POLLHUP;

		nty_poll_inner(&fds, 1, 1);
		ret = send(fd, ((char*)buf)+sent, len-sent, flags);
		//printf("send --> len : %d\n", ret);
		if (ret <= 0) {			
			break;
		}
		sent += ret;
	}

	if (ret <= 0 && sent == 0) return ret;
	
	return sent;
}


ssize_t nty_sendto(int fd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen) {


	int sent = 0;

	while (sent < len) {
		struct pollfd fds;
		fds.fd = fd;
		fds.events = POLLOUT | POLLERR | POLLHUP;

		nty_poll_inner(&fds, 1, 1);
		int ret = sendto(fd, ((char*)buf)+sent, len-sent, flags, dest_addr, addrlen);
		if (ret <= 0) {
			if (errno == EAGAIN) continue;
			else if (errno == ECONNRESET) {
				return ret;
			}
			printf("send errno : %d, ret : %d\n", errno, ret);
			assert(0);
		}
		sent += ret;
	}
	return sent;
	
}

ssize_t nty_recvfrom(int fd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen) {

	struct pollfd fds;
	fds.fd = fd;
	fds.events = POLLIN | POLLERR | POLLHUP;

	nty_poll_inner(&fds, 1, 1);

	int ret = recvfrom(fd, buf, len, flags, src_addr, addrlen);
	if (ret < 0) {
		if (errno == EAGAIN) return ret;
		if (errno == ECONNRESET) return 0;
		
		printf("recv error : %d, ret : %d\n", errno, ret);
		assert(0);
	}
	return ret;

}




int nty_close(int fd) {
#if 0
	nty_schedule *sched = nty_coroutine_get_sched();

	nty_coroutine *co = sched->curr_thread;
	if (co) {
		TAILQ_INSERT_TAIL(&nty_coroutine_get_sched()->ready, co, ready_next);
		co->status |= BIT(NTY_COROUTINE_STATUS_FDEOF);
	}
#endif	
	return close(fd);
}
```





# Protobuf

Protocol buffers 是⼀种语⾔中⽴，平台⽆关，可扩展的序列化数据的格式，可⽤于通信协议，数据存储等。Protocol buffers 在序列化数据⽅⾯，它是灵活的，⾼效的。相⽐于 XML 来说，Protocol buffers 更加⼩巧，更加快速，更加简单。⼀旦定义了要处理的数据的数据结构之后，就可以利⽤ Protocol buffers 的代码⽣成⼯具⽣成相关的代码。甚⾄可以在⽆需重新部署程序的情况下更新数据结构。只需使⽤
Protobuf 对数据结构进⾏⼀次描述，即可利⽤各种不同语⾔或从各种不同数据流中对你的结构化数据轻松读写。Protocol buffers 很适合做数据存储或 RPC 数据交换格式。可⽤于通讯协议、数据存储等领域的语⾔⽆关、平台⽆关、可扩展的序列化结构数据格式。  



## 1. ProtoBuf 协议的⼯作流程  

![image-20231219223837857](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231219223837857.png)



可以看到，对于序列化协议来说，使⽤⽅只需要关注业务对象本身，即 idl 定义（.proto），序列化和反序列化的代码只需要通过⼯具⽣成即可。 

 ## 2. protobuf的编译安装  

https://github.com/protocolbuffers/protobuf  

> 解压: 
> tar zxvf protobuf-cpp-3.8.0.tar.gz  
>
> 编译 
>
> cd protobuf-3.8.0/
> ./configure
> make
> sudo make install 
>
> sudo ldconfig
>
> 显示版本信息  
>
> protoc --version  
>
> 将proto⽂件⽣成对应的.cc和.h⽂件  
>
> protoc -I=/路径1 --cpp_out=./路径2 /路径1/addressbook.proto
> 路径1为.proto所在的路径
> 路径2为.cc和.h⽣成的位置
>
> 将指定proto⽂件⽣成.pb.cc和.pb.h
> protoc -I=./ --cpp_out=./ test.proto  
>
> 将对应⽬录的所有proto⽂件⽣成.pb.cc和.pb.h
> protoc -I=./ --cpp_out=./ *.proto  

编译:

> g++ -std=c++11 -o list_people list_people.cc addressbook.pb.cc -lprotobuf -lpthread  

**protobuf option部分选项**  

option optimize_for = LITE_RUNTIME;
optimize_for是⽂件级别的选项，Protocol Buffer定义三种优化级别SPEED/CODE_SIZE/LITE_RUNTIME。缺省情况下是SPEED。  

1. SPEED: 表示⽣成的代码运⾏效率⾼，但是由此⽣成的代码编译后会占⽤更多的空间  
2. CODE_SIZE: 和SPEED恰恰相反，代码运⾏效率较低，但是由此⽣成的代码编译后会占⽤更少的空间，通常⽤于资源有限的平台，如Mobile  
3. LITE_RUNTIME: ⽣成的代码执⾏效率⾼，同时⽣成代码编译后的所占⽤的空间也是⾮常少。这是以牺牲Protocol Buffer提供的反射功能为代价的。因此我们在C++中链接Protocol Buffer库时仅需链接libprotobuf-lite，⽽⾮libprotobuf。  



## 3. 标量数值类型  

| .proto Type | Notes                                                        | C++ Type | Java Type   | Go Type |
| ----------- | ------------------------------------------------------------ | -------- | ----------- | ------- |
| double      | double                                                       | double   | float64     |         |
| float       | float                                                        | float    | float32     |         |
| int32       | 使⽤变⻓编码，对于负值的效率很低，如果你的域 有可能有负值，请使⽤sint64替代 | int32    | int         | int32   |
| uint32      | 使⽤变⻓编码                                                 | uint32   | int         | uint32  |
| uint64      | 使⽤变⻓编码                                                 | uint64   | long        | uint64  |
| sint32      | 使⽤变⻓编码，这些编码在负值时⽐int32⾼效的多                | int32    | int         | int32   |
| sint64      | 使⽤变⻓编码，有符号的整型值。编码时⽐通常的 int64⾼效。     | int64    | long        | int64   |
| fixed32     | 总是4个字节，如果数值总是⽐总是⽐2^28⼤的 话，这个类型会⽐uint32⾼效。 | uint32   | int         | uint32  |
| fixed64     | 总是8个字节，如果数值总是⽐总是⽐2^56⼤的 话，这个类型会⽐uint64⾼效。 | uint64   | long        | uint64  |
| sfixed32    | 总是4个字节                                                  | int32    | int         | int32   |
| sfixed64    | 总是8个字节                                                  | int64    | long        | int64   |
| bool        | bool                                                         | boolean  | bool        |         |
| string      | ⼀个字符串必须是UTF-8编码或者7-bit ASCII编 码的⽂本。        | string   | String      | string  |
| bytes       | 可能包含任意顺序的字节数据。                                 | string   | ByteStri ng | []byte  |



## 4. Protobuf 的数据组织  

在上⾯的讨论中, 我们了解了 Protobuf 所使⽤的 Varints 编码和 Zigzag 编码的编码原理, 本节我们来讨论 Protobuf 的数据组织⽅式, ⾸先来看⼀个例⼦, 假设客户端和服务端使⽤ protobuf 作为数据交换格式, proto 的具体定义为  

```protobuf
syntax = "proto3";
package pbTest;
message Request {
int32 age = 1;
}
```

Request 中包含了⼀个名称为 name 的字段, 客户端和服务端双⽅都⽤同⼀份相同的 proto ⽂件是没有任何问题的, 假设客户端⾃⼰将 proto ⽂件做了修改, 修改后的 proto ⽂件如下  

```protobuf
syntax = "proto3";
package pbTest;
message Request {
int32 age_test = 1;
}
```

在这种情形下, 服务端不修改应⽤程序仍能够正确地解码, 原因在于序列化后的 Protobuf 没有使⽤字段名称, ⽽仅仅采⽤了字段编号, 与 json xml 等相⽐, Protobuf 不是⼀种完全⾃描述的协议格式, 即接收端在没有 proto ⽂件定义的前提下是⽆法解码⼀个 protobuf 消息体的, 与此相对的,json xml 等协议格式是完全⾃描述的, 拿到了 json 消息体, 便可以知道这段消息体中有哪些字段, 每个字段的值分别是什么, 其实对于客户端和服务端通信双⽅来说, 约定好了消息格式之后完全没有必要在每⼀条消息中都携带字段名称, Protobuf 在通信数据中移除字段名称, 这可以⼤⼤降低消息的⻓度, 提⾼通信效率, Protobuf 进⼀步将通信线路上消息类型做了划分, 如下表所示  

| Type | Meaning          | Used For                                                 |
| ---- | ---------------- | -------------------------------------------------------- |
| 0    | Varint           | int32, int64, uint32, uint64, sint32, sint64, bool, enum |
| 1    | 64-bit           | fixed64, sfixed64, double                                |
| 2    | Length-delimited | string, bytes, embedded messages, packed repeated fields |
| 3    | Start group      | groups (deprecated)                                      |
| 4    | End group        | groups (deprecated)                                      |
| 5    | 32-bit           | fixed32, sfixed32, float                                 |

对于 int32, int64, uint32 等数据类型在序列化之后都会转为 Varints 编码, 除去两种已标记为 deprecated 的类型, ⽬前 Protobuf 在序列化之后的消息类型(wire-type) 总共有 4种, Protobuf 除了存储字段的值之外, 还存储了字段的编号以及字段在通信线路上的格式类型(wiretype), 具体的存储⽅式为  

field_num << 3 | wire type  

即将字段标号逻辑左移 3 位, 然后与该字段的 wire type 的编号按位或, 在上表中可以看到, wiretype 总共有 6 种类型, 因此可以⽤ 3 位⼆进制来标识, 所以低 3 位实际上存储了其后所跟的数据的wire type, 接收端可以利⽤这些信息, 结合 proto ⽂件来解码消息结构体, 我们以上⾯ proto 为例来看⼀段 Protobuf 实际序列化之后的完整⼆进制数据, 假设 age 为 5, 由于 age 在 proto ⽂件中定义的是 int32 类型, 因此序列化之后它的 wire type 为 0, 其字段编号为 1, 因此按照上⾯的计算⽅式,即 1 << 3 | 0, 所以其类型和字段编号的信息只占 1 个字节, 即 00001000, 后⾯跟上字段值 5 的Varints 编码, 所以整个结构体序列化之后为  

![image-20231219225545352](C:\Users\zhen\AppData\Roaming\Typora\typora-user-images\image-20231219225545352.png)

有了字段编号和 wire type, 其后所跟的数据的⻓度便是确定的, 因此 Protobuf 是⼀种⾮常紧密的数据组织格式, 其不需要特别地加⼊额外的分隔符来分割⼀个消息字段, 这可⼤⼤提升通信的效率, 规避冗余的数据传输  

