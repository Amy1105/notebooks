    负载平衡是指在多个后端服务器之间有效地分配网络流量。
   

* Prerequisites  先决条件

     在NGINX Plus Release 5及更高版本中，NGINX Plus可以代理和负载平衡传输控制协议（TCP）流量。TCP是许多流行应用程序和服务的协议，如LDAP、MySQL和RTMP。
    在NGINX Plus Release 9及更高版本中，NGINX Plus可以代理和负载平衡UDP流量。UDP（用户数据报协议）是许多流行的非事务性应用程序的协议，如DNS、syslog和RADIUS。

    首先，您需要配置反向代理，以便NGINX Plus或NGINX开源可以将TCP连接或UDP数据报从客户端转发到上游组或代理服务器。

* Configuring Reverse Proxy 配置反向代理
  
    * 创建顶级流｛｝块：
    * 在顶级流｛｝上下文中为每个虚拟服务器定义一个或多个服务器｛｝配置块。
    * 在每个服务器的服务器｛｝配置块中，包含listen指令，以定义服务器侦听的IP地址和/或端口。对于UDP流量，还包括UDP参数。由于TCP是流上下文的默认协议，侦听指令中没有TCP参数：
    * 包含proxy_pass指令以定义代理服务器或服务器将流量转发到的上游组：
    * 如果代理服务器有多个网络接口，您可以选择将NGINX配置为在连接到上游服务器时使用特定的源IP地址。如果NGINX后面的代理服务器被配置为接受来自特定IP网络或IP地址范围的连接，这可能会很有用。包括proxy_bind指令和相应网络接口的IP地址
    * 您可以选择调整两个内存缓冲区的大小，NGINX可以在其中放置来自客户端和上游连接的数据。如果数据量较小，可以减少缓冲区，从而节省内存资源。如果有大量数据，可以增加缓冲区大小以减少套接字读/写操作的数量。一旦在一个连接上接收到数据，NGINX就会读取并通过另一个连接转发。缓冲区由proxy_buffer_size指令控制：

* Configuring TCP or UDP Load Balancing 配置TCP或UDP负载平衡


* Configuring Health Checks 配置健康检查

* On-the-Fly Configuration 飞行配置

    上游服务器组可以使用NGINX Plus REST API轻松地在现场重新配置。使用此界面，您可以查看上游组或特定服务器中的所有服务器，修改服务器参数，以及添加或删除上游服务器



通过发送定期健康检查来监控上游组中HTTP服务器的健康状况，包括NGINX Plus中可定制的活动健康检查。







