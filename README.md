# 数据库设计规范

本规范整理自网络，适用于 MySQL 数据库系统，TiDB 可作部分参考。

## 修订记录

| 版本号   | 发布人     | 更新日期       | 备注  |
| ----- | ------- | ---------- | --- |
| 0.0.1 | zhiweio | 2021-08-06 | 初稿  |
| 0.0.2 | zhiweio | 2022-11-04 | 修订  |

## 目录

- [数据库设计规范](#数据库设计规范)
  - [修订记录](#修订记录)
  - [目录](#目录)
  - [数据库命名](#数据库命名)
    - [概述](#概述)
    - [基本命名规范](#基本命名规范)
      - [语言](#语言)
      - [大小写](#大小写)
      - [单词分隔](#单词分隔)
      - [保留字](#保留字)
      - [命名长度](#命名长度)
      - [字段名称](#字段名称)
    - [数据库对象命名规范](#数据库对象命名规范)
      - [表](#表)
      - [字段](#字段)
      - [主键](#主键)
      - [唯一建](#唯一建)
      - [索引](#索引)
  - [数据库结构设计](#数据库结构设计)
    - [主键约束](#主键约束)
    - [NULL 值](#null-值)
    - [字符集](#字符集)
    - [数据类型](#数据类型)
      - [字符型](#字符型)
      - [数值型](#数值型)
      - [枚举型](#枚举型)
      - [日期和时间](#日期和时间)
      - [大字段](#大字段)
      - [类型参考](#类型参考)
    - [数据规则](#数据规则)
      - [完整性规则](#完整性规则)
      - [一致性规则](#一致性规则)
      - [准确性规则](#准确性规则)
    - [常用预定义字段参考](#常用预定义字段参考)
      - [固定字段](#固定字段)
      - [常用字段](#常用字段)
    - [索引](#索引-1)
      - [索引创建](#索引创建)
      - [索引使用](#索引使用)
  - [SQL 规范](#sql-规范)
  - [行为规范](#行为规范)
  - [FAQ](#faq)
  - [引用](#引用)

## 数据库命名

### 概述

使用统一的数据库命名规范，使数据库命名风格标准化，以便于阅读、理解和继承。本规范主要通过基本命名规范和数据库对象的命名规范两方面来阐述。 

### 基本命名规范

#### 语言

* 命名宜使用拼音简写和英文，不宜使用中文、数字或者特殊字符；

* 当出现对象名重名且不同类型对象时，宜加类型前缀或后缀以示区别； 

* 须见名知意，使用名词不是动词。

#### 大小写

使用小写。

#### 单词分隔

命名的各单词之间可以使用下划线 `_` 进行分隔。 

#### 保留字

命名禁止使用 SQL 保留字。 

#### 命名长度

表名、字段名、视图名长度应限制在 32 个字符内。

#### 字段名称

* 同一个字段名在一个数据库中只能代表一个意思；
  
  > 比如 telephone 在一个表中代表“电话号码”的意 思，在另外一个表中就不能代表“手机号码”的意思。

* 不同的表用于相同内容的字段应该采用同样的名称，字段类型定义。

### 数据库对象命名规范

#### 表

* 表名使用` t_` 作为前缀，如果是数仓建模，改用 `ods_`、`dwd_`、`dws_`、`dim_`、`ads_` 等前缀开头；

* 临时表名称必须以 `tmp_` 为前缀，并以日期为后缀，如 `tmp_orders_20221104`；

* 有关联的表名需统一名称前缀，如 `user` 表和 `user_login` 表；

* 建表必须有注释。

#### 字段

* 禁止使用未约定成俗，难以理解的缩写；

* 避免使用与表名相同的字段名，这会在编写查询时造成混淆；

* 字段必须有 comment。

#### 主键

前缀为 `pk_<主键标识>`。

#### 唯一建

前缀为 `uk_<构成的字段名>`。

#### 索引

前缀为 `idx_<构成的字段名>`，如果复合索引的构成字段较多，则只包含第一个字段，并添加序号。

## 数据库结构设计

### 主键约束

* 表必须有主键（PRIMARY KEY）；

* 物理主键：命名为 `id`，类型为 `INT` 或 `BIGINT`，且为 `auto_increment`（在 TiDB 中使用 `auto_random`），且主键值禁止被更新；

* 业务主键：用业务字段约定表的唯一性，确定其增删改逻辑，建表之初必须确认业务主键，建立<u>唯一索引</u>；

* 禁止使用外键；

* 关联表的父表应有业务主健，业务主健字段或组合字段应满足非空属性和唯一性要求。

### NULL 值

* 对于字段能否 NULL，应该在 SQL 建表脚本中明确指明，不应使用缺省；

* 原则上字段必须都是 `NOT NULL` 属性，业务可以根据需要定义 `DEFAULT` 值
  
  > 使用 `NULL` 值会存在每一行都会占用额外存储空间、数据迁移容易出错、聚合函数计算结果偏差等问题。

### 字符集

* 数据库本身库、表、列所有字符集必须保持一致，为 `utf8` 或 `utf8mb4`；

* MySQL 及兼容 MySQL 的数据库，采用 `utf8mb4` 字符集。

### 数据类型

#### 字符型

* 固定长度的字串类型采用 `CHAR`，长度不固定的字串类型采用 `VARCHAR`，避免在长度不固定的情况下采用 `CHAR`类型；

* 存储文本时，少用 `TEXT/BLOB`，`VARCHAR` 的性能会比 `TEXT` 好很多。

#### 数值型

* 在存储数字数据时，应该充分考虑数据的长度选择合适的类型进行存储，同时为数据的扩展保留一定的空间。；
* 在 SQL 中使用 `SMALLINT` 类型需要注意数据项的范围。例如，想看月销售总额，总额字段类型是 `SMALLINT` ，如果总额超过 32767，不宜进行计算操作；
* 使用 `TINYINT` 用于 status、 type、 flag 等含义的字段；
* 使用 `DECIMAL` 代替 `FLOAT/DOUBLE` 存储精确浮点数，对于有小数位的，使用 `DECIMAL`，如 `DECIMAL(9,6)`，最多精确到 6 位小数；
* 整型定义中不添加长度，比如使用 `INT`，而不是 `INT(4)`；

#### 枚举型

禁止使用使用 `ENUM`、`SET` 类型，枚举值字段使用 `TINYINT` 类型存储，建议设置默认值。

#### 日期和时间

* 系统时间：由数据库产生的系统时间首选数据库的日期型，如 `DATE`、`TIMESTAMP` 类型；

* 外部时间：由数据导入或外部应用程序产生的日期时间类型采用 `VARCHAR` 类型，时间戳用 `BIGINT`；

* 时间类型尽量选取 `TIMESTAMP`；
  
  > 因为 `DATETIME` 占用 8 字节，`TIMESTAMP` 仅占用 4 字节，但是范围为 `1970-01-01 00:00:01` 到 `2038-01-01 00:00:00`。

* 考虑选用 `INT`/`BIGINT` 来存储时间，使用 SQL 函数 `unix_timestamp()` 和 `from_unixtime()` 来进行转换。

#### 大字段

如无特别需要，避免使用大字段（`BLOB`，`CLOB`，`LONG`，`TEXT`，`IAMGE` 等）。

> * 这些数据类型比较浪费硬盘和内存空间。在加载表数据时，会读取大字段到内存里从而浪费内存空间，影响系统性能。
> 
> * `InnoDB` 中当一行记录超过 8098 字节时，会将该记录中选取最长的一个字段将其 768 字节放在原始 `page` 里，该字段余下内容放在 `overflow-page` 里，不幸的是在 `compact` 行格式下，原始 `page` 和 `overflow-page` 都会加载。
> 
> * 在 TiDB 中开启了 `Titan` 后可以忽略这一点，相见：[Titan 介绍 | PingCAP Docs](https://docs.pingcap.com/zh/tidb/stable/titan-overview)。

#### 类型参考

| 类型（同义词）                                     | 存储长度(BYTES) | 最小值(SIGNED/UNSIGNED) | 最大值(SIGNED/UNSIGNED)      |
| ------------------------------------------- | ----------- | -------------------- | ------------------------- |
| **整形数字**                                    |             |                      |                           |
| TINYINT                                     | 1           | -128/0               | 127/255                   |
| SMALLINT                                    | 2           | -32,768/0            | 32767/65,535              |
| MEDIUMINT                                   | 3           | -8,388,608/0         | 8388607/16,777,215/       |
| INT(INTEGER)                                | 4           | -2,14,7483,648/0     | 2147483647/4,294,967,295/ |
| BIGINT                                      | 8           | -2^63/0              | 2^63-1/2^64-1             |
| **小数支持**                                    |             |                      |                           |
| FLOAT[(M[,D])]                              | 4 or 8      | -                    |                           |
| DOUBLE[(M[,D])]<br>(REAL, DOUBLE PRECISION) | 8           | -                    |                           |
| **时间类型**                                    |             |                      |                           |
| DATETIME                                    | 8           | 1001-01-01 00:00:00  | 9999-12-31 23:59:59       |
| DATE                                        | 3           | 1001-01-01           | 9999-12-31                |
| TIME                                        | 3           | 00:00:00             | 23:59:59                  |
| YEAR                                        | 1           | 1001                 | 9999                      |
| TIMESTAMP                                   | 4           | 1970-01-01 00:00:00  |                           |

### 数据规则

#### 完整性规则

1. 列完整性
   
   * 所有字段通过 Y/N 标识字段能否为空，标识为 N 的字段不能为空；
   * 设置默认值；

2. 数据集完整性
   
   * 如果在两个有关联的数据表中，那么一个数据表的外键一定在另一个数据表中的主键中可以找到。

#### 一致性规则

1. 数字编码化，如币种、城市码；
2. 属性一致，如性别，默认-1 未知：-1，男：1，女：0；
3. 采集方法一致，如列入日期和移除日期成对出现。

#### 准确性规则

1. 甄选数据来源
   
   * 选取权威数据源；
   * 多数据源时，需要考虑多源合并；

2. 异常值处理
   
   * 合法：不在范围内的数据，强制为最大值，比如异常年龄不可超过 200 岁；价格不可以为负；数据、日期时间字段出现汉字；
   * 干预：不合法值、离群值，直接报错，比如同比、环比监控激增；

**时效性规则**

1. 数据的公布日期和发生日期均要采集；
2. 重要数据加快轮巡频率；
3. 留存数据创建日期（create_time）和数据更新日期（update_time）（TiDB 数仓表只设置 `update_ts`） 。

### 常用预定义字段参考

仅供参考，每张表必须包含固定字段，如需使用，按标准添加常用字段。

#### 固定字段

| 字段名称      | 字段中文名  | 字段类型      | 默认值               | 备注                          |
| --------- | ------ | --------- | ----------------- | --------------------------- |
| id        | 自增主键   | BIGINT    |                   | auto_increment              |
| create_ts | 创建时间   | TIMESTAMP | CURRENT_TIMESTAMP |                             |
| update_ts | 记录更新时间 | TIMESTAMP | CURRENT_TIMESTAMP | ON UPDATE CURRENT_TIMESTAMP |

#### 常用字段

| 字段名称         | 字段中文名   | 字段类型          | 默认值 | 备注            |
| ------------ | ------- | ------------- | --- | ------------- |
| url          | 数据链接    | VARCHAR(1024) |     |               |
| source       | 数据来源    | VARCHAR(20)   |     | 使用数字编码，而非文字说明 |
| remarks      | 备注      | TEXT          |     |               |
| address      | 地址等含义   | VARCHAR(1024) |     |               |
| content      | 内容等含义   | TEXT          |     |               |
| sex          | 性别      | TINYINT       | -1  | 未知：-1，男：1，女：0 |
| company_name | 公司机构等含义 | VARCHAR(512)  |     |               |
| person_name  | 个人姓名    | VARCHAR(128)  |     |               |
| department   | 部门等含义   | VARCHAR(512)  |     |               |

### 索引

#### 索引创建

* 【强制】索引建在区分度高的字段上面，不在低基数列上建立索引；
  
  > 示例：在雇员表的“性别”列上只有“男”与“女”两个不同值，因此就无必要建立索引，因为建立索引不 但不会提高查询效率，反而会严重降低更新速度。
  > 
  > 区分度计算：区分度的公式是`count(distinct col)/count(*)`，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，[参考文章](https://tech.meituan.com/2014/06/30/mysql-index.html)。

* 【强制】索引字段必须为 `NOT NULL`，`NULL` 值会影响优化器对索引的选择；

* 【强制】不要对 `TEXT`，`BLOB` 字段建立索引；

* 【强制】单个索引中每个索引记录的长度不能超过 64KB；

* 【建议】在频繁进行排序或分组（即进行 `group by` 或 `order by` 操作）的列上建立索引，有多个列时考虑建立复合索引（compound index）；

* 【建议】在经常进行 `JOIN` 的连接列上建立索引，而不经常连接的字段则由优化器自动生成索引；

* 【建议】数量限制：索引数量控制在 5 个以内，最多不可以超过表字段个数的 20%（考虑索引写入成本）；

* 【建议】区分度最大的字段放在前面；
  
  > 说明：存在非等号和等号混合时，在建索引时，请把等号条件的列前置。如：`where c>? and d=?` 那么即使 `c` 的区分度更高，也必须把 `d` 放在索引的最前列，即索引 `idx_d_c`。
  > 
  > 如果 `where a=? and b=?`，如果 `a` 列的几乎接近于唯一值，那么只需要单建 `idx_a` 索引即可。
- 【建议】对字符串长的字段使用前缀索引；
  
  > 前缀索引说白了就是对文本的前几个字符（具体是几个字符在建立索引时指定）建立索引，这样建立起来的索引更小，所以查询更快；
  > 
  > 前缀索引能有效减小索引文件的大小，提高索引的速度；
  > 
  > 但是前缀索引也有它的坏处：`MySQL` 不能在 `ORDER BY` 或 `GROUP BY` 中使用前缀索引，也不能把它们用作覆盖索引(`Covering Index`)。
  > 
  > ```sql
  > # 建立前缀索引的语法：
  > ALTER TABLE table_name ADD KEY(column_name(prefix_length));
  > ```
* 【建议】合理创建联合索引（避免冗余），`(a,b,c)` 相当于 `(a)`、`(a,b)`、`(a,b,c)`；

* 【建议】合理利用覆盖索引来进行查询操作，避免回表（Lookup）。
  
  > 能够建立索引的种类分为主键索引、唯一索引、普通索引三种，而覆盖索引只是一种查询的一种效果，用`explain`的结果，`extra`列会出现：`using index`。

#### 索引使用

* 【强制】索引列不能参与计算（数学运算和函数运算）；
  
  > 如：`from_unixtime(create_time) = ’2014-05-29’` 就不能使用到索引，`B+Tree` 中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。此语句应该写成`create_time = unix_timestamp(’2014-05-29’)`。

* 【强制】`WHERE` 条件中必须使用合适的类型，避免 `MySQL` 进行隐式类型转化，导致索引失效；
  
  > `f_business_id` 定义为字符型 案例说明中 `where` 条件未加 ‘xxxx’ 引号，会导致索引失效。
  > 
  > ```sql
  > `f_business_id` varchar(40) DEFAULT NULL COMMENT '业务id ';
  > select f_business_id FROM human.t_task WHERE f_business_id = 433423882127424;
  > ```

* 【建议】不使用 `%` 前导的查询，如 `like "%ab"`；
  
  > 说明：索引文件具有`B-Tree`的最左前缀匹配特性，如果左边的值未确定，那么无法使用此索引。

* 【建议】不使用负向查询，如 `NOT IN`/`LIKE`；
  
  > 说明：使用负向查询是指使用负向运算符，如：`NOT`、`!=`、`<>`、`NOT EXISTS`、`NOT IN `以及 `NOT LIKE` 等等。因为通过索引有顺序的结构，可以有效的利用二分查找法，快速找到对等的数据，但若使用负向查询，则无法利用索引结构做二分查找，只好全表扫描。

* 【参考】最左前缀匹配原则，非常重要的原则；
  
  > `mysql` 会一直向右匹配直到遇到范围查询 `(>、<、between、like)` 就停止匹配，比如 `a = 1 and b = 2 and c > 3 and d = 4` 如果建立 `(a,b,c,d)` 顺序的索引，`d` 是用不到索引的，如果建立 `(a,b,d,c)` 的索引则都可以用到，`a,b,d ` 的顺序可以任意调整。

* 【参考】创建索引时避免有如下极端误解：
  
  > 1. 宁滥勿缺。认为一个查询就需要建一个索引；
  > 2. 宁缺勿滥。认为索引会消耗空间、严重拖慢更新和新增速度；
  > 3. 抵制唯一索引。认为业务的惟一性一律需要在应用层通过“先查后插”方式解决。

## SQL 规范

* 【强制】`SELECT` 语句只获取需要的字段，不允许使用 `SELECT *`；

* 【强制】`INSERT` 语句必须显式的指明字段名称，不允许使用 `INSERT INTO table()`；

* 【强制】禁止单条 `SQL ` 语句同时更新多个表；

* 【强制】拒绝使用复杂的 `SQL`，将大的 `SQL` 拆分成多条简单 `SQL` 分步执行，简单的 `SQL` 容易使用到 `MySQL` 的 `query cache`；

* 【强制】超过三个表禁止 `JOIN`。需要 `JOIN` 的字段，数据类型保持绝对一致；多表关联查询时，保证被关联的字段需要有索引；

* 【强制】不要使用 `count(列名)` 或 `count(常量)` 来替代 `count(*)`，`count(*)` 是 `SQL92` 定义的标准统计行数的语法，跟数据库无关，跟 NULL 和非 NULL 无关；
  
  > 说明：`count(*)` 会统计值为 NULL 的行，而 `count(列名)` 不会统计此列为 NULL 值的行。

* 【强制】`count(distinct col)` 计算该列除 NULL 之外的不重复行数。
  
  > 注意： `count(distinct col1, col2)` 如果其中一列全为 NULL，那么即使另一列有不同的值，也返回为 0。

* 【强制】使用 `ISNULL()` 来判断是否为 NULL 值；

* 【强制】对于数据库中表记录的查询和变更，只要涉及多个表，都需要在列名前加表的别名（或表名）进行限定；

* 【建议】SQL 语句中表的别名前加 as，并且以 t1、t2、t3、...的顺序依次命名；

* > 说明：`count(*)` 会统计值为 NULL 的行，而 `count(列名)` 不会统计此列为 NULL 值的行。

* 【建议】利用延迟关联或者子查询优化超多分页场景；
  
  > 说明：`MySQL ` 并不是跳过 `offset` 行，而是取 `offset+N` 行，然后返回放弃前 `offset` 行，返回 N 行，那当 `offset` 特别大的时候，效率就非常的低下，要么控制返回的总页数，要么对超过特定阈值的页数进行 `SQL` 改写。
  > 
  > 示例：先快速定位需要获取的 `id` 段，然后再关联：
  > 
  > ```sql
  > SELECT a.* FROM 表1 a`, `(select id from 表1 where 条件 LIMIT 100000,20 ) b where a.id=b.id
  > ```

* 【建议】禁止使用 OR 条件，改为 IN 查询；

* 【建议】尽量避免 `IN` 操作，若使用 `IN` 查询，查询条件不要超过 500 个；

* 【建议】不要在 SQL 语句中进行数学或函数等业务逻辑运算；

* 【建议】禁止使用子查询，建议将子查询转换成关联查询；

* 【建议】不要在大表上使用子查询（n+1），尽量不使用 `JOIN` 关联；

* 【建议】尽量减少和数据库的访问次数，使用批量处理方式。

## 行为规范

* 【强制】开发人员禁止拥有任何环境 `DLL` 权限账号；

* 【强制】版本发布需在指定目录提交线上`SQL`变更需求，必须详细注明提交人、日期、需求说明；

* 【强制】大数据量的数据操作存在锁表等风险的需要提出风险预警，避开业务高峰期执行；

* 【强制】禁止直接在线上库执行后台管理和统计类查询。

## FAQ

`INT[M]`，`M` 值代表什么含义？

注意数值类型括号后面的数字只是表示宽度而跟存储范围没有关系，比如 `INT(3) ` 默认显示3位，空格补齐，超出时正常显示，`Python`、`Java` 客户端等不具备这个功能。

什么是覆盖索引？

`InnoDB` 存储引擎中，`secondary index` 二级索引中没有直接存储行地址，存储主键值。如果用户需要查询 `二级索引` 中所不包含的数据列时，需要先通过 `二级索引` 查找到主键值，然后再通过主键查询到其他数据列，因此需要查询两次，即回表。

覆盖索引的概念就是查询可以通过在一个索引中完成，覆盖索引效率会比较高，主键查询是天然的覆盖索引。

合理的创建索引以及合理的使用查询语句，当使用到覆盖索引时可以获得性能提升。

```sql
SELECT email,uid FROM user_email WHERE uid=xx
```

如果 `uid` 不是主键，适当时候可以将索引添加为`index(uid,email)`，以获得性能提升

## 引用

* [MySQL 数据库设计规范](https://github.com/guanguans/notes/blob/master/MySQL/MySQL数据库设计规范.md)
* [数据库规范 - 知学云产品开发者文档中心](http://developer.zhixueyun.com/standard/database/#_3)
* [国家食品药品监管信息化标准 数据库设计规范](https://www.nmpa.gov.cn/directory/web/nmpa/images/uL28jQgyv2+3biyei8xrnmt7ajqNX3xPS4rz7uOWjqS5wZGY=.pdf)
