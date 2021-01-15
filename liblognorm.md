# liblognorm rsyslog rb 测试工具
## 安装前准备
**安装前 这个工具有两个依赖需要先安装**
### 安装libestr、libee
```shell
wget http://libestr.adiscon.com/files/download/libestr-0.1.11.tar.gz
tar xvf libestr-0.1.11.tar.gz 
cd libestr-0.1.11
./configure CC="gcc -m64" --prefix=/usr/local/libestr --libdir=/usr/lib64
make && make install
wget http://www.libee.org/files/download/libee-0.4.1.tar.gz
cd ../libee-0.4.1
./configure --prefix=/usr/local/libee CC="gcc -m64" PKG_CONFIG_PATH="/usr/lib64/pkgconfig" --libdir=/usr/lib64
make && make install
```
### 安装libfastjson(CentOS5选择0.99.0版本)
``` shell
wget http://download.rsyslog.com/libfastjson/libfastjson-0.99.8.tar.gz
tar xvf libfastjson-0.99.8.tar.gz 
cd libfastjson-0.99.8
./configure --prefix=/usr/local/libfastjson CC="gcc -m64" PKG_CONFIG_PATH="/usr/lib64/pkgconfig" --libdir=/usr/lib64
make && make install
```
## 安装liblognorm
```shell
wget http://www.liblognorm.com/download/files/download/liblognorm-2.0.5.tar.gz
cd liblognorm-2.0.5
./configure --prefix=/usr/local/liblognorm CC="gcc -m64" PKG_CONFIG_PATH="/usr/lib64/pkgconfig" --libdir=/usr/lib64
make && make install
ln -s /usr/local/liblognorm/bin/lognormalizer /usr/bin/lognormalizer
    
```