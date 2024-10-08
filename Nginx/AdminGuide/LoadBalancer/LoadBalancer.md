# HTTP负载平衡

    通过多种算法和高级功能（如慢启动和会话持久性）在web或应用程序服务器组之间平衡HTTP流量。

    跨多个应用程序实例的负载平衡是一种常用的技术，用于优化资源利用率、最大化吞吐量、减少延迟和确保容错配置。

    根据需求观看NGINX Plus负载平衡和扩展网络研讨会，深入了解NGINX用户用于构建大规模、高可用web服务的技术。

    NGINX和NGINX Plus可以在不同的部署场景中用作非常高效的HTTP负载均衡器。

## 将HTTP流量代理到一组服务器

    首先需要使用上游指令定义该组。该指令放置在http上下文中。
    要将请求传递给服务器组，请在proxy_pass指令（或这些协议的fastcgi_pass、memcached_pass、scgi_pass或uwsgi_pass指令）中指定组的名称。

## 选择负载平衡方法
    NGINX开源支持四种负载平衡方法，NGINX Plus增加了另外两种方法：

### 循环-
    请求在服务器上均匀分布，并考虑服务器权重。默认情况下使用此方法（没有启用它的指令）：

### 最少连接–
    向活动连接数最少的服务器发送请求，同样考虑服务器权重：

### IP哈希-
    根据客户端IP地址确定向其发送请求的服务器。在这种情况下，使用IPv4地址的前三个八位字节或整个IPv6地址来计算哈希值。该方法保证来自同一地址的请求到达同一服务器，除非该服务器不可用。

    如果需要从负载平衡轮换中临时删除其中一台服务器，可以用down参数标记它，以保留客户端IP地址的当前哈希值。此服务器处理的请求会自动发送到组中的下一个服务器：

### 通用哈希-
    请求发送到的服务器由用户定义的密钥确定，该密钥可以是文本字符串、变量或组合。例如，密钥可以是成对的源IP地址和端口，也可以是本例中的URI：

    hash指令的可选一致性参数启用ketama一致性哈希负载平衡。请求根据用户定义的哈希键值在所有上游服务器上均匀分布。如果将上游服务器添加到上游组或从上游组中删除，则只会重新映射几个键，从而在负载平衡缓存服务器或其他累积状态的应用程序的情况下最大限度地减少缓存缺失。


### 最少时间（仅限NGINX Plus）-
    对于每个请求，NGINX Plus会选择平均延迟最低、活动连接数量最少的服务器，其中最低平均延迟是根据包含以下哪些参数来计算的：

    header–从服务器接收第一个字节的时间
    last_byte–从服务器接收完整响应的时间
    last_byte飞行中–考虑到不完整的请求，从服务器接收完整响应的时间

### 随机-
    每个请求都将被传递给随机选择的服务器。如果指定了这两个参数，首先，NGINX会根据服务器权重随机选择两个服务器，然后使用指定的方法选择其中一个服务器：

    least_conn–活动连接数最少
    least_time=header（NGINX Plus）–从服务器接收响应标头的最短平均时间（$upstream_header_time）
    least_time=last_byte（NGINX Plus）-从服务器接收完整响应的最短平均时间（$upstream_response_time）

    随机负载平衡方法应用于分布式环境，其中多个负载平衡器将请求传递到同一组后端。对于负载平衡器可以全面查看所有请求的环境，请使用其他负载平衡方法，如轮询、最少连接和最少时间。

    注意：在配置除轮转之外的任何方法时，请将相应的指令（hash、ip_hash、least_conn、least_time或random）放在上游｛｝块中的服务器指令列表上方。

## 服务器权重
    默认情况下，NGINX使用轮转方法根据组中服务器的权重在服务器之间分发请求。服务器指令的权重参数设置服务器的权重；默认值为1:


