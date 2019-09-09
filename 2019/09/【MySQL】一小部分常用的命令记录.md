# 【MySQL】一小部分常用的命令记录

## 前言

在Github上发现了一个挺好玩的repo，打算按照其`README.md`在服务器上部署到自己的VPS时，看到过程需要配置数据库。好嘛，之前搜索过的一下又不记得了，因此，做下笔记，记下**当时部署时使用到的**一些简单的配置命令

## 指令

**查看数据库**

```
show databases;
```

**创建数据库**

```
create database 数据库名称;
```

**创建数据库时指定字符编码**

指定为utf-8

```
CREATE DATABASE 数据库名称 DEFAULT CHARACTER SET utf8 COLLATE utf8_general_ci;
```

指定为GBK

```
create database 数据库名称 DEFAULT CHARACTER SET gbk COLLATE gbk_chinese_ci;
```

**添加用户**

```
create user "用户名"@"服务器地址" identified by "密码";
```

此处服务器地址可以为

`%` 代表任意情境下可登录

`localhost` 代表仅本地登录

`8.8.8.8` 代表仅这个IP地址可登录

**授权用户访问数据库**

```
grant 权限名称 on `数据库名`.`表名` to '用户名'@'服务器地址';

flush privileges;  # 刷新权限
```

其中权限名称可以是

`all privileges`  指增删改查等所有权限

`select` 查询权限

`insert` 写入权限

`delete` 删除权限

`update` 更新权限

其中表名可以是

`*` 指数据库下的所有表

## 尾巴

嘛~暂时就只是这些了，因为也只是简单的新建一个数据库给应用而已，不涉及到一些增删改查的命令，所以也就只有这些建库建用户的命令而已~在这里做个记录，以便自己和需要用到的小伙伴能在需要时用得上