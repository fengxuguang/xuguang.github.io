---
title: shell脚本清空指定目录下的文件内容
tags:
  - shell
abbrlink: 3cbd56d
date: 2024-08-20 17:49:53
---

想要情况指定文件夹下的所有文件的内容，但是保留文件应该怎么做呢？

通过编写 shell 脚本获取指定文件夹下所有 .log 结尾的文件，写入空数据，脚本如下`truncateFileContent.sh`：

```shell
#!/bin/bash

# 获取指定的文件夹路径
dir=$1

cd $dir

# 循环遍历目录下的所有文件
for file in *.log
do
	# 判断是否为文件，不是则跳过
	if [ -f file ]
	then
		echo "" > $file
	fi
done

echo $dir + "文件夹下的所有 .log 文件内容已清空！"
```

使用姿势：

```shell
sh ./truncateFileContent.sh /opt/data/test/
```

