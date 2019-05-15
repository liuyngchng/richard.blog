# 1. 综述
 `sunbox` 框架用于处理数据密集型的业务，可以帮助企业极大地提高生产力,主要提供以下几点功能。  
表 1 框架功能

| No | function | Note |
| -- | -- | -- |
| 1 | ORM | 数据库的 `CRUD`
| 2 | 分库分表 | 简单易用的分表策略，支持横向扩展 |
| 3 | 埋点 | 可以轻易地在方法中进行业务埋点 |
| 4 | 定时任务 | 轻松配置定时任务，进行数据的聚合 |
# 2. 项目结构
项目结构如表 2 所示。  
表 2 项目结构  

| No | file/dir | Note |
| -- | -- | -- |
| 1 | pom.xml | pom-parent，配置公共依赖，以及开源项目的版本 |
| 2 | sunbox-agent | 数据库 |
| 3 | sunbox-api | API, 接口定义 |
| 4 | sunbox-base | 文件上传，以及 接口返回数据类型配置的抽象（xml，json） |
| 5 | sunbox-batch-recharge | 批量充值? psw_en， native 实现了一个报文摘要获取的方法？ |
| 6 | sunbox-cloud | |
| 7 | sunbox-core | REST API 框架抽象，埋点代码块逻辑判断实现，各种结构化数据和非结构化数据库连接池管理，DAO 层抽象，分区表抽象，线程池的封装， XSS 拦截，常用的拦截器的实现，SQL 语句执行抽象，|
| 8| sunbox-import | `CFCA` 认证？ |
| 9 | sunbox-jmx | agent 实现？ 监控系统资源？ |
| 10 | sunbox-plugins | 第三方服务封装，如 `com.aliyun.oss`， `token` 认证，充电桩业务，人脸识别，OCR ，netty 封装，支付回调， 阿里，微信api 的封装 |
| 11 | sunbox-pom | 是一个父工程，定义了其子工程，可按需打包 |
| 12 | sunbox-sqlmanagement | SQL 语句管理，如根据datasource 关键字定位数据源|
| 13 | sunbox-test | 几种注解的封装，用于 REST API |
| 14 | sunbox-tools | 与 kshop 框架交互工具类 |
| 15 | sunbox-web | web 常用的一些静态资源， js,css, 字体 |
| 16 | sunbox-web-parent | 在线商城使用的一些静态资源 |
# 3. Best practice
## 3.1 Model
对应数据库中的表结构
## 3.2 VO
## 3.3 DBManager
## 3.4 Scheduled task
## 3.5 埋点
## 3.6 分库分表
# 4. mvn package
```
cd {WORKSPACE_DIR}  
git pull xxxxxx  
sudo echo 'SUNBOX_ROOT={WORKSPACE_DIR}' >> /etc/profile  
source /etc/profile  
cd {WORKSPACE_DIR}/sunbox-pom/  
mvn clean compile package -Dsunbox-group-version=1.0
```
这样 所有的包已经安装在自己的本地 `maven` 仓 `～/.m2.repository` le ,可以开始开发工作了
