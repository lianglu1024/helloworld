两张表：`student`和`class`

```
mysql> select * from student;
+----+-----+-------+----------+
| id | age | name  | class_id |
+----+-----+-------+----------+
|  1 |  18 | zk    |        1 |
|  2 |  17 | jack  |        2 |
|  3 |  18 | lucs  |        3 |
|  4 |  18 | helen |        4 |
|  5 |  17 | bob   |        3 |
|  6 |  19 | json  |        2 |
+----+-----+-------+----------+
mysql> select * from class;
+----+-------+--------------+
| id | level | total_people |
+----+-------+--------------+
|  1 | A     |           20 |
|  2 | B     |           30 |
|  3 | C     |           40 |
|  4 | D     |           45 |
|  5 | E     |           50 |
+----+-------+--------------+
```

## join

join是笛卡尔积，如果表A有n条数据，表B有m条数据，那么A join B就是n*m条数据；B join A等价于A join B

```
mysql> select * from student join class;
+----+-----+-------+----------+----+-------+--------------+
| id | age | name  | class_id | id | level | total_people |
+----+-----+-------+----------+----+-------+--------------+
|  1 |  18 | zk    |        1 |  1 | A     |           20 |
|  1 |  18 | zk    |        1 |  2 | B     |           30 |
|  1 |  18 | zk    |        1 |  3 | C     |           40 |
|  1 |  18 | zk    |        1 |  4 | D     |           45 |
|  1 |  18 | zk    |        1 |  5 | E     |           50 |
|  2 |  17 | jack  |        2 |  1 | A     |           20 |
|  2 |  17 | jack  |        2 |  2 | B     |           30 |
|  2 |  17 | jack  |        2 |  3 | C     |           40 |
|  2 |  17 | jack  |        2 |  4 | D     |           45 |
|  2 |  17 | jack  |        2 |  5 | E     |           50 |
|  3 |  18 | lucs  |        3 |  1 | A     |           20 |
|  3 |  18 | lucs  |        3 |  2 | B     |           30 |
|  3 |  18 | lucs  |        3 |  3 | C     |           40 |
|  3 |  18 | lucs  |        3 |  4 | D     |           45 |
|  3 |  18 | lucs  |        3 |  5 | E     |           50 |
|  4 |  18 | helen |        4 |  1 | A     |           20 |
|  4 |  18 | helen |        4 |  2 | B     |           30 |
|  4 |  18 | helen |        4 |  3 | C     |           40 |
|  4 |  18 | helen |        4 |  4 | D     |           45 |
|  4 |  18 | helen |        4 |  5 | E     |           50 |
|  5 |  17 | bob   |        3 |  1 | A     |           20 |
|  5 |  17 | bob   |        3 |  2 | B     |           30 |
|  5 |  17 | bob   |        3 |  3 | C     |           40 |
|  5 |  17 | bob   |        3 |  4 | D     |           45 |
|  5 |  17 | bob   |        3 |  5 | E     |           50 |
|  6 |  19 | json  |        2 |  1 | A     |           20 |
|  6 |  19 | json  |        2 |  2 | B     |           30 |
|  6 |  19 | json  |        2 |  3 | C     |           40 |
|  6 |  19 | json  |        2 |  4 | D     |           45 |
|  6 |  19 | json  |        2 |  5 | E     |           50 |
+----+-----+-------+----------+----+-------+--------------+
30 rows in set (0.00 sec)
```

加上过滤条件（除了用on，还可以用where，对left join不可以使用）

```
mysql> select * from student join class on student.class_id=class.id;
+----+-----+-------+----------+----+-------+--------------+
| id | age | name  | class_id | id | level | total_people |
+----+-----+-------+----------+----+-------+--------------+
|  1 |  18 | zk    |        1 |  1 | A     |           20 |
|  2 |  17 | jack  |        2 |  2 | B     |           30 |
|  3 |  18 | lucs  |        3 |  3 | C     |           40 |
|  4 |  18 | helen |        4 |  4 | D     |           45 |
|  5 |  17 | bob   |        3 |  3 | C     |           40 |
|  6 |  19 | json  |        2 |  2 | B     |           30 |
+----+-----+-------+----------+----+-------+--------------+
6 rows in set (0.00 sec)
```

## left join
如果A left join B，那么就以A为基准，A表行数据需要全部输出，B表若没有对应行去匹配，输出NULL

另外，A left join B 等价于 B right join A

```
mysql> select * from student left join class on student.class_id=class.id;
+----+-----+-------+----------+------+-------+--------------+
| id | age | name  | class_id | id   | level | total_people |
+----+-----+-------+----------+------+-------+--------------+
|  1 |  18 | zk    |        1 |    1 | A     |           20 |
|  2 |  17 | jack  |        2 |    2 | B     |           30 |
|  6 |  19 | json  |        2 |    2 | B     |           30 |
|  3 |  18 | lucs  |        3 |    3 | C     |           40 |
|  5 |  17 | bob   |        3 |    3 | C     |           40 |
|  4 |  18 | helen |        4 |    4 | D     |           45 |
+----+-----+-------+----------+------+-------+--------------+
6 rows in set (0.00 sec)

mysql> select * from class left join student on student.class_id=class.id;
+----+-------+--------------+------+------+-------+----------+
| id | level | total_people | id   | age  | name  | class_id |
+----+-------+--------------+------+------+-------+----------+
|  1 | A     |           20 |    1 |   18 | zk    |        1 |
|  2 | B     |           30 |    2 |   17 | jack  |        2 |
|  3 | C     |           40 |    3 |   18 | lucs  |        3 |
|  4 | D     |           45 |    4 |   18 | helen |        4 |
|  3 | C     |           40 |    5 |   17 | bob   |        3 |
|  2 | B     |           30 |    6 |   19 | json  |        2 |
|  5 | E     |           50 | NULL | NULL | NULL  |     NULL |
+----+-------+--------------+------+------+-------+----------+
7 rows in set (0.00 sec)
```
