# Redis基础

## 安装

```shell
wget https://github.com/redis/redis/archive/7.2.3.tar.gz

tar -zxvf redis-7.2.3.tar.gz

yum -y install cpp binutils glibc glibc-kernheaders glibc-common glibc-devel gcc make

cd redis-7.2.3

make

make install
```

## 运行

```
redis-server

redis-cli
```

