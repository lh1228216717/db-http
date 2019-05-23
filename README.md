# ORACLE

## ORACLE 触发器 新旧值
``` sql
   id  name     create_date
              1   张三     2018-01-20 00:00:00
              
       1. 新增记录：
              
                  :old.name   为空
                  :new.name   张三
       
       2. 修改记录
              
              2.1.修改id字段值（不修改name的情况下）
              
                :old.name   张三
                :new.name   张三
              
              2.2 修改name字段值（修改name的情况下）
              
                :old.name   张三
                :new.name   李四
                
       3. 删除记录
       
                :old.name   李四
                :new.name   空

```
## ORACLE 触发器语法
```
Oracle触发器用法实例详解
本文实例讲述了Oracle触发器用法。分享给大家供大家参考，具体如下：

一、触发器简介

触发器的定义就是说某个条件成立的时候，触发器里面所定义的语句就会被自动的执行。

因此触发器不需要人为的去调用，也不能调用。

然后，触发器的触发条件其实在你定义的时候就已经设定好了。

这里面需要说明一下，触发器可以分为语句级触发器和行级触发器。

详细的介绍可以参考网上的资料，简单的说就是语句级的触发器可以在某些语句执行前或执行后被触发。而行级触发器则是在定义的了触发的表中的行数据改变时就会被触发一次。

具体举例：

1、 在一个表中定义的语句级的触发器，当这个表被删除时，程序就会自动执行触发器里面定义的操作过程。这个就是删除表的操作就是触发器执行的条件了。
2、 在一个表中定义了行级的触发器，那当这个表中一行数据发生变化的时候，比如删除了一行记录，那触发器也会被自动执行了。

二、触发器语法

触发器的语法：


create [or replace] tigger 触发器名 触发时间 触发事件
on 表名
[for each row]
begin
 pl/sql语句
end
其中：

触发器名：触发器对象的名称。由于触发器是数据库自动执行的，因此该名称只是一个名称，没有实质的用途。
触发时间：指明触发器何时执行，该值可取：
before：表示在数据库动作之前触发器执行;
after：表示在数据库动作之后触发器执行。
触发事件：指明哪些数据库动作会触发此触发器：
insert：数据库插入会触发此触发器;
update：数据库修改会触发此触发器;
delete：数据库删除会触发此触发器。
表 名：数据库触发器所在的表。
for each row：对表的每一行触发器执行一次。如果没有这一选项，则只对整个表执行一次。

触发器能实现如下功能：

功能：

1、 允许/限制对表的修改
2、 自动生成派生列，比如自增字段
3、 强制数据一致性
4、 提供审计和日志记录
5、 防止无效的事务处理
6、 启用复杂的业务逻辑

举例

1)、下面的触发器在更新表tb_emp之前触发，目的是不允许在周末修改表：


create or replace trigger auth_secure before insert or update or DELETE
on tb_emp
begin
  IF(to_char(sysdate,'DY')='星期日') THEN
    RAISE_APPLICATION_ERROR(-20600,'不能在周末修改表tb_emp');
  END IF;
END;
/
2)、使用触发器实现序号自增

创建一个测试表：

create table tab_user(
  id number(11) primary key,
  username varchar(50),
  password varchar(50)
);
创建一个序列：
复制代码 代码如下:
create sequence my_seq increment by 1 start with 1 nomaxvalue nocycle cache 20;
创建一个触发器：

CREATE OR REPLACE TRIGGER MY_TGR
 BEFORE INSERT ON TAB_USER
 FOR EACH ROW--对表的每一行触发器执行一次
DECLARE
 NEXT_ID NUMBER;
BEGIN
 SELECT MY_SEQ.NEXTVAL INTO NEXT_ID FROM DUAL;
 :NEW.ID := NEXT_ID; --:NEW表示新插入的那条记录
END;
向表插入数据:

insert into tab_user(username,password) values('admin','admin');
insert into tab_user(username,password) values('fgz','fgz');
insert into tab_user(username,password) values('test','test');
COMMIT;
查询表结果：SELECT * FROM TAB_USER;



3)、当用户对test表执行DML语句时，将相关信息记录到日志表

--创建测试表
CREATE TABLE test(
  t_id  NUMBER(4),
  t_name VARCHAR2(20),
  t_age NUMBER(2),
  t_sex CHAR
);
--创建记录测试表
CREATE TABLE test_log(
  l_user  VARCHAR2(15),
  l_type  VARCHAR2(15),
  l_date  VARCHAR2(30)
);
创建触发器：

--创建触发器
CREATE OR REPLACE TRIGGER TEST_TRIGGER
 AFTER DELETE OR INSERT OR UPDATE ON TEST
DECLARE
 V_TYPE TEST_LOG.L_TYPE%TYPE;
BEGIN
 IF INSERTING THEN
  --INSERT触发
  V_TYPE := 'INSERT';
  DBMS_OUTPUT.PUT_LINE('记录已经成功插入，并已记录到日志');
 ELSIF UPDATING THEN
  --UPDATE触发
  V_TYPE := 'UPDATE';
  DBMS_OUTPUT.PUT_LINE('记录已经成功更新，并已记录到日志');
 ELSIF DELETING THEN
  --DELETE触发
  V_TYPE := 'DELETE';
  DBMS_OUTPUT.PUT_LINE('记录已经成功删除，并已记录到日志');
 END IF;
 INSERT INTO TEST_LOG
 VALUES
  (USER, V_TYPE, TO_CHAR(SYSDATE, 'yyyy-mm-dd hh24:mi:ss')); --USER表示当前用户名
END;
/
--下面我们来分别执行DML语句
INSERT INTO test VALUES(101,'zhao',22,'M');
UPDATE test SET t_age = 30 WHERE t_id = 101;
DELETE test WHERE t_id = 101;
--然后查看效果
SELECT * FROM test;
SELECT * FROM test_log;
运行结果如下：



3)、创建触发器，它将映射emp表中每个部门的总人数和总工资

--创建映射表
CREATE TABLE dept_sal AS
SELECT deptno, COUNT(empno) total_emp, SUM(sal) total_sal
FROM scott.emp
GROUP BY deptno;
--创建触发器
CREATE OR REPLACE TRIGGER EMP_INFO
 AFTER INSERT OR UPDATE OR DELETE ON scott.EMP
DECLARE
 CURSOR CUR_EMP IS
  SELECT DEPTNO, COUNT(EMPNO) AS TOTAL_EMP, SUM(SAL) AS TOTAL_SAL FROM scott.EMP GROUP BY DEPTNO;
BEGIN
 DELETE DEPT_SAL; --触发时首先删除映射表信息
 FOR V_EMP IN CUR_EMP LOOP
  --DBMS_OUTPUT.PUT_LINE(v_emp.deptno || v_emp.total_emp || v_emp.total_sal);
  --插入数据
  INSERT INTO DEPT_SAL
  VALUES
   (V_EMP.DEPTNO, V_EMP.TOTAL_EMP, V_EMP.TOTAL_SAL);
 END LOOP;
END;
--对emp表进行DML操作
INSERT INTO emp(empno,deptno,sal) VALUES('123','10',10000);
SELECT * FROM dept_sal;
DELETE EMP WHERE empno=123;
SELECT * FROM dept_sal;
显示结果如下：



4)、创建触发器，用来记录表的删除数据

--创建表
CREATE TABLE employee(
  id  VARCHAR2(4) NOT NULL,
  name VARCHAR2(15) NOT NULL,
  age NUMBER(2)  NOT NULL,
  sex CHAR NOT NULL
);
--插入数据
INSERT INTO employee VALUES('e101','zhao',23,'M');
INSERT INTO employee VALUES('e102','jian',21,'F');
--创建记录表(包含数据记录)
CREATE TABLE old_employee AS SELECT * FROM employee;
--创建触发器
CREATE OR REPLACE TRIGGER TIG_OLD_EMP
 AFTER DELETE ON EMPLOYEE
 FOR EACH ROW --语句级触发，即每一行触发一次
BEGIN
 INSERT INTO OLD_EMPLOYEE VALUES (:OLD.ID, :OLD.NAME, :OLD.AGE, :OLD.SEX); --:old代表旧值
END;
/
--下面进行测试
DELETE employee;
SELECT * FROM old_employee;
5)、创建触发器，利用视图插入数据

--创建表
CREATE TABLE tab1 (tid NUMBER(4) PRIMARY KEY,tname VARCHAR2(20),tage NUMBER(2));
CREATE TABLE tab2 (tid NUMBER(4),ttel VARCHAR2(15),tadr VARCHAR2(30));
--插入数据
INSERT INTO tab1 VALUES(101,'zhao',22);
INSERT INTO tab1 VALUES(102,'yang',20);
INSERT INTO tab2 VALUES(101,'13761512841','AnHuiSuZhou');
INSERT INTO tab2 VALUES(102,'13563258514','AnHuiSuZhou');
--创建视图连接两张表
CREATE OR REPLACE VIEW tab_view AS SELECT tab1.tid,tname,ttel,tadr FROM tab1,tab2 WHERE tab1.tid = tab2.tid;
--创建触发器
CREATE OR REPLACE TRIGGER TAB_TRIGGER
 INSTEAD OF INSERT ON TAB_VIEW
BEGIN
 INSERT INTO TAB1 (TID, TNAME) VALUES (:NEW.TID, :NEW.TNAME);
 INSERT INTO TAB2 (TTEL, TADR) VALUES (:NEW.TTEL, :NEW.TADR);
END;
/
--现在就可以利用视图插入数据
INSERT INTO tab_view VALUES(106,'ljq','13886681288','beijing');
--查询
SELECT * FROM tab_view;
SELECT * FROM tab1;
SELECT * FROM tab2;
6)、创建触发器，比较emp表中更新的工资


--创建触发器
set serveroutput on;
CREATE OR REPLACE TRIGGER SAL_EMP
 BEFORE UPDATE ON EMP
 FOR EACH ROW
BEGIN
 IF :OLD.SAL > :NEW.SAL THEN
  DBMS_OUTPUT.PUT_LINE('工资减少');
 ELSIF :OLD.SAL < :NEW.SAL THEN
  DBMS_OUTPUT.PUT_LINE('工资增加');
 ELSE
  DBMS_OUTPUT.PUT_LINE('工资未作任何变动');
 END IF;
 DBMS_OUTPUT.PUT_LINE('更新前工资 ：' || :OLD.SAL);
 DBMS_OUTPUT.PUT_LINE('更新后工资 ：' || :NEW.SAL);
END;
/
--执行UPDATE查看效果
UPDATE emp SET sal = 3000 WHERE empno = '7788';
运行结果如下：
```
## over 函数
```
create table s_score
(   s_id number(6)
   ,score number(4,2)
);
insert into s_score values(001,98);
insert into s_score values(002,66.5);
insert into s_score values(003,99);
insert into s_score values(004,98);
insert into s_score values(005,98);
insert into s_score values(006,80);

select
    s_id 
   ,score
   ,rank() over(order by score desc) rank               --按照成绩排名，纯排名
   ,dense_rank() over(order by score desc) dense_rank   --按照成绩排名，相同成绩排名一致
   ,row_number() over(order by score desc) row_number   --按照成绩依次排名
   ,ntile(3) over (order by score desc) group_s         --按照分数划分成绩梯队
from s_score;
复制代码

```
## oracle 循环
```
BEGIN
for i in 10000074..10000075 loop
insert into s_jxc_jewelryprocess (id,...)
values (i,...);
end loop;
end;
```
## oracle trunc函数
```
作用与格式化数据
trunc('数据',‘所要保留小数’)
    select trunc(123.567,2) from dual;--123.56,将小数点右边指定位数后面的截去;
```
## wm_concat函数 把多行结果合并为一行显示
```
select wm_concat(字段) from dual;
```
## oracle round 四舍五入函数有两个参数和单一参数两种用法
>* 两个参数 round(m,d) 其中m为待四舍五入的数值，d为计算精度也就是四舍五入时保留的小数，d还可以取负值这时表示在整数位置四舍五入 -1 表示对数值进行十位以内的四舍五入 -2 表示对数值进行百位的四舍五入
>* 单一参数round(m) m看做待进行四舍五入的数值，精度为0 可以看做 round(m,0)
```
select round(250.14,-2) from dual -- 结果300
select round(45.14,-1) from dual -- 结果 50
   select round(123.567,2) from dual;--四舍五入保留两位小数
```
## oralce 随机排序


