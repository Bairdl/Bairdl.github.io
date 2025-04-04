---
description: Lua 入门所需要掌握的一些基础知识，基于 Lua5.3。
tags:
  - lua
---

# Lua 快速入门

Lua 代码文件的后缀名为 `.lua` 。

## Lua 基础

Lua 的编程方式有交互式编程与脚本式编程，与 python 类型。  

### 注释

单行注释：`--` 。

``` lua
-- 单行注释
```

多行注释： `--[[ --]]` 。

``` lua
-- [[
多行注释
--]]
```

**Lua 不支持嵌套注释**，即多个多行注释嵌套是不合法的。

### 标识符

**Lua 大小写敏感。**
以英文字母或下划线开头，后面可以加上字母、下划线、数字。  
**不要使用下划线加大写字母的标识符，其通常为保留字或表示常量。**

### 关键字

``` plaintext
and not or
if elseif else 
do then end
true false
nil
repeat until
local
return
for while in 
function
goto
break
```

### 全局变量

默认情况下，变量都是全局，局部变量需要显式声明。  

当想删除一个全局变量时，将其赋值为 `nil` 。

## Lua 数据类型

**Lua 是动态类型语言，变量没有类型定义，只需要为变量赋值。**  
值可以存储在变量中，作为参数传递或结果返回。  

Lua 中有 8 个基本类型分别为：

- nil：表示一个无效值，在条件表达式中为 false 。
- boolean：包含两个值 true 和 false。
- number：表示双精度类型的实浮点数。
- string：字符串。
- function：由 C 或 Lua编写的函数。
- userdata：表示任意存储在变量中的 C 数据结构。
- thread：表示执行的独立线路，用于执行协同程序。
- table：关联熟手数组，数组的索引可以是数字、字符串或表类型。
  - `local t = {}` 。

### nil

表示空值，没有任何有效值。
对于全局变量和 table，nil 还有删除作用。给全局变量或 table 表中的变量赋一个 nil 值，即表示将它们删除。  

**nil 作比较时应该加上双引号 `"`。**（因为 `type()` 函数返回的值实际上是一个 `string` 类型。）

### boolean

只有两个值：false 和 true。
**只有 nil 与 false 在条件表达式中为false，其他值都为true。0也为true。**

### number

Lua 默认只有一种 number 类型：double 。

以下写法都是合法的。

``` lua
print(type(2))
print(type(2.2))
print(type(0.2))
print(type(2e+1))
print(type(0.2e-1))
print(type(7.8263692594256e-06))
```

### string

字符串由一对双引号或单引号，也可以使用两个方括号 `[[]]` 来表示。

``` lua
html = [[
<html>
<head></head>
<body>
    <a href="http://www.runoob.com/">菜鸟教程</a>
</body>
</html>
]]
print(html)
```

**在一个数字字符串上进行算法操作时，Lua 会尝试将其转换为一个数字进行计算。**

使用 `#` 可以计算字符串的长度。

```  lua
> len = "www.runoob.com"
> print(#len)
14
> print(#"www.runoob.com")
14
```

### table

Lua中，`table` 的创建是通过“构造表达式”来完成，最简单的构造表达式是 `{}`，用来创建一个空表。也可以在表中添加一些数据，直接初始化表。

``` lua
-- 创建一个空的 table
local tbl1 = {}
 
-- 直接初始表
local tbl2 = {"apple", "pear", "orange", "grape"}
```

Lua 中的表实际上是一个关联数组，数组的索引可以是数字也可以其他类型。

**在Lua 中，表的默认索引一般以 1 为开始。**

**`table` 不会固定长度大小，在新数据添加时 `table`长度会自动增长，没初始的 `table` 都是 `nil`。**

### function

在 Lua 中，函数是一等公民，是可以像其他数据一样被处理的实例。函数可以被存到变量里，作为参数传递。

`function` 可以通过匿名函数的方式通过参数传递。

``` lua
-- function_test2.lua 脚本文件
function testFun(tab,fun)
        for k ,v in pairs(tab) do
                print(fun(k,v));
        end
end


tab={key1="val1",key2="val2"};
testFun(tab,
function(key,val)--匿名函数
        return key.."="..val;
end
);
```

