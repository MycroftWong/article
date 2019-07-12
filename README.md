# nginx

因为在学习`Spring Boot`的过程中，想搭建网页，而且了解到`Spring Boot`常用做微服务，并非`MVC`，而且`nginx`更适合做静态网页，在使用`Spring Boot`搭建网页时也遇到些问题，所以驱动我学习`nginx`。

## 目的

* 搭建静态网站
* 使用`nginx`实现反向代理，实现动静分离
* 使用`nginx`强转`HTTP`为`HTTPS`

## 指令

### `docker`启动`nginx`

```
## 这个是网上搜到的使用docker启动nginx的命令
docker run  -d -P --name nginxweb -v /root/nginx/nginx.conf:/etc/nginx/nginx.conf -v /root/nginx/conf.d/:/etc/nginx/conf.d/ -v /www:/www -v /root/nginx/logs/:/home/nginx/logs/ nginx

## 下面是我自己在windows 10上使用docker启动nginx的命令，出现的问题时，不知道什么原因，总提示配置文件出错，但是配置文件在Cent OS上检查是没有问题的
docker run -d -p 80:80 -P --name nginx -v D:/nginx/docker/nginx.conf:/etc/nginx/nginx.conf -v D:/nginx/docker/conf.d/:/etc/nginx/conf.d/ -v D:/nginx/docker/www:/www -v D:/nginx/docker/logs/:/home/nginx/logs/ nginx
```

### `nginx`简单的命令

```shell
## 启动
nginx
## 检查配置文件
### 1. 检查默认的配置文件 /etc/nginx/nginx.conf
nginx -t

### 2. 检查指定的配置文件 /home/nginx/nginx.conf
nginx -t -c /home/nginx/nginx.conf

## 重启
nginx -s reload

## 停止
nginx -s stop
```

## TODO

* 如何设置静态资源访问权限，就算是最简单的`HTML`也可能希望设置访问权限，例如在需要在`header`中添加`token`


## 参考

[nginx动静分离之后，设置默认主页](https://www.cnblogs.com/jcici/p/9990504.html)

[nginx error_page详解](https://blog.csdn.net/hyk_lk/article/details/65936042)

[检查nginx配置文件是否正确](https://blog.csdn.net/weixin_38823568/article/details/81296901)

[Nginx https配置 和 反向代理到spring boot和vue.js](https://segmentfault.com/a/1190000016760251)

[SpringBoot + Nginx 配置HTTPS的一次经历](https://blog.csdn.net/fenglin0429/article/details/81347634)

[Nginx配置http强制跳转到https](https://blog.csdn.net/fjinhao/article/details/75909702)

[Nginx代理https强制http跳转https](https://blog.csdn.net/weixin_41679874/article/details/86249970)