```sql
select '' from dual order by dbms_random.value();
--随机函数对每一行都会执行随机运行函数，会造成全表扫描
```
## oracle 随机数
>* select  dbms_random.value   from dual
>* select  dbms_random.value(low,high)   from dual 用来返回一个大于low小于high的数字
## between 范围比较函数
>* 作为范围比较条件 DBMS(数据库操作系统对 between ... and ... 进行了查询优化使用他进行范围查询将会得到比其他方式更好的性能，因此在范围查询时 应该优先使用 between and )
```sql
where 字段名 between 左范围值（大于等于） and 右范围值（小于等于）
```
## having 
>* 在having语句中不能包含未分组的列名

## 结果集过滤函数
```sql
select * from (
 select   row_number() over(order by brankid asc) row_num,
   a.* from s_ErpAndBrankData a
) erp where  erp.row_num>=3 and erp.row_num<=1000
```
## concat 字段拼接函数
>* concat会将非字符串类型转换为字符串类型
```sql
select concat('123',concat('123','abc')) from dual
```
## oracle 不允许不带from 子句的select语句
>* select * from dual;
>* dual 系统表
## oracle distinct 是对整个结果集进行数据重复抑制的
>* 下面的sql 是对查出来 name和age都相同的行进行去重 不是单一的 认为如果name这一列有相同就去重
```sql
select distinct name,age from t_person;
```
## oracle 计算字符串长度函数 length
```sql
select length('name') from dual;
```
## oracle 取字符串子串函数 substr(主字符串,子串的起始位置[下标0和1都一样]，要截取的长度)
```sql
select substr('name',1,2) from dual;-- 结果 na
select substr('name',0,2) from dual;-- 结果 na
```
## abs(表达式) 绝对值
```sql
select abs(-5) from dual; --结果5
```
## ceil(表达式)
>* 天花板函数向上取整
```sql
select ceil(2.59) from dual;
```
## floor(表达式)
>* 向下取整
```sql
select floor(2.25) from dual;
```
## sign 求符号
>* sign(表达式) 用来返回一个数值的符号 如果数值大于0 返回1 小于0 返回-1 等于0 返回0
```sql
select sign(-0.5) from dual;
```
## 整数余数
>* 返回两个数整除后的余数
```sql
select mod(5,2) from dual;
```
## 字符串转小写 lower(表达式)
```sql
select lower('ABC') from dual;
```
## 字符串转大写 upper(表达式)
```sql
select upper('abc') from dual;
```
## 截去字符串左侧空格ltrim(表达式)
```sql
select ltrim('   a') from dual;
```
## 截去字符串右侧空格rtrim(表达式)
```sql
select rtrim('a    ') from dual;
```
## 截去字符串两侧空格trim(表达式)
```sql
select trim('    a   ') from dual;
```
## 填充函数 rpad(char1,n,[,char2]) lpad(char1,n,[,char2])
>* 若第三个参数不填则默认按照空格填充
```sql
select rpad('abc',5,'*') from dual; --结果 abc**
select lpad('abc',5,'*') from dual; --结果 **abc
```
## 返回当月最后一天的日期
```sql
select last_day(to_date('2019-04-15','yyyy-mm-dd')) from dual; --结果返回2019/4/30
```
## 计算日期之间相差天数
```sql
select datediff(date1,date2) from dual; --计算date1和date2之间相差的天数
```
## 计算最大值最小值
```sql
--最大值
select greatest(2,4,5,9,100) from dual;
--最小值
select least(100,200,3.5,5,3.51) from dual;
```
## 辅助函数
```sql
select user from dual; --查看系统当前登录的用户名
select userenv('isdba') from dual;--返回当前登系统的角色是否为DBA
select userenv('language') from dual;--返回当前用户使用的字符集 格式为 语言.字符集
select userenv('terminal') from dual;--返回当前用户的操作系统表示
select userenv('sessionid') from dual;--返回当前登系统用户的回话标识
select userenv('entryid') from dual;-- 返回当前用户的认证标识
select userenv('lang') from dual;--返回当前用户使用的语言
select userenv('instance') from dual;--返回当前实例标识
```
## not 语法
>* select * from dual where not (1=1) 效果等同与 1<>1

