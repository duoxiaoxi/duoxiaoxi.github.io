# Oracle





## 函数

> n: 代表输入一个数字
>
> d: 代表输入一个日期
>
> s: 代表输入一个字符串
>
> n:xxx: xxx为对这个参数的描述

### 数值型函数

`abs(n)` `mod(n, n)` `sign(n)` `ceil(n)` `floor(n)` `round(n, n)` `trunc(n, n)`

`sin(n)` `cos(n)` `tan(n)` `sqrt(n)` `exp(n)` `log(n, n)` `power(n, n)`

### 字符型函数

`chr(n)` `ascii(s)` `upper(s)` `lower(s)` `initcap(s)`

`length(s)` 

`substr(s, n:pos, n:len)`

`concat(s, s)`

`instr(s, s:sub, n:pos, n:occ)`

`replace(s, s:sub, s:rel)`

`lpad(s, n, s:pad)` `rpad(s, n, s:pad)`

`trim([LEADING|TRAILING|BOTH], [SUB from], s)`

`ltrim(s, s:del_set)` `rtrim(s, s:del_set)`







## PL/SQL

### HelloWorld

```plsql
begin
  dbms_output.put_line('hello, world');  
end;
```



### 块

`PL/SQL程序`的基本单位是`块`, `块`分为了三个部分: `声明部分`, `执行部分`, `异常处理部分`. 语法如下:

```plsql
[declare]
-- 声明部分
begin
-- 执行部分
[exception]
-- 异常处理部分
end;
```

声明部分是可以省略的... 也就是没有声明的情况, 异常处理也是可以省略的... 最简单的情况也就是我们的==HelloWorld==小程序了... 接下来我们写一个完整的程序:

```plsql
-- 完整的程序
declare      -- 声明部分
  v_name varchar2(10);		-- PLSQL程序范围内的每一条子语句都应该以;结尾
begin        -- 执行部分
  select ename
    into v_name				 -- 将查询到的结果存储到某个变量当中 很明显要单行单列才行
    from emp 
    where mgr is null;
  dbms_output.put_line('没有上司的人是: ' || v_name);
exception    -- 异常处理部分
  when no_data_found then
    dbms_output.put_line('没有找到记录');  
  when too_many_rows then
    dbms_output.put_line('对应数据过多');
end;         -- 代表结束
```

这些代码一看就能懂 需要注意的是 ==通过SELECT语句向变量中存储值的话 要求结果集必须单行的 也就是只能有一条记录... 这是无论何时都应该满足的==



### 注释

* 单行注释: `-- xxx`
* 多行注释: `/* xxx */`



### 变量

**声明的语法**:

```plsql
declare
变量名 [constant] 变量类型 [not null {:= | default} expression]
begin ... end;
```

**标量类型变量**:

```plsql
-- 标量类型变量的声明
declare
  v_empno     number(4);
  v_ename     varchar2(10)       := 'KING';       -- 设置默认值方法 1
  v_job       varchar2(9)        default 'SSS';   -- 设置默认值方法 2
  v_mgr       number(4)     not null := 1234;     -- 有 not null 就必须设置默认值了
  v_hiredate  date          not null default '24-1月-99';
  v_sal       number(7,2);
  v_comm      emp.comm%type;                      -- 引用表字段的类型
  v_deptno    dept.deptno%type;
begin
  select ename, job, hiredate
    into v_ename, v_job, v_hiredate
    from emp
    where mgr is null;
  dbms_output.put_line('ename = ' || v_ename);
  dbms_output.put_line('job = ' || v_job);
  dbms_output.put_line('hiredate = ' || v_hiredate);
end;
```

**记录类型**(就是若干字段构成一种类型):

```plsql
-- 记录类型的变量
declare
  -- 定义一种记录类型(几个字段的集合)
  type my_type is record (
    -- 括号里是若干标量的定义
    v_ename    emp.ename%type,
    v_job      emp.job%type,
    v_sal      emp.sal%type
  );
  -- 根据刚才定义的类型 去定义一个变量
  my_var1       my_type;
  -- %rowtype代表某张表的所有字段构成的类型 而不是几个字段
  -- 然后起内部的属性名称全是该表的字段名称...
  my_var2       emp%rowtype;
begin
  select ename, job, sal
    into my_var1
    from emp
    where ename = 'KING';
  select * into my_var2 from emp where mgr is null;
  dbms_output.put_line(my_var1.v_ename);
  dbms_output.put_line(my_var2.ename);
end;
```

**数组类型**:

```plsql
-- 关联数组类型的变量
declare
  -- 创建一种映射表 内容为字符串类型 下标为数字类型
  type my_map is table of emp.ename%type index by binary_integer;
  type my_map2 is table of emp%rowtype index by varchar2(32);
  my_var      my_map;
  my_var2     my_map2;
begin
  my_var(1) := 'KING1';
  my_var(2) := 'KING2';
  select * into my_var2('a') from emp where ename like '%KING%';
  select * into my_var2('b') from emp where ename like '%KING%';
  dbms_output.put_line(my_var(1));
  dbms_output.put_line(my_var2('b').ename);
end;
```





```plsql
-- 定义类型
type obj_name is record (prop1_name prop1_type, prop2_name prop2_type);
type arr_name is varray of value_type;
type map_name is table of value_type index by key_type;
-- 声明变量
var_name var_type := default_value;
```





```plsql
if condition then
  statements
elsif condition then
  statements
else
  statementes
end if;
```















