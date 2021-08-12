reference
 https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl.html

The online DDL feature provides support for in-place table alterations and concurrent DML. Benefits of this feature include:

在线ddl变更的特性提供了原地表结构变更与并发dml场景下的支持。具体的优点包括以下几点：

- Improved responsiveness and availability in busy production environments, where making a table unavailable for minutes or hours is not practical.

  提高了业务高峰期下，数据库的响应能力与可用性。原本这是不可以在业务高峰期下进行的，因为这种表结构的变更会导致表出现分钟级甚至小时级别不可用的情况。

- The ability to adjust the balance between performance and concurrency during DDL operations using the LOCK clause. See The LOCK clause.
 
  在使用 LOCK 子句条件下，可调节DDL 操作期间数据库性能和并发ddl操作 之间的平衡。详细可以查看 LOCK 字句的说明文档。
  https://dev.mysql.com/doc/refman/5.7/en/innodb-online-ddl-performance.html#innodb-online-ddl-locking-options
- Less disk space usage and I/O overhead than the table-copy method.
  
  比拷贝表数据的dml方法使用的磁盘空间资源和io资源 远远小得多。

Typically, you do not need to do anything special to enable online DDL. By default, MySQL performs the operation in place, as permitted, with as little locking as possible.

通常情况下，你不需要做任何特殊的操作去启用 online DDL 在线表结构变更的设置。默认情况下，MySQL 在允许的情况下都会选择 原地的变更，尽可能用最少的锁去实现 表结构的变更。

You can control aspects of a DDL operation using the ALGORITHM and LOCK clauses of the ALTER TABLE statement. These clauses are placed at the end of the statement, separated from the table and column specifications by commas. For example:

你可以通过使用 ALTER TABLE中的  ALGORITHM and LOCK clauses 来对这个ddl操作的某些方面进行控制。这些字句放置在你需要发起的statement语句的最后，通过逗号与你的表、列描述分隔开。例如：
```
ALTER TABLE tbl_name ADD PRIMARY KEY (column), ALGORITHM=INPLACE, LOCK=NONE;
```
The LOCK clause is useful for fine-tuning the degree of concurrent access to the table. The ALGORITHM clause is primarily intended for performance comparisons and as a fallback to the older table-copying behavior in case you encounter any issues. For example:

LOCK字句有微调 ddl 操作对这张表上相关并发读写操作影响程度的作用。ALGORITHM 字句的使用主要体现在提升了变更的性能，一旦发生任何不支持的情况，都将回退使用旧的表拷贝复制方法来进行表结构变更。例如：

- To avoid accidentally making the table unavailable for reads, writes, or both, specify a clause on the ALTER TABLE statement such as LOCK=NONE (permit reads and writes) or LOCK=SHARED (permit reads). The operation halts immediately if the requested level of concurrency is not available.

  为了避免造成表不可以读或不可写，或两者均不能进行的情况，在alter语句中指定 如  LOCK=NONE (无锁，允许读写操作并行) or LOCK=SHARED (共享锁，运行并发读).如果在操作的语句中存在不支持使用指定对应的LOCK模式的情况，该操作将会被立即终止。
  - LOCK=DEFAULT (by default， if not specify the LOCK CLAUSE，mysql performs the operation with as little locking as possible using none or shared or EXCLUSIVE)

    默认情况下mysql自带的lock模式，默认情况下，如果没有显式得指定lock=什么内容，mysql将自行选择lock模式来实现最少程度的锁变更。
  - LOCK=NONE (permit reads and writes)
  -  LOCK=SHARED (permit reads)共享锁。
   

- To compare performance between algorithms, run a statement with ALGORITHM=INPLACE and ALGORITHM=COPY. Alternatively, run a statement with the `old_alter_table` configuration option disabled and enabled.

   如果想要比较算法之间的性能差异，可以用不同的算法 INPLACE 和 COPY 运行相同的一句alter statement.或者，在启用与未启用old_alter_table此变量的情况下，进行alter statement进行对比。
> 在启用old_alter_table此变量时，服务器不会使用优化的处理 ALTER TABLE 操作的方法

> https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html#sysvar_old_alter_table

- To avoid tying up the server with an ALTER TABLE operation that copies the table, include ALGORITHM=INPLACE. The statement halts immediately if it cannot use the in-place mechanism.

  为避免使用复制表的 ALTER TABLE 操作占用服务器，请尽可能带上 ALGORITHM=INPLACE的字句。如果在操作的语句中存在不支持使用指定对应的ALGORITHM=INPLACE模式的情况，该操作会被立即终止。
