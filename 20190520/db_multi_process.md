# 1. Context
## 1.1 上次扩容情况   
 2018-05-17 晚上扩容数据库，解决了磁盘不够用的问题，  5 个 1TB 的盘，目前只使用了一个。
由于 161 机器配置较高，为了进一步利用机器资源，同时提升 MySQL IO 性能， 准备进行如下修改
 - 启动多个 `mysqld` 进程
 - 每个 `mysqld` 进程只读 1 个数据库，共启动 5 个 `mysqld` 进程

## 1.2 现状  
```
[root@yhb-db1 sunbox]# df -h
Filesystem      Size  Used Avail Use% Mounted on
/dev/xvda3      170G  155G  5.6G  97% /
tmpfs            32G   52K   32G   1% /dev/shm
/dev/xvda1      194M   40M  144M  22% /boot
/dev/xvde1      493G  272G  196G  59% /data
/dev/xvdh1     1008G  312G  646G  33% /game
/dev/xvdf1     1008G  116G  841G  13% /account
/dev/xvdg1     1008G  200M  957G   1% /api_order
/dev/xvdi1     1008G  200M  957G   1% /market
/dev/xvdj1     1008G  200M  957G   1% /sun
[root@yhb-db1 sunbox]# ps -ef | grep mysqld --color
root        537    506  0 15:43 pts/0    00:00:00 grep mysqld --color
root      15064   3636  0 May17 ?        00:00:00 /bin/sh /usr/bin/mysqld_safe --defaults-file=/game/my.cnf --datadir=/game/mysql
mysql     15337  15064 71 May17 ?        1-22:09:58 /usr/sbin/mysqld --defaults-file=/game/my.cnf --basedir=/usr --datadir=/game/mysql --plugin-dir=/usr/lib64/mysql/plugin --user=mysql --log-error=/var/log/mysqldgame.log --pid-file=/var/run/mysqld/mysqldgame.pid --socket=/var/lib/mysql/mysql.sock --port=3306
root      31342   4520  0 May17 ?        00:00:00 /bin/sh /usr/bin/mysqld_safe --defaults-file=/etc/my.cnf --datadir=/var/lib/mysql
mysql     31593  31342  0 May17 ?        00:12:42 /usr/sbin/mysqld --defaults-file=/etc/my.cnf --basedir=/usr --datadir=/var/lib/mysql --plugin-dir=/usr/lib64/mysql/plugin --user=mysql --log-error=/var/log/mysqld.log --pid-file=/var/run/mysqld/mysqld.pid --socket=/var/lib/mysql/mysql.sock --port=3307
[root@yhb-db1 sunbox]#
```
# 2. MySQL 多实例实施  
实施计划如下:

| No | item | note |
| -- | -- | -- |
| 1 | 时间 | 2019-05-24 |
| 2 | 是否需要停机 | 是 |
| 3 | 预计停机时长 | 7h |
## 2.1 停服务
```
stop all service          ##  停止所所有服务
/etc/init.d/mysqld stop   ##  停止数据库服务
```
## 2.2 拷贝文件
game            
```
cd /game                      ## 1
tar -cf mysql.tar mysql       ## 2 预计需要 1.5h
```
account          
```
cp /game/mysql.tar /account   ## 3 预计需要 1.5h
cd /account
tar -xf mysql.tar             ## 4 预计需要 1h
```
api_order     
```
cp /game/mysql.tar /api_order ## 5 与 4 同时执行， 预计需要 1.5h
cd /api_order
tar -xf mysql.tar             ## 6 预计需要 1h
```
market        
```

cp /account/mysql.tar /market    ## 7 在 4 完成后执行，预计需要 1.5h
cd /market
tar -xf mysql.tar               ## 8 预计需要 1h

```
sunbox
```
cp /api_order/mysql.tar /sunbox  ## 9 在 6 完成后执行，预计需要 1.5h
cd /sunbox
tar -xf mysql.tar                ## 10 预计需要 1h
```
## 2.3 启动多个 mysqld 进程
启动 game
```
/usr/bin/mysqld_safe --defaults-file=/game/my.cnf --datadir=/game/mysql
```
启动 account
```
vim /accout/my.cnf
%s/port=3306/port=3307/ig
/usr/bin/mysqld_safe --defaults-file=/account/my.cnf --datadir=/account/mysql
```
启动 api_order
```
vim /api_order/my.cnf
%s/port=3306/port=3308/ig
/usr/bin/mysqld_safe --defaults-file=/api_order/my.cnf --datadir=/api_order/mysql
```
启动 market
```
vim /market/my.cnf
%s/port=3306/port=3309/ig
/usr/bin/mysqld_safe --defaults-file=/market/my.cnf --datadir=/market/mysql
```
启动 sunbox
```
vim /sun/my.cnf
%/port=3306/port=3310/ig
/usr/bin/mysqld_safe --defaults-file=/sun/my.cnf --datadir=/sun/mysql
```
检查端口监听，检查 jdbc 连接
```
netstat -anpl | grep 3306
netstat -anpl | grep 3307
netstat -anpl | grep 3308
netstat -anpl | grep 3309
netstat -anpl | grep 3310
mysql -h 10.179.201.161 -u root --port=3310  -p*****
```
## 2.4 修改 service 配置
```
cd /home/sunbox/client/conf
vim application-db.properties   # 配置多个数据库 jdbc 数据源
```
# 3. 风险及应对措施
- game 库保留全量库的数据，暂不删除;
- 等所有的 5 个端口的 mysqld 的监听正常，且服务正常后， 再开始删除 acount api_order mysqld 实例中的其他数据库;
- 若其他实例启动失败，则恢复默认配置;

# 4. 新增网络策略配置
## 4.1 DMZ 区机器可自由互访
以下机器
```
172.16.30.50
172.16.30.51
172.16.30.52
172.16.30.53
172.16.30.55
```
可访问 `172.16.30.57` 的 3306, 3307, 3308, 3309, 3310 端口
## 4.2 DMZ -> 核心区新开端口策略  
需要开通 `DMZ`区的机器 57 到 161 MySQL 实例的以下端口的入访权限

| source | target | target port |
| -- | -- | -- |
| 172.16.30.57 | 10.179.201.161 | 3306, 3307, 3308, 3309, 3310 |
`DMZ` 区的其他机器访问 57, 由 57 作为代理来访问核心区的服务
