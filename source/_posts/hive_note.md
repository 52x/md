##Hive Note ##

### Hive 实现 WordCount 示例程序 ###

```
CREATE TABLE docs (lines STRING);
LOAD DATA INPUT 'docs' OVERWRITE INTO TABLE docs;
CREATE TABLE word_counts AS
SELECT word, count(1) As count FROM 
  (SELECT explode(split(line, 's')) AS word FROM docs) w
GROUP BY word
ORDER BY word;
```

### Hive 中的命名空间 ###

| 命名空间        | 使用权限           | 描述  |
| ------------- |:-------------:| ---------------:|
| hivevar      | 可读/可写 | (Hive v0.8.0 以及以后的版本) 用户自定义变量 |
| hiveconf      | 可读/可写      |  Hive 相关的配置属性 |
| system | 可读/可写  |    Java 定义的配置属性 |
| env    |  只可读           |   Shell 环境定义的环境变量       |

用户可以自定义变量以便在 Hive 脚本中使用。

### Hive 中的命令

1. `hive -e "select * from tablename limit 10"; > /tmp/tabledata`  运行 sql 语句，将输出结果重定向到文件中
2. `hive -f /path/to/file/queries.hql`；在 hive shell 中可以使用 `source /path/to/file/queries.hql;` 来执行一个脚本文件
3. 用户不需要退出 hive CLI 就可以执行简单的 bash shell 命令。只要在命令前加上 ！ 并且以分号(;)结尾。
4. 使用 `set hive.cli.print.current.db=true;` 显示当前所在的数据库.
5. 一般在 Hive 中不允许删除一个包含有表的数据库，一种做法是先删除数据库中的所有表，再删除数据库；另外一种做法是在删除语句后面加上关键字 cascade;
6. 通过拷贝一张已经存在的表的表模式（无需拷贝数据）来创建新表: ```CREATE TABLE IF NOT EXISTS mydb.employees2 LIKE mydb.employees;```
7. 外部表 CREATE EXTERNAL TABLE .... LOCATION ''，通过 external 说明创建的是外部表，location指示该表的数据位于哪个路径。外部表，Hive 不认为自己完全拥有这份数据，因此，删除该表并不会删除这份数据，只会删除该表的元数据。
8. 可以手动设置 Hive 为严格(strict)模式，此时，如果查询语句中没有指定分区过滤，就会禁止提交这个任务。当然也可以设置为非严格(nostrict)模式。set hive.mapred.mode=strict;
9. SHOW PARTITIONS tablename; 查看表分区， SHOW PARTITIONS tablename PARTITIONS(partition1='') 查看某个分区下的数据
10. 导出数据：按照原格式导出， `hadoop fs -cp source_path target_path`, 或者使用
  ```
  INSERT OVERWRITE LOCAL DIRECTORY '/tmp/ca_employees' 
  SELECT name, salary, address FROM employees WHERE se.state='CA';
  ```

###概念说明
1. 管理表和外部表： **管理表**一般指的是我们自己创建的表，有时称为内部表，这种表 Hive 或多或少会控制着数据的生命周期。这些表的额数据存储在有配置项 hive.metastore.warehouse.dir 所定义的目录的子目录下。当我们删除一个管理表时，Hive 也会删除这个表中的数据；**外部表** 删除外部表时，只是删除了外部表的元数据。
