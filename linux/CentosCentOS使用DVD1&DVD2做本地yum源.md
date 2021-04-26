# CentosCentOS使用DVD1&DVD2做本地yum源
**CentOS6以上版本一般都会提供一个DVD1和一个DVD2镜像，使用DVD1即可安装使用CentOS了，DVD2中存放了一些额外的软件包，本文介绍如何制作和使用本地yum仓库**

## 1. 合并 CentsOS 6 的两个镜像

**说明**
### 相关目录
/mnt/dvd1和/mnt/dvd2 用于挂载 Centos 镜像

/mnt/dvd3 合并后的镜像文件

/mnt/iso ISO储存

``` shll
mkdir -p /mnt/dvd1 /mnt/dvd2 /mnt/dvd3 /mnt/iso
```
### 上传Centos 镜像大服务器, 挂载Centos镜像文件

``` shell
mount -o loop /mnt/iso/CentOS-6.5-x86_64-bin-DVD1.iso /mnt/dvd1
mount -o loop /mnt/iso/CentOS-6.5-x86_64-bin-DVD2.iso /mnt/dvd2

```

### 拷贝文件
**首先, 拷贝第一张DVD中的所有文件到 /mnt/dvd3 目录下，然后, 只拷贝第二张 DVD 中 Packages 目录下的所有RPM文件到  /mnt/dvd3/Packages 目录下**
``` shell
cp  -av  /mnt/dvd1  /mnt/dvd3 
cp  -v  /mnt/dvd2/Packages/*.rpm  /mnt/dvd3/Packages/
```
### 合并 TRANS.TBL
**将DVD2中TRANS.TBL的信息追加到DVD1中TRANS.TBL后面, 并排序保存**

```shell
cat  /mnt/dvd2/Packages/TRANS.TBL  >>  /mnt/dvd3/Packages/TRANS.TBL 
mv  /mnt/dvd3/Packages/{TRANS.TBL,TRANS.TBL.BAK} 
sort  /mnt/dvd3/Packages/TRANS.TBL.BAK  >  /mnt/dvd3/Packages/TRANS.TBL 
rm  -rf  /mnt/dvd3/Packages/TRANS.TBL.BAK
```
**dvd3已经是合并后的文件了，可以用作本地源和做成ISO使用。**

## 2. 生成新的Yum配置文件
``` shell
vi /etc/yum.repos.d/CentOS-Media.repo
```
``` .repo
[c6-media]
name=CentOS-\$releasever - Media
baseurl=file:///mnt/dvd3
gpgcheck=0
enabled=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-6
```
## 3. 更新YUM源
``` shell
yum clean all
yum upgrade
```