### thread

在 Lua 中最主要的线程是协同程序（coroutine）。它跟线程差不多，拥有自己独立的栈、局部变量和指令指针，可以跟其他协同程序共享全局变量和其他大部分数据。

线程和协程的区别：线程可以多个运行，而协程任意时刻只能运行一个，并且处于运行状态的协程只有被挂起时才会暂停。

### userdata

`userdata` 是一种用户自定义数据，用于表示一种由应用程序或 C/C++ 语言库所创建的类型，可以将任意 C/C++ 的任意数据类型的数据（通常是 struct 和指针）存储到 Lua 变量中调用。

## Lua 变量

Lua 变量有三种类型：全局变量、局部变量、表中的域。  
Lua 默认的变量类型是全局变量，即使是在函数或语句块中声明的也是全局变量，除非显式都将其声明为局部变量。  
**变量的默认值都为 `nil`**。

``` lua
-- test.lua 文件脚本
a = 5               -- 全局变量
local b = 5         -- 局部变量

function joke()
    c = 5           -- 全局变量
    local d = 6     -- 局部变量
end

joke()
print(c,d)          --> 5 nil

do 
    local a = 6     -- 局部变量
    b = 6           -- 对局部变量重新赋值
    print(a,b);     --> 6 6
end

print(a,b)      --> 5 6
```

### 赋值语句

Lua 可以对多个变量同时赋值，变量列表和值列表的各个元素使用逗号分开，赋值语句右边的值会依次赋值给左边的变量。

遇到赋值语句，Lua 会先计算右边的所有值，然后再执行赋值操作，所以可以通过以下方式来交换变量的值。

``` lua
x, y = y, x                     -- swap 'x' for 'y'
a[i], a[j] = a[j], a[i]         -- swap 'a[i]' for 'a[j]'
```

当变量个数与值的个数不一致时，如果 变量个数 > 值的个数，按变量个数补足nil：变量个数 < 值的个数，多余的值会被忽略。

### 索引

对 table 的索引有两种方式：`[]`和 `.` 。

``` lua
t[i]
t.i                 -- 当索引为字符串类型时的一种简化写法
gettable_event(t,i) -- 采用索引访问本质上是一个类似这样的函数调用
```

## Lua 循环

Lua 的循环类型有三种：`while` 、 `for` 和 `repeat...unitl`。

### while 循环

当判断条件为 `true` 时会重复执行循环体语句。  

``` lua
while(condition)
do
   statements
end
```

### for 循环

可以指定重复执行的次数。

Lua 中 for 语句有两类：数值 for 循环、泛型 for 循环。

#### 数值 for 循环

``` lua
for var=exp1,exp2,exp3 do  
    <执行体>  
end  
```

var 从 exp1 变化到 exp2，每次变化以 exp3 为步长递增 var，并执行一次 "执行体"。exp3 是可选的，如果不指定，默认为1。

**for中的表达式都在循环开始前进行一次求值，循环中不再进行求值。**

#### 泛型 for 循环

通过一个迭代器函数来遍历所有值。

``` lua
--打印数组a的所有值  
a = {"one", "two", "three"}
for i, v in ipairs(a) do
    print(i, v)
end 
```

### repeat...until 循环

`repeat...until` 循环的条件语句在当前循环结束后才判断，类似与`do...while`。  
但是其为 **条件判断语句为 `true` 时停止执行**。  

``` lua
repeat
   statements
until( condition )
```

### 循环控制语句

Lua 支持以下循环控制语句：

- `break`：退出当前循环。
- `goto`：将程序的控制点转移到一个标签处。

#### break 语句

break 语句插入在循环体中，用于退出当前循环或语句，并开始脚本执行紧接着的语句。  
如果使用循环嵌套，break语句将停止最内层循环的执行，并开始执行的外层的循环语句。  

#### goto 语句

goto 语句允许将控制流程无条件地转到被标记的语句处。

