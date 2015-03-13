---
layout: post
title: '[Docker] Generate Nginx .conf File from Docker Containers'
---

之前一段时间我一直纠结使用Docker创建多个Web容器的Nginx自动反向转发问题。

之后我得出了一个最小化解决方案，从当前运行的容器获取需要反向转发的本机地址。

比如说:

Web容器内部使用3000端口，`fig`会自动分配一个随机的主机端口与之对应。

我们只需要通过Docker API获取当前运行的容器信息，筛选内部暴露端口为3000的容器，从他们的主机端口生成一份 Nginx Upstream 文件到一个固定位置，在主要Nginx配置文件内部 include 这个 Upstream 文件。

每次容器发生变化，用一个固定脚本更新这个文件，然后让Nginx重载即可。

为此我写了一个脚本，最初为Node.js版，但是考虑到Docker主机一般为最小化主机，专门安装Node.js不科学。

因此之后写了一个Python版本。

地址如下: [https://gist.github.com/yanke-guo/919a660fcfa56c79fe06](https://gist.github.com/yanke-guo/919a660fcfa56c79fe06)

假设主要 Nginx 配置文件如下:


```
include includes/my-tibird-com-upstream.conf;

server {
  listen 80;
  server_name  tibird.com;
  location / {
         ....
         ....
        }
```

每次容器变动的时候，跑一个小的脚本即可：

```bash
#!/bin/bash

set -u
set -e

CONFIG_PATH=/etc/nginx/includes
CONFIG_FILENAME=my-tibird-com-upstream.conf
CONFIG_FILE=$CONFIG_PATH/$CONFIG_FILENAME

mkdir -p $CONFIG_PATH

./util/gen-nginx-upstream.py -p 3000 -n my-tibird > $CONFIG_FILE

nginx -s reload
```

I've thought out the proper way to reverse proxy Docker web containers with Nginx;

Then I got a minimal solution, regenerate nginx upstream file and reload each time container changes.

For example:

Rails web containers uses 3000 port internally, `fig` will map random host ports to them.

The easiest way is to get container informations via Docker API, filter out containers who expose 3000 internally, and generate a Nginx upstream file from their external ports.

The main Nginx config file include that upstream file.

Each time container changes, update the upstream file with a script and reload Nginx.

At first I wrote a Node.js version. Considering usually Docker host is a minimal host, install a Node.js is not sensible, I write a Python version, which is more commonly installed.

Link: [https://gist.github.com/yanke-guo/919a660fcfa56c79fe06](https://gist.github.com/yanke-guo/919a660fcfa56c79fe06)

Assume main nginx config file looks like this:

```
include includes/my-tibird-com-upstream.conf;

server {
  listen 80;
  server_name  tibird.com;
  location / {
         ....
         ....
        }
```

Run a small shell script each time containers change：

```bash
#!/bin/bash

set -u
set -e

CONFIG_PATH=/etc/nginx/includes
CONFIG_FILENAME=my-tibird-com-upstream.conf
CONFIG_FILE=$CONFIG_PATH/$CONFIG_FILENAME

mkdir -p $CONFIG_PATH

./util/gen-nginx-upstream.py -p 3000 -n my-tibird > $CONFIG_FILE

nginx -s reload
```
