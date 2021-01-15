# RabbitMQ 3.3.5-1 安装 el6.5
## 1. 安装编译工具以及epel 扩展
``` shell
yum -y install epel-release
yum -y install ncurses ncurses-base ncurses-devel ncurses-libs ncurses-static ncurses-term ocaml-curses ocaml-curses-devel 
yum –y install openssl-devel zlib-devel 
yum -y install make ncurses-devel gcc gcc-c++ unixODBC unixODBC-devel openssl openssl-devel
```
**如果遇到 Cannot retrieve metalink for repository: epel. Please verify its path and try again **

**修改`/etc/yum.repos.d/epel.repo`**

- 打开/etc/yum.repos.d/epel.repo，将
    ``` repo
    [epel]
    name=Extra Packages for Enterprise Linux 6 - $basearch
    #baseurl=http://download.fedoraproject.org/pub/epel/6/$basearch
    mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=$basearch
    ```
- 修改为
    ``` repo
    [epel]
    name=Extra Packages for Enterprise Linux 6 - $basearch
    baseurl=http://download.fedoraproject.org/pub/epel/6/$basearch
    #mirrorlist=https://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=$basearch
    ```
## 2.安装Erlang
``` shell
yum -y install Erlang

```
- 查看是否安装完成
	``` shell
	cd /usr/local/bin
	erl
  Erlang R14B04 (erts-5.8.5) [source] [64-bit] [smp:2:2] [rq:2] 	[async-threads:0] [kernel-poll:false]
  Eshell V5.8.5  (abort with ^G)
	```
## 2.安装RabbitMQ 3.3.5-1
``` shell
wget https://www.rabbitmq.com/releases/rabbitmq-server/v3.3.5/rabbitmq-server-3.3.5-1.noarch.rpm
rpm -ivh rabbitmq-server-3.3.5-1.noarch.rpm
```
- rabbitmq 的安装目录
	``` shell
	[root@localhost bin]# whereis rabbitmq
	rabbitmq: /etc/rabbitmq /usr/lib/rabbitmq
	```
## 3. 非 root 权限启动 rabbitmq

**安装完成之后 rabbitmq 会在 /usr/sbin/ 目录下加入所需要的命令，如 rabbitmq-server，默认会以rabbitmq用户启动，而使用该用户需要root权限。但是我们在生产机器上不可能一直使用 root，因此使用普通用户启动 rabbitmq 非常有必要**

1. 删除/usr/sbin/rabbitmq*
	``` shell
	rm -f /usr/sbin/rabbitmq*
	```
2. 添加软链
	``` shell
	# 注意：rabbitmq_server-3.3.5 换成你安装的对应版本
    ln -s /usr/lib/rabbitmq/lib/rabbitmq_server-3.3.5/sbin/rabbitmqctl /usr/sbin/rabbitmqctl
    ln -s /usr/lib/rabbitmq/lib/rabbitmq_server-3.3.5/sbin/rabbitmq-env /usr/sbin/rabbitmq-env
    ln -s /usr/lib/rabbitmq/lib/rabbitmq_server-3.3.5/sbin/rabbitmq-server /usr/sbin/rabbitmq-server
    ln -s /usr/lib/rabbitmq/lib/rabbitmq_server-3.3.5/sbin/rabbitmq-defaults /usr/sbin/rabbitmq-defaults
    ln -s /usr/lib/rabbitmq/lib/rabbitmq_server-3.3.5/sbin/rabbitmq-diagnostics /usr/sbin/rabbitmq-diagnostics
    ln -s /usr/lib/rabbitmq/lib/rabbitmq_server-3.3.5/sbin/rabbitmq-plugins /usr/sbin/rabbitmq-plugins
	```
3. 修改文件用户权限
	``` shell
	chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/mnesia/
	```
## 4. 启动rabbitmq
``` shell
su  - rabbitmq
rabbitmq-server  -detached #后台运行
#rabbitmq-server  start #前台运行
```
## 5. 启动web页面管理
``` shell
-bash-4.1$ rabbitmq-plugins enable rabbitmq_management
The following plugins have been enabled:
  mochiweb
  webmachine
  rabbitmq_web_dispatch
  amqp_client
  rabbitmq_management_agent
  rabbitmq_management
Plugin configuration has changed. Restart RabbitMQ for changes to take effect.
```
## 6. 添加管理用户并登录web
**guest 用户是登不进的**
- 添加普通用户
    ``` shell
    -bash-4.1$ rabbitmqctl add_user testuser qwe123
    Creating user "testuser" ...
    ...done
    ```
- 分配管理员权限
    ``` shell
    -bash-4.1$ rabbitmqctl set_user_tags testuser administrator
    Setting tags for user "testuser" to [administrator] ...
    ...done
    ```
- 查询用户是否正常添加
    ``` shell
    -bash-4.1$ rabbitmqctl list_users
    Listing users ...
    guest	[administrator]
    testuser	[administrator]
    ```

## 7. web 登录
**浏览器**
- **IP:15672** 
- 输入刚刚创建的账户和密码
- 登入

## 8. rabbitmq config 配置
** /etc/config**

``` shell
[root@localhost mnesia]# find / -name "rabbitmq.config.example"
/usr/share/doc/rabbitmq-server-3.3.5/rabbitmq.config.example
[root@localhost mnesia]# cp -avx /usr/share/doc/rabbitmq-server-3.3.5/rabbitmq.config.example /etc/rabbitmq/rabbitmq.config
`/usr/share/doc/rabbitmq-server-3.3.5/rabbitmq.config.example' -> `/etc/rabbitmq/rabbitmq.config'
[root@localhost rabbitmq]# chown rabbitmq:rabbitmq rabbitmq.config 
```

## 9. rabbitmq 非root启动问题

1. Error description: cannot_log_to_file,"/var/log/rabbitmq/rabbit@localhost.log"
    ``` shell
    chown -R rabbitmq:rabbitmq /var/log/rabbitmq
    chown -R rabbitmq:rabbitmq /var/log/rabbitmq/*
    ```
    
2. /usr/sbin/rabbitmq-server: line 84: /var/lib/rabbitmq/mnesia/rabbit@localhost.pid: Permission denied
    ``` shell
    chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/
    chown -R rabbitmq:rabbitmq /var/lib/rabbitmq/mnesia/*
    ```
    
3.   {cannot_delete_plugins_expand_dir, ["/var/lib/rabbitmq/mnesia/rabbit@localhost-plugins-expand" {cannot_delete, "/var/lib/rabbitmq/mnesia/rabbit@localhost-plugins-expand",eacces}]}}
    ``` shell
    chown  rabbitmq:rabbitmq  /var/lib/rabbitmq
    chown -R rabbitmq:rabbitmq  /var/lib/rabbitmq/
    ```
    
4. [rabbitmq@localhost ~]$ rabbitmq-plugins enable rabbitmq_management Error: {cannot_write_enabled_plugins_file,"/etc/rabbitmq/enabled_plugins",eacces}
    ``` shell
    chown  rabbitmq:rabbitmq /etc/rabbitmq
    chown  rabbitmq:rabbitmq /etc/rabbitmq/enabled_plugins
    ```
   
     ​                      

