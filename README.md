## 1. MySQL dump
```
mysql -h -u -p -Dsunbox -e 'source /a/b/c.sql' > ./log/mysqldump.log &
```
数据库databasefoo的表table1,table2以sql形式导入foo.sql中,
```
mysqldump -uroot -pmysql databasefoo table1 table2 > foo.sql
```
