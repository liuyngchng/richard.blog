# 1. Context
## 1.1 DB info
| Key | Value |
| -- | -- |
| DB | MySQL |
| IP | 10.179.201.161 |
| 实例 | 1 |
| 数据库数量 | 5 |
## 1.2 Disk space
```shell
[root@yhb-db1 mysql]# du -h --max-depth=1  
31G     ./account                          
636K    ./performance_schema               
75G     ./market                           
687M    ./backup                           
68M     ./game                             
27G     ./api_order                        
15M     ./mysql                            
12G     ./sunbox                           
147G    .                                  
[root@yhb-db1 mysql]#                      
```
- 目前数据库服务器上的磁盘是 6 个月的数据量所占的磁盘空间
- 每个数据库所占磁盘约为 50GB
- 按照 5 年的线性数据量进行预估.
# 2. Disk 扩容方案  
## 2.1 数据库磁盘空间预估  

| No | db | Disk Space | Note |
| -- | -- | -- | -- |
| 1  | account | 500GB | 100GB/Y * 5 |
| 2 | api_order | 500GB | 100GB/Y * 5 |
| 3 | game | 500GB | 100GB/Y * 5 |
| 4 |  market | 500GB | 100GB/Y * 5 |
| 5 | sunbox | 500GB | 100GB/Y * 5 |
| 6 | 总计 | 5 TB | 5 块 500GB 的磁盘, 另需 2.5TB 用于数据备份 |
## 2.2 磁盘种类
- 推荐 SSD 盘，读取速度快，不易损坏
- 若使用普通磁盘，推荐做 RAID
## 2.3 磁盘块数
采用不同数据库挂载不同磁盘的策略，共需要 5 块最终可用容量为 500GB 的磁盘
# 3. 扩容步骤
## 3.1 停 Service
```
ps -ef | grep java  -- 查看进程
./stop.sh           -- 逐个停服务
ps -ef | grep java  -- 检查是否全部停止
```
## 3.2 停 DB
```
ps -ef | grep mysqld        -- get process
systemctl stop mysql        -- stop process
ps -ef | grep mysqld        -- check process
```
## 3.3 Mount disk
```
mount disk1 /data/mysql/acount
mount disk2 /data/mysql/game
mount disk3 /data/mysql/api_order
mount disk4 /data/mysql/market
mount disk5 /data/mysql/sunbox
```
## 3.4 Check space
```
cd /data/mysql/*
du -h --max-depth =1
```
## 3.5 Copy data
```
cd /var/lib
sudo tar -cf /data/mysql1.tar ./mysql  # 162上测试，./mysql 为 100GB，打包耗时 1h， /data 挂载的硬盘跟 ./mysql 挂载盘不同，机器配置与162相同
cd /data
tar -xf mysql.tar
```
## 3.6 Start MySQL
设置数据目录
```
vi /et/my.conf
datadir=/data/mysql
```
关闭 bin-log
```
# log-bin=mysql-bin
```
启动服务
```
mysqld_safe --datadir=/data/mysql
mysql -uroot -p
```
## 3.7 Start Service
```
cd ../service/
./start.sh
```
# 4. DB backup  

| No| DB | disk | backup disk |  
| -- | -- | -- | -- |  
| 1 | account | /data/mysql/account | /data/mysql/api_order |
| 2 | account | /data/mysql/api_order | /data/mysql/api_order |
| 3 | account | /data/mysql/game | /data/mysql/market |
| 4 | account | /data/mysql/market | /data/mysql/sunbox |
| 5 | account | /data/mysql/sunbox | /data/mysql/account |
