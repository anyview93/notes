# 1.释放内存

## 1.1.释放网页缓存(To free pagecache):

```shell
sync; echo 1 > /proc/sys/vm/drop_caches 
```

## 1.2.释放目录项和索引(To free dentries and inodes):

```shell
sync; echo 2 > /proc/sys/vm/drop_caches 
```

## 1.3.释放网页缓存，目录项和索引（To free pagecache, dentries and inodes）

```shell
sync; echo 3 > /proc/sys/vm/drop_caches
```

# 2. 创建软连接（快捷方式）

ln -s 源文件 目标文件

```shell
ln -s /data/thunisoft/mongodb/mongodb-linux-x86_64-3.0.6/bin/mongod /usr/local/bin/
```

