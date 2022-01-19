## Bind With Dnstap

!!! 为什么要使用dnstap
    原因很简单，dns bind的配置很灵活，同样的dns bind的日志也很灵活，但是我把所有的日志选项开启了以后有一个内容我发现我拿不到，那就是dns解析的response结果我拿不到。就好比说我请求www.baidu.com这个地址，我日志中是看不到响应的dns解析的。但是在dnsmasq的日志中其实就是可以看到的。这个就是dnsmasq本身自身的一个特性，但是bind并不支持。这个功能在后续对接安全的日志审计中还是比较重要的。所以说为了该功能，我分别调研了`dnscap`以及`dnstap`，最后选择`dnstap`实现该方案。不管哪种方案都是使用抓包来实现的，只不过上述方案根据用户需求做了一系列针对dns抓包的优化。

> dnstap是一种高效，灵活的抓取dns流量和记录dns日志的解决方案。它支持包含bind在内的多种dns实现方案。具体可以参考[官方网站](http://dnstap.info)

默认如果使用yum安装bind dns的话，dnstap默认是不会被编译进去的。因此需要自己进行重新编译。注意支持dnstap的版本为bind9.11往后的版本。我们这里选用9.16版本。首先安装一些会用到的依赖的lib库

```shell
yum -y install automake autoconf libtool libtool-ltdl-devel libevent libevent-devel
```

dnstap依赖三个包，分别为：

- https://github.com/google/protobuf
- https://github.com/protobuf-c/protobuf-c
- https://github.com/farsightsec/fstrm

在安装带有dnstap的bind之前，我们需要先安装上对应的依赖环境

```shell
git clone https://github.com/google/protobuf
git clone https://github.com/protobuf-c/protobuf-c
git clone https://github.com/farsightsec/fstrm 
```
首先clone一下对应的仓库，然后进入每一个目录中，执行下面的命令进行编译安装。

```shell
autoreconf -i
configure
make; make install 
```
在编译protobuf-c的时候，你可能会遇到如下的报错

```shell
checking for protobuf... no
checking for protobuf... no
configure: error: Package requirements (protobuf >= 2.6.0) were not met:

No package 'protobuf' found

Consider adjusting the PKG_CONFIG_PATH environment variable if you
installed software in a non-standard prefix.

Alternatively, you may set the environment variables protobuf_CFLAGS
and protobuf_LIBS to avoid the need to call pkg-config.
See the pkg-config man page for more details.
```
但是其实我们安装的版本是够得，报这个错的原因只是因为预编译的时候它没找对位置。因此在预编译的时候我们指定一下位置就可以了。
```shell
./configure PKG_CONFIG_PATH=/usr/local/lib/pkgconfig
```
然后再`make && make install`就可以了。

## 准备PKCS11

> [Bind Pkcs11](http://ftp.iij.ad.jp/pub/network/isc/bind9/cur/9.16/doc/arm/html/advanced.html#pkcs-11-cryptoki-support)

使用native pkcs11，需要在编译的时候，添加上编译参数
```shell
./configure --enable-native-pkcs11 \
  --with-pkcs11=provider-library-path
```
pkcs11依赖hsm，这里我们需要安装[SoftHSM](https://github.com/opendnssec/SoftHSMv2)
```shell
git clone https://github.com/opendnssec/SoftHSMv2.git
cd SoftHSMv2/
sh autogen.sh
yum -y install openssl openssl-devel
./configure --with-crypto-backend=openssl --prefix=/opt/pkcs11/usr
make && make install

# 初始化token，********代表你输入的pin值
$ /opt/pkcs11/usr/bin/softhsm2-util --init-token 0 --slot 0 --label softhsmv2
=== SO PIN (4-255 characters) ===
Please enter SO PIN: ********
Please reenter SO PIN: ********
=== User PIN (4-255 characters) ===
Please enter user PIN: ********
Please reenter user PIN: ********
The token has been initialized and is reassigned to slot 554785532
```

## 编译安装

准备安装包
```shell
# https://www.isc.org/bind/ 官方下载页
cd /usr/src
wget https://downloads.isc.org/isc/bind9/9.16.21/bind-9.16.21.tar.xz
tar xf bind-9.16.21.tar.xz
cd bind-9.16.21
```
预编译
```shell
# 安装部分必要的依赖
yum install openldap openldap-devel mysql-devel postgresql postgresql-devel python-devel libuv libuv-devel python-ply libcap-devel libdb libdb-devel -y
# 预编译
./configure \
--prefix=/usr \
--exec-prefix=/usr \
--bindir=/usr/bin \
--sbindir=/usr/sbin \
--localstatedir=/var \
--includedir=/usr/include \
--libdir=/usr/lib64 \
--datadir=/usr/share \
--mandir=/usr/share/man \
--sysconfdir=/etc \
--libexecdir=/usr/libexec \
--sharedstatedir=/var/lib \
--infodir=/usr/share/info \
--build=x86_64-redhat-linux-gnu \
--host=x86_64-redhat-linux-gnu \
--with-dlz-ldap=yes \
--with-dlz-postgres=yes \
--with-dlz-mysql=yes \
--with-dlz-filesystem=yes \
--with-dlz-bdb=yes \
--with-python=/usr/bin/python \
--with-libtool \
--with-pic \
--with-lmdb=no \
--with-gssapi=yes \
--enable-dnstap \
--enable-dnsrps \
--with-pkcs11=/opt/pkcs11/usr/lib/softhsm/libsofthsm2.so \
--enable-native-pkcs11 \
CFLAGS='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic' \
LDFLAGS='-Wl,-z,relro' \
CPPFLAGS='-DDIG_SIGCHASE'
make && make install 
```


## 配置dnstap

dnstap的配置需要配置在bind主配置文件`/etc/named.conf`的`option`段落下。一共有四个指定需要配置

- dnstap(required)：必填参数，目的是开启dnstap
- dnstap-output(required)：指定输出的位置，可以是unix socket，也可以是一个file文件。
- dnstap-identity(optional)：value为string类型，主要目的是为抓取到的数据打标签，如果没有指定的话，默认使用hostname
- dnstap-version(optional)：也是可选字段，指定抓取数据的版本号。如果没有指定的话，那么将会使用bind的release version


## 附录1：参考文档


- [Using DNSTAP with BIND](https://kb.isc.org/docs/aa-01342)