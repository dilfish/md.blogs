---
title: "Mac 下为 Clion 生成 nginx 的 CMakeLists.txt 文件"
date: 2020-03-26T10:53:25+08:00
draft: false
---

## CLion + nginx + Mac

```cmake
project(nginx)
cmake_minimum_required(VERSION 3.1)

INCLUDE_DIRECTORIES(./)
INCLUDE_DIRECTORIES(/usr/include/libxml2)
INCLUDE_DIRECTORIES(/usr/local/Cellar/pcre/8.44/include)
INCLUDE_DIRECTORIES(/Users/seanzhang/Downloads/nginx-1.17.9/deps/openssl-1.1.1e/include)
INCLUDE_DIRECTORIES(./objs)
INCLUDE_DIRECTORIES(./src/core)
INCLUDE_DIRECTORIES(./src/event)
INCLUDE_DIRECTORIES(./src/os/unix)
INCLUDE_DIRECTORIES(./src/http)
INCLUDE_DIRECTORIES(./src/http/modules)
INCLUDE_DIRECTORIES(./src/mail)

LINK_DIRECTORIES(/Users/seanzhang/Downloads/nginx-1.17.9/deps/openssl-1.1.1e)

aux_source_directory(. SRC_LIST)
aux_source_directory(./src/core SRC_LIST)
aux_source_directory(./src/event SRC_LIST)
aux_source_directory(./src/os/unix SRC_LIST)
aux_source_directory(./src/http SRC_LIST)
aux_source_directory(./src/http/modules SRC_LIST)


set(SRC_LIST ${SRC_LIST} ./src/event/modules/ngx_kqueue_module.c)
set(SRC_LIST ${SRC_LIST} ./objs/ngx_modules.c)

list(REMOVE_ITEM SRC_LIST ./src/os/unix/ngx_linux_init.c ./src/os/unix/ngx_linux_sendfile_chain.c)
list(REMOVE_ITEM SRC_LIST ./src/os/unix/ngx_freebsd_init.c ./src/os/unix/ngx_freebsd_sendfile_chain.c)
list(REMOVE_ITEM SRC_LIST ./src/os/unix/ngx_solaris_init.c ./src/os/unix/ngx_solaris_sendfilev_chain.c)
list(REMOVE_ITEM SRC_LIST ./src/core/ngx_thread_pool.c)
list(REMOVE_ITEM SRC_LIST ./src/os/unix/ngx_file_aio_read.c)
list(REMOVE_ITEM SRC_LIST ./src/os/unix/ngx_linux_aio_read.c)
list(REMOVE_ITEM SRC_LIST ./src/os/unix/ngx_thread_cond.c)
list(REMOVE_ITEM SRC_LIST ./src/os/unix/ngx_thread_mutex.c)
list(REMOVE_ITEM SRC_LIST ./src/os/unix/ngx_thread_id.c)
list(REMOVE_ITEM SRC_LIST ./src/http/modules/ngx_http_dav_module.c)
list(REMOVE_ITEM SRC_LIST ./src/http/modules/ngx_http_geoip_module.c)
list(REMOVE_ITEM SRC_LIST ./src/http/modules/ngx_http_degradation_module.c)
list(REMOVE_ITEM SRC_LIST ./src/http/modules/ngx_http_grpc_module.c)
list(REMOVE_ITEM SRC_LIST ./src/http/modules/ngx_http_image_filter_module.c)
list(REMOVE_ITEM SRC_LIST ./src/http/modules/ngx_http_stub_status_module.c)

add_executable(${PROJECT_NAME} ${SRC_LIST})

TARGET_LINK_LIBRARIES (${PROJECT_NAME} dl pthread crypto ldap lber pcre ssl crypto dl pthread z xml2 xslt exslt)
```