**`goto` 语句只能在同一个函数内跳转，不能跨函数或跨文件跳转。**

goto 语法格式

``` lua
goto Label
```

Label 语法格式

``` lua
:: Label ::
```

实例

```lua
local a = 1
::label:: print("--- goto label ---")

a = a+1
if a < 3 then
    goto label   -- a 小于 3 的时候跳转到标签 label
end
```

结果为：

``` text
--- goto label ---
--- goto label ---
```

使用 `goto` 来实现 `continue` 的功能。

``` lua
for i=1, 3 do
    if i <= 2 then
        print(i, "yes continue")
        goto continue
    end
    print(i, " no continue")
    ::continue::
    print([[i'm end]])
end
```

## Lua 流程控制

流程控制语句通过程序设定一个或多个条件语句来设定。在条件为 true 时执行指定程序代码，在条件为 false 时执行其他指定代码。

控制结构的条件表达式结果可以是任何值，Lua认为false和nil为假，true和非nil为真。**Lua 中 0 为 true。**  

Lua 提供的流程控制语句有 ：

- `if`
- `elseif`
- `else`

示例：

``` lua
if( 布尔表达式 1)
then
   --[ 在布尔表达式 1 为 true 时执行该语句块 --]

elseif( 布尔表达式 2)
then
   --[ 在布尔表达式 2 为 true 时执行该语句块 --]

elseif( 布尔表达式 3)
then
   --[ 在布尔表达式 3 为 true 时执行该语句块 --]
else 
   --[ 如果以上布尔表达式都不为 true 则执行该语句块 --]
end

```

## Lua 函数

在Lua中，函数是对语句和表达式进行抽象的主要方法。既可以用来处理一些特殊的工作，也可以用来计算一些值。  

### 函数定义

``` lua
optional_function_scope function function_name( argument1, argument2, argument3..., argumentn)
    function_body
    return result_params_comma_separated
end

```

解析：

- `optional_function_scope`: 该参数是可选的指定函数是全局函数还是局部函数，未设置该参数默认为全局函数，如果你需要设置函数为局部函数需要使用关键字 local。
- `function_name`: 指定函数名称。
- `argument1, argument2, argument3..., argumentn`: 函数参数，多个参数以逗号隔开，函数也可以不带参数。
- `function_body`: 函数体，函数中需要执行的代码语句块。
- `result_params_comma_separated`: 函数返回值，Lua语言函数可以返回多个值，每个值以逗号隔开。

### 可变参数

Lua 函数可以接受可变数目的参数，和 C 语言类似，在函数参数列表中使用三点 ... 表示函数有可变的参数。  

``` lua
function add(...)  
local s = 0  
  for i, v in ipairs{...} do   --> {...} 表示一个由所有变长参数构成的数组  
    s = s + v  
  end  
  return s  
end  
print(add(3,4,5,6,7))  --->25
```

获取参数数量：`#{...}` 、`select("#", ...)` 。

**固定参数必须放在可变参数之前。**  

通常在遍历变长参数的时候只需要使用 `{…}`，然而变长参数可能会包含一些 `nil`，那么就可以用 `select` 函数来访问变长参数了：`select('#', …)` 或者 `select(n, …)`

- `select('#', …)` 返回可变参数的长度。
- `select(n, …)` 用于返回从起点 n 开始到结束位置的所有参数列表。
调用 `select` 时，必须传入一个固定实参 selector(选择开关) 和一系列变长参数。如果 selector 为数字 n，那么 select 返回参数列表中从索引 `n` 开始到结束位置的所有参数列表，否则只能为字符串 `#`，这样 select 返回变长参数的总数。

``` lua
function f(...)
    a = select(3,...)  -->从第三个位置开始，变量 a 对应右边变量列表的第一个参数
    print (a)
    print (select(3,...)) -->打印所有列表参数
end

f(0,1,2,3,4,5)
```

## Lua 运算符

运算符是一个特殊的符号，用于告诉解释器执行特定的数学或逻辑运算。Lua提供了以下几种运

算符类型：

- 算术运算符
- 关系运算符
- 逻辑运算符
- 其他运算符

