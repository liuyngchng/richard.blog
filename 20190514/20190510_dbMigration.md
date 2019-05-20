# 1. Context
## 1.1 DB info

| Key | Value |
| --- | --- |
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

| No | db | Disk Space | Note | disk |
| --- | --- | --- | --- | --- |
| 1  | account | 500GB | 100GB/Y * 5 | f |
| 2 | api_order | 500GB | 100GB/Y * 5 | g |
| 3 | game | 500GB | 100GB/Y * 5 | h |
| 4 |  market | 500GB | 100GB/Y * 5 | i |
| 5 | sunbox | 500GB | 100GB/Y * 5 | j |
| 6 | 总计 | 5 TB | 5 块 500GB 的磁盘, 另需 2.5TB 用于数据备份 |  * |

## 2.2 磁盘种类
- 推荐 SSD 盘，读取速度快，不易损坏
- 若使用普通磁盘，推荐做 RAID

## 2.3 磁盘块数
采用不同数据库挂载不同磁盘的策略，共需要 5 块最终可用容量为 500GB 的磁盘
# 3. 预扩容
由于涉及到 5 个库的迁移，先用 1 个数据量较小的库进行迁移，查看服务是否正常，进行验证迁移的可行性。
```
ps -ef | grep java  -- 查看进程
cd /home/sunbox/
./stop-a.sh           -- 逐个停服务
ps -ef | grep java  -- 检查是否全部停止
ps -ef | grep mysqld        -- get process
systemctl stop mysql        -- stop process
ps -ef | grep mysqld        -- check process
cd /var/lib/mysql
mv game game_bck            -- mv dir
mount disk2 /var/lib/mysql/game -- mount disk
cd /var/lib/mysql/
du -h --max-depth =1          -- check disk space
cp -r game_bck game           -- hard copy mysql file
systemctl start MySQL         -- start mysql
grep '源端IP为' app-service.log --color
curl http://172.16.30.51:18807/app/json/app_game/loadGameByCode -d  'gameTypeCode=DZP000000&token=80ADEBF0-F411-49B4-93CD-32767D9F8925'               -- read game database
```
# 4. 扩容步骤
## 4.1 停 Service
```
ps -ef | grep java  -- 查看进程
./stop.sh           -- 逐个停服务
ps -ef | grep java  -- 检查是否全部停止
```
## 4.2 停 DB
```
ps -ef | grep mysqld        -- get process
systemctl stop mysql        -- stop process
ps -ef | grep mysqld        -- check process
```

## 4.3 Mount disk
```
cd /var/lib/mysql
mv account acccout_bck
mv api_order api_order_bck
mv game game_bck
mv market market_bck
mv sunbox sunbox_bck
mount disk1 /var/lib/mysql/acount
mount disk2 /var/lib/mysql/game
mount disk3 /var/lib/mysql/api_order
mount disk4 /var/lib/mysql/market
mount disk5 /var/lib/mysql/sunbox
/etc/init.d/start mysqld
```
## 4.4 Check space
```
cd /var/lib/mysql/
du -h --max-depth =1
```
## 4.5 Copy data
```
cd /var/lib
sudo tar -cf /data/mysql.tar ./mysql  # 162上测试，./mysql 为 100GB，打包耗时 1h， /data 挂载的硬盘跟 ./mysql 挂载盘不同，机器配置与162相同
cd /data
tar -xf mysql.tar
```
## 4.6 Start MySQL
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
mysql -uroot -p    --check service
```
## 4.7 Start Service
```
cd /home/sunbox/
sh ./start-b.sh
```
## 4.8 Regression Test
测试线上业务是否正常  
# 5. DB backup  

| No| DB | disk | backup disk |  
| --- | --- | --- | --- |
| 1 | account | /data/mysql/account | /data/mysql/api_order |
| 2 | api_order | /data/mysql/api_order | /data/mysql/api_order |
| 3 | game | /data/mysql/game | /data/mysql/market |
| 4 | market | /data/mysql/market | /data/mysql/sunbox |
| 5 | sunbox | /data/mysql/sunbox | /data/mysql/account |
# 6. RollBack
## 6.1 Stop Service
```
cd ./service
sh ./stop.sh
```
## 6.2 Start MySQL
```
/etc/init.d/stop mysqld
unmount disk
mv account_bck acccout
mv api_order_bck api_order
mv game_bck game
mv market_bck market
mv sunbox_bck sunbox
/etc/init.d/start mysqld
```
## 6.3 Start Service
```
cd ./service
sh ./stop.sh
```
## 6.4 Regression Test
测试线上业务是否正常  
## 7. service
## 7.1 服务清单  
涉及到的服务器清单如下所示。

| SSH | user | psword | note | env |
| --- | --- | --- | --- | --- |
| ssh sunbox@172.16.30.50 | root |   | nginx、web服务、报表服务 | prod |
| ssh sunbox@172.16.30.51 | root |   | nginx、App服务、中控服务、绑卡定时任务服务 | prod |
| ssh sunbox@172.16.30.52 | root |   | nginx、App服务、中控服务、绑卡定时任务服务 | prod |
| ssh sunbox@172.16.30.53 | root |   | nginx、App服务、中控服务、绑卡定时任务服务 | prod |
| ssh sunbox@172.16.30.55 | root |   | 定时任务task | prod |
## 7.2 启动顺序

```
50 -> 51 -> 52 -> 53 -> 55
```
## 7.3 停止顺序

```
55 -> 53 -> 52 -> 51 -> 50
```

# 7. 验证服务
token kshop 生成，由kshop进行登录操作，操作完调用sunbox 的接口， post ，在post 的参数里或者cookie 里传 token。
由 AppLoginInterceptor  进行解码，验证token 是否正确
```
grep '源端IP为' app-service.log --color
curl http://172.16.30.51:18807/app/json/app_game/loadGameByCode -d  'gameTypeCode=DZP000000&token=80ADEBF'
```
