## Centos7 python3.10 编译安装步骤

### 1、准备安装环境
 > yum -y install wget gcc zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel readline-devel tk-devel gdbm-devel db4-devel libpcap-devel xz-devel libffi-devel

### 2、升级gcc
###### 第一步: 安装centos-release-scl
> sudo yum install centos-release-scl
###### 第二步: 安装devtoolset 
- 注意事项，如果想安装7.版本的，就改成devtoolset-7-gcc，以此类推
> sudo yum install devtoolset-8-gcc*
###### 第三步: 激活对应的devtoolset
- 所以你可以一次安装多个版本的devtoolset，需要的时候用下面这条命令切换到对应的版本
> scl enable devtoolset-8 bash
###### 第四步: 查看版本
- 大功告成，查看一下gcc版本
> gcc -v
###### 注意事项:gcc如果没有切换只对本次会话有效
- 1.切换gcc版本
 补充: 这条激活命令只对本次会话有效，重启会话后还是会变回原来的4.8.5版本，要想随意切换可按如下操作。
 首先,安装的devtoolset是在 /opt/sh 目录下的,如图
 每个版本的目录下面都有个 enable 文件，如果需要启用某个版本，只需要执行
 >  source ./enable
 所以要想切换到某个版本，只需要执行
 > source /opt/rh/devtoolset-8/enable
 可以将对应版本的切换命令写个shell文件放在配了环境变量的目录下，需要时随时切换，或者开机自启
- 2.直接替换旧的gcc
  旧的gcc是运行的 /usr/bin/gcc，所以将该目录下的gcc/g++替换为刚安装的新版本gcc软连接，免得每次enable
  > mv /usr/bin/gcc /usr/bin/gcc-4.8.5
   ln -s /opt/rh/devtoolset-8/root/bin/gcc /usr/bin/gcc
   mv /usr/bin/g++ /usr/bin/g++-4.8.5
   ln -s /opt/rh/devtoolset-8/root/bin/g++ /usr/bin/g++
   gcc --version
   g++ --version


### 2、升级openssl
###### 1、下载最新的openssl (https://www.openssl.org/source 有很多其他版本，找一个最新的)
> wget https://www.openssl.org/source/openssl-1.1.1c.tar.gz
###### 2、解压并编译安装
> tar -zxvf openssl-1.1.1c.tar.gz
 cd openssl-1.1.1c
./config --prefix=/usr/local/openssl   #如果此步骤报错,需要安装perl以及gcc包
 make && make install
 mv /usr/bin/openssl /usr/bin/openssl.bak
 ln -sf /usr/local/openssl/bin/openssl /usr/bin/openssl
 echo "/usr/local/openssl/lib" >> /etc/ld.so.conf
 ldconfig -v                    # 设置生效
###### 3、查看确认版本
> openssl version

### 3、准备编译python
###### 1、python官网下载最新版本的python https://www.python.org/downloads/
选择最新的 Gzipped source tarball 版本
###### 2、解压安装包
> tar zxvf Python-3.10.2.tgz
###### 3、修改文件，支持SSL模块（非常重要 3.7以后的版本必须修改）
可以有效解决pip安装包时的 ssl 错误
HTTPS URL because the SSL module is not available.
在 Setup 文件内添加以下内容默认是被注释状态
> vim Modules/Setup
SSL=/usr/local/openssl
_ssl _ssl.c \
		-DUSE_SSL -I$(SSL)/include -I$(SSL)/include/openssl \
        -L$(SSL)/lib -lssl -lcrypto


###### 4、编译安装（非常重要，必须制定正确的编译参数）
--with-openssl=/usr/local/openssl : 定位到系统的openssl路径
--enable-optimizations : 启动编译优化
--enable-shared : 表示启用动态库版本,用于兼容一些库。即是说，在大多数 Unix 系统上（除了 Mac OS X 之外），共享库的路径不是绝对路径。 因此，如果我们在非标准位置安装 Python，为了不和相同版本的系统 Python 产生干扰，我们需要配置非标准位置安装的 Python共享库的路径，或者通过设置运行时的环境变量，如 LD_LIBRARY_PATH。 为了避免这个问题，我们最好避免使用 `--enable-shared`。
> ./configure --prefix=/usr/local/python3 --with-openssl=/usr/local/openssl --enable-optimizations --enable-shared
make && make install

使用 "--enable-shared" 参数会导致编译完成后，直接运行python3 报找不到so库的问题通过下面代码解决这个问题

>echo "/usr/local/python3/lib" > /etc/ld.so.conf.d/python3.10.conf
ldconfig


###### 5、配置访问环境
创建软链接
> ln -s /usr/local/python3/bin/python3 /usr/bin/python3
> ln -s /usr/local/python3/bin/pip3 /usr/bin/pip3

此时执行python3命令可能还得报错: error while loading shared libraries



### 3、使用python3 或者 pip3 命令查询安装结果
> python3 -V
> pip3 -V 

###### 笔记参考：
- https://blog.csdn.net/whatday/article/details/98053179
- https://www.qahome.top/posts/centos7%E7%BC%96%E8%AF%91python3/
- https://zhuanlan.zhihu.com/p/50838802
- https://blog.csdn.net/csdn18740599042/article/details/108951385