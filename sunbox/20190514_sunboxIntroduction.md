# 1. 简介

 `sunbox` 框架主要用于处理数据密集型的业务，可以将企业的业务从繁杂的数据库操作中解放出来，
 帮助企业极大地提高生产力。

    `sunbox` 框架是领域驱动设计(DDD:Domain-Driven Design)思想的实践，可快速响应互联网行业快速变化的业务需求和版本迭代，使得软件系统更加灵活，同时能够将企业开发人员从繁重的业务开发中解放出来。
 同类型的框架如 `JdonFramework`

## 1.1 功能模块

`sunbox` 框架主要功能如表 1 所示。

表 1 功能模块  

| No | function | Note |
| -- | -- | -- |
| 1 | `ORM` | 数据库的 `CRUD` |
| 2 | 分库分表 | 简单易用的分库分表策略，支持横向扩展 |
| 3 | 埋点 | 可以轻易地在类的方法中进行业务埋点，使业务代码更加优雅 |
| 4 | 定时任务 | 轻松配置定时任务，进行数据的聚合、`ETL` |
| 5 | `CI` , `CD`| 提供简单易用的持续集成、持续部署 |
| 6 | 服务器资源监控 | 提供对服务器硬盘、`CPU` 等资源的监控 |
| 7 | 服务监控 | 提供对服务的监控，灵活的触发机制配置，如短信报警等 |

## 1.2 主要特点
### 1.2.1 灵活性

框架能够灵活适应企业的各种业务，基本覆盖了目前90% 以上的业务类型，这也是 DDD 设计的优势之一。

### 1.2.2 易用性

学习成本低，上手快，学习曲线平滑。对于部分特殊的业务，框架本身无法满足的，用户可以自己扩展。基本上大多数业务中，用户使用本框架，会像搭乐高积木一样，实现自己的功能和业务场景。
框架集成了整个开发环境下的所有需要用到的环境配置、持续集成、系统监控等功能，可为企业解决大量的人力、运维、软件成本。

### 1.2.3 可伸缩性

框架采用了模块化的设计， 用户可按需选择自己需要的模块，最小化集成项目所需要的部分，这一点类似于 Spring 全家桶中的各个模块的关系。 

### 1.2.4 良好的性能

本框架在企业内部线上，以及部分大型企业线上，均有长期的运行实践， QPS 达到 1000 ～ 10，000时，吞吐量也不会下降，响应时间小于 100ms，系统能够正常稳定运行。

## 1.3 底层
| No | function | Note |
| -- | -- | -- |
| 1 | 数据源封装 | 将 `MySQL`, `SQLServer`, `Oracle`, `SyBase`, `MongoDB`, `SQLServer` 以统一的 `API` 进行封装，提供简单易用的 API 进行调用 |
| 2 | ORM | 提供统一的 ORM API， 隐藏了不同数据源的差异性,简单易用 |
| 3 | 完善的文档 | 提供完善的开发文档，以及最佳实践 Demo |
| 4 |  持续更新升级 | `sunbox` 框架由专业团队维护，跟踪市场变化，满足企业的各种需求 |
| 5 | 技术支持 | 由专业团队提供技术支持 |

# 2. 系统架构
## 2.1 功能模块

![sunboxArch](./sunbox_architech.png)

## 2.2 架构设计  

![sunboxProjArch](./sunbox_proj_arch.png)

## 2.3 项目清单
项目结构如表 2 所示。  
表 2 项目清单  

| No | file/dir | Note |
| -- | -- | -- |
| 1 | pom.xml | pom-parent，配置公共依赖，以及开源项目的版本 |
| 2 | sunbox-agent | 服务监控，如数据库连接数等 |
| 3 | sunbox-api | 领域通用 API |
| 4 | sunbox-base | 框架配置，实现 |
| 5 | sunbox-core | 框架核心实现，`REST API` 领域知识抽象|
| 6| sunbox-import | 自动化部署实现 |
| 7 | sunbox-jmx | agent 实现， 通过 `JMX` 获取 `JVM` 信息 |
| 8 | sunbox-plugins | 第三方服务封装 |
| 9 | sunbox-pom | 是一个父工程，定义了其子工程，可按需打包 |
| 10 | sunbox-sqlmanagement | SQL 语句管理|
| 11 | sunbox-test | 测试 |
| 12 | sunbox-tools | 工具类 |
| 13 | sunbox-web | web 常用的一些静态资源 |

# 3. Best practice
## 3.1 Model  

接口层传值对象，为了方便与 VO 互相转换，对应 table 的 schema

| No | item | model | db |
| :---- | :--- |:--- |:--- |
| 1 | type | Integer | int |
| 2 | type | String | varchar |
| 3 | name | myNme | my_name |

## 3.2 VO  

对应 table 的 schema
Integer <-> int
String <-> varchar
update 方法，可直接将数据刷入 DB
```
VO vo = new VO();
vo.setFieldA(a);
vo.load();
vo.update();
```
对应数据库中的表结构， 例如数据库表结构如下，
```
mysql> desc acct_card_activate;
+------------------+-------------+------+-----+---------+-------+
| Field            | Type        | Null | Key | Default | Extra |
+------------------+-------------+------+-----+---------+-------+
| id               | int(11)     | NO   | PRI | NULL    |       |
| user_id          | varchar(64) | NO   |     | NULL    |       |
| main_card_no     | varchar(50) | NO   |     | NULL    |       |
| card_type        | int(11)     | NO   |     | NULL    |       |
| custom_name      | varchar(50) | YES  |     | NULL    |       |
| custom_phone     | varchar(50) | YES  |     | NULL    |       |
+------------------+-------------+------+-----+---------+-------+
14 rows in set (0.04 sec)
mysql>
```
对应的 Model 如下
```
public class AcctCardActivateModel {
    private Integer id;
    private  String userId;
    private  String mainCardNo;
    private  Integer cardType;
    private  String customName;
    private  String customPhone;
}
```

