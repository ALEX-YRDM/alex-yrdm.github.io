# Redis进阶

## 持久化

- RDB (Redis Database Backup file) 快照

默认保存在当前运行目录 

save命令,阻塞所有命令来执行保存操作

bgsave: 后台异步执行 开启子进程执行 fork

停机时默认触发一次RDB

- AOF

## 主从

## 哨兵

## 分片集群

