+++

[toc]

+++

# `SQL`语法

## 基本数据类型

### 域和属性

- 在数据库中，域是具有**相同数据类型**的**有意义**的**值的集合**，属性是**域的实例**
  对应于编程语言，可以不严谨地把域看作数据类型，把属性看作变量，把某一行中的某一属性值看作变量的取值
- 域的组成：
  - 相同数据类型：要求一个域必须基于一个**基本数据类型**上作约束
  - 有意义：用户可以自定义约束，限制取值范围，使属性的取值符合现实
- 域的第一要求(或者说域的第一范式)：域是**原子性**的，即属性的**每一个取值都不可分割**
  例如，如果某一个取值是元组或值列表，那就不符合域的第一范式，因为元组是可分的

### 数值类型

- 整型：各种数据库都支持不同大小的整型类型，如小整型、整型、大整型，但命名不同
- 定点数：`decimal(P, D)`或`numeric(P, D)`
  - 在`SQL`标准中它们有细微差别：`numeric`会严格保证**运算过程中精度不变**，而`decimal`在存储时会分配更多空间以便**运算时提高精度**，只在显示时收敛精度；各种数据库对它们的实现和标准也有差别
    但在实践中，可以认为没什么差别
  - `P`表示**总的有效位数**
  - `D`**大于`0`**时，表示**小数点后**的有效位数；**小于`0`**时，表示**小数点前**的有效位数；不能等于`0`
- 浮点数：比定点数**运算更快**，但会**损失精度**，适用于存储近似值
  各种数据库的命名不同

### 字符串类型

- 各种数据库的字符串类型命名一致
- `char(size)`：定长字符串，**存储长度**固定为`size`的字符串，无论值的长度是多少
- `varchar(size)`：可变长字符串，最大存储长度为`size`，实际存储长度和值的长度相等
- 字符串**数据**是用**单引号**括起的字符串
  **表名、列名、属性名**等**标识符**是用**双引号**括起的字符串(当属性名中存在空格时，需要用`""`括起)
  但应该遵循命名规范：用小写字母、不同单词用下划线连接

### 其它类型

- 文本类型：数据库通过在表外存储大文本，在表内存储对应的指针来实现文本类型，存储效率较低
  各种数据库命名不一致
- 布尔类型：
- 日期和时间：
  - `DATE`：日期类型
  - `TIME`：时间类型
  - 日期时间类型：各种数据库命名不同
  - `TIMESTAMP`：时间戳类型
  - 字面量：为符合规范，尽量使用标准格式的字面量`YYYY-MM-DD HH:MM:SS`或**显式转换**

### 类型转换

- 隐式类型转换：
- 显式类型转换：

### 不同数据库的实现

待查

## 表达式

### 表达式的本质

- 表达式就是一个函数，只不过是编程语言通过一些语法糖对表达式进行了简化
  **表达式的输入输出都是值**，输入可以是一组值，输出只能是标量值

- **原子表达式**<a id=expl_atomexpr></a>：指那些不能再分割的表达式，基于原子表达式可以通过运算符、标量值函数组合得到新的表达式

  - 字面量
  - 列引用：即某个表中某个属性的名称

- **`Sql`语句的输入输出都是表**，是多组值，一个表经过表达式，相当于输入一个列向量(每个元素是一组值)，表达式会对该列向量按元素求值，返回一个新的列向量(每个元素是一个标量值)
  例如：

  ```sql
  -- e 是一个元组
  1			-- f(e) = 1			1是一个整型字面量
  attr		-- f(e) = e.attr	attr是一个列引用
  attr > 1	-- f(e) = boolean(e.attr > 1)
  
  -- f(表) ~ f([e1, e2, ...]) ~ [f(e1), f(e2), ...]
  ```

### 数值表达式及部分函数

- 标量值：指单个不可分割的值
  绝大多数数据库对返回一个复合类型(如集合等)的支持很有限，这里讨论的大多是**返回标量值的表达式**
  所以可以将标量值看作一个数值表达式的返回值

- **值列表**：用`()`括住一系列标量值，表示由这些值组成的**集合**

