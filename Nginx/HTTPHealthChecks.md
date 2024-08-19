    通过发送定期健康检查来监控上游组中HTTP服务器的健康状况，包括NGINX Plus中可定制的活动健康检查。
    NGINX和NGINX Plus可以持续测试您的上游服务器，避免出现故障的服务器，并将恢复的服务器优雅地添加到负载平衡组中。

* Passive Health Checks  被动健康检查

    对于被动健康检查，NGINX和NGINX Plus会在事务发生时进行监控，并尝试恢复失败的连接。如果事务仍然无法恢复，NGINX开源和NGINX Plus会将服务器标记为不可用，并暂时停止向其发送请求，直到它再次标记为活动

    fail_timeout–设置服务器必须发生多次失败尝试才能标记为不可用的时间，以及服务器标记为不可使用的时间（默认值为10秒）。

    max_fails–设置在fail_timeout期间必须发生的失败尝试次数，以便服务器标记为不可用（默认值为1次尝试）。

    请注意，如果组中只有一台服务器，则fail_timeout和max_fails参数将被忽略，服务器永远不会被标记为不可用。

    * Server Slow Start
        最近恢复的服务器很容易被连接淹没，这可能会导致服务器再次标记为不可用。慢速启动允许上游服务器在恢复或可用后逐渐将其权重从零恢复到标称值。这可以通过上游服务器指令的slow_start参数来实现：

        请注意，如果组中只有一台服务器，则slow_start参数将被忽略，服务器永远不会被标记为不可用。慢启动是NGINX Plus独有的。

* Active Health Checks 主动健康检查
  
    NGINX Plus可以通过向每台服务器发送特殊的健康检查请求并验证正确的响应来定期检查上游服务器的健康状况。

    默认情况下，NGINX Plus每五秒钟向后端组中的每台服务器发送一次“/”请求,如果发生任何通信错误或超时（服务器的响应状态代码在200到399的范围之外），则健康检查失败。服务器被标记为不健康，NGINX Plus在再次通过健康检查之前不会向其发送客户端请求.

    您可以选择指定另一个端口进行健康检查，例如，用于监视同一主机上许多服务的健康状况。使用health_check指令的port参数指定新端口

    在上游服务器组中，使用zone指令定义共享内存区域：


    该区域在所有工作进程之间共享，并存储上游组的配置。这使工作进程能够使用同一组计数器来跟踪组中服务器的响应。活动健康检查的默认值可以用health_check指令的参数覆盖：

    `location / {
    proxy_pass   http://backend;
    health_check interval=10 fails=3 passes=2;
    }`

    在这里，interval参数将健康检查之间的延迟从默认的5秒增加到10秒。fails参数要求服务器在三次健康检查中失败，才能标记为不健康（高于默认值）。最后，passs参数意味着服务器必须连续通过两次检查才能再次标记为健康，而不是默认检查。

    您还可以使用keepalive_time参数启用连接缓存-在TLS上游的情况下，不会对每个健康检查探测都进行完整的TLS握手，并且可以在指定的时间段内重用连接：
    `location / {
    proxy_http_version 1.1;
    proxy_set_header   Connection "";
    proxy_pass         https://backend;
    health_check       interval=1 keepalive_time=60s;
    }`

    * 指定请求的URI

    使用health_check指令的uri参数设置要在健康检查中请求的uri：

    `location / {
    proxy_pass   http://backend;
    health_check uri=/some/path;
    }`

    指定的URI将附加到上游块中为服务器设置的服务器域名或IP地址。对于上面声明的示例后端组中的第一个服务器，健康检查请求URI“http://backend1.example.com/some/path".

    * 定义自定义条件

    您可以设置响应必须满足的自定义条件，服务器才能通过健康检查。这些条件在匹配块中定义，该块在health_check指令的match参数中引用。

        在http{}级别，指定match{}块并命名它，例如server_ok：

        通过指定匹配参数和匹配块的名称，参考health_check指令中的块：
    `http {
    #...
    match server_ok {
        status 200-399;
        body   !~ "maintenance mode";
    }
    server {
        #...        
        location / {
            proxy_pass   http://backend;
            health_check match=server_ok;
        }
    }
    }`

    match指令使NGINX Plus能够检查状态代码、标头字段和响应正文。使用此指令，可以验证状态是否在指定范围内，响应是否包含标头，或者标头或正文是否与正则表达式匹配。match指令可以包含一个状态条件、一个正文条件和多个标头条件。响应必须满足匹配块中定义的所有条件，服务器才能通过健康检查。

    例如，以下match指令匹配具有状态代码200、Content-Type标头中的确切值text/html和文本Welcome to nginx的响应！在体内：

    `match welcome {
    status 200;
    header Content-Type = text/html;
    body   ~ "Welcome to nginx!";
}`


以下示例使用感叹号（！）定义响应必须通过健康检查的特征。在这种情况下，当状态代码不是301、302、303或307，并且没有Refresh标头时，健康检查通过。

    `match not_redirect {
    status ! 301-303 307;
    header ! Refresh;
}`


    * 强制性健康检查
  
  默认情况下，当新服务器添加到上游组时，NGINX Plus会认为它是健康的，并立即向其发送流量。但对于某些服务器，特别是如果它们是通过API接口或DNS解析添加的，那么在允许它们处理流量之前，最好先进行健康检查。

  强制参数要求每个新添加的服务器在NGINX Plus向其发送流量之前通过所有配置的健康检查。
  当与慢速启动相结合时，它为新服务器提供了更多的时间来连接到数据库并“预热”，然后再要求其处理全部流量。

  强制性健康检查可以标记为持久性，以便在重新加载配置时记住之前的状态。指定持久参数和强制参数：

    `upstream my_upstream {
        zone   my_upstream 64k;
        server backend1.example.com slow_start=30s;
    }

    server {
        location / {
            proxy_pass   http://my_upstream;
            health_check mandatory persistent;
        }
    }`

  这里指定了health_check指令的强制和持久参数以及server指令的slow_start参数。使用API或DNS接口添加到上游组的服务器被标记为不健康，并且在通过健康检查之前不接收流量；此时，它们开始在30秒内接收逐渐增加的流量。如果重新加载NGINX Plus配置，并且在重新加载之前服务器被标记为健康，则不会执行强制健康检查，并且服务器状态被认为是正常的。