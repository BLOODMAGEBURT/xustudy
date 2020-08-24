### `Mysql`学习笔记

[TOC]

#### 1、`SQL`语句分类

- `DQL` 查询
- `DML ` 操作
- `DDL` 定义
- `TCL` 事务控制
- `DCL` 权限

#### 2、`Mysql` 常用命令

```mysql
show databases;

use dbName;

show tables;

show create table tableName;

```

#### 3、查询、排序

~~~mysql
# 查询名字中第二个字母是A的人
``` _ 代表一个， %代表多个```
select ename from emp where ename like '_A%';

# 查询名字中带_的人
``` \ 代表转义字符```
select ename from emp where ename like '%\_%';

# 按照工资降序排列，当工资相同时再按照名字升序排列
```多个排序规则时， 先匹配前面的，如果前面相等，再匹配后面的```
select ename,sal from emp order by sal desc, ename asc;

# 找出工作岗位是SALESMAN的员工，并且要求按照薪资的降序排列
```执行顺序是 from - where - select - order by```
select ename,sal,job from emp where job='SALESMAN' order by sal desc;
``` 因此 也可以先起别名 然后 按 别名 排序```
select ename,sal as salary,job from emp where job='SALESMAN' order by salary desc;

~~~

#### 4、分组/聚合函数

> count 计数
>
> sum 求和
>
> avg 平均值
>
> max 最大值
>
> min 最小值
>
> 注意： **所有的分组函数都是对“某一组" 数据进行操作的**
>
> 又名：多行处理函数，指的是输入多行，输出一行；
>
> **分组函数，自动忽略null值。**
>
> **分组函数，不能直接出现在where子句中。**
>
> 单行处理函数： `ifnull`("可能为null的字段"，"如果为null,则处理为什么值")，可以对null值经行预处理。
>
> `select ename, (sal+ ifnull(comm,0))*12 as yearsal from emp;`

~~~mysql
# 找出总人数
select count(*) from emp;

# 找出总工资
select sum(sal) from emp;

# 分组函数，自动忽略null值
select count(comm) from emp;
```与以下语句是同等作用的```
select count(comm) from emp where comm is not null;

# 找出工资高于平均工资的员工
``` 执行顺序 where - group by - avg ```
``` 所以在 where语句中不能使用avg函数```
select ename,sal from emp where sal > (select avg(sal) from emp);

# count(*) 与 count("字段")的区别
``` count(*)统计的是总记录的数量```
select count(*) from emp;
``` count(字段)统计的是 字段中不为null的数据的数量```
select count(comm) from emp;
~~~

group by 和 having

> group by 是对**某个** 或 **某些** 字段 进行分组
>
> having 是对分组结果进行二次过滤
>
> 注意：
>
> **分组函数一般都会和group by 一起使用，**
>
> **并且任何一个分组函数都是在 group by执行之后才会执行**
>
> **当一条语句中没有group by的话， 整张表自成一组**
>
> 规则：
>
> **当一条语句中包含group by时，select 后面只能跟分组函数 和 参与分组的字段**
>
> 





排序规则如下：

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200818153132.png)

~~~mysql
# 找出每个工作岗位的最高薪资
``` 此语句 在oracle会报错，在mysql查询结果无意义```
```因为 ename字段 没有参与分组，所以select 结果随机显示 ```
select ename,job,max(sal) from emp group by job;

# 找出每个部门，不同工作岗位的最高薪资
```多字段联合分组```
select job,deptno,max(sal) from emp group by deptno,job;

~~~

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200818161431.png)

~~~mysql
# 找出每个部门的最高薪资，要求显示薪资大于3000的数据
 select deptno,max(sal) from emp group by deptno having max(sal) > 3000;
``` 但是上面的语句效率低：好不容易分组，然后又不要其中的一部分```
``` 不如最开始的时候就不要，然后再分组，如下使用where过滤```
select deptno, max(sal) from emp where sal > 3000 group by deptno;

# 找出每个部门的平均薪资，要求显示平均薪资大于2900的数据
```此时，因为分组函数不能使用在where子句中, 所以使用having```
select deptno, avg(sal) from emp group by deptno having avg(sal)>2900;
~~~

distinct去重

```mysql
# 统计岗位的数量
select count(distinct job) from emp;
```

#### 5、连接查询