### 算术运算符

- `+` ：加法
- `-` ： 减法
- `*` ： 乘法
- `/` ：除法
- `%` ：取余
- `^` ： 乘幂
- `-` ：负号
- `//` ：整除运算

### 关系运算符

- `==` ：等于
- `~=` ：不等于
- `>` ：大于
- `<` ：小于
- `>=` ：大于等于
- `<=` ：小于等于

### 逻辑运算符

- `and` ：逻辑与
- `or` ：逻辑或
- `not` ：逻辑非

### 其他运算符

- `..`　：连接两个字符串
- `#` ：一元运算符，返回字符串或表的长度。

### 运算符优先级

从高到低的顺序。

``` text
^
not    - (unary)
*      /       %
+      -
..
<      >      <=     >=     ~=     ==
and
or
```

除了 `^` 和 `..` 外所有的二元运算符都是左连接的。  

## Lua 字符串

Lua 中的字符串可以包含任意字符，包括字母、数字、符号、空格以及其他特殊字符。

字符串表示方式有三种：

``` lua
str1 = '字符串1'
str2 = "字符串2"
str3 = [[字符串3]]
```

### 转义字符

- `\0` ：空字符 NULL
- `\ddd`：1到3位八进制所代表的任意字符
- `\xhh`：1到2位十六进制所代表的任意字符

### 相关函数

- 计算长度
  - `string.len`：用于计算只包含 ASCII 字符的字符串长度。
  - `utf8.len`
- 大小写转换
  - `string.upper(arg)`：全部转换为大写字母。
  - `string.lower(arg)`：全部转换为小写字母。
- 字符串替换
  - `string.gsub(mainStr, findStr, repStr, num)`：num 为替换次数（忽略则全部替换）。
- 字符串查找
  - `string.find(mainStr, findStr, init, plain)`：
    - init 为搜索的起始位置；plain 表示是否使用简单模式，为false时表示使用正则表达式，默认为 false 。
    - 返回子串的起始索引和结束索引；不存在则返回 nil 。
- 字符串反转
  - `string.reverse(arg)`
- 字符串格式化
  - `string.format(...)`
- 字符串转换
  - `string.char(arg)`：将整型数字转换为字符，并连接。
  - `string.byte(arg, int)`：转换指定位置的字符为整数值，默认为第一个字符。
- 重复字符串
  - `string.rep(str, n)`。
- 链接字符串
  - `..`
- 字符串查找
  - `string.gmatch(str, pattern)`
    - 返回一个迭代器函数。每次调用这个函数，返回一个在字符串 str 中找到的下一个符合 pattern 描述的子串。
    - 如果没有找到，返回  nil 。
  - `string.match(str, pattern, init)`
    - 只寻找第一个配对。
    - init 指定搜索的起点，默认为1.
    - 如果没有找到，返回 nil。
- 字符串截取
  - `string.sub(str, start, end)`
    - end 默认为 -1 。

## Lua 数组

Lua 中并没有专门的数组类型，而是使用一种被称为 "table" 的数据结构来实现数组的功能。

Lua 数组的索引键值可以使用整数表示，数组的大小不是固定的。

在 Lua 索引值是以 1 为起始，但你也可以指定 0 开始。

### 一维数组

``` lua
-- 创建一个数组
local myArray = {10, 20, 30, 40, 50}

-- 修改数组元素
myArray[2] = 25

-- 添加新元素到数组末尾
myArray[#myArray + 1] = 60

-- 删除第三个元素
table.remove(myArray, 3)

-- 循环遍历数组
for i = 1, #myArray do
    print(myArray[i])
end
```

### 多维数组

``` lua
-- 初始化数组
array = {}
maxRows = 3
maxColumns = 3
for row=1,maxRows do
   for col=1,maxColumns do
      array[row*maxColumns +col] = row*col
   end
end

-- 访问数组
for row=1,maxRows do
   for col=1,maxColumns do
      print(array[row*maxColumns +col])
   end
end
```

[Lua快速入门下](./Lua快速入门2.md)