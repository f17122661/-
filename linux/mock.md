# mock 
## mock 安装
``` shell
yum install mock
```
## mock 配置
``` shell
#建立 builder 用户
useradd  builder -d /home/builder
#mock组 添加 builder 用户
usermod -a -G mock builder
#添加编译目录
su builder
cd /home/builder
mkdir -p ./rpmbuild/{SOURCES,SPECS}
```
## mock yum镜像源 配置

**/etc/mock/*.cfg
``` conf
config_opts['root'] = 'centos-7-x86_64'
config_opts['target_arch'] = 'x86_64'
config_opts['legal_host_arches'] = ('x86_64',)
config_opts['chroot_setup_cmd'] = 'install bash bzip2 coreutils cpio diffutils system-release findutils gawk gcc gcc-c++ grep gzip info make patch redhat-rpm-config rpm-build sed shadow-utils tar unzip util-linux which xz'
config_opts['dist'] = 'el7'  # only useful for --resultdir variable subst
config_opts['macros']['%dist'] = ".el7"
config_opts['%centos_ver'] = "7"
config_opts['macros']['%centos_ver'] = "7"
config_opts['macros']['%rhel'] = "7"
config_opts['macros']['%el7'] = "1"
config_opts['macros']['%redhat'] = "7"
config_opts['macros']['%_vendor'] = "redhat"
config_opts['macros']['%_vendor_host'] = "redhat"
config_opts['macros']['%_host'] = "x86_64-redhat-linux-gnu"

# no ccache in base repo
config_opts['plugin_conf']['ccache_enable'] = False
config_opts['plugin_conf']['yum_cache_enable'] = False

config_opts['yum.conf'] = """
[main]
cachedir=/var/cache/yum
keepcache=1
debuglevel=2
reposdir=/dev/null
logfile=/var/log/yum.log
retries=20
obsoletes=1
gpgcheck=0
assumeyes=1
syslog_ident=mock
syslog_device=

# repos

[base]
name=centos 7 x86_64 - base
baseurl=http://mirrors.aliyun.com/centos/7/os/x86_64/
enabled=1
gpgcheck=0
cost=2000
includepkgs=*.x86_64 *.noarch glibc.i686 glibc-devel.i686 nss-softokn-freebl*.i686

[updates]
name=centos 7 x86_64 - updates
baseurl=http://mirrors.aliyun.com/centos/7/updates/x86_64/
enabled=1
gpgcheck=0
cost=2000
includepkgs=*.x86_64 *.noarch glibc.i686 glibc-devel.i686 nss-softokn-freebl*.i686

#[extras]
#name=centos 7 x86_64 - extras 
#baseurl=http://mirrors.aliyun.com/centos/7/extras/x86_64/
#enabled=0
#gpgcheck=0
#cost=2500
#includepkgs=*.x86_64 *.noarch glibc.i686 glibc-devel.i686 #nss-softokn-freebl*.i686

# collectd
#- enable the EPEL repository (http://dl.fedoraproject.org/pub/epel/) in the
#  configuration files for your target systems (/etc/mock/*.cfg).

[collectd]
name=collectd 5.9.0 x86_64 -base
baseurl=https://dl.fedoraproject.org/pub/epel/7/x86_64/
enabled=1
cost=2000
"""
```
## collectd 源码下载
**5.9.0**
```shell
cd /rpmbuild/SOURCES
wget https://storage.googleapis.com/collectd-tarballs/collectd-5.9.0.tar.bz2
```
## collectd collectd.spec下载
**5.9.0**
```shell
cd /home/builder
mkdir -p /home/builder/collectd/.git/
cd /home/builder/collectd/.git/
git clone https://github.com/collectd/collectd.git

cp contrib/redhat/collectd.spec /home/builder/rpmbuild/SPECS/
```

## build the SRPM
```shell
su builder
mock -r centos-7-x86_64 --buildsrpm --spec ~/rpmbuild/SPECS/collectd.spec \
    --sources ~/rpmbuild/SOURCES/
``` 
## build the RPMs
```shell
mock -r centos-7-x86_64  --no-clean --rebuild \
/var/lib/mock/centos-7-x86_64/result/collectd-5.9.0-1.el7.src.rpm

```