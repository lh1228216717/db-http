# 基于ORACLE存储过程的创建及调用

+ 存储过程相当于java语言中的方法
+ PLSQL是一种过程语言sql，可以使sql具有程序判断的逻辑
+ PLSQL不区分大小写 

## PLSQL程序结构
+ PLSQL可以分为三个部分，变量声明部分、可执行部分、异常处理部分
```plsql
declare 
  -- [变量声明部分，声明变量、游标]
  i integer;
begin
  -- [可执行部分]
  
  --[异常处理]
  
end;
```
+ 其中 declare部分用来声明变量和游标（结果集类型变量），如果没有声明，可以省略declare部分

##  PLSQL 之 Hello world
+ dbms_output.put_line('hello world'); 向控制台输出 hello world
+ 在plsql中，字符、字符串均使用 单引号包裹

## 变量
+ PLSQL编程中常见变量分两大类
+ 1 普通数据类型[char,varchar2,date,number,boolean,long]
+ 2 特殊变量类型[引用型变量,记录型变量]
```
引用型：变量类型取决于表中字段类型
记录型：变量接收的值为一整条记录的值
```
### 声明变量的方式
+ 变量名 变量类型(变量长度) 列如： v_name varchar2(20);

#### 普通变量赋值方式
+ 1 直接赋值语句 := 例如 v_name := '张三'
+ 2 语句赋值，使用 select...into...赋值 例如 select 值 into 变量

```plsql
declare 
  v_name varchar2(20) := '張三';
  v_age number;
  v_addr varchar2(200);
begin
 v_age := 18;
 select '上海' into v_addr from dual; --语句赋值
 dbms_output.put_line('name:'||v_name||'年齡'||v_age||'地址'||v_addr);
end;
```

###  引用型变量
+ 引用变量的类型和长度取决于表字段的类型和长度
+ 变量名 表名.列名%TYPE指定类型的长度和类型 例如 v_name table.colunm%TYPE;
+ 示例

```plsql
declare 
  v_name t_alloy.alname%type;
  v_code t_alloy.alcode%type;

begin
select t.alname,t.alcode into v_name,v_code from t_alloy t where t.alid = 163;
 dbms_output.put_line('v_name:'||v_name||'v_code'||v_code);
end;
```

### 记录型变量
+ 接收表中的一整行记录，相当于java中的一个对象
+ 变量名称 表名%ROWTYPE  例如 v_alloy t_alloy%ROWTYPE
+ 示例

```plsql
declare 
  v_alloy t_alloy%ROWTYPE;

begin
select * into v_alloy from t_alloy t where t.alid = 163;
 dbms_output.put_line('v_name:'||v_alloy.alname||'v_code'||v_alloy.alcode);
end;
```
## 流程控制
### 条件分支
+ 示例

```plsql
declare
  v_count number;

begin
  select count(1) into v_count from t_alloy;
  if v_count > 100 then
    dbms_output.put_line('成色表数据超过100条为' || v_count);
  elsif v_count > 20 then
    dbms_output.put_line('成色表数据为20-100条 为' || v_count);
  else
    dbms_output.put_line('成色表数据不超过20条为' || v_count);
  end if;

end;

```
+ 上述需注意 ELSIF 与java编程中ELSE IF少一个 E 且 连贯为 ESLIF
+ IF ENDIF; 注意 ENDIF 后面需要加 分号;

### 循环
+ LOOP 循环示例 

```plsql
declare
  v_count number := 1;

begin

  loop
    exit when v_count > 40;
    dbms_output.put_line(v_count);
    v_count := v_count + 1;
  end loop;

end;
```
+ FOR LOOP 循环实例

```plsql
declare
i number :=3;
begin
 for y in 0..i loop
   dbms_output.put_line(y);
   end loop;
end;

```
+ WHILE LOOP 循环

```plsql
declare
  i number := 0;
begin
  while i <= 10 loop
    dbms_output.put_line(i);
    i := i + 1;
  end loop;
end;

```
## 游标
+ 游标就是用于临时存储一个查询返回的多行数据，通过便利游标，可以逐行访问处理该结果集的行数
+ 游标使用方式 声明 、打开、读取、关闭

### 语法
#### 游标声明
+ CURSOR 游标名[(参数列表)] IS 查询语句；

#### 游标打开
+ OPEN 游标名;

#### 游标的取值
+ FETCH 游标名 INTO 便利列表;

#### 游标的关闭
+ CLOSE 游标名;

### 游标属性
属性 | 返回值 |  说明 
-|-|-
 %ROWCOUNT	 | 返回整型		|获得FETCH返回的行数
 %FOUND              |  返回boolean    | 最近的FETCH语句返回一行为TRUE，否则为FALSE
 $NOTFOUND        | 返回boolean     |与%FOUND相反
 %ISOPEN	     |  返回boolean    |游标打开返回TRUE，否则返回FALSE	

