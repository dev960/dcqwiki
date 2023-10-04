---
title: "nginx离线安装(国产)"
date: 2023-09-14T21:37:56+08:00
lastmod: 2023-09-14T21:37:56+08:00
draft: false
tags: ["nginx", "国产"]
categories: ["笔记"]
author: "dcq"

contentCopyright: '[Creative Commons Attribution-ShareAlike License](https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License)'

---

# 1. 依赖：

   wget https://nginx.org/download/nginx-1.20.2.tar.gz

   wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/pcre-8.44.tar.gz
   tar xzf pcre-8.44.tar.gz

   wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/zlib-1.2.11.tar.gz
   tar xzf zlib-1.2.11.tar.gz

   wget https://buildpack.oss-cn-shanghai.aliyuncs.com/static/r6d/nginx/nginx-compile-lib/openssl-1.1.1l.tar.gz
   tar xzf openssl-1.1.1l.tar.gz

# 2. 安装步骤

```shell
cd /eflex/packages/env
tar -xvf pcre-8.44.tar.gz
cd /eflex/packages/env/pcre-8.44
./configure
make
make install
```

```shell
cd /eflex/packages/env
tar -xvf openssl-1.1.1l.tar.gz
cd /eflex/packages/env/openssl-1.1.1l
./config
make
make install
```

```shell
cd /eflex/packages/env
tar -xvf zlib-1.2.11.tar.gz
cd /eflex/packages/env/zlib-1.2.11
./config
make
make install
```

# 3. 编译：

```shell
./configure --prefix=/eflex/app/packages/nginx --with-http_ssl_module --with-openssl=../../env/openssl-1.1.1l --with-pcre=../../env/pcre-8.44  --with-zlib=../../env/zlib-1.2.11
```

# 4. 运行：

```shell
$ make && make install
./sbin/nginx -c ../conf/nginx.conf
```

#### 2. 防火墙开启

> sudo ufw enable
> 
> sudo ufw default deny
> 
> sudo ufw allow 22/tcp
> 
> sudo ufw allow 2231/tcp
> 
> sudo ufw allow 8000/tcp