## oracle中可以直接用 ‘+’来进行日期的加减法 其计算单位为天
```sql
select to_date('2019-04-08','yyyy-mm-dd')+2/24 from dual
```
## 计算子字符串的位置 instr(主字符串，子字符串)
>* 如果子字符串存在与主字符串中则返回第一次出现在主字符串中的位置，如果不存在则返回0
```sql
select instr('abc','a') from dual;
```
## 字符串替换
>* 可以删除字符串中间的空格
```sql
select replace('aababcc','aa','c') from dual; --结果cbabcc
```
## ADD_MONTHS(date,number)
>* 其中参数date为待计算的日期，参数number为要增加的月份数，如果number为负数则表
示进行日期的减运算
```sql
select Add_months(to_date('2019-04-09','yyyy-mm-dd'),2)-1 from dual
```
## to_char(date,format) 格式化日期
|占位符|说明
|:|:|
|YEAR |年份（英文拼写），比如NINETEEN NINETY-EIGHTYYYY 4位年份，比如1998
|YYY |年份后3位，比如998
|YY |年份后2位，比如98
|Y |年份后1位，比如8
|IYYY |符合ISO标准的4位年份，比如1998
|IYY |符合ISO标准的年份后3位，比如998
|IY |符合ISO标准的年份后2位，比如98
|I |符合ISO标准的年份后1位，比如8
|Q |以整数表示的季度，比如1
|MM |月份的名称，比如2月
|MONTH |月份的名称，补足9个字符
|RM |罗马表示法的月份，比如VIII
|WW |日期属于当年的第几周，比如30
|W |日期属于当月的第几周，比如2
|IW |日期属于当年的第几周（按照ISO标准），比如30
|D |日期属于周几，以整数表示，返回值范围为1至7
|DAY| 日期属于周几，以名字的形式表示，比如星期五
|DD |日期属于当月的第几天，比如2
|DDD |日期属于当年的第几天，比如168
|DY |日期属于周几，以名字的形式表示，比如星期五
|HH |小时部分（12小时制）
|HH12| 小时部分（12小时制）
|HH24 |小时部分（24小时制）
|MI |分钟部分
|SS |秒部分
|SSSSS| 自从午夜开始的秒数
## HEXTORAW()、RAWTOHEX()
> * HEXTORAW()用于将十六进制格式的数据转换为原始值，而RAWTOHEX()函数用来将原始
值转换为十六进制格式的数据
## TO_MULTI_BYTE()、TO_SINGLE_BYTE()
> * TO_MULTI_BYTE()函数用于将字符串中的半角字符转换为全角字符，而TO_SINGLE_BYTE()
函数则用来将字符串中的全角字符转换为半角字符
## COALESCE ( expression,value1,value2……,valuen) 为逻辑函数【判断是否为空并且填充】
>* 如果expression字段不为空返回expression 如果为空 判断value1是否为空不为空则返回 。。。
## case
>* 第一种用法
```sql
CASE expression
WHEN value1 THEN returnvalue1
WHEN value2 THEN returnvalue2
WHEN value3 THEN returnvalue3
……
ELSE defaultreturnvalue
END
```
>* 第二种用法
```sql
SELECT
FName,
FWeight,
(CASE
WHEN FWeight<40 THEN 'thin'
WHEN FWeight>50 THEN 'fat'
ELSE 'ok'
END) as isnormal
FROM T_Person
```
## 使用触发器解决新增数据时 主键的方案
```sql
CREATE OR REPLACE TRIGGER trigger_personIdAutoInc
BEFORE INSERT ON T_Person
FOR EACH ROW
DECLARE
BEGIN
SELECT seq_PersonId.NEXTVAL INTO:NEW.FID FROM DUAL;
END trigger_personIdAutoInc;
```
## 开窗函数在oracle中被称为分析函数 【2003年被iso制定为sql标准  】
>* COUNT(*) OVER(PARTITION BY FCITY) 在函数后面加上over 表示这个函数不再是它本身而是一个开窗函数
>* 开窗函数对于查询结果的每一行都返回所有符合条件的行的条数 over()括号中还经常添加选项，用以改变进行聚合运算的窗口范围，如果over关键字后的括号中的选项为空，则开窗函数会对结果集中的所有行进行聚合运算
>* over(PARTITION BY 字段) 表示按照字段分区进行聚合计算 
```sql
SELECT FName,FCITY, FAGE, FSalary,
COUNT(*) OVER(PARTITION BY FCITY),
COUNT(*) OVER(PARTITION BY FAGE)
FROM T_Person
```
>* over(ORDER BY 字段名 RANGE|ROWS BETWEEN 边界规则1 AND 边界规则2)
RANGE表示按照值的范围进行范围的定义，而ROWS表示按照行的范围进行范围的定义

