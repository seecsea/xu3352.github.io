---
layout: post
title: "Python 3 源码安装及问题解决"
tagline: ""
description: "`python3` 快捷的源码安装步骤, 解决 `openssl` 和 `zipimport.ZipImportError` 问题"
date: '2018-05-15 11:15:24 +0800'
category: python
tags: python
---
> {{ page.description }}

# 安装步骤

```bash
# 0. 依赖包
$ yum -y install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel

# 1. 下载安装包 (如果下载不了, 可以手动传)
$ wget --no-check-certificate https://www.python.org/ftp/python/3.6.5/Python-3.6.5.tgz

# 2. 解压
$ tar -zxvf Python-3.6.5.tgz

# 3. 编译
$ cd Python-3.6.5
$ ./configure --prefix=/usr/local/python3

# 4. 执行安装
$ make && make install

# 5. 创建软链(好像是3.6之后的自带了pip, 因为3.5.1的是没有的)
$ ln -s /usr/local/python3/bin/python3 /usr/bin/python3
$ ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3

# [可选]如果你想默认就使用 python3
# 建议先把 python2 先备份 (如果 yum 不好使, 记得改为 python2 运行)
$ mv /usr/bin/python /usr/bin/python2
$ ln -s /usr/local/python3/bin/python3 /usr/bin/python
```

查看 `python` 版本号:
```bash
$ python3 -V
Python 3.6.5

$ pip3 -V
pip 9.0.1 from /usr/local/python3/lib/python3.6/site-packages (python 3.6)
```

# 安装模块包

```bash
# 安装 SomePackage 模块
$ python -m pip install SomePackage

# 安装指定版本模块
$ python -m pip install SomePackage==1.0.4    # specific version

# 安装最低版本模块
$ python -m pip install "SomePackage>=1.0.4"  # minimum version

# 升级模块
$ python -m pip install --upgrade SomePackage

# 卸载模块
$ python -m pip uninstall SomePackage
```

如果按上面的安装步骤, 那么这里应该是: 

`python3 -m pip install SomePackage` 或 `pip3 install SomePackage`

# 其他方式
如果想要更简单的, 可以试试这个: (可能也不好使)
```bash
$ sudo yum install python34-setuptools
$ sudo easy_install-3.4 pip
```

# 遇到的问题
最开始的时候是安装的 `3.6.1` 版本的, 安装过程每太注意日志, 后面无意间试了一下安装模块包的时候悲剧了, 意思是无法支持 `ssl`, 无法安装模块包 

## ssl 问题
```bash
$ python3 -m pip install flask
pip is configured with locations that require TLS/SSL, however the ssl module in Python is not available.
Collecting flask
  Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)': /simple/flask/
  Could not fetch URL https://pypi.python.org/simple/flask/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.python.org', port=443): Max retries exceeded with url: /simple/flask/ (Caused by SSLError("Can't connect to HTTPS URL because the SSL module is not available.",)) - skipping
  Could not find a version that satisfies the requirement flask (from versions: )
No matching distribution found for flask
```
尝试过设置把域名设置为信任, 没有好使

搜索到的解决办法基本都是先安装 `openssl` 包, 然后重新编译

有的还需要改一下 `./Modules/Setup.dist` 设置

- 检查 `openssl` 
```bash
# 已安装成功 (使用 `yum -y install openssl openssl-devel` 方式安装)
$ yum list | grep openssl
openssl.i686                     0.9.8e-40.el5_11       installed
openssl.x86_64                   0.9.8e-40.el5_11       installed
openssl-devel.i386               0.9.8e-40.el5_11       installed
openssl-devel.x86_64             0.9.8e-40.el5_11       installed
```

- 修改 源码目录下的文件  `./Modules/Setup.dist`
<pre>
# Socket module helper for socket(2)
_socket socketmodule.c

# Socket module helper for SSL support; you must comment out the other
# socket line above, and possibly edit the SSL variable:
SSL=/usr/local/ssl   # 默认路径, 如果手动安装到别的路径, 这里需要改
_ssl _ssl.c \
    -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
    -L$(SSL)/lib -lssl -lcrypto
</pre>


`make` 时还是同样的的错误日志:
```bash
Failed to build these modules:
_decimal    _ssl

...

./Modules/_ssl.c:65: 警告：忽略 #pragma GCC diagnostic
./Modules/_ssl.c: In function ‘SSL_SESSION_has_ticket’:
./Modules/_ssl.c:221: 错误：‘SSL_SESSION’ 没有名为 ‘tlsext_ticklen’ 的成员
./Modules/_ssl.c: In function ‘SSL_SESSION_get_ticket_lifetime_hint’:
./Modules/_ssl.c:227: 错误：‘SSL_SESSION’ 没有名为 ‘tlsext_tick_lifetime_hint’ 的成员
make: *** [Modules/_ssl.o] 错误 1
```

