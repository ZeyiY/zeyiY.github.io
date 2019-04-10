---
title: MySQL语句中IN和EXISTS的区别和使用场景
date: 2019-04-01 23:40:41
tags:
---
## MySQL SQL语句中IN 和 EXISTS的区别和使用场景

转载：https://blog.csdn.net/wqc19920906/article/details/79800374

### IN 语句：只执行一次

确定给定的值是否与子查询或列表中的值相匹配。

in在查询的时候，首先查询子查询的表，然后将内表和外表做一个笛卡尔积，然后按照条件进行筛选。所以相对内表比较小的时候，in的速度较快。

### EXISTS语句：执行外表结果.length次

指定一个子查询，检测行的存在。遍历循环外表，然后看外表中的记录有没有和内表的数据一样的。匹配上就将结果放入结果集中。

使用exists关键字进行查询的时候，首先，我们先查询的不是子查询的内容，而是查我们的主查询的表。

然后，根据表的每一条记录，执行以下语句，依次去判断where后面的条件是否成立。

如果成立则返回true不成立则返回false。如果返回的是true的话，则该行结果保留，如果返回的是false的话，则删除该行，最后将得到的结果返回。

### 区别及应用场景

in 和 exists的区别: 

1. 如果子查询得出的结果集记录较少，主查询中的表较大且又有索引时应该用in。
2.  反之如果外层的主查询记录较少，子查询中的表大，又有索引时使用exists。
3. 其实我们区分in和exists主要是造成了驱动顺序的改变(这是性能变化的关键)，如果是exists，那么以外层表为驱动表，先被访问，如果是IN，那么先执行子查询，所以我们会以驱动表的快速返回为目标，那么就会考虑到索引及结果集的关系了 。
4. IN时不对NULL进行处理。
5. in 是把外表和内表作hash 连接，而exists是对外表作loop循环，每次loop循环再对内表进行查询。
6. 一直以来认为exists比in效率高的说法是不准确的。

### not in 和not exists

​    如果查询语句使用了not in 那么内外表都进行全表扫描，没有用到索引；而not extsts 的子查询依然能用到表上的索引。所以无论那个表大，用not exists都比not in要快。

### 场景示例

1. IN 查询示例

```
select * from A
where id in(select id from B)
```

查询过程类似于以下过程

```
List resultSet=[];
Array A=(select * from A);
Array B=(select id from B);

for(int i=0;i<A.length;i++) {
   for(int j=0;j<B.length;j++) {
      if(A[i].id==B[j].id) {
         resultSet.add(A[i]);
         break;
      }
   }
}
return resultSet;
```

2. exists 查询示例

```
select a.* from A a 
where exists(select 1 from B b where a.id=b.id)
```

查询过程类似于以下过程

```
List resultSet=[];
Array A=(select * from A)

for(int i=0;i<A.length;i++) {
   if(exists(A[i].id) {    //执行select 1 from B b where b.id=a.id是否有记录返回
       resultSet.add(A[i]);
   }
}
return resultSet;
```

3. exists 在子查询中使用 NULL 仍然返回结果集

select * from TableIn where exists(select null)
等同于： select * from TableIn

4. 

```
users表有1000条记录，id自增，id都大于0

select * from users where exists (select * from users limit 0); --输出1000条记录

select * from users where exists (select * from users where id < 0); --输出0条记录
```

exists查询的本质，只要碰到有记录，则返回true；所以limit根本就不会去管，或者说执行不到。

5. exists不可以完全代替in

```
--没有关联字段的情况：枚举常量
select * from areas where id in (4, 5, 6);

--没有关联字段的情况：这样exists对子查询，要么全true，要么全false
select * from areas where id in (select city_id from deals where deals.name = 'xxx'); 
```

6. exists 优化

在许多基于基础表的查询中，为了满足一个条件，往往需要对另一个表进行联接。 
在这种情况下，使用exists(或not exists)通常将提高查询的效率。

```
（低效） 
select ... from table1 t1 where t1.id > 10 and pno in (select no from table2 where name like 'www%'); 
（高效） 
select ... from table1 t1 where t1.id > 10 and exists (select 1 from table2 t2 where t1.pno = t2.no and name like 'www%'); 
```

7. 用not exists替代not in

在子查询中，not in子句将执行一个内部的排序和合并。 
无论在哪种情况下，not in都是最低效的 (因为它对子查询中的表执行了一个全表遍历)。 
为了避免使用not in，我们可以把它改写成外连接(Outer Joins)或not exists。 

8. 用exists替换distinct

当提交一个包含一对多表信息的查询时,避免在select子句中使用distinct. 一般可以考虑用exists替换 

exists使查询更为迅速,因为RDBMS核心模块将在子查询的条件一旦满足后,立刻返回结果.

```
（低效） 
select distinct d.dept_no, d.dept_name from t_dept d, t_emp e where d.dept_no = e.dept_no; 
（高效） 
select d.dept_no, d.dept_name from t_dept d where exists (select 1 from t_emp where d.dept_no = e.dept_no); 
```

9. 用表连接替换exists

```
（低效） 
select ename from emp e where exists (select 1 from dept where dept_no = e.dept_no and dept_cat = 'W'); 
SELECT ENAME 
（高效） 
select ename from dept d, emp e where e.dept_no = d.dept_no and dept_cat = 'W'; 
```

