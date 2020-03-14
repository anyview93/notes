<center><h1>数据库优化-explain</h1></center>

# 1.explain

~~~plsql
EXPLAIN SELECT * FROM tenk1;

                         QUERY PLAN
-------------------------------------------------------------
 Seq Scan on tenk1  (cost=0.00..458.00 rows=10000 width=244)
~~~

- 估计的启动成本。这是输出阶段可以开始之前花费的时间，例如，在排序节点中进行排序的时间。
- 估计总费用。这是在假设计划节点已运行完成（即，检索到所有可用行）的假设下得出的。实际上，节点的父节点可能会停止读取所有可用行（请参阅下面的`LIMIT`示例）。
- 此计划节点输出的估计行数。同样，假定该节点运行完毕。
- 此计划节点输出的行的估计平均宽度（以字节为单位）。

## 2.analyze

~~~plsql
EXPLAIN ANALYZE SELECT *
FROM tenk1 t1, tenk2 t2
WHERE t1.unique1 < 10 AND t1.unique2 = t2.unique2;

                                                           QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------
 Nested Loop  (cost=4.65..118.62 rows=10 width=488) (actual time=0.128..0.377 rows=10 loops=1)
   ->  Bitmap Heap Scan on tenk1 t1  (cost=4.36..39.47 rows=10 width=244) (actual time=0.057..0.121 rows=10 loops=1)
         Recheck Cond: (unique1 < 10)
         ->  Bitmap Index Scan on tenk1_unique1  (cost=0.00..4.36 rows=10 width=0) (actual time=0.024..0.024 rows=10 loops=1)
               Index Cond: (unique1 < 10)
   ->  Index Scan using tenk2_unique2 on tenk2 t2  (cost=0.29..7.91 rows=1 width=244) (actual time=0.021..0.022 rows=1 loops=10)
         Index Cond: (unique2 = t1.unique2)
 Planning time: 0.181 ms
 Execution time: 0.501 ms
~~~

​	`EXPLAIN`实际上会执行查询，然后显示每个计划节点内累积的真实行数和真实运行时间，以及与普通`EXPLAIN`显示的估计相同的估计。

​	在上面的嵌套循环计划中，内部索引扫描将对每个外部行执行一次。在这种情况下，`循环`值报告该节点的执行总数，并且显示的实际时间和行值是每次执行的平均值。这样做是为了使数字与显示成本估算的方式具有可比性。乘以`loops`值即可得出该节点实际花费的总时间。在上面的示例中，我们总共花费了0.220毫秒对`tenk2`执行索引扫描。

​	在某些情况下，`EXPLAIN ANALYZE会`显示计划节点执行时间和行数以外的其他执行统计信息。例如，“排序”和“哈希”节点提供了额外的信息

------

​	`EXPLAIN`有一个`BUFFERS`选项，可以与`ANALYZE`一起使用，以获取更多的运行时统计信息：

~~~plsql
EXPLAIN (ANALYZE, BUFFERS) SELECT * FROM tenk1 WHERE unique1 < 100 AND unique2 > 9000;

                                                           QUERY PLAN
---------------------------------------------------------------------------------------------------------------------------------
 Bitmap Heap Scan on tenk1  (cost=25.08..60.21 rows=10 width=244) (actual time=0.323..0.342 rows=10 loops=1)
   Recheck Cond: ((unique1 < 100) AND (unique2 > 9000))
   Buffers: shared hit=15
   ->  BitmapAnd  (cost=25.08..25.08 rows=10 width=0) (actual time=0.309..0.309 rows=0 loops=1)
         Buffers: shared hit=7
         ->  Bitmap Index Scan on tenk1_unique1  (cost=0.00..5.04 rows=101 width=0) (actual time=0.043..0.043 rows=100 loops=1)
               Index Cond: (unique1 < 100)
               Buffers: shared hit=2
         ->  Bitmap Index Scan on tenk1_unique2  (cost=0.00..19.78 rows=999 width=0) (actual time=0.227..0.227 rows=999 loops=1)
               Index Cond: (unique2 > 9000)
               Buffers: shared hit=5
 Planning time: 0.088 ms
 Execution time: 0.423 ms
~~~

​	`BUFFERS`提供的数字有助于确定查询的哪些部分是I / O最密集的部分。

### 3.参考

​	[PostgreSQL官方文档](https://www.postgresql.org/docs/9.6/using-explain.html)

