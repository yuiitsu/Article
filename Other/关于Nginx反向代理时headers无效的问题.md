# 关于Nginx反向代理时headers无效的问题

这真是一个巨坑

Nginx在做反向代理时，一般会设置host和ip，如果你的请求的headers里有值，它是可以同时转发过去的，但是，默认情况下，并不是所有headers的fields它都会转发，fields里带有_的，Nginx视为不合法，自动抛弃不发了。如：

```bash
access_token: abcde
```

查询文档，里面有这么一段：

> | Syntax:  | underscores_in_headers on \| off; |
> | :------- | --------------------------------- |
> | Default: | underscores_in_headers off;       |
> | Context: | http, server`                     |
>
> Enables or disables the use of underscores in client request header fields. When the use of underscores is disabled, request header fields whose names contain underscores are marked as invalid and become subject to the [ignore_invalid_headers](http://nginx.org/en/docs/http/ngx_http_core_module.html#ignore_invalid_headers) directive.

意思很明白，想要支持下划线(_)的headers fields，就需要将underscores_in_headers设置为on。

但是，如果只是设置它，会发现，并没用，因为还要设置一项：

```bash
proxy_pass_request_headers on;
```

OK，完成的例子：

```bash 
server {
	...
	underscores_in_headers on;
	
	location / {
		proxy_pass_request_headres on;
		proxy_pass http://server;
	}
}
```



