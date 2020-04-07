---
title: Laradock 使用 Swoole
date: 2019.03.19 18:33:28
tags: [docker]
categories: 后端
---

- 环境：mac
- docker 版本：2.0.0.3（31259）
- laradock 版本 ：254a9ae19490b53fc8f0647d81b7581c312f0724
- laravel 版本：5.4.*
- laravel-swoole 版本：^2.5

1. 首先我们需要在 `laradock` 的`.env`文件下面修改`WORKSPACE_INSTALL_SWOOLE=true`
1. 重新 build 一下虚拟机 `docker-compose build workspace`
1. 重新 build 好了之后，启动
1. 进入虚拟机检查一下是否安装成功了 `php -m | grep swoole`，如果打印出了`swoole`，就证明安装成功了
1. 接下来，我们要修改一下 nginx 的配置文件 ` `
   ```
        map $http_upgrade $connection_upgrade {
            default upgrade;
            ''      close;
        }
        upstream laravels {
            # Connect IP:Port
            server workspace:1215 weight=5 max_fails=3 fail_timeout=30s;
            keepalive 16;
        }
        server {

            listen 80;
        #    listen [::]:80 ipv6only=on;

            server_name yourdomain.com;
            root /var/www/swoole/public;
            index index.php index.html index.htm;
            error_log /var/www/swoole_error.log;

            location = /index.php {
                # Ensure that there is no such file named "not_exists"
                # in your "public" directory.
                try_files /not_exists @swoole;
            }

            location / {
                 try_files $uri $uri/ @swoole;
            }

            location @swoole {
                set $suffix "";

                if ($uri = /index.php) {
                    set $suffix ?$query_string;
                }

                proxy_set_header Host $http_host;
                proxy_set_header Scheme $scheme;
                proxy_set_header SERVER_PORT $server_port;
                proxy_set_header REMOTE_ADDR $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header Upgrade $http_upgrade;
                proxy_set_header Connection $connection_upgrade;

                # IF https
                # proxy_set_header HTTPS "on";

                proxy_pass http://laravels$suffix;
            }

            location ~ /\.ht {
                deny all;
            }

            location /.well-known/acme-challenge/ {
                root /var/www/letsencrypt/;
                log_not_found off;
            }
        }
     ```
    这份配置文件，是参照官方文档的，这里面有个很关键的地方，就是修改 upsteam 那里，`server workspace:1215`。因为我们 Nginx 的运行是跟 laravel 的环境在不同一台机子的，所以你必须修改这里的upsteam，不然就会造成502。
1. 接下来，我们进入我们的 laravel 项目，安装一下`laravel-swoole`，`composer require laravel-swoole`
1. 然后接下来，我们可以修改一下 `.env` 文件，让`laravel-swoole`变成守护进程启动还有指定 swoole 代理的host，`SWOOLE_HTTP_HOST=workspace
SWOOLE_HTTP_DAEMONIZE=true`，端口我没有修改，默认是`1215`，如果有需要可以自行修改，记得修改nginx。
1. `php artisan swoole:http start`，启动swoole，打开我们的网页
1. 我修改了host，所以我用的是自定义的域名，打开之后如果你看到
    ![image.png](https://upload-images.jianshu.io/upload_images/121543-3b377e95b14e238c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
恭喜你，成功了。另外，假设你发现启动了swoole 之后，性能反而变慢了，那就要进行一些参数调优了，具体可以参照官方的文档[swoole](https://wiki.swoole.com/wiki/page/520.html)，这里就不展开讲了。