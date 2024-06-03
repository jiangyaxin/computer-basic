# 路由截断规则

**location 中的 proxy_pass 的 uri**

如果 `proxy_pass` 的 `url` 不带 `uri`

如果尾部是`/`，则会截断匹配的`uri`

如果尾部不是`/`，则不会截断匹配的`uri`

如果`proxy_pass`的`url`带`uri`，则会截断匹配的`uri`

示例：

```text
#-------servers配置--------------------
location / {
    echo $uri    #回显请求的uri
}

#--------proxy_pass配置---------------------
location /t1/ { proxy_pass http://servers; }    #正常，不截断
location /t2/ { proxy_pass http://servers/; }    #正常，截断
location /t3  { proxy_pass http://servers; }    #正常，不截断
location /t4  { proxy_pass http://servers/; }    #正常，截断
location /t5/ { proxy_pass http://servers/test/; }    #正常，截断
location /t6/ { proxy_pass http://servers/test; }    #缺"/"，截断
location /t7  { proxy_pass http://servers/test/; }    #含"//"，截断
location /t8  { proxy_pass http://servers/test; }    #正常，截断
#---------访问----------------------
for i in $(seq 6)
do
    url=http://localhost/t$i/doc/index.html
    echo "-----------$url-----------"
    curl url
done

#--------结果---------------------------
----------http://localhost:8080/t1/doc/index.html------------
/t1/doc/index.html

----------http://localhost:8080/t2/doc/index.html------------
/doc/index.html

----------http://localhost:8080/t3/doc/index.html------------
/t3/doc/index.html

----------http://localhost:8080/t4/doc/index.html------------
/doc/index.html

----------http://localhost:8080/t5/doc/index.html------------
/test/doc/index.html

----------http://localhost:8080/t6/doc/index.html------------
/testdoc/index.html

----------http://localhost:8080/t7/doc/index.html------------
/test//doc/index.html

----------http://localhost:8080/t8/doc/index.html------------
/test/doc/index.html
```

# 限流

## 控制流速

`ngx_http_limit_req_module` 模块提供限制请求处理速率能力。

1. 正常流量：

   `limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s;`

   * `key` ：定义限流对象，`binary_remote_addr` 是一种key，表示基于 `remote_addr`(客户端IP) 来做限流，`binary_` 的目的是压缩内存占用量。
   * `zone`：定义共享内存区来储存访问信息，one:10m 表示一个大于为 10M，名字为 one 的内存区域，1M 能储存16000 IP 地址的访问信息，10M 可以存储 16W IP地址访问信息。
   * `rate`：设置最大访问速率，rate=10r/s 表示每秒最多处理10个请求。Nginx 实际上以毫秒为粒度来跟踪请求信息，因此 10r/s 实际上是限制：每100毫秒处理一个请求。这意味着，自上一个请求处理完后，若后续100毫秒内又有请求到达，将拒绝处理该请求。如果限制的频率低于1r/s，则可以使用r/m，如30r/m。

2. 突发流量：

   `limit_req zone=one burst=5 nodelay;`

   * `zone=one`：设置使用哪个配置区域来做限制，与上面`limit_req_zone` 里的name对应。
   * `burst=5`：burst爆发的意思，这个配置的意思是设置一个大小为5的缓冲区当有大量请求（爆发）过来时，超过了访问频次限制的请求可以先放到这个缓冲区内。
   * `nodelay`：如果设置，超过访问频次而且缓冲区也满了的时候就会直接返回503，如果没有设置，则所有请求会等待排队。

示例：

```text
http {
    limit_req_zone $binary_remote_addr zone=one:10m rate=1r/s; 
    server {
        location /search/ {
            limit_req zone=one; 
        }
}
```

## 并发限制

`ngx_http_limit_conn_module`提供限制单个IP的请求数，只有在服务器处理了请求并且已经读取了整个请求头时，连接才被计数。

示例：

```text
limit_conn_zone $binary_remote_addr zone=perip:10m;
limit_conn_zone $server_name zone=perserver:10m;

server {
    ...
    limit_conn perip 10;
    limit_conn perserver 100;
}
```

* `limit_conn perip 10` 作用的key是 `$binary_remote_addr`，表示限制单个IP同时最多能持有10个连接。

* `limit_conn perserver 100` 作用的key是 `$server_name`，表示虚拟主机(server) 同时能处理并发连接的总数。

# 实战

## 设置白名单

使用  `ngx_http_geo_module` 和 `ngx_http_map_module` 两个工具模块即可搞定。

```text
geo $limit {
    default 1;
    10.0.0.0/8 0;
    192.168.0.0/24 0;
    172.20.0.35 0;
}
map $limit $limit_key {
    0 "";
    1 $binary_remote_addr;
}
limit_req_zone $limit_key zone=myRateLimit:10m rate=10r/s;
```

* `geo` 对于白名单(子网或IP都可以) 将返回0，其他IP将返回1。

* `map` 将 `$limit` 转换为 `$limit_key`，如果是 `$limit` 是0(白名单)，则返回空字符串；如果是1，则返回客户端实际IP。

* `limit_req_zone` 限流的key不再使用 `$binary_remote_addr`，而是 `$limit_key` 来动态获取值。如果是白名单，`limit_req_zone` 的限流key则为空字符串，将不会限流；若不是白名单，将会对客户端真实IP进行限流。