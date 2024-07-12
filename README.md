# Squid OpenSSL - Docker

因为基于GnuTLS的squid的`tls-cert`加载证书的时候不能包含中间证书，造成有些客户端无法正确验证，所以需要用这个基于OpenSSL的squid，这样就可以加载中间证书了。

# 基本使用

如果想要基于https的代理可以新建一个`conf.d`文件夹然后在里面增加一个`https.conf`文件：

```
https_port 9443 tls-cert=/path/to/cert/domain-fullchain.cer tls-key=/path/to/cert/domain.key
http_access allow all
```

然后创建`docker-compose.yml`如下：

```yml
services:
  app:
    image: ecat/squid-openssl
    restart: always
    ports:
      - "9443:9443"
    volumes:
      - "./conf.d:/etc/squid/conf.d"
      - "./log:/var/log/squid"
      - "/path/to/cert:/path/to/cert:ro"
```

上面的`/path/to/cert`是域名`domain`的位置，需要映射到容器里面。

这样启动后就可以通过`https://domain:9433`进行基于https的代理了。

> 可以通过`openssl s_client -connect domain:9443 -showcerts`命令查看ServerHello返回的证书。

代理测试可以用curl命令：

```sh
curl -v --proxy https://domain:9433  https://www.baidu.com
```

如果代理加了验证可以加上`--proxy-user user:password`参数。


# 参考

- [Let's Encrypt certificate for client to Squid proxy encryption - Help - Let's Encrypt Community Support](https://community.letsencrypt.org/t/lets-encrypt-certificate-for-client-to-squid-proxy-encryption/206978/10)
- [squid : http_port configuration directive](http://www.squid-cache.org/Doc/config/http_port/)