- 基本算术运算：`+ - * / % ^`

- `CASE`：对输入值从上到下地进行逻辑判断，若成立则返回指定的标量值

  ```sql
  -- 较灵活的写法
  CASE
  	WHEN logic_expr1(input) THEN value1
  	WHEN ... THEN ...
  	ELSE default_value		-- 若不加则为ELSE null
  END
  
  -- 较偷懒的写法, 只支持等值比较
  case input
  	WHEN val1 THEN ret_val1
  	WHEN ... THEN ...
  	ELSE default_value
  END
  ```

- **`null`**：空值，表示一种不确定的状态
  若参与**运算则一定返回`null`**，参与**一般比较一定会返回`false`**，会**被聚集函数无视**
  只能通过`is null`和`is not null`判断是否为空

  - [空值处理](#more_nullfc)<a id='expl_nullfc'></a>：涉及空值的函数

- [聚集函数值](#more_gfunc)<a id='expl_gfunc'></a>：对**多行**进行**聚集**，**返回标量值**
  很多函数都只接受一行数据的输入，输出一个标量值
  而聚集函数**接受多行数据的输入**，仍输出一个标量值

### 逻辑表达式

- 连接逻辑表达式的运算符：`and、or、not`
- 连接数值表达式的运算符：
  - `is null`和`is not null`：若左值是/不是空则返回真
  - `in`和`not in`，前跟标量值，后跟值列表，若左值属于/不属于该值列表则返回真
  - `between left_val and right_val`：闭区间比较
- 连接数值表达式和子查询语句的函数：
  - `any()/some()`
  - `all()`

## 数据完整性约束

### 限制不可重复

- `primary key`：声明主键，主键能够**唯一标识元组**，天生包括了**非空约束**及**唯一性约束**
  为了性能，虽然关系是一个集合，但数据库不会时时刻刻保证没有重复，除非添加了主码约束
  添加后，若插入元组中的主键已经存在，将会报错

  ```sql
  -- 建表时声明 主键
  (1) <属性名> <属性类型> primary key		这样的主键只有一个属性
  (2) primary key (attr1, attr2, ...)		允许声明一组属性作为主键
  ```

- `unique`：声明唯一性约束，被约束的**属性值的集合**只能存在至多一次
  和主键不同，它**允许值为空，且允许有多个记录的该属性的值为空**

  ```sql
  -- 建表时 声明方式与主键类似, 但unique可声明多次, 被约束的属性组可以重叠、甚至重复
  (1) <属性名> <属性类型> unique
  (2) unique (attr1, attr2, ...)		这时它们会作为一个整体受到唯一性约束
  ```

### 限制输入范围

- `not null`：声明非空约束，**一个非空约束只能针对一个属性**，即只能用上述的第一种方式声明
  字面意义，若没有默认值约束，则插入元组时必须指定该属性的值，且不能为`null`

- `default(value)`：声明默认值约束，一个默认值约束只能针对一个属性，即只能用上述的第一种方式声明
  平时就相当于`default (null)`，指定后将在插入元组且未指定值时填充默认值
  声明`default`约束必须指定默认值`value`

- `check(logic_expr)`：声明检查约束，检查约束必须指定逻辑表达式，插入元组时会检查，若违反约束则会报错
  实际上检查约束无论用哪一种方法声明都无所谓，因为是通过表达式来约束属性的
  最好是使用第二种方式声明

### 外码

- `foreign key (<参照属性组>) refereces <被参照表>(<被参照属性组>)`：声明外键约束，其中被参照属性组必须是被参照表的主键
  声明后，`<参照属性组>`的值要么是被参照表中**已有的主键值**，要么为**`null`**
  外键的声明方式有点特别：

  ```sql
  (1) <属性名> <属性类型> references <关系名>(<属性名>)
  (2) foreign key <属性名> references <关系名>(<属性名>)
  ```

  需要注意，如果**一个表被其它表参照，那么它是不能被删除的**

### 约束对象

- 约束实际上是一个数据库对象，是可以起名的(默认是系统起名)，起名也有两种方式(都是在键前添加`constraint <约束名>`)：

  ```sql
  (1) <属性名> <属性类型> constraint <约束名> references tbl_name(attr_name)
  	<属性名> <属性类型> constraint <约束名> unique
  (2) constraint <约束名> foreign key () references ..()
  	constraint <约束名> primary key ()
  ```

  实践中，最好自己起约束名

- 最后对比一下两种声明方式：

  - 第一种声明方式**针对一个属性**，可以用空格分隔并**声明多个约束**
    可以看作是在一个属性后列出其约束
  - 第二种声明方式针对含多个属性的约束，一条只能**声明一个约束**，但能**约束一个属性组**

## `DDL`(数据定义语言)

### 数据库对象

- `database`：数据库
- `schema`：模式
- `table`：表
- `view`：视图
- `function`：函数
- `constraint`：约束

### 通用语法

- 通用语法是指，所有数据库对象创建/更改/删除时都需要用到的主语句，例如`create、drop`等

- `create`：创建某种数据库对象

  ```sql
  create <type> <obj_name>
  ```

- `drop`：删除某种数据库对象
  ```sql
  drop <type> <obj_name>
  ```

- `alter`：更新某种数据库对象的结构
  在实践中，`alter`尽量作为最后手段，一切结构尽量在创建时定义完整

  ```sql
  alter <type> <obj_name>
  ```

### 关于表

- 创建表需要在表名后附加元组，表示每一列的属性名、属性类型，以及表的约束
  创建表时添加约束的语法已在前面详细讲过

  ```sql
  create table <table_name> (
      <attr_name> <attr_type> <attr_constraint>,
      <attr_constraint>
  );
  ```

- 更新表的结构：
  ```sql
  alter table <table_name>	-- 后跟子句:
  add column <column_name>					-- 添加列
  drop column <column_name> 				-- 删除列
  -- 重命名列
  add constraint <constraint_name> (<attrs>)	-- 添加约束, 定义方式同上
  drop constraint <constraint_name>			-- 删除约束
  
  ```

## 查询语句

### `FROM`

- `FROM`子句用于**添加数据源**，后**跟表或视图**，有两种添加数据源的方式

- 笛卡尔积：直接用`','`连接表，`FROM`会将它们进行笛卡尔积后交给下一个子句
  这样的`FROM`通常配合`WHERE`子句进行限制

- 内连接：用**`JOIN ON`**子句连接数据源，这样的子句必须跟`ON`及一个逻辑表达式
  会将两表进行**笛卡尔积**并**只保留使该逻辑表达式为真的行**，再交给下一个子句

  ```sql
  FROM t1 JOIN t2 ON <logic_expr>
  ```

  - 自然连接：部分数据库支持，等价于`ON`后跟一系列**同名属性的等值比较**的内连接
    因此不需要跟`ON`

    ```sql
    FROM t1 NATURAL JOIN t2
    ```

  - 交叉连接：部分数据库的语法，等价于直接进行笛卡尔积
    ```sql
    FROM t1 CROSS JOIN t2
    ```

- 外连接：在**内连接的基础**上，保留那些不满足逻辑表达式的行，连接后未知的值设为`NULL`
  分为`LEFT JOIN、RIGHT JOIN、FULL JOIN`
  分别是：保留左表、保留右表、保留两边的表
  使用外连接要注意，如果要进行多表连接，那么一个内连接很有可能丢弃掉外连接保留下来的行(因为部分属性是`NULL`，很容易经过逻辑表达式后返回假)

### `WHERE`

- `WHERE`子句用于进一步限制由`FROM`得来的查询结果，后**跟逻辑表达式**
  逻辑表达式的细节不再赘述

### `GROUP BY`

- `GROUP BY`子句用于**根据某些属性**进行**分组**，需要和`SELECT`等语句区分开，`GROUP BY`后**跟一系列属性名**(列引用)，而**不是值列表**
  后跟多个属性时，`GROUP BY`会根据**元组的等值比较**规则来分组
  分组后由于**域的原子性**这个第一范式，只能选择**分组依据**或者**聚集函数值**作为[**原子表达式**](#expl_atomexpr)
- 很多时候用户会在使用`GROUP BY`后`SELECT`那些不能作为原子表达式的属性，并误以为是`SELECT`的问题，实际上是`GROUP BY`后能作为原子表达式的内容变为了分组依据或聚集函数值

### `HAVING`

- `HAVING`子句用于**过滤分组**，后**跟逻辑表达式**
  前面说了，`WHERE`只能用于限制由`FROM`得来的查询结果
  而又有分组后再根据聚集函数值来限制结果的需求，因此有了`HAVING`子句
  一旦**使用`HAVING`**子句则**默认使用了`GROUP BY`**(没有`GROUP BY`时将整个表视为一组)，因此如`GROUP BY`中说的那样，**只能用分组依据或聚集函数值作为原子表达式**

### `SELECT`

- `SELECT`是整个查询语句的核心，后**跟一个数值表达式列表**，也就是值列表，`SELECT`会对输入表的**每一行计算值列表的结果**，并输出查询结果呈现给用户
- `SELECT`的返回值是一个表，不是视图，不要因视图的创建语法而误判
- `DISTINCT`关键字：只能在`SELECT`后添加，用于对得到的结果**去重**
  去重规则：将整一行视为元组，若存在两行完全一致，则去重
  可配合`count()`统计不重复的值的数量

### `ORDER BY`

- `ORDER BY`子句用于**排序**，后**跟值列表**，`ORDER BY`会像`SELECT`那样对`SELECT`输出的表的每一行计算值列表的值，并根据结果排序
  当值列表中有多个值时，排序规则是进行**元组的比较**，即从左到右，若前一个值相等则比较后一个值，从此类推下去

- 自定义排序规则：因为跟的是数值表达式列表，可以**配合`CASE`**表达式来**自定义排序**规则

- 默认下，会对计算结果进行**升序排序**，即**`ASC`**
  如要**降序排序**，需要在表达式后添加**`DESC`**，会单独对该表达式值进行降序排序

- `NULLS FIRST`和`NULLS LAST`：部分数据库支持，会将计算结果为`NULL`的行放在最前/最后
  ```sql
  SELECT * FROM t
  ORDER BY attr1, NULLS FIRST
  ```

- 虽然`ORDER BY`在`SELECT`后执行，但它实际上能看到所有没被`SELECT`选中的属性
  所以`ORDER BY`仍能使用`SELECT`能使用的那些属性作为原子表达式

### `LIMIT`

- `LIMIT`子句用于限制表的显示前若干行，后跟一个整型表示要显示的行数

### 子查询

- 不相关子查询
  - `WITH AS`：
- 相关子查询

### 集合查询

- `UNION`
- `INTERSECT`
- `EXCEPT`或`MINUS`
- `ALL`关键字：集合运算默认进行去重，将`ALL`添加在上述语句后，会保留所有重复的行

## 其它`DML`

### `INSERT INTO`

### `UPDATE`

### `DELETE`

## `SQL`函数

### [常见空值处理函数](#expl_nullfc)<a id='more_nullfc'></a>

- `coalesce(...)`：返回从左到右第一个非`null`值(除非不存在非`null`值)，要求所有值的数据类型相同
  不想无视某些为空值的属性时，可用`coalesce()`

- `nullif(a, b)`：若`a=b`，返回`null`，否则返回`a`
  前面说到空值一旦参与运算即返回空且会被聚集函数无视，可以通过`nullif`忽略某些非空值

- `is [not] distinct with`：空值安全比较，相比于`=`和`!=`，将`null`视为一般值
  即`null = null`返回真/`null != null`返回真

- `nulls first`和`nulls last`：空值排序，将该字段放在`order by`的某个属性后面，会把为`null`的元组放在最前/最后显示

  ```sql
  
  ```

### [常见聚集函数](#expl_gfunc)<a id='more_gfunc'></a>

- 固有的聚集函数(所有数据库都支持)：`count()、avg()、sum()、min()、max()`
- 统计类聚集函数：`variance()、stddev()、corr(x, y)`
- 字符串聚集函数：`string_agg(..., delim_char)`
- 自定义聚集函数