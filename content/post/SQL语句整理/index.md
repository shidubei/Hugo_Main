+++
title = 'SQL语句整理'
date = 2024-10-30T11:00:11+08:00
draft = true
+++

# SQL语句整理&试题分析

## 1.Select

**在SQL语句中,想要查询数据库中的信息,必须要有的关键字就是Select,其基础的语句结构如下:**

```sql
Select {[All,Distinct]} [column.../*] From (table,...) {Where (search
condition)} {And/Or (search condition)} Group By(column...) 
Having(column..) Order By(column...)
```

 **下面是针对Select 语句形式的一一解释**

### 1.1 Select */ Distinct From table

**Select * from customer: 表示列出customer表中的所有列和所有信息,并没有对表中数据做筛选**

**Select id,name from customer: 表示列出customer表中的Id和name列,其余列不做显示**

**Select Distinct race from customer: 表示列出customer表,race列中 没有重复的元素, Distinct关键字表示截然不同,独立的值**

### 1.2 Select 条件 Where [Not]

**Where关键字通常用于Select 筛选需要条件的时候,也经常和其它关键字连用**

**Select * from customer where race = 'CN' 表示筛选customer表中所有满足race='CN'条件的数据**

| ID  | Name | Race |
| --- | ---- | ---- |
| 01  | John | CN   |
| 02  | Tom  | CN   |
| 03  | Jack | SG   |

**筛选之后为:**

| ID  | Name | Race |
| --- | ---- | ---- |
| 01  | John | CN   |
| 02  | Tom  | CN   |

**Select * from customer where not (race='CN')同样的,加上Not 关键字自后,筛选的结果就会除去所有满足条件的数据**

| ID  | Name | Race |
| --- | ---- | ---- |
| 03  | Jack | SG   |

#### 1.2.1 Select 条件 And/Or/Between/Like/In

**Select * from customer where race ='CN' and age>=21**

**And关键字通常用于设置筛选条件,筛选必须同时满足and前面条件和后面条件的数据**

**Select * from customer where race= 'CN' or age>=21**

**Or关键字通常用于设置筛选条件,筛选满足or前后之一条件的数据,即满足race='CN' 或者 age>=21(包括既满足race='CN'和age>=21的数据)**

**Select * from customer where age between 20 and 23**

**Between关键字通常用于设置筛选条件,筛选出满足范围条件的数据**

**Select * from customer where name like 'J%'**

**Like关键字通常用于模糊匹配,筛选出所有满足类似条件的数据. %表示替代字符.这里的'J%'表示所有以J开头的字符串**

**Select * from customer where name like '%J%'**

**'%J%'则表示包含J字符的数据**

**Select * from customer where race in ('CN','SG')**

**In 关键字也表示范围条件,但范围通常是In括号中的值,对于字符串筛选可以用In,筛选所有满足In范围的数据**

### 1.3 Select 别名 AS

**除了对数据进行条件筛选,我们也可以通过别名机制设置我们想显示的信息**

**Select id AS CustomerID,name from customer. 将id这一列别名为CustomerID,那么我们的显示就会是**

| CustomerID | Name |
| ---------- | ---- |
| 01         | John |
| 02         | Tom  |
| 03         | Jack |

### 1.4 Select 嵌套查询(子查询) Sub Query

**在SQL中,以一个表的查询结果作为一个表(这个表可以是另一个表,也可以是当前表)的查询条件,我们叫做子查询/嵌套查询.和外表类似而不同的是,Sub Query只用在两个表的情况**

#### 1.4.1 同表子查询

**Select employee_name from employees where salary > (select salary from employees where emplyee_name ='zhongyi')**

**上述语句的作用就是筛选所有工资大于'zhongyi'的employee,这就是同表的嵌套查询**

#### 1.4.2 跨表子查询

**除此之外,子查询除了返回一条数据外,也可以返回多行数据,通常与In关键字连用.子查询也可以跨表使用**

**Select employee_name from employees where department_id in (select depaertment_id from departments where location = 'CN')**

**这里就表示查询所有employees中的department_id和departments表中department_id条件相符合的数据.和外键查询类似**

### 1.5 Select 空 Null

**对于数据库查询,我们的数据中有时候会出现空值,当我们要去查询这些空值条件时,就要用Null关键字**

**Select * from customer where email is Null**

### 1.6 Select 排序 Order By [DESC]

### 1.7 Select 计数 Count

**Count 关键字会计算筛选条件返回后的数据的行数,注意是计算数据的行数**

**Count(`*`)计算所有行**

**Count(column)计算当前列中不为null的所有行数**

**Count(Distinct column) 计算当前列中唯一且不为空的行数**

### 1.8 Select 平均值 AVG

**AVG关键字会计算所选列的平均值,会自动忽略null值,如果所有值都是null,则返回null**

### 1.9 Select 总和 SUM

**SUM关键字会计算所选列的综合,将一列中的非null值相加,需要注意的是,当SUM关键字作为结果显示,同样作为结果显示的列需要搭配order by使用**

## 2. Join

### 2.1 Inner Join

### 2.2 Left Join

### 2.3 Self Join

## 3. Where/ Having/ On的区别

**Where通常是在行数据被查询之前进行条件过滤,在使用聚合函数(Count,AVG,SUM)的时候不能使用Where**

**Having用于过滤分组后的数据,通常和Group By一起使用,对聚合的结果进行筛选,其过滤过程是在查询后的,可以和聚合函数(Count,AVG,SUM)一起使用**

**On通常在连接表时指定连接条件,在Inner Join 的时候可以被Where 替代, 但在外连接时候不能被替代**

## 3. Union

## 4. Limit

## 5. SQL语句限制

**对于SQL语句,我们什么时候用Where,什么时候用On,什么时候用Having,Group By等等,好像都没有一个明确的概念.因此,这一小节主要讲解SQL语句的工作原理以及部分关键字的使用限制.**

**首先我们要明确的是SQL语句关键字执行的阶段**

* **FROM: FROM关键字表示SQL执行期确定数据源的阶段,即确定要进行操作的数据来自哪张表,也是最早执行的阶段**

* **JOIN/ON: JOIN和ON关键字也在确定数据源的阶段,但通常表示着连接外表的时候,ON关键字表示外表的连接条件,比如On Order.CustomerID = Customer.CustomerI**

* **Where: 在确定完数据源之后,要提取数据,可能会对数据进行过滤,这时候要用到Where关键字,数据库根据Where之后的条件对要取出的数据进行过滤,然后进行到下一阶段**

* **Group By/聚合函数(SUM/COUNT/AVG): 在过滤好数据之后,可能要对数据进行分组和聚合,这就需要Group By关键字和部分聚合函数发挥作用.SQL根据Group By后面指定的列进行分组,利用聚合函数将分组的结果进行聚合.这是这个阶段的事情.因此不能在Where的时候使用聚合的原因就是,聚合的阶段发生在Where过滤之后.**

* **Having: 在我们分组聚合后,要对数据进行进一步的过滤,这需要Having 关键字.Having 关键字通常对聚合后的结果进行过滤,而不是像Where在原始数据进行过滤.且Having关键字后面不能出现select中没有的列**

* **Select: 在所有的数据过滤-分组-再过滤完毕后.需要选择列输出的阶段,就用到Select 关键字了.虽然Select 关键字写在最前,但执行时一般是在最后的阶段执行.**

* **Distinct: 要对数据进行去重的时候使用**

* **Order By: 我们要求数据有顺序的输出时,要用到Order By,这基本是等前面所有阶段完成后才执行的**

* **LIMIT: 要限制展示的数据多少行的时候使用**
