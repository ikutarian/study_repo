## 慢查询日志

### 相关参数

慢查询日志和这几个参数有关：

| 参数名称            | 意义                                                   |
| ------------------- | ------------------------------------------------------ |
| slow_query_log      | 慢查询日志的记录开关                                   |
| slow_query_log_file | 慢查询日志的存放位置                                   |
| long_query_time     | 慢查询时间阀值，单位是秒。超过这个时间就被认定为慢查询 |

可以利用以下 SQL 语句查看 `slow_query_log` 和 `slow_query_log_file` 的相关信息

```mysql
mysql> show variables like '%slow_query%';
+---------------------+-----------------------------------+
| Variable_name       | Value                             |
+---------------------+-----------------------------------+
| slow_query_log      | OFF                               |
| slow_query_log_file | /var/lib/mysql/localhost-slow.log |
+---------------------+-----------------------------------+
2 rows in set (0.01 sec)
```

利用以下 SQL 查看 `long_query_time` 的信息

```mysql
mysql> show variables like '%long_query%';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
1 row in set (0.00 sec)
```

### 开启慢查询

#### 临时开启

**注意：该方法只是临时生效，MySQL 重启后就会失效**

打开慢查询记录开关

```mysql
mysql> set global slow_query_log = on;
Query OK, 0 rows affected (0.00 sec)
```

设置慢查询时间阀值，比如设置为 1 秒

```mysql
mysql> set global long_query_time = 1;
Query OK, 0 rows affected (0.00 sec)
```

> **MySQL 中 long_query_time 的值如何确定呢？**
>
> 线上业务一般建议把 `long_query_time` 设置为 1 秒，如果某个业务的 MySQL 要求比较高的 QPS，可设置慢查询为 0.1 秒。发现慢查询及时优化或者提醒开发改写。
>
> 一般测试环境建议 `long_query_time` 设置的阀值比生产环境的小，比如生产环境是 1 秒，则测试环境建议配置成 0.5 秒。便于在测试环境及时发现一些效率低的 SQL。
>
> 甚至某些重要业务测试环境 `long_query_time` 可以设置为 0，以便记录所有语句。并留意慢查询日志的输出，上线前的功能测试完成后，分析慢查询日志每类语句的输出，重点关注 `Rows_examined`（语句执行期间从存储引擎读取的行数），提前优化。

#### 永久开启

打开 MySQL 配置文件 `/etc/my.cnf` ，增加以下信息

```ini
[mysqld]
slow_query_log = ON
long_query_time = 1
```

然后重启 MySQL

```shell
service mysqld restart
```

### 验证慢查询

执行一条耗时 2 秒的 SQL 语句

```mysql
SELECT sleep(2);
```

查看慢查询日志的内容

```
# Time: 2020-07-30T14:56:42.684211Z
# User@Host: root[root] @ localhost []  Id:     8
# Query_time: 2.000816  Lock_time: 0.000000 Rows_sent: 1  Rows_examined: 1
SET timestamp=1596121000;
SELECT sleep(2);
```

### 慢查询日志的格式

这里对上方的执行结果详细描述一下

- `Time`：慢查询发生的时间
- `User@Host`：客户端用户和 IP
- `Query_time`：查询耗费时间
- `Lock_time`：等待锁的时间
- `Rows_sent`：语句返回的行数
- `Rows_examined`：语句执行期间从存储引擎读取的行数
- `SET timestamp=1596121000;`：当前时间戳
- `SELECT sleep(2);`：慢查询的 SQL 语句

## 利用 EXPLAIN 分析SQL语句

通过慢查询日志可以知道有哪些 SQL 语句执行很慢，但是怎么知道它们为什么执行这么慢呢？