### 无参数游标取值示例
```plsql
declare
  --定义游标
  CURSOR V_ALLOY IS
    SELECT T.ALNAME, T.ALCODE FROM T_ALLOY T;
  --定义引用类型变量 接收游标行数据
  v_name t_alloy.alname%TYPE;
  v_code t_alloy.alcode%TYPE;
begin

  --打开游标
  OPEN V_ALLOY;
  --开始循环
  loop
    --游标取值
    FETCH V_ALLOY
      INTO v_name, v_code;
    --结束条件
    exit when V_ALLOY%notfound;
  
    DBMS_OUTPUT.put_line(v_name || v_code);
  end loop;
  --结束循环
  --关闭游标
  CLOSE V_ALLOY;

end;
```

### 带参数的游标取值示例
```plsql
declare
  --定义游标
  CURSOR V_ALLOY(acode t_alloy.alcode%TYPE) IS
    SELECT T.ALNAME, T.ALCODE FROM T_ALLOY T WHERE T.ALCODE = acode;
  --定义引用类型变量 接收游标行数据
  v_name t_alloy.alname%TYPE;
  v_code t_alloy.alcode%TYPE;
begin

  --打开游标
  OPEN V_ALLOY('011011');
  --开始循环
  loop
    --游标取值
    FETCH V_ALLOY
      INTO v_name, v_code;
    --结束条件
    exit when V_ALLOY%notfound;
  
    DBMS_OUTPUT.put_line(v_name || v_code);
  end loop;
  --结束循环
  --关闭游标
  CLOSE V_ALLOY;

end;

```

## 存储过程
+ PLSQL是将一个个PLSQL的业务处过程存储起来进行复用，这些被存储起来的PLSQL程序称之为存储过程

```plsql
 CREATE OR REPLACE PROCEDURE 过程名称[(参数列表)] IS
 BEGIN
 
 END [过程名称]
```
###  根据参数类型划分存储过程

#### 不带参数存储过程
+ 实例

```plsql
create or replace procedure p_hello is
-- 变量声明区域
begin
  dbms_output.put_line('hello world');
  
end p_hello;
```
+ 存储过程中不需要 declare ，declare用在语句块中
+ IS 和 AS 可以互用
+ 调用

```plsql
--打开 TEST windows

begin
p_hello;
  
end;
```

#### 带输入参数
+ 实例

```plsql
create or replace procedure p_queryAlloyCode(code in t_alloy.alcode%TYPE) as
  -- 变量声明区域
  v_name      t_alloy.alname%type;
  v_printName t_alloy.alprintname%type;
begin
  select a.alname, a.alprintname
    into v_name, v_printName
    from t_alloy a
   where a.alcode = code;
  dbms_output.put_line(v_name || '---' || v_printName);
end;

```
+ 注意输入参数的存储过程声明方式  create or replace procedure 过程名(参数名 in 参数类型) as begin end;


#### 带输入输出参数
+ 实例

```plsql
create or replace procedure p_queryAlloyCode(code   in t_alloy.alcode%TYPE,
                                             v_name out t_alloy.alname%type) as
  -- 变量声明区域

begin
  select a.alname into v_name from t_alloy a where a.alcode = code;

end;

```
+ 调用

```plsql
declare
v_name t_alloy.alname%type;
begin
  -- Call the procedure
  p_queryalloycode('011011',v_name);
  dbms_output.put_line(v_name);
end;

```
## 存储函数
### 声明
+ 将业务逻辑 存储起来 并返回
+ create or replace function 函数名称(参数名称 参数类型) return 返回值类型 is begin end;
+ 参数类型默认输入类型可以省略
+ 实例 

```plsql
create or replace function getAlloyCode(code  varchar2) return t_alloy%rowtype is

  v_alloy t_alloy%rowtype;
  
begin
   select * into v_alloy from t_alloy a where a.alcode = code;
  return v_alloy;
end ;
```
### 调用
+ 调用
+ 函数可以在SQL语句里直接调用

```plsql
declare
  -- Non-scalar parameters require additional processing 
  alloy t_alloy%rowtype;
begin
  -- Call the function
  alloy := getalloycode('011011');
  dbms_output.put_line(alloy.alname);
end;

```
### 存过过程与存储函数的区别
+ SQL可以 直接调用存储函数，不能直接调用存储过程
## 扩展内容
### 游标和FOR LOOP 扩展 
+ 实例

```plsql
declare
  cursor v_alloy is
    select * from t_alloy;
begin
  for alloy in v_alloy loop
    dbms_output.put_line(alloy.alcode || '----' || alloy.alname);
  end loop;
end;

```

### 系统引用类型游标使用及便利
```plsql
declare
  v_alloy sys_refcursor;
  alloy   t_alloy%rowtype;
begin
  open v_alloy for
    select * from t_alloy;

  loop
    fetch v_alloy
      into alloy;
    exit when v_alloy%notfound;
    dbms_output.put_line(alloy.alname);
  end loop;

  close v_alloy;
end;

```