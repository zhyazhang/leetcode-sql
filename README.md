# leetcode-sql

#### 601.体育馆的人流量

`Stadium表`

```
+---------------+---------+
| Column Name   | Type    |
+---------------+---------+
| id            | int     |
| visit_date    | date    |
| people        | int     |
+---------------+---------+
visit_date 是表的主键
每日人流量信息被记录在这三列信息中：序号 (id)、日期 (visit_date)、 人流量 (people)
每天只有一行记录，日期随着 id 的增加而增加
```

`Sql Schema`

```mysql
Create table If Not Exists Stadium (id int, visit_date DATE NULL, people int);
Truncate table Stadium;
insert into Stadium (id, visit_date, people) values ('1', '2017-01-01', '10');
insert into Stadium (id, visit_date, people) values ('2', '2017-01-02', '109');
insert into Stadium (id, visit_date, people) values ('3', '2017-01-03', '150');
insert into Stadium (id, visit_date, people) values ('4', '2017-01-04', '99');
insert into Stadium (id, visit_date, people) values ('5', '2017-01-05', '145');
insert into Stadium (id, visit_date, people) values ('6', '2017-01-06', '1455');
insert into Stadium (id, visit_date, people) values ('7', '2017-01-07', '199');
insert into Stadium (id, visit_date, people) values ('8', '2017-01-09', '188');
```

编写一个 SQL 查询以找出每行的人数大于或等于 `100` 且 `id` 连续的三行或更多行记录。返回按 `visit_date` **升序排列** 的结果表。

```
输入：
Stadium 表:
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 1    | 2017-01-01 | 10        |
| 2    | 2017-01-02 | 109       |
| 3    | 2017-01-03 | 150       |
| 4    | 2017-01-04 | 99        |
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+
输出：
+------+------------+-----------+
| id   | visit_date | people    |
+------+------------+-----------+
| 5    | 2017-01-05 | 145       |
| 6    | 2017-01-06 | 1455      |
| 7    | 2017-01-07 | 199       |
| 8    | 2017-01-09 | 188       |
+------+------------+-----------+
解释：
id 为 5、6、7、8 的四行 id 连续，并且每行都有 >= 100 的人数记录。
请注意，即使第 7 行和第 8 行的 visit_date 不是连续的，输出也应当包含第 8 行，因为我们只需要考虑 id 连续的记录。
不输出 id 为 2 和 3 的行，因为至少需要三条 id 连续的记录。
```



```mysql
select
    id,
    visit_date,
    people
from (
    select
        *,
        count(*) over(partition by rk) as rk2
    from (
        select
            *,
            id - rank() over(order by id) as rk
        from (
            select *
            from Stadium
            where people>=100
        )as t1 ##筛选出符合流量大于100的记录
    )as t2 ##若id连续则rank排名相同
)as t3 ##以rank的排名分组计数
where rk2>=3;
```



#### 626.换座位

`Seat表`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| id          | int     |
| name        | varchar |
+-------------+---------+
Id是该表的主键列。
该表的每一行都表示学生的姓名和ID。
Id是一个连续的增量。
```



`Sql Schema`

```mysql
Create table If Not Exists Seat (id int, student varchar(255));
Truncate table Seat;
insert into Seat (id, student) values ('1', 'Abbot');
insert into Seat (id, student) values ('2', 'Doris');
insert into Seat (id, student) values ('3', 'Emerson');
insert into Seat (id, student) values ('4', 'Green');
insert into Seat (id, student) values ('5', 'Jeames');
```

编写SQL查询来交换每两个连续的学生的座位号。如果学生的数量是奇数，则最后一个学生的id不交换。按 `id` **升序** 返回结果表。

```
输入: 
Seat 表:
+----+---------+
| id | student |
+----+---------+
| 1  | Abbot   |
| 2  | Doris   |
| 3  | Emerson |
| 4  | Green   |
| 5  | Jeames  |
+----+---------+
输出: 
+----+---------+
| id | student |
+----+---------+
| 1  | Doris   |
| 2  | Abbot   |
| 3  | Green   |
| 4  | Emerson |
| 5  | Jeames  |
+----+---------+
解释:
请注意，如果学生人数为奇数，则不需要更换最后一名学生的座位。
```



```mysql
select 
    id,
    if(mod(id,2)=1,
        lead(student,1,student) over(order by id),
        lag(student,1) over(order by id)) as student
from Seat as s
```

#### 1484. 按日期分组销售产品

`Activities`：

```
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| sell_date   | date    |
| product     | varchar |
+-------------+---------+
此表没有主键，它可能包含重复项。
此表的每一行都包含产品名称和在市场上销售的日期。
```

```mysql
Create table If Not Exists Activities (sell_date date, product varchar(20))
Truncate table Activities
insert into Activities (sell_date, product) values ('2020-05-30', 'Headphone')
insert into Activities (sell_date, product) values ('2020-06-01', 'Pencil')
insert into Activities (sell_date, product) values ('2020-06-02', 'Mask')
insert into Activities (sell_date, product) values ('2020-05-30', 'Basketball')
insert into Activities (sell_date, product) values ('2020-06-01', 'Bible')
insert into Activities (sell_date, product) values ('2020-06-02', 'Mask')
insert into Activities (sell_date, product) values ('2020-05-30', 'T-Shirt')
```

编写一个 SQL 查询来查找每个日期、销售的不同产品的数量及其名称。每个日期的销售产品名称应按词典序排列。返回按 `sell_date` 排序的结果表。查询结果格式如下例所示。

```
输入：
Activities 表：
+------------+-------------+
| sell_date  | product     |
+------------+-------------+
| 2020-05-30 | Headphone   |
| 2020-06-01 | Pencil      |
| 2020-06-02 | Mask        |
| 2020-05-30 | Basketball  |
| 2020-06-01 | Bible       |
| 2020-06-02 | Mask        |
| 2020-05-30 | T-Shirt     |
+------------+-------------+
输出：
+------------+----------+------------------------------+
| sell_date  | num_sold | products                     |
+------------+----------+------------------------------+
| 2020-05-30 | 3        | Basketball,Headphone,T-shirt |
| 2020-06-01 | 2        | Bible,Pencil                 |
| 2020-06-02 | 1        | Mask                         |
+------------+----------+------------------------------+
解释：
对于2020-05-30，出售的物品是 (Headphone, Basketball, T-shirt)，按词典序排列，并用逗号 ',' 分隔。
对于2020-06-01，出售的物品是 (Pencil, Bible)，按词典序排列，并用逗号分隔。
对于2020-06-02，出售的物品是 (Mask)，只需返回该物品名。
```

```mysql
select 
    sell_date, 
    count(distinct product) as num_sold, 
    group_concat(distinct product order by product asc separator ',') as products
from Activities
group by sell_date
```





