##### 5.1、分类

- 内连接
  - 等值连接
  - 非等值连接
  - 自连接
- 外连接
  - 左连接
  - 右连接

~~~mysql
# 找出每一个员工的部门名称，要求显示员工名和部门名
``` 取别名 可以 增加可读性，也可以提供运行效率， 就不会在两个表里查同一个字段了```
select e.ename, d.dname from emp e, dept d where e.deptno = d.deptno;

```新语法 join on 如下```
select e.ename,d.dname from emp e join dept d on e.deptno = d.deptno;


# （非等值连接） 找出每个员工的工资等级，要求显示员工名，工资，工资等级

select e.ename, e.sal,s.grade from emp e join salgrade s on e.sal between s.losal and s.hisal;

# （自连接）找出每个员工的上级领导，要求显示员工名和对应的领导名
select e.ename, e2.ename as leader from emp e join emp e2 on e.mgr = e2.empno;
~~~

外连接

```mysql
# 找出每个员工的上级领导，要求所有员工都必须显示出来员工名，领导名
select e.ename, e2.ename as leader from emp e left join emp e2 on e.mgr = e2.empno;

# 找出每个部门的员工数量， 要求显示部门名称，员工数量


# 找出哪个部门没有员工，要求显示部门名称

```

#### 6、子查询

- where嵌套子查询
- from嵌套子查询
- select嵌套子查询

#### 7、Union

可以将查询结果集相加

~~~mysql
# 找出工作岗位是SALESMAN和MANAGER的员工？
select ename, job from emp where job='MANAGER' or job='SALESMAN';
``` 还有第二种方式 ```
select ename, job from emp where job in ('MANAGER', 'SALESMAN');
``` 第三种方式，使用union ```
select ename, job from emp where job='MANAGER' 
union 
select ename, job from emp where job='SALESMAN';

~~~

#### 8、视图（View），了解即可

> 以不同的角度，看待同一张表的数据, 可以看作是一个虚拟表
>
> 只能以`DQL`语句创建视图，也即select语句
>
> 视图的作用，就是隐藏表的实现细节

创建视图、删除视图

```mysql
create view myview as select empno, ename from emp;
# 删除
drop view myview;
```

对视图进行增删改查，会影响到原表数据

```mysql
# 显示视图,因为视图也是一张表，只不过是一张特殊表
show table status where comment='view';
# 根据视图查询
select * from myview;
# 根据视图修改
update myvew set ename = 'hello world' where empno='6379';
# 根据视图删除
delete from myview where empno='45';
```

视图的具体作用

保密级别较高的系统，对外只提供视图供程序员操作

```mysql
# 隐藏表的实现细节
create view myview2 as select ename a, deptno b, sal c from emp;
```

#### 9、设计三范式

> 三范式的目的 就是为了 减少 数据的冗余。
>
> 实际开发中，以满足用户的 需求为主，有的时候会拿冗余换取执行速度。

- 每张表都要有主键

- 非主键字段对主键 不能 **部分** 依赖

  *多对多 三张表，关系表两个外键*

  ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200821113249.png)

- 非主键字段对主键 不能 **传递** 依赖

  *一对多 两张表，多的一方加外键*

  ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200821113416.png)

#### 10、一对一表设计方案

1. 第一种：主键共享

   ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200821112946.png)

2. 第二种：外键唯一

   ![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/20200821113041.png)

#### 11、索引

##### 11.1、什么是索引？有什么用？

索引就相当于是一本书的目录，通过目录可以快速找到对应的资源

查询一张表的时候有两种检索方式：

- 全表扫描
- 根据索引扫描

索引为什么能提高检索的效率，最根本的原因就是缩小了扫描的范围。

~~~mysql
# 添加索引，其实是给 某一个字段 或者 某些字段 添加索引
``` 当ename没有添加索引的时候，会进行全表扫描，扫描ename字段中所有的值```
``` 当ename上添加索引的时候，会根据索引扫描，快速定位```
select ename,sal from emp where ename = 'sara';

~~~

索引也需要数据库进行维护，所以不能乱加



##### 11.2、怎么创建索引对象?怎么删除索引对象？

##### 11.3、什么时候考虑给字段加索引？

- 数据量庞大
- 改字段很少进行`DML`操作
- 该字段经常出现在where子句中

