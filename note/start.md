# How to start a yii2-advanced project.

## 一、初始化一个项目
```shell
php ini
```
![start](https://github.com/pklim101/yii2codelib/blob/master/note/imagelib/start.jpg)


## 二、配置数据库
```shell
vim common/config/main-local.php
```
## 三、用户登录
### 1. 创建用户表
用户表scheme路径：vendor\mdmsoft\yii2-admin\migrations\schema-mysql.sql
```SQL
CREATE TABLE `user` (
`id` int(11) NOT NULL AUTO_INCREMENT COMMENT '自增ID', 
`username` varchar(255) NOT NULL COMMENT '用户名', 
`auth_key` varchar(32) NOT NULL COMMENT '自动登录key', 
`password_hash` varchar(255) NOT NULL COMMENT '加密密码', 
`password_reset_token` varchar(255) DEFAULT NULL COMMENT '重置密码token', 
`email` varchar(255) NOT NULL COMMENT '邮箱', 
`role` smallint(6) NOT NULL DEFAULT '10' COMMENT '角色等级', 
`status` smallint(6) NOT NULL DEFAULT '10' COMMENT '状态', 
`created_at` int(11) NOT NULL COMMENT '创建时间', 
`updated_at` int(11) NOT NULL COMMENT '更新时间', 
PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8 COMMENT='用户表';
```
### 2. 访问frontend，先注册个用户；
### 3. 登录即可使用项目。