## 3.3 DBManager  

```
import sunbox.core.conn.DBManager
DBManager db=new DBManager(poolName);
db.sendQuery("SELECT xxxx FROM account.acct_card_activate WHERE xxxx");
List<AcctAccountYoudi> list = db.getObjects(AcctAccountYoudi.class);
```

## 3.4 Scheduled task

处理一些定时任务，实现数据聚合、业务补偿等
```
INSERT INTO `sunbox`.`base_stat_task` (`task_code`, `task_type`, `task_config`, `task_name`, `task_method`, `exec_period`, `status`, `create_time`, `create_by`, `modify_time`) VALUES ('acct.youdi_active_vice_card', '1', 'cron=0 0 2 * * ?', 'xxxxx', 'sunbox.hbsy.action.task.ViceCardActiveTaskAction', '每天凌晨2点执行一次', '0', '2019-05-20 15:40:07', 'webmaster', '2019-05-20 15:40:07');
```

## 3.5 BuriedIn business

埋点业务  

```
class MyClass {
   public void myMethod() {
     doSomething();
     Application.doStaticExecute("market.before.start.order.activity", this);
     doOtherThings();
     Application.doExecute("market.valid.order.activity", this);
     needCleanSomething();
   }
}
```

## 3.6 Partition

实现数据库、表的横向扩展、自动分区  
```
SELECT
    *
FROM
    sunbox.app_conf_detail
WHERE
    name LIKE '%account%';
account.database.tablepatition = 64
```

# 4. mvn package
```
cd {WORKSPACE_DIR}  
git pull xxxxxx  
sudo echo 'SUNBOX_ROOT={WORKSPACE_DIR}' >> /etc/profile  
source /etc/profile  
cd {WORKSPACE_DIR}/sunbox-pom/  
mvn clean compile package -Dsunbox-group-version=1.0
```
这样 所有的包已经安装在自己的本地 `maven` 仓 `～/.m2.repository` 了 ,可以开始开发工作了

# 5. FAQ
## 5.1 ORM 与 开源 ORM 的区别  

sunbox 框架的 ORM 更加轻量级， 比 `MyBatis`， `Hibernate` 配置更简单，而且学习曲线更加平滑，上手更快 散列  

## 5.2 分库分表的键值有那些？

- 年，月，日
- id
- hash 散列

## 5.3 什么是埋点？  

埋点是`sunbox`框架的一个重要特性， 类似与 hook 程序，或者我们经常用到的 `AOP`， 只不过埋点实在 method 的方法体内部执行埋点逻辑。

## 5.4 定时任务的优势在哪里？

- `sunbox` 框架的定时任务配置更加灵活，与`sunbox`框架提供的其他特性可以结合使用;
- 与 linux `crontab` 比较，部署之后无需进行其他配置，提供的是`one-stop`的解决方案
- 与常用的 `Spring` 的定时任务相比较， `Spring` 需要启动`Spring`容器，而 `sunbox`框架定时任务更加轻量;
-

## 5.5 CI 、CD 为什么不使用第三方开源框架？

- 第三方开源框架也会涉及到 `license` 协议的问题，对于一些大客户来讲，可能有潜在法律风险;
- `sunbox` 的 `CI`、`CD` 是 `one-stop`的解决方案，无需额外的环境搭建，节约了人力和时间成本;
- `sunbox` 对第三方`CI`、`CD` 框架的集成是开放的;
- `sunbox`的`CI`、`CD`更加轻量级;

## 5.6 监控为什么不使用第三方开源框架

原因同 `5.5` 节。

## 5.7 ORM Model 是否有工具可自动生成

- 目前尚无此功能
- 后续可以添加
- `sunbox` 的数据类型映射关系相对来讲更简单，数字、字符串 2 种常用的类型，基本上不需要工具，也可以写出来`VO`类。

## 5.8 为什么提供 `DBManager` 这种直接写`SQL`的功能？

- 主要为了考虑`sunbox`框架的扩展性
- 在一些特定的、极其少见的复杂 `case` 下， 简单的映射关系可能无法满足业务需求，这时候就可以借助`DBManager` 直接实现业务功能

## 5.9 数据库连接池是否有`cache`功能？

- 框架采用的是配置信息的2级缓存机制;
- 第1级为`JVM`运行时缓存，`HashMap`;
- 第2级为`Redis`缓存;
- 当 1,2 级缓存均未命中时，读取数据库，同时依靠版本管理机制来刷新1,2级缓存

## 5.10 框架代码是否采取了加密机制？

- 目前尚未对`sunbox`框架代码进行加密
- 后续可以完善这部分的内容;

## 5.11 框架的认证授权机制是怎样的？

- 框架有内置的认证授权机制;
- 企业内部使用`sunbox`框架的多个项目之间可以实现`SSO`;
- 框架对集成第三方的`CA`系统开放;

## 5.12 是否有可视化的运维界面？
- 框架内部提供常规的监控界面;
- 可以实现常规的运营需求;

## 5.13 框架的学习曲线是什么样的？

- `sunbox`框架提供有完善的中文开发文档，学习曲线比较平缓;
- 对于有2～3年开发经验的开发人员，可轻松上手;