先后尝试过的版本:`3.6.1` `3.6.5` `3.5.1` 这3个版本, 都是上面同样的问题...

同样尝试过增加参数也都不好使: (`--with-ssl` 估计是很老的版本支持过)
```bash
./configure --prefix=/usr/local/python3 --enable-shared --with-ssl
```

## ssl 解决
直到看到了这篇文章:[How to Compile and Install Python with OpenSSL Support?](https://techglimpse.com/install-python-openssl-support-tutorial/), 然后我尝试着手动源码重新安装了 `openssl-1.0.2e` 版本, 最终解决该问题

<span style="color:red">也就是说: yum -y install openssl openssl-devel 安装的 0.9.8e 版本不行!!!</span>

- 手动安装 `openssl-1.0.2e`:
```bash
$ cd /tmp
$ wget http://www.openssl.org/source/openssl-1.0.2e.tar.gz
$ tar xzvf openssl-1.0.2e.tar.gz
$ cd openssl-1.0.2e
$ ./config --prefix=/usr/local/openssl --openssldir=/usr/local/openssl
$ make && make install
```
注意:不适用参数编译时, 默认安装目录为: `/usr/local/ssl`, 这里我们安装到了 `/usr/local/openssl`, 后面也需要对应的修改

<span style="color:red">删除老的源码目录, 重新解压一遍! </span>

`/usr/local/python3` 这个目录我也一起删除掉了

- 修改 `./setup.py`: (默认的openssl路径不改也可以)
```bash
        # Detect SSL support for the socket module (via _ssl)
        search_for_ssl_incs_in = [
                              '/usr/local/openssl/include', # 修改为新目录
                              '/usr/local/openssl/include/openssl',  # 新增
                              '/usr/contrib/ssl/include/'
                             ]
```

- 修改 `./Modules/Setup.dist`:
<pre>
# Socket module helper for socket(2)
_socket socketmodule.c

# Socket module helper for SSL support; you must comment out the other
# socket line above, and possibly edit the SSL variable:
SSL=/usr/local/openssl   # 这里改为我们指定的目录
_ssl _ssl.c \
    -DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
    -L$(SSL)/lib -lssl -lcrypto
</pre>

然后重新编译, 安装 (过程参考最上面的安装步骤)
```bash
$ ./configure --prefix=/usr/local/python3
$ make
$ make install
```

检查 `ssl` 是否安支持, 没报错就是好使了
```bash
$ python3
Python 3.6.5 (default, May 17 2018, 21:24:08)
[GCC 4.1.2 20080704 (Red Hat 4.1.2-55)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import ssl
>>> exit()
```

再安装一个模块包试试: `pip3 install flask` 成功!

## zipimport.ZipImportError 错误
到达这一步, `python3` 好像已经安装好了, 而 `pip3` 安装的时候失败

```bash
zipimport.ZipImportError: can't decompress data; zlib not available
make: *** [install] 错误 1
```

这个主要是最开始的 `zlib-devel` 依赖没解决导致的问题; 安装依赖包的时候可能忽略了一个问题, 那就是提示:  

<span style="color:red">warning: rpmts_HdrFromFdno: Header V3 DSA signature: NOKEY, key ID e8562897</span>

一般警告我们是忽略的, 所以导致了后面的问题, 解决方案请看这里: [yum 使用警告](https://xu3352.github.io/linux/2018/02/01/yum-centos-5-repo-disabled#yum-%E4%BD%BF%E7%94%A8%E8%AD%A6%E5%91%8A)

```bash
# 针对 Centos5, 导入 gpgkey
$ rpm --import http://mirrors.aliyun.com/centos/RPM-GPG-KEY-CentOS-5
```

然后重新安装依赖包后, 查看依赖包是否安装成功:
```bash
[root@iZ23wbfnv7pZ ~]# yum list zlib*
Installed Packages
zlib.x86_64                    1.2.3-7.el5                 installed
zlib-devel.x86_64              1.2.3-7.el5                 installed
```

---
参考：
- [python 官网](https://www.python.org/)
- [Installing Python Modules](https://docs.python.org/3/installing/index.html)
- [Linux安装python3.6](https://www.cnblogs.com/kimyeee/p/7250560.html)
- [CentOS 7 安装 Python3、pip3](https://ehlxr.me/2017/01/07/CentOS-7-%E5%AE%89%E8%A3%85-Python3%E3%80%81pip3/)
- [源码编译安装Python3及问题解决](http://chowyi.com/2017/05/04/%E6%BA%90%E7%A0%81%E7%BC%96%E8%AF%91%E5%AE%89%E8%A3%85Python3%E5%8F%8A%E9%97%AE%E9%A2%98%E8%A7%A3%E5%86%B3/)
- [How to Compile and Install Python with OpenSSL Support?](https://techglimpse.com/install-python-openssl-support-tutorial/)
- [菜鸟教程:Python 3 基础语法](http://www.runoob.com/python3/python3-tutorial.html)

