# rsyslog v8.39 安装

## 1.centos7 rpm 安装

### 1.1 下载v8-stable源

``` shell
cd /etc/yum.repos.d/
wget http://rpms.adiscon.com/v8-stable/rsyslog.repo
yum install rsyslog
```
这种不是很推荐 系统本身携带了rsyslog 可以通过升级的方式来安装指定版本的rsyslog
进入这个网站下载rsyslog版本依赖和插件**http://rpms.adiscon.com/v8-stable/epel-7/x86_64/RPMS/ **
``` shell
cd ~
mkdir rsyslog 
cd rsyslog 
wget http://rpms.adiscon.com/v8-stable/epel-7/x86_64/RPMS/libee-0.4.1-1.el7.x86_64.rpm
wget http://rpms.adiscon.com/v8-stable/epel-7/x86_64/RPMS/libestr-0.1.10-1.el7.x86_64.rpm
wget http://rpms.adiscon.com/v8-stable/epel-7/x86_64/RPMS/libfastjson4-0.99.8-1.el7.centos.x86_64.rpm
wget http://rpms.adiscon.com/v8-stable/epel-7/x86_64/RPMS/liblognorm1-1.1.3-1.el7.x86_64.rpm
wget http://rpms.adiscon.com/v8-stable/epel-7/x86_64/RPMS/liblognorm5-2.0.6-1.el7.x86_64.rpm
wget http://rpms.adiscon.com/v8-stable/epel-7/x86_64/RPMS/rsyslog-mmnormalize-8.39.0-4.el7.x86_64.rpm
wget http://rpms.adiscon.com/v8-stable/epel-7/x86_64/RPMS/rsyslog-pgsql-8.39.0-4.el7.x86_64.rpm
```
### 1.2 安装
#### 1.2.1 升级 系统自带的rsyslog
查询自带版本
```shell
sudo rsyslogd -ver
rsyslogd 7.4.7, compiled with:
	FEATURE_REGEXP:				Yes
	FEATURE_LARGEFILE:			No
	GSSAPI Kerberos 5 support:		Yes
	FEATURE_DEBUG (debug build, slow code):	No
	32bit Atomic operations supported:	Yes
	64bit Atomic operations supported:	Yes
	Runtime Instrumentation (slow code):	No
	uuid support:				Yes

See http://www.rsyslog.com for more information.
```
**升级**
```shell
rpm -Uvh rsyslog-pgsql-8.39.0-4.el7.x86_64.rpm
```
#### 1.2.2 安装插件
**安装插件**
```shell
rpm -ivh rsyslog-mmnormalize-8.39.0-4.el7.x86_64.rpm #解析插件
rpm -ivh rsyslog-pgsql-8.39.0-4.el7.x86_64.rpm #数据库写入插件
```
#### 1.3 启动
```shell
systemctl enable rsyslog.service
systemctl start rsyslog.service
```
## 2 安装过程中遇到的问题

**遇到问题**

```shell
warning: rsyslog-pgsql-8.39.0-4.el7.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID e00b8985: NOKEY
error: Failed dependencies:
	libpq.so.5()(64bit) is needed by rsyslog-pgsql-8.39.0-4.el7.x86_64
	rsyslog = 8.39.0-4.el7 is needed by rsyslog-pgsql-8.39.0-4.el7.x86_64
```
** 没有安装libpq.so.5**
安装 liblognorm5-2.0.6-1.el7.x86_64.rpm 但是这个依赖 libfastjson4-0.99.8-1.el7.centos.x86_64.rpm
解决过程
```shell
 rpm -ivh liblognorm5-2.0.6-1.el7.x86_64.rpm
 rpm -ivh liblognorm5-2.0.6-1.el7.x86_64.rpm
```
**然后再进行升级**
遇到这个问题的原因是没有安装postgres 要提前安装postgres 来解决这个问题 看pgsql 安装文档
```shell
rpm -Uvh rsyslog-pgsql-8.39.0-4.el7.x86_64.rpm
warning: rsyslog-pgsql-8.39.0-4.el7.x86_64.rpm: Header V3 RSA/SHA1 Signature, key ID e00b8985: NOKEY
error: Failed dependencies:
	libpq.so.5()(64bit) is needed by rsyslog-pgsql-8.39.0-4.el7.x86_64
	rsyslog = 8.39.0-4.el7 is needed by rsyslog-pgsql-8.39.0-4.el7.x86_64
```








