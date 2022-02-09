## Centos7 自带的 python2.7 安装pip

#### 1、安装 epel-release
> yum -y install epel-release
#### 2、查询源中是否存在python2.7的pip 注意版本号
> yum yum search python-pip
#### 3、执行安装 如果上一步查询到了多个版本这里得指定版本
> yum install -y python-pip
#### 4、检查安装结果
> pip -V