|边界规则 |  | |
|:|:|
|可取值|说明|示例|
|CURRENT ROW| 当前行|
|N PRECEDING |前N行 |2 PRECEDING
|UNBOUNDED PRECEDING| 一直到第一条记录
|N FOLLOWING |后N行 |2 FOLLOWING
|UNBOUNDED FOLLOWING| 一直到最后一条记录
>* 这里的开窗函数“SUM(FSalary) OVER(ORDER BY FSalary ROWS BETWEEN
UNBOUNDED PRECEDING AND CURRENT ROW)”表示按照FSalary进行排序，然后计算从第
一行（UNBOUNDED PRECEDING）到当前行（CURRENT ROW）的和，这样的计算结果就是按照
工资进行排序的工资值的累积和
```sql
SELECT FName, FSalary,
SUM(FSalary) OVER(ORDER BY FSalary ROWS BETWEEN UNBOUNDED PRECEDING
AND CURRENT ROW)
FROM T_Person;
```
>* 按照 “RANGE”进行
范围定位,则按照一定范围内所有与当前值相同的累积和
```sql
SELECT FName, FSalary,
SUM(FSalary) OVER(ORDER BY FSalary ROWS BETWEEN UNBOUNDED PRECEDING
AND CURRENT ROW)
FROM T_Person;
```
>* 然后计算从当前行前两行（2
PRECEDING）到当前行后两行（2 FOLLOWING）的工资和
```sql
SELECT FName, FSalary,
SUM(FSalary) OVER(ORDER BY FSalary ROWS BETWEEN 2 PRECEDING AND 2
FOLLOWING)
FROM T_Person;
```
>* 表示按照FSalary进行排序，然后计算从当前行后一行（1
FOLLOWING）到后三行（3 FOLLOWING）的工资和。注意最后一行没有后续行，其计算结果为
空值NULL而非0。
```sql
SELECT FName, FSalary,
SUM(FSalary) OVER(ORDER BY FSalary ROWS BETWEEN 1 FOLLOWING AND 3
FOLLOWING)
FROM T_Person;
```
>* “SUM(FSalary) OVER(ORDER BY FName RANGE BETWEEN UNBOUNDED
PRECEDING AND CURRENT ROW)”等价于“SUM(FSalary) OVER(ORDER BY FName)”
```sql
SELECT FName, FSalary,
SUM(FSalary) OVER(ORDER BY FName RANGE BETWEEN UNBOUNDED PRECEDING AND
CURRENT ROW)
FROM T_Person;
```
>*  WITH
```sql
WITH person_tom AS
(
SELECT * FROM T_Person
WHERE FName='TOM'
)
SELECT * FROM T_Person
WHERE FAge=person_tom.FAge
OR FSalary=person_tom.FSalary

--with 别名
WITH person_tom(F1,F2,F3) AS
(
SELECT FAge,FName,FSalary FROM T_Person
WHERE FName='TOM'
)
SELECT * FROM T_Person
WHERE FAge=person_tom.F1
OR FSalary=person_tom.F3
```
## 注意
>* 所有的DBMS都支持用单引号包围的形式定义的子字符串 所以建议都应该使用单引号的形式
>* to_char()用来将时间日期函数或者数值类型的数据转化为字符串
>* to_date()函数用来将字符串转化为时间类型的函数
>* to_number()函数用来将字符串类型转化为数值类型
## 索引与约束
### 索引
```sql
create index 索引名称 on 表名 (列名)--创建
drop index 索引名 --oracle 不要求指定表名 指定索引名称即可
```
>* DBMS一般采用自上而下的的顺序解析where语句 有可能过滤醉的的数据条件应放在前面
>* select * 避免使用 DBMS解析中会将* 解析为所有的列名这意味着消耗更多的时间
>* 尽量将多余的sql压缩到一条sql中
```xml
每次执行sql时都要建立网络链接权限校验进行sql语句的查询优化，发送执行结果这个过程是耗时的
场景 一般用于修改删除增加
```
>* 避免在索引列上使用计算
```sql
select * from dual d where d.age*12 >2500 --导致全表扫描
sleect * from dual d wehre d.age> 2500/12 --避免在索引列上使用计算
```
>* 不能在索引列上使用函数 函数也是一种计算会造成全表扫描
>*  union all 替换 union 保证两个结果集不会有重复数据的情况下 使用union all替换union
>* 避免索引列进行隐式转化 age 为string类型  where 子句 age = 1 导致不走索引  [age= '1'(正确写法)]
>* 防止检索范围过宽 如果DBMS优化器认为检索范围过宽那么将会放弃索引查找而是用全表扫描
```xml
使用 is not null 或者不等于判断，可能会造成优化器假设匹配的记录数太多放弃使用索引
使用 like的时候 a%将会使用索引，而a%c和%c则会使用全表扫描   
```
### 约束
>* 增加唯一约束
```sql
--添加唯一约束
alter table  表名 add constraint 唯一约束名 unique(字段一....)
--删除唯一约束
alert table 表名 drop constraint 
--条件约束
create table t_person(fname varchar2(20),fnumber varchar(20) ,constraint ck_1 check(fname <> fnumber))

```
## null 的概述
>* DBMS中空值 表示一个字段的值为‘未知'
>* null值与比较用算符 比较运算符会把空值过滤掉
>* 如果空值出现在任何计算字段中，那么计算结果永远是空
>* 如果空值出现在任何和字符串相关计算的字段中，那么计算结果永远是空
>* 如果空值出现在普通函数中，那么计算结果永远是空
>* 如果空值出现在聚合函数中，那么计算结果会忽略空值
## oracle 递归
>* 运算符prior 被放置与等号前后的位置，决定查询时的顺序
如果prior 被置于connect by 子句中等号前面，则从根节点到叶节点进行检索，是一种自顶向下的顺序
如果prior 被置于connect by 子句中等号后面，则从叶节点到根节点进行检索，是一种自底向上的顺序
>* sys_connect_by_path展现层级关系
```sql
select * from s_base_alloy a connect by  prior a.al_id=  a.al_fatherid --父级向子级检索
select * from s_base_alloy a connect by  a.al_id= prior a.al_fatherid  --子级向父级检索
select sys_connect_by_path(a.al_name,'>'),
a.al_id,a.al_fatherid from s_base_alloy a connect by prior a.al_id=  a.al_fatherid 
```
## 实用sql
```sql
select to_char(trunc(sysdate,'d')+1,'yyyy-mm-dd') from dual; -- 获取当前系统时间所在星期一是几号
```
```sql
create table t_person as select * from t_person1 ;--复制t_person1中的数据到t_person中
```
```sql
select trunc(to_date('2019-05-05','yyyy-mm-dd'),'mm') from dual;--所选日期当月第一天
--trunc()可以用来将日期截取到任意精度 因此使用trunc()函数截取到月就可以获得当月的第一天 
select last_day(to_date('2019-05-05','yyyy-mm-dd')) from dual;--所选日期当月最后一天
```