## 服务器慢启动

    服务器慢启动功能可防止最近恢复的服务器被连接淹没，这可能会超时并导致服务器再次标记为失败。

    在NGINX Plus中，慢启动允许上游服务器在恢复或可用后逐渐将其权重从0恢复到标称值。这可以通过server指令的slow_start参数来实现：

    时间值（此处为30秒）设置NGINX Plus将服务器连接数增加到最大值的时间。

    请注意，如果组中只有一台服务器，则服务器指令的max_fails、fail_timeout和slow_start参数将被忽略，服务器永远不会被视为不可用。

## 启用会话持久性

    会话持久性意味着NGINX Plus识别用户会话，并将给定会话中的所有请求路由到同一上游服务器。

    NGINX Plus支持三种会话持久化方法。这些方法是用sticky指令设置的。（对于NGINX开源的会话持久性，请使用如上所述的hash或ip_hash指令。）

### Sticky cookie-
    NGINX Plus在上游组的第一个响应中添加会话cookie，并标识发送响应的服务器。客户端的下一个请求包含cookie值，NGINX Plus将请求路由到对第一个请求做出响应的上游服务器：

    在该示例中，srv_id参数设置cookie的名称。可选的expires参数设置浏览器保留cookie的时间（此处为1小时）。可选域参数定义了设置cookie的域，可选路径参数定义了为其设置cookie的路径。这是最简单的会话持久化方法。

### Sticky route ——
    NGINX Plus在收到第一个请求时为客户端分配一条“路由”。将所有后续请求与服务器指令的路由参数进行比较，以识别请求所代理的服务器。路由信息来自cookie或请求URI。

### Sticky learn method ——
    NGINX Plus首先通过检查请求和响应来查找会话标识符。然后NGINX Plus“学习”哪个上游服务器对应哪个会话标识符。通常，这些标识符在HTTP cookie中传递。如果请求包含已“学习”的会话标识符，NGINX Plus会将请求转发到相应的服务器：


### 限制连接数量 max_conns

如果已达到max_conns限制，则将请求放入队列中进行进一步处理，前提是还包含队列指令以设置队列中可以同时处理的最大请求数：



## 配置健康检查


## 与多个工作进程共享数据


### 设置区域大小


## 使用DNS配置HTTP负载平衡--
    可以在运行时使用DNS修改服务器组的配置
    `http {
        resolver 10.0.0.1 valid=300s ipv6=off;
        resolver_timeout 10s;
        server {
            location / {
                proxy_pass http://backend;
            }
        }
        upstream backend {
            zone backend 32k;
            least_conn;
            # ...
            server backend1.example.com resolve;
            server backend2.example.com resolve;
        }
    } `

    服务器指令的resolve参数告诉NGINX Plus定期将backend1.example.com和backend2.example.com域名重新解析为IP地址。

    解析器指令定义了NGINX Plus向其发送请求的DNS服务器的IP地址（此处为10.0.0.1）。默认情况下，NGINX Plus会按照记录中的生存时间（TTL）指定的频率重新解析DNS记录，但您可以使用有效参数覆盖TTL值；在该示例中为300秒或5分钟。

    可选的ipv6=off参数意味着只有IPv4地址用于负载平衡，尽管默认情况下支持解析IPv4和ipv6地址。

    如果域名解析为多个IP地址，则这些地址将保存到上游配置并进行负载平衡。在我们的示例中，服务器根据最小连接负载平衡方法进行负载平衡。如果服务器的IP地址列表发生了变化，NGINX Plus会立即开始跨新地址集进行负载平衡。

## Microsoft Exchange服务器的负载平衡

    在NGINX Plus Release 7及更高版本中，NGINX Plus可以将Microsoft Exchange流量代理到服务器或一组服务器，并对其进行负载平衡。

* 要设置Microsoft Exchange服务器的负载平衡，请执行以下操作：
    在位置块中，使用proxy_pass指令配置到上游Microsoft Exchange服务器组的代理：
* 为了使Microsoft Exchange连接传递到上游服务器，在位置块中将proxy_http_version指令值设置为1.1，将proxy_set_header指令设置为Connection“”，就像保活连接一样：


TCP和UDP负载平衡

## 将NGINX配置为邮件代理服务器

## 动态模块