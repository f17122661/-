# virtualenv离线安装
## 离线安装virtualenv 
**1. 下载安装包**
```shell
curl -o https://files.pythonhosted.org/packages/49/65/cc3ad660d8b41ce19d34db825d00e022ba72415df1abf8a24984f9cfe50b/virtualenv-16.7.0.tar.gz
```
**2. 将安装包拷贝到离线机器上, 然后使用下列命令进行安装
```shell
tar zxvf virtualenv-16.7.0.tar.gz
cd   virtualenv-16.7.0.tar.gz
su root
python setup.py install
```