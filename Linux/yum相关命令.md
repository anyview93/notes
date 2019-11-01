# 1.清理yum缓存

## 1.1. 清理/var/cache/yum的headers

```shell
yum clean headers 
```

## 1.2. 清理/var/cache/yum下的软件包

```shell
yum clean packages
```

## 1.3.清理yum元数据

```shell
yum clean metadata
```

# 2.yum下载软件包到本地 

```shell
yum install --downloadonly --downloaddir=/home/java java
```

# 3.释放内存

## 1.释放网页缓存(To free pagecache):

```shell
sync; echo 1 > /proc/sys/vm/drop_caches 
```

## 2.释放目录项和索引(To free dentries and inodes):

```shell
sync; echo 2 > /proc/sys/vm/drop_caches 
```

## 3.释放网页缓存，目录项和索引（To free pagecache, dentries and inodes）

```shell
sync; echo 3 > /proc/sys/vm/drop_caches
```

# 4. 创建软连接（快捷方式）
ln -s 源文件 目标文件
```
ln -s /data/thunisoft/mongodb/mongodb-linux-x86_64-3.0.6/bin/mongod /usr/local/bin/
```