#linux 内存压力测试工具 memtester
## 1. 安装
``` shell
cd ~
mkdir stress
mkdir memtester
wget http://pyropus.ca/software/memtester/old-versions/memtester-4.3.0.tar.gz
tar -zxvf memtester-4.3.0.tar.gz
cd memtester-4.3.0
make&&make install
```
## 2. 使用
**申请10GB 内存做1次测试**
```shell
memtester 10GB 1
```
**内存压力测试**

主要想对内存进行压力测试，以上只是试用，可以申请大内存，放入后台无限测试

```shell
nohup memtester 2G > /tmp/memtest.log &
```
## 3. 查看内存使用率
**系统自带的 free 配合 watch 命令**
```shell
watch -n 2 -d free -m # 两秒钟刷新一次 内存以MB的形式显示
```