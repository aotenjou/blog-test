---
title: Java学习之路4
date: 2024-09-17 22:44:53
tags:
- 'Javaweb'
- '课外'
- '收纳分布式项目'
---
 <!-- more -->
# HTTP协议
- 基于TCP
- 一次请求一次响应
- 无状态协议

### 请求数据格式
**请求行**:
- 请求数据第一行
- 请求方式(GET POST)请求路径,协议.
**请求头**:
- 第二行开始
- key: value

    HOST 请求主机名
    User-Agent 浏览器版本
    Accept 浏览器能接受的资源类型 */*所有
    Accept-Language
    Accept-Encoding
    Content-Type
    Content-Length

**请求体**
- POST请求(GET无请求体)
- POST大小无限制

### 响应数据格式
**响应行**:
- 响应数据第一行
- 协议
- 状态
  
    1xx 响应中
    2xx 成功
    3xx redirect
    4xx client error
    5xx server error

- 描述
**响应头**
- key:value

    content type  html/text/json
    content length 长度
    content encoding 压缩算法
    cache control 如何缓存
    set cookie 设置cookie
**响应体**
- 响应正文:json数据

### 协议解析
