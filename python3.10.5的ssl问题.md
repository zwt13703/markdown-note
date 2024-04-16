# Python3.10.5的ssl问题
#### 首先是因为这个问题
```angular2html
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.")) - skipping
```
网上找了一些办法后也没解决

后来终于解决了，以下是解决方案
### 先安装依赖项

```
sudo yum install gcc openssl-devel bzip2-devel libffi-devel zlib-devel readline-devel sqlite-devel wget
```

### 安装 openssl-1.1.1n
目前已知python3.10.5 可以使用openssl-1.1.1n，其他1.1.1、1.1.1w、1.1.1s及更高版本均有ssl问题。
```bash
#下载openssl-1.1.1n 解压
cd /server/ocr
wget https://www.openssl.org/source/openssl-1.1.1n.tar.gz
tar -zxf openssl-1.1.1n.tar.gz
# 编译安装
cd ./openssl-1.1.1n
./config --prefix=/usr/local/openssl
make && make install
### 替换旧版本openSSL
mv /usr/bin/openssl /usr/bin/openssl.old
mv /usr/lib64/openssl /usr/lib64/openssl.old
mv /usr/lib64/libssl.so /usr/lib64/libssl.so.old
ln -s /usr/local/openssl/bin/openssl /usr/bin/openssl
ln -s /usr/local/openssl/include/openssl /usr/include/openssl
ln -s /usr/local/openssl/lib/libssl.so /usr/lib64/libssl.so
# 将 /usr/local/openssl/lib 目录添加到动态链接器的配置文件中，
# 并通过运行 ldconfig 命令使系统重新加载动态链接库缓存，
# 以便系统能够找到并链接到新添加的 OpenSSL 库
echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
ldconfig -v
#查看OpenSSL版本
openssl version
#OpenSSL 1.1.1n 15 Mar 2022
```

### 安装 python3.10.5

1. 下载安装包
```bash
cd /server/ocr
wget https://www.python.org/ftp/python/3.10.5/Python-3.10.5.tgz
```
2. 解压并进入目录
```bash
tar -zxvf Python-3.10.5.tgz
cd Python-3.10.5
```
3. 配置和编译
```bash
# --with-openssl=/usr/local/openssl 是指向openssl，必须需要
./configure --with-openssl=/usr/local/openssl
make -j $(nproc)
```
4. 安装
```bash
sudo make altinstall
```
5. 检查安装
```bash
python3.10 --version
```
6. 验证openssl模块是否关联
```bash
python3.10
# 在python终端输入
import ssl
# 没报错就可以，报错了需要寻找错误输出问题原因.
# 这里如果报错，那后面python安装https的依赖包肯定会出错
# Can't connect to HTTPS URL because the SSL module is not available.
```
### 参考文档：
[掘金-踩坑小结：Linux安装python环境 、安装OpenSSL](https://juejin.cn/post/7091573912428871710)
[CSDN-centos7 python3.12.1 报错 No module named _ssl，坑我几个小时](https://blog.csdn.net/Amio_/article/details/126716818)
[51CTO博客-python3.10带openssl](https://blog.51cto.com/huwei555/6140266)
[CSDN-python3.10及以上版本编译安装ssl模块](https://blog.csdn.net/ye__mo/article/details/129436629)
