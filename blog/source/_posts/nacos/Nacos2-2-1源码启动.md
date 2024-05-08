---
title: Nacos2.2.1源码启动
tags:
  - nacos
abbrlink: 89a74d17
date: 2024-04-19 16:35:39
---

## 启动流程

1. 找到 Nacos 的控制台启动类

   ![image-20240419163826445](../../images/articles/nacos/nacos-start.png)

 2. 配置启动选项（默认 cluster 集群模式启动，会报错，需要将其改为 单机模式 standalone）

    -Dnacos.standalone=true

    ![image-20240419164224322](../../images/articles/nacos/nacos-param.png)

 3. 修改 application.properties 配置文件

    ```propert
    # 给下面两个属性随机设值
    nacos.core.auth.server.identity.key=111
    nacos.core.auth.server.identity.value=111
    
    # 打开此属性
    nacos.core.auth.plugin.nacos.token.secret.key=SecretKey012345678901234567890123456789012345678901234567890123456789
    ```

    
