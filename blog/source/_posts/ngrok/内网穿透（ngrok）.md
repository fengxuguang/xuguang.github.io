---
title: 内网穿透（ngrok）
tags:
  - ngrok
abbrlink: 36bbd933
date: 2024-06-04 12:02:50
---

## 一、ngrok 官网地址

https://ngrok.com/，使用 github 账号登录。

## 二、下载

登录成功后下载对应软件

![image.png](../..//images/articles/ngrok/ngrok.jpg)

## 三、获取鉴权 token

![image-20240604120711574](../../images/articles/ngrok/ngrok-token.jpg)

## 四、使用

![image.png](../../images/articles/ngrok/ngrok-use.jpg)

安装包下载后解压，点击“ngrok.exe”文件执行。

执行后会弹出 DOS 命令框

![image.png](../../images/articles/ngrok/ngrok-help.jpg)

1. 输入如下命令进行 token 授权

```shell
ngrok authtoken <token>
```

2. 暴露服务端口

```shell
ngrok http 10088
```

![image.png](../../images/articles/ngrok/ngrok-info.jpg)
