# 在运行时控制NGINX进程

    了解处理流量的NGINX进程，以及如何在运行时控制它们

## 主进程 和 工作进程

    NGINX有一个主进程和一个或多个工作进程。如果启用了缓存，缓存加载器和缓存管理器进程也会在启动时运行。

    主进程的主要目的是读取和评估配置文件，以及维护工作进程。

    工作进程实际处理请求。NGINX依靠依赖于操作系统的机制在工作进程之间高效地分发请求。
    工作进程的数量由nginx.conf配置文件中的worker_processs指令定义，可以设置为固定数量，也可以配置为自动调整到可用CPU内核的数量。

## 控制Nginx

    要重新加载配置，您可以停止或重新启动NGINX，或向主进程发送信号。可以通过运行带有-s参数的nginx命令（调用nginx可执行文件）来发送信号。
       
* `nginx -s quit`   优雅地关闭,停止 nginx 进程并等待工作进程完成当前请求的服务；此命令应在启动 nginx 的同一用户下执行
    
* `nginx -s stop`   立即关闭

*  `nginx -s reload` 重新加载配置文件,配置文件中所做的更改只有在将重新加载配置的命令发送到 nginx 或重新启动后才会应用；一旦主进程收到重新加载配置的信号，它就会检查新配置文件的语法有效性并尝试应用其中提供的配置。如果成功，主进程将启动新的工作进程并向旧工作进程发送消息，要求它们关闭。否则，主进程将回滚更改并继续使用旧配置。旧工作进程收到关闭命令后，将停止接受新连接并继续为当前请求提供服务，直到所有此类请求都得到服务。此后，旧工作进程将退出。

* `nginx -s reopen` 重新打开日志文件（SIGUSR1信号）
   
* kill实用程序也可用于直接向主进程发送信号。默认情况下，主进程的进程ID会写入nginx.pid文件，该文件位于/usr/local/nnginx/logs或/var/run目录中。
   

# 创建NGINX Plus和NGINX配置文件

## 指令 
    简单单行指令均以封号结尾；
    其他指令充当“容器”，将相关指令组合在一起，用花括号（{}）括起来；这些通常被称为块；


## 特定功能配置文件

    为了使配置更易于维护，我们建议您将其拆分为一组存储在/etc/nginx/conf.d目录中的特定功能文件，并使用nginx.conf主文件中的include指令引用特定功能文件的内容。

    `include conf.d/http;`
    `include conf.d/stream;`
    `include conf.d/exchange-enhanced;`

## 上下文 Contexts

    一些顶级指令（称为上下文）将适用于不同流量类型的指令组合在一起：

* events–常规连接处理
* http–http流量
* 邮件–邮件流量
* 流–TCP和UDP流量

    置于这些上下文之外的指令被称为处于主上下文中。

## 虚拟服务器

    在每个流量处理上下文中，您都包含一个或多个服务器块来定义控制请求处理的虚拟服务器。您可以在服务器上下文中包含的指令因流量类型而异。

    对于HTTP流量（HTTP上下文），每个服务器指令都控制对特定域或IP地址的资源请求的处理。服务器上下文中的一个或多个位置上下文定义了如何处理特定的URI集。

    对于邮件和TCP/UDP流量（邮件和流上下文），服务器指令分别控制到达特定TCP端口或UNIX套接字的流量的处理。

## 继承
    一般来说，子上下文（一个包含在另一个上下文（其父级）中）继承了父级包含的指令设置。某些指令可能出现在多个上下文中，在这种情况下，您可以通过将指令包含在子上下文中来覆盖从父上下文继承的设置。例如，请参阅proxy_set_header指令。

## 重新加载配置

    要使对配置文件的更改生效，必须重新加载它。您可以重新启动nginx进程，也可以发送reload信号来升级配置，而不会中断当前请求的处理。有关详细信息，请参阅在运行时控制NGINX进程。

    使用NGINX Plus，您可以在上游组中的服务器之间动态重新配置负载平衡，而无需重新加载配置。您还可以使用NGINX Plus API和密钥值存储来动态控制访问，例如基于客户端IP地址。