# nginx配置WebSocket服务的一些配置信息

在server的前面增加websocket的参数信息，防止socket短时间断开

```
map $http_upgrade $connection_upgrade{
                default upgrade ;
                '' close;
        }
```

配置拦截信息

```
        location /cloud/ws {
                proxy_pass http://192.168.1.200:15613/ ;
                proxy_set_header       Host $host;
                proxy_set_header  X-Real-IP  $remote_addr;
                proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_http_version 1.1;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection "upgrade";
        }

```



------

如果后端还有使用spring cloud gateway进行的websocket的代理，则要填写spring cloud gateway的地址，其次，gateway中的跨域设置会导致

```java
2022-10-13 15:28:38.581 ERROR 111997 --- [or-http-epoll-5] o.s.w.s.adapter.HttpWebHandlerAdapter    : [c9a7e7bb-18] Error [java.lang.UnsupportedOperationException] for HTTP GET "/ws/1/1580460487718604800/", but ServerHttpResponse already committed (200 OK)
2022-10-13 15:28:38.583 ERROR 111997 --- [or-http-epoll-5] r.n.http.server.HttpServerOperations     : [c9a7e7bb-1, L:/192.168.1.111:82 - R:/10.10.10.2:56170] Error finishing response. Closing connection

java.lang.UnsupportedOperationException: null
        at org.springframework.http.ReadOnlyHttpHeaders.set(ReadOnlyHttpHeaders.java:106) ~[spring-web-5.3.23.jar:5.3.23]
        Suppressed: reactor.core.publisher.FluxOnAssembly$OnAssemblyException:
Error has been observed at the following site(s):
        *__checkpoint ⇢ org.springframework.cloud.gateway.filter.WeightCalculatorWebFilter [DefaultWebFilterChain]
        *__checkpoint ⇢ HTTP GET "/ws/1/1580460487718604800/" [ExceptionHandlingWebHandler]
Original Stack Trace:
                at org.springframework.http.ReadOnlyHttpHeaders.set(ReadOnlyHttpHeaders.java:106) ~[spring-web-5.3.23.jar:5.3.23]
                at org.springframework.cloud.gateway.filter.factory.DedupeResponseHeaderGatewayFilterFactory.dedupe(DedupeResponseHeaderGatewayFilterFactory.java:141) ~[spring-cloud-gateway-server-3.1.4.jar:3.1.4]
                at org.springframework.cloud.gateway.filter.factory.DedupeResponseHeaderGatewayFilterFactory.dedupe(DedupeResponseHeaderGatewayFilterFactory.java:130) ~[spring-cloud-gateway-server-3.1.4.jar:3.1.4]
                at org.springframework.cloud.gateway.filter.factory.DedupeResponseHeaderGatewayFilterFactory$1.lambda$filter$0(DedupeResponseHeaderGatewayFilterFactory.java:93) ~[spring-cloud-gateway-server-3.1.4.jar:3.1.4]
                at reactor.core.publisher.MonoRunnable.call(MonoRunnable.java:73) ~[reactor-core-3.4.23.jar:3.4.23]
                at reactor.core.publisher.MonoRunnable.call(MonoRunnable.java:32) ~[reactor-core-3.4.23.jar:3.4.23]
                at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.subscribeNext(MonoIgnoreThen.java:228) [reactor-core-3.4.23.jar:3.4.23]
                at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.onComplete(MonoIgnoreThen.java:203) [reactor-core-3.4.23.jar:3.4.23]
                at reactor.core.publisher.FluxHide$SuppressFuseableSubscriber.onComplete(FluxHide.java:147) [reactor-core-3.4.23.jar:3.4.23]
                at reactor.core.publisher.MonoIgnoreThen$ThenIgnoreMain.onComplete(MonoIgnoreThen.java:209) [reactor-core-3.4.23.jar:3.4.23]
                at reactor.core.publisher.FluxDoOnEach$DoOnEachSubscriber.onComplete(FluxDoOnEach.java:223) [reactor-core-3.4.23.jar:3.4.23]
                at reactor.core.publisher.Operators.complete(Operators.java:137) [reactor-core-3.4.23.jar:3.4.23]
```

去掉跨越配置后，虽可以正常连接，但发现只有发送，没有回复。（待后续配置测试）

