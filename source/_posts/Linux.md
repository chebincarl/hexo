---
layout: title
title: Linux
date: 2019-04-02 14:15:59
tags: Linux
---
Linux的cheat sheets

<!--more-->

# 查看Linux系统版本信息

## 查看Linux内核版本命令（两种方法）：
```Bash
cat /proc/version
```
```Bash
uname -a
```

## 查看Linux系统版本的命令（3种方法）：
```Bash
lsb_release -a    即可列出所有版本信息。
```

```Bash
cat /etc/redhat-release  这种方法只适合Redhat系的Linux。
```

```Bash
cat /etc/issue  CentOS 7中无信息。
```

# 用户

由于日常使用时root用户权限过大，所以添加一个用户供日常使用，或者供他人使用。有些软件（例如Elasticsearch）不支持root用户启动。

## 查看用户和用户组
用户列表文件：/etc/passwd
用户组列表文件：/etc/group

查看系统中有哪些用户
```Bash
cut -d : -f 1 /etc/passwd
```

查看可以登录系统的用户
```Bash
cat /etc/passwd | grep -v /sbin/nologin | cut -d : -f 1
```

查看用户操作：
```Bash
w (需要root权限)   查看当前活跃的用户列表
```

查看某一用户
```Bash
w 用户名
```

查看登录用户
```Bash
who
```

查看用户登录历史记录
```Bash
last
```

查看登录日志
```Bash
who /var/log/wtmp
```

## 创建
1、创建新用户
```Bash
adduser [用户名]
```
2、修改新用户的密码
```Bash
passwd [用户名]
```
## 授权
新创建的用户并不能使用sudo命令，需要给此用户添加授权。
1、添加sudoers文件可写权限
```Bash
chmod -v u+w /etc/sudoers
```
2、修改sudoers文件
```Bash
vim /etc/sudoers
[用户名]    ALL=(ALL)    ALL（如需新用户使用sudo时不用输密码，把最后一个ALL改为NOPASSWD:ALL即可）
```
3、收回sudoers文件可写权限
```Bash
chmod -v u-w /etc/sudoers
```