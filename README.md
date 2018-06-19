# php-architecture
# 系统重构及架构调整方案


## 全站 HTTP2HTTPS

### Why Https
https://developers.google.com/web/fundamentals/security/encrypt-in-transit/why-https?hl=zh-cn

### Https Steps
- step1 - 购买并部署服务器证书

> 登录 https://cn.globalsign.com/support/support_vetting_dvssl.html 联系对方销售购买ca证书

> Nginx 安装服务器证书，Url: https://cn.globalsign.com/support/download/Nginx.pdf

- step2 － Nginx 转发所有http请求到https

```
server {
    listen 80;
    server_name api.smallmitao.com;
    add_header Strict-Transport-Security max-age=31536000;
    return 301 https://api.smallmitao.com$request_uri;
}
``` 

## 安全(数据库)

### 现存安全隐患
- 1、生产库、测试库 同存于同一台rds实例
- 2、测试库帐号可用于dump生产库
- 3、敏感信息未做脱敏处理(姓名、手机号、身份证等)

### 解决方法
- 1、将生产库独立一台rds，测试机器中安装Mysql Server
- 2、编写脱敏SQL脚本、定时从生产rds中脱敏并同步生产数据

### 优点
- 防止程序员登录生产库排查数据问题
- 防止敏感信息泄漏

## 数据库服务器编排
- Mysql Master Server(主数据库) 
- Mysql Slave Server(从数据库)
- Mysql Sync Server(数据库同步服务端，脱敏同步)
- MasterDB -> SlaveDB <- SyncServer

## 代码风格
```
请严格遵守 https://psr.phphub.org/
1 基础编码规范
2 编码风格规范
```

## 轻API，重LIB
```
1 API 负责数据的输入、类型验证、调用CORE、输出结果
2 CORE 负责所有业务的低耦合实现
3 这种设计大大增加了代码可重用性
例 伪代码
<?php
// 接口文件
用户名 ＝ 获取用户名
密码 ＝ 获取用户名
结果 ＝ 用户名密码类型验证()
IF 结果为真 THEN
    结果2 ＝ User::login(用户名, 密码)
    IF 结果2 为真
        输出正确信息
    ELSE 
        输出错误信息
    ENDIF
ELSE
输出错误信息
ENDIF
```

## 程序发布流程
```
1 从主分支git checkout 出新分支。
2 分支命名规范为 hotfix_*, dev_*,例如: hotfix_user_login_err, dev_add_user_login_log
3 git fetch origin
4 git merge origin/master （必须解决完所有冲突）
5 git checkout master
6 git merge hotfix_user_login_err --ff-only （保证无冲突 有冲突请回到第三步）
7 git tag -a 20180501_ hotfix_user_login_err -m '修复用户登录异常的bug'
8 git push origin 20180501_ hotfix_user_login_err
```

## 数据更新流程
```
1 新建仓库 命名为: dba
2 仓库下新建 [20180501]分支名称／hotfix_user_login_err.sql
3 git add .
4 git commit -m "修复登录异常的用户数据"
5 git push
```

## 一键启动开发环境(docker-compose)

```
git clone https://github.com/luyunhua/dnmp.git
git checkout -b dev_laravel_support origin/dev_laravel_support
修改docker-compose.yml 中volumes目录映射、conf/conf.d/目录下新增新的配置文件 可拷贝一份进行修改
docker-compose up -d
docker-compose down
```




