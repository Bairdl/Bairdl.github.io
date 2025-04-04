---
description: 介绍Lua 的迭代器、表、模块与包、元表、协同程序等。
tags:
  - lua
---

# Lua 快速入门下

[Lua 快速入门上](./Lua快速入门1.md)

## Lua 迭代器

在 Lua 中迭代器是一种支持指针类型的结构，它可以遍历集合的每一个元素。

### 泛型 for 迭代器

泛型 for 在内部保存三个值：迭代函数、状态常量、控制变量。

其语法格式如下：

``` lua
for k, v in pairs(t) do
    print(k, v)
end
```

k, v 为变量列表， `pairs(t)` 为表达式列表。

#### 泛型执行过程

1. 初始化，计算 `in` 后面表达式的值，表达式应该返回泛型 `for` 需要的三个值：迭代函数、状态常量、控制变量；与多值赋值一样，如果表达式返回的结果个数不足三个会自动用 nil 补足，多出部分会被忽略。
2. 将状态常量和控制变量作为参数调用迭代函数（注意：对于 for 结构来说，状态常量没有用处，仅仅在初始化时获取他的值并传递给迭代函数）。
3. 将迭代函数返回的值赋给变量列表。
4. 如果返回的第一个值为nil循环结束，否则执行循环体。
5. 回到第二步再次调用迭代函数。

### 无状态的迭代器

无状态的迭代器是指不保留任何状态的迭代器，因此在循环中我们可以利用无状态迭代器避免创建闭包花费额外的代价。

每一次迭代，迭代函数都是用两个变量（状态常量和控制变量）的值作为参数被调用，一个无状态的迭代器只利用这两个值可以获取下一个元素。

这种无状态迭代器的典型的简单的例子是 ipairs，它遍历数组的每一个元素，元素的索引需要是数值。

迭代的状态包括被遍历的表（循环过程中不会改变的状态常量）和当前的索引下标（控制变量），ipairs 和迭代函数都很简单，我们在 Lua 中可以这样实现：

``` lua
function iter (a, i)
    i = i + 1
    local v = a[i]
    if v then
       return i, v
    end
end
 
function ipairs (a)
    return iter, a, 0
end
```

当 Lua 调用 ipairs(a) 开始循环时，他获取三个值：迭代函数 iter、状态常量 a、控制变量初始值 0；然后 Lua 调用 iter(a,0) 返回 1, a[1]（除非 a[1]=nil）；第二次迭代调用 iter(a,1) 返回 2, a[2]……直到第一个 nil 元素。

### 多状态的迭代器

很多情况下，迭代器需要保存多个状态信息而不是简单的状态常量和控制变量，最简单的方法是使用闭包，还有一种方法就是将所有的状态信息封装到 table 内，将 table 作为迭代器的状态常量，因为这种情况下可以将所有的信息存放在 table 内，所以迭代函数通常不需要第二个参数。

``` lua
array = {"Google", "Runoob"}

function elementIterator (collection)
   local index = 0
   local count = #collection
   -- 闭包函数
   return function ()
      index = index + 1
      if index <= count
      then
         --  返回迭代器的当前元素
         return collection[index]
      end
   end
end

for element in elementIterator(array)
do
   print(element)
end
```

## Lua table (表)

table 是 Lua 的一种数据结构用来帮助我们创建不同的数据类型，如：数组、字典等。

Lua table 使用关联型数组，你可以用任意类型的值来作数组的索引，但这个值不能是 nil。

Lua table 是不固定大小的，你可以根据自己需要进行扩容。

Lua也是通过table来解决模块（module）、包（package）和对象（Object）的。

### 表的构造

构造器是创建和初始化表的表达式。表是Lua特有的功能强大的东西。最简单的构造函数是{}，用来创建一个空表。可以直接初始化数组:

``` lua
-- 初始化表
mytable = {}

-- 指定值
mytable[1]= "Lua"

-- 移除引用
mytable = nil
-- lua 垃圾回收会释放内存
```

### 表操作

- `table.concat(table, sep, start, end)`
  - 将表中从 start 到 end 范围内的元素连接起来，使用分隔符 sep 间隔。
- `table.insert(table, pos, value)`
  - 在指定位置 pos 处插入值为 value 的元素。
  - pos 可省略，默认为表的末尾。
- `table.remove(table,pos)`
  - 删除指定位置 pos 处的元素。
  - pos 可省略，默认为末尾元素。
- `table.sort(table)`
  - 对表进行升序排序。

## Lua 模块与包

模块类似于一个封装库，可以把一些公用的代码放在一个文件里，以 API 接口的形式在其他地方调用，有利于代码的重用和降低代码耦合度。

Lua 的模块是由变量、函数等已知元素组成的 table，因此创建一个模块很简单，就是创建一个 table，然后把需要导出的常量、函数放入其中，最后返回这个 table 就行。

``` lua
-- 文件名为 module.lua
-- 定义一个名为 module 的模块
module = {}
 
-- 定义一个常量
module.constant = "这是一个常量"
 
-- 定义一个函数
function module.func1()
    io.write("这是一个公有函数！\n")
end
 
local function func2()
    print("这是一个私有函数！")
end
 
function module.func3()
    func2()
end
 
return module
```

### require 函数

Lua提供了一个名为require的函数用来加载模块。

``` lua
require("<模块名>")

-- 或者
require "<模块名>"
```

执行 require 后会返回一个由模块常量或函数组成的 table，并且还会定义一个包含该 table 的全局变量。

``` lua
-- test_module.lua 文件
-- module 模块为上文提到到 module.lua
require("module")
 
print(module.constant)
 
module.func3()

-- test_module2.lua 文件
-- module 模块为上文提到到 module.lua
-- 别名变量 m
local m = require("module")
 
print(m.constant)
 
m.func3()
```

### 加载机制

对于自定义的模块，模块文件不是放在哪个文件目录都行，函数 `require` 有它自己的文件路径加载策略，它会尝试从 Lua 文件或 C 程序库中加载模块。

`require` 用于搜索 Lua 文件的路径是存放在全局变量 `package.path` 中，当 Lua 启动后，会以环境变量 LUA_PATH 的值来初始这个环境变量。如果没有找到该环境变量，则使用一个编译时定义的默认路径来初始化。

如果找过目标文件，则会调用 `package.loadfile` 来加载模块。否则，就会去找 C 程序库。

搜索的文件路径是从全局变量 `package.cpath` 获取，而这个变量则是通过环境变量 LUA_CPATH 来初始。

搜索的策略跟上面的一样，只不过现在换成搜索的是 `so` 或 `dll` 类型的文件。如果找得到，那么 `require` 就会通过 `package.loadlib` 来加载它。

### C 包

与Lua中写包不同，C包在使用以前必须首先加载并连接，在大多数系统中最容易的实现方式是通过动态连接库机制。

Lua在一个叫`loalibd`的函数内提供了所有的动态连接的功能。这个函数有两个参数:库的绝对路径和初始化函数。

``` lua
local path = "/usr/local/lua/lib/libluasocket.so"
local f = loadlib(path, "luaopen_socket")
```

`loadlib` 函数加载指定的库并且连接到 Lua，然而它并不打开库（也就是说没有调用初始化函数），反之他返回初始化函数作为 Lua 的一个函数，这样我们就可以直接在Lua中调用他。

如果加载动态库或者查找初始化函数时出错，`loadlib` 将返回 nil 和错误信息。我们可以修改前面一段代码，使其检测错误然后调用初始化函数：

``` lua
local path = "/usr/local/lua/lib/libluasocket.so"
-- 或者 path = "C:\\windows\\luasocket.dll"，这是 Window 平台下
local f = assert(loadlib(path, "luaopen_socket"))
f()  -- 真正打开库
```

一般情况下我们期望二进制的发布库包含一个与前面代码段相似的 stub 文件，安装二进制库的时候可以随便放在某个目录，只需要修改 stub 文件对应二进制库的实际路径即可。

将 stub 文件所在的目录加入到 LUA_PATH，这样设定后就可以使用 require 函数加载 C 库了。

## Lua 元表（Metatable）

在 Lua table 中我们可以访问对应的 key 来得到 value 值，但是却无法对两个 table 进行操作(比如相加)。

因此 Lua 提供了元表(Metatable)，允许我们改变 table 的行为，每个行为关联了对应的元方法。

例如，使用元表我们可以定义 Lua 如何计算两个 table 的相加操作 a+b。

当 Lua 试图对两个表进行相加时，先检查两者之一是否有元表，之后检查是否有一个叫 `__add`的字段，若找到，则调用对应的值。 `__add` 等即时字段，其对应的值（往往是一个函数或是 table）就是"元方法"。

有两个很重要的函数来处理元表：

- `setmetatable(table,metatable)`: 对指定 table 设置元表(metatable)，如果元表(metatable)中存在 `__metatable` 键值，setmetatable 会失败。
- `getmetatable(table)`: 返回对象的元表(metatable)。

``` lua
mytable = {}                          -- 普通表 
mymetatable = {}                      -- 元表
setmetatable(mytable,mymetatable)     -- 把 mymetatable 设为 mytable 的元表

-- 简写
mytable = setmetatable({},{})

getmetatable(mytable)                 -- 这会返回 mymetatable
```

### `__index` 元方法

当你通过键来访问 table 的时候，如果这个键没有值，那么Lua就会寻找该table的metatable（假定有metatable）中的`__index` 键。如果`__index`包含一个表，Lua会在表中查找相应的键。

如果`__index`包含一个函数的话，Lua就会调用那个函数，table和键会作为参数传递给函数。

`__index` 元方法查看表中元素是否存在，如果不存在，返回结果为 nil；如果存在则由 `__index` 返回结果。

``` lua
mytable = setmetatable({key1 = "value1"}, {
  __index = function(mytable, key)
    if key == "key2" then
      return "metatablevalue"
    else
      return nil
    end
  end
})

print(mytable.key1,mytable.key2)

-- 简写
mytable = setmetatable({key1 = "value1"}, { __index = { key2 = "metatablevalue" } })
print(mytable.key1,mytable.key2)
```

实例解析：

- mytable 表赋值为 {key1 = "value1"}。
- mytable 设置了元表，元方法为 `__index`。
- 在mytable表中查找 key1，如果找到，返回该元素，找不到则继续。
- 在mytable表中查找 key2，如果找到，返回 `metatablevalue`，找不到则继续。
- 判断元表有没有`__index`方法，如果`__index`方法是一个函数，则调用该函数。
- 元方法中查看是否传入 "key2" 键的参数（mytable.key2已设置），如果传入 "key2" 参数返回 "metatablevalue"，否则返回 mytable 对应的键值。

Lua 查找一个表元素时的规则，其实就是如下 3 个步骤:

1. 在表中查找，如果找到，返回该元素，找不到则继续
2. 判断该表是否有元表，如果没有元表，返回 nil，有元表则继续。
3. 判断元表有没有 `__index` 方法，如果`__index` 方法为 nil，则返回 nil；如果 `__index` 方法是一个表，则重复 1、2、3；如果`__index` 方法是一个函数，则返回该函数的返回值。

### `__newindex` 元方法

`__newindex` 元方法用来对表更新，`__index`则用来对表访问 。

当你给表的一个缺少的索引赋值，解释器就会查找`__newindex` 元方法：如果存在则调用这个函数而不进行赋值操作。

``` lua
mymetatable = {}
mytable = setmetatable({key1 = "value1"}, { __newindex = mymetatable })

print(mytable.key1)

mytable.newkey = "新值2"
print(mytable.newkey,mymetatable.newkey)

mytable.key1 = "新值1"
print(mytable.key1,mymetatable.key1)

--[[
    value1
    nil    新值2
    新值1    nil
--]]
```

以上实例中表设置了元方法 __newindex，在对新索引键（newkey）赋值时（mytable.newkey = "新值2"），会调用元方法，而不进行赋值。而如果对已存在的索引键（key1），则会进行赋值，而不调用元方法__newindex。

### 为表添加操作符

``` lua
-- 自定义计算表中最大键值函数 table_maxn，即返回表最大键值
function table_maxn(t)
    local mn = 0
    for k, _ in pairs(t) do
        if type(k) == "number" and k > mn then
            mn = k
        end
    end
    return mn
end

-- 两表相加操作
mytable = setmetatable({ 1, 2, 3 }, {
  __add = function(mytable, newtable)
    local max_key_mytable = table_maxn(mytable)
    for i = 1, table_maxn(newtable) do
      table.insert(mytable, max_key_mytable + i, newtable[i])
    end
    return mytable
  end
})

secondtable = {4, 5, 6}

mytable = mytable + secondtable

for k, v in ipairs(mytable) do
    print(k, v)
end

-- 输出
--[[
1    1
2    2
3    3
4    4
5    5
6    6
--]]
```

|模式|描述|
|----|----|
|__add| 对应的运算符 '+'.|
|__sub| 对应的运算符 '-'.|
|__mul| 对应的运算符 '*'.|
|__div| 对应的运算符 '/'.|
|__mod| 对应的运算符 '%'.|
|__unm| 对应的运算符 '-'.|
|__concat| 对应的运算符 '..'.|
|__eq| 对应的运算符 '=='.|
|__lt| 对应的运算符 '<'.|
|__le| 对应的运算符 '<='.|

### `__call` 元方法

__call 元方法在 Lua 调用一个值时调用。

``` lua
-- 计算表中最大值，table.maxn在Lua5.2以上版本中已无法使用
-- 自定义计算表中最大键值函数 table_maxn，即计算表的元素个数
function table_maxn(t)
    local mn = 0
    for k, v in pairs(t) do
        if mn < k then
            mn = k
        end
    end
    return mn
end

-- 定义元方法__call
mytable = setmetatable({10}, {
  __call = function(mytable, newtable)
        sum = 0
        for i = 1, table_maxn(mytable) do
                sum = sum + mytable[i]
        end
    for i = 1, table_maxn(newtable) do
                sum = sum + newtable[i]
        end
        return sum
  end
})
newtable = {10,20,30}
print(mytable(newtable))

-- 输出：70

```

### `__tostring` 元方法

__tostring 元方法用于修改表的输出行为。

``` lua
mytable = setmetatable({ 10, 20, 30 }, {
  __tostring = function(mytable)
    sum = 0
    for k, v in pairs(mytable) do
                sum = sum + v
        end
    return "表所有元素的和为 " .. sum
  end
})
print(mytable)

-- 输出
--[[
表所有元素的和为 60
--]]
```

## Lua 协同程序（coroutine）

Lua 协同程序(coroutine)与线程比较类似：拥有独立的堆栈，独立的局部变量，独立的指令指针，同时又与其它协同程序共享全局变量和其它大部分东西。

协同程序可以理解为一种特殊的线程，可以暂停和恢复其执行，从而允许**非抢占式的多任务处理**。

### 基本语法

协同程序由 `coroutine` 模块提供支持。

使用协同程序，你可以在函数中使用 `coroutine.create` 创建一个新的协同程序对象，并使用 `coroutine.resume` 启动它的执行。协同程序可以通过调用 `coroutine.yield` 来主动暂停自己的执行，并将控制权交还给调用者。

coroutine 的状态有三种：dead，suspended，running。

|方法|描述|
|----|----|
|`coroutine.create()` |创建 coroutine，返回 coroutine， 参数是一个函数，当和 resume 配合使用的时候就唤醒函数调用。|
|`coroutine.resume()` |重启 coroutine，和 create 配合使用。|
|`coroutine.yield()` |挂起 coroutine，将 coroutine 设置为挂起状态，这个和 resume 配合使用能有很多有用的效果。|
|`coroutine.status()` |查看 coroutine 的状态。|
|`coroutine.wrap()`| 创建 coroutine，返回一个函数，一旦你调用这个函数，就进入 coroutine，和 create 功能重复。|
|`coroutine.running()`| 返回正在跑的 coroutine，一个 coroutine 就是一个线程，当使用running的时候，就是返回一个 coroutine 的线程号。|

``` lua
function foo (a)
    print("foo 函数输出", a)
    return coroutine.yield(2 * a) -- 返回  2*a 的值
end
 
co = coroutine.create(function (a , b)
    print("第一次协同程序执行输出", a, b) -- co-body 1 10
    local r = foo(a + 1)
     
    print("第二次协同程序执行输出", r)
    local r, s = coroutine.yield(a + b, a - b)  -- a，b的值为第一次调用协同程序时传入
     
    print("第三次协同程序执行输出", r, s)
    return b, "结束协同程序"                   -- b的值为第二次调用协同程序时传入
end)
        
print("main", coroutine.resume(co, 1, 10)) -- true, 4
print("--分割线----")
print("main", coroutine.resume(co, "r")) -- true 11 -9
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- true 10 end
print("---分割线---")
print("main", coroutine.resume(co, "x", "y")) -- cannot resume dead coroutine
print("---分割线---")

-- 输出 
--[[
第一次协同程序执行输出    1    10
foo 函数输出    2
main    true    4
--分割线----
第二次协同程序执行输出    r
main    true    11    -9
---分割线---
第三次协同程序执行输出    x    y
main    true    10    结束协同程序
---分割线---
main    false    cannot resume dead coroutine
---分割线---
--]]
```

### 生产者-消费者示例

``` lua
local newProductor

function productor()
     local i = 0
     while true do
          i = i + 1
          send(i)     -- 将生产的物品发送给消费者
     end
end

function consumer()
     while true do
          local i = receive()     -- 从生产者那里得到物品
          print(i)
     end
end

function receive()
     local status, value = coroutine.resume(newProductor)
     return value
end

function send(x)
     coroutine.yield(x)     -- x表示需要发送的值，值返回以后，就挂起该协同程序
end

-- 启动程序
newProductor = coroutine.create(productor)
consumer()

-- 输出 
--[[
1
2
3
4
5
6
7
8
9
10
11
12
13
……
--]]
```

### 线程和协同程序的区别

线程与协同程序的主要区别在于，一个具有多个线程的程序可以同时运行几个线程，而协同程序却需要彼此协作的运行。

在任一指定时刻只有一个协同程序在运行，并且这个正在运行的协同程序只有在明确的被要求挂起的时候才会被挂起。

协同程序有点类似同步的多线程，在等待同一个线程锁的几个线程有点类似协同。

主要区别归纳如下：

- 调度方式：线程通常由操作系统的调度器进行抢占式调度，操作系统会在不同线程之间切换执行权。而协同程序是非抢占式调度的，它们由程序员显式地控制执行权的转移。

- 并发性：线程是并发执行的，多个线程可以同时运行在多个处理器核心上，或者通过时间片轮转在单个核心上切换执行。协同程序则是协作式的，只有一个协同程序处于运行状态，其他协同程序必须等待当前协同程序主动放弃执行权。

- 内存占用：线程通常需要独立的堆栈和上下文环境，因此线程的创建和销毁会带来额外的开销。而协同程序可以共享相同的堆栈和上下文，因此创建和销毁协同程序的开销较小。

- 数据共享：线程之间可以共享内存空间，但需要注意线程安全性和同步问题。协同程序通常通过参数传递和返回值来进行数据共享，不同协同程序之间的数据隔离性较好。

- 调试和错误处理：线程通常在调试和错误处理方面更复杂，因为多个线程之间的交互和并发执行可能导致难以调试的问题。协同程序则在调试和错误处理方面相对简单，因为它们是由程序员显式地控制执行流程的。

总体而言，线程适用于需要并发执行的场景，例如在多核处理器上利用并行性加快任务的执行速度。而协同程序适用于需要协作和协调的场景，例如状态机、事件驱动编程或协作式任务处理。选择使用线程还是协同程序取决于具体的应用需求和编程模型。

## Lua 文件 I/O

Lua I/O 库用于读取和处理文件。分为简单模式（和C一样）、完全模式。

- 简单模式（simple model）拥有一个当前输入文件和一个当前输出文件，并且提供针对这些文件相关的操作。
- 完全模式（complete model） 使用外部的文件句柄来实现。它以一种面对对象的形式，将所有的文件操作定义为文件句柄的方法

简单模式在做一些简单的文件操作时较为合适。但是在进行一些高级的文件操作的时候，简单模式就显得力不从心。例如同时读取多个文件这样的操作，使用完全模式则较为合适。

语法格式

``` lua
file = io.open (filename [, mode])
```

|模式|描述|
|----|----|
|r| 以只读方式打开文件，该文件必须存在。|
|w| 打开只写文件，若文件存在则文件长度清为0，即该文件内容会消失。若文件不存在则建立该文件。|
|a| 以附加的方式打开只写文件。若文件不存在，则会建立该文件，如果文件存在，写入的数据会被加到文件尾，即文件原先的内容会被保留。（EOF符保留）|
|r+| 以可读写方式打开文件，该文件必须存在。|
|w+| 打开可读写文件，若文件存在则文件长度清为零，即该文件内容会消失。若文件不存在则建立该文件。|
|a+| 与a类似，但此文件可读可写。|
|b| 二进制模式，如果文件是二进制文件，可以加上b。|
|+| 号表示对文件既可以读也可以写。|

### 简单模式

简单模式使用标准的 I/O 或使用一个当前输入文件和一个当前输出文件。

``` lua
-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 设置默认输入文件为 test.lua
io.input(file)

-- 输出文件第一行
print(io.read())

-- 关闭打开的文件
io.close(file)

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 设置默认输出文件为 test.lua
io.output(file)

-- 在文件最后一行添加 Lua 注释
io.write("--  test.lua 文件末尾注释")

-- 关闭打开的文件
io.close(file)
```

`io.read()` 参数

|模式|描述|
|----|----|
|"*n"|读取一个数字并返回它。例：file.read("*n")。|
|"*a"|从当前位置读取整个文件。例：file.read("*a")|
|"*l"|（默认）读取下一行，在文件尾 (EOF) 处返回 nil。例：file.read("*l")|
|number|返回一个指定字符个数的字符串，或在 EOF 时返回 nil。例：file.read(5)|

`io` 相关方法：

- `io.tmpfile()` :返回一个临时文件句柄，该文件以更新模式打开，程序结束时自动删除。
- `io.type(file)`: 检测obj是否一个可用的文件句柄
- `io.flush()`: 向文件写入缓冲中的所有数据
- `io.lines(optional file name)`: 返回一个迭代函数，每次调用将获得文件中的一行内容，当到文件尾时，将返回 nil，但不关闭文件。

### 完全模式

通常我们需要在同一时间处理多个文件。我们需要使用 `file:function_name` 来代替 `io.function_name` 方法。

``` lua
-- 以只读方式打开文件
file = io.open("test.lua", "r")

-- 输出文件第一行
print(file:read())

-- 关闭打开的文件
file:close()

-- 以附加的方式打开只写文件
file = io.open("test.lua", "a")

-- 在文件最后一行添加 Lua 注释
file:write("--test")

-- 关闭打开的文件
file:close()
```

其他方法:

- `file:seek(optional whence, optional offset)`: 设置和获取当前文件位置,成功则返回最终的文件位置(按字节),失败则返回nil加错误信息。参数 whence 值可以是:
  - "set": 从文件头开始
  - "cur": 从当前位置开始[默认]
  - "end": 从文件尾开始
  - offset:默认为0
- 不带参数`file:seek()`则返回当前位置,`file:seek("set")`则定位到文件头,`file:seek("end")`则定位到文件尾并返回文件大小。
- `file:flush()`: 向文件写入缓冲中的所有数据
- `io.lines(optional file name)`: 打开指定的文件 filename 为读模式并返回一个迭代函数，每次调用将获得文件中的一行内容，当到文件尾时，将返回 nil，并自动关闭文件。
  - 若不带参数时 io.lines() <=> io.input():lines(); 读取默认输入设备的内容，但结束时不关闭文件，

``` lua
-- 以只读方式打开文件
file = io.open("test.lua", "r")

file:seek("end",-25)
print(file:read("*a"))

-- 关闭打开的文件
file:close()
```

## Lua 错误处理

错误类型有：语法错误，运行错误。

### 语法错误

语法错误通常是由于对程序的组件（如运算符、表达式）使用不当引起的。  
例如 `a==2` ，执行后会出现  `lua: test.lua:2: syntax error near '=='` 。

### 运行错误

运行错误是程序可以正常执行，但是会输出报错信息。  

``` lua
function add(a,b)
   return a+b
end

add(10)

-- 输出
--[[
lua: test2.lua:2: attempt to perform arithmetic on local 'b' (a nil value)
stack traceback:
    test2.lua:2: in function 'add'
    test2.lua:5: in main chunk
    [C]: ?
--]]
```

### 错误处理

可以使用 `assert` 和 `error` 来处理错误。

``` lua
local function add(a,b)
   assert(type(a) == "number", "a 不是一个数字")
   assert(type(b) == "number", "b 不是一个数字")
   return a+b
end
add(10)

-- 输出
--[[
lua: test.lua:3: b 不是一个数字
stack traceback:
    [C]: in function 'assert'
    test.lua:3: in local 'add'
    test.lua:6: in main chunk
    [C]: in ?

实例中assert首先检查第一个参数，若没问题，assert不做任何事情；否则，assert以第二个参数作为错误信息抛出。
--]]
```

#### error 函数

`error(message, level)`

功能：终止正在执行的函数，并返回message的内容作为错误信息(error函数永远都不会返回)

通常情况下，error会附加一些错误位置的信息到message头部。

Level参数指示获得错误的位置:

- Level=1[默认]：为调用error位置(文件+行号)
- Level=2：指出哪个调用error的函数的函数
- Level=0:不添加错误位置信息

#### pcall 和 xpcall、debug

Lua中处理错误，可以使用函数pcall（protected call）来包装需要执行的代码。

pcall接收一个函数和要传递给后者的参数，并执行，执行结果：有错误、无错误；返回值true或者或false, errorinfo。

语法格式

``` lua
if pcall(function_name, ….) then
-- 没有错误
else
-- 一些错误
end
```

示例

``` lua
> =pcall(function(i) print(i) end, 33)
33
true
   
> =pcall(function(i) print(i) error('error..') end, 33)
33
false        stdin:1: error..
```

pcall以一种"保护模式"来调用第一个参数，因此pcall可以捕获函数执行中的任何错误。

通常在错误发生时，希望落得更多的调试信息，而不只是发生错误的位置。但pcall返回时，它已经销毁了调用桟的部分内容。

Lua提供了xpcall函数，xpcall接收第二个参数——一个错误处理函数，当错误发生时，Lua会在调用桟展开（unwind）前调用错误处理函数，于是就可以在这个函数中使用debug库来获取关于错误的额外信息了。

debug库提供了两个通用的错误处理函数:

- debug.debug：提供一个Lua提示符，让用户来检查错误的原因
- debug.traceback：根据调用桟来构建一个扩展的错误消息

``` lua
function myfunction ()
   n = n/nil
end

function myerrorhandler( err )
   print( "ERROR:", err )
end

status = xpcall( myfunction, myerrorhandler )
print( status)

-- 输出
--[[
ERROR:    test2.lua:2: attempt to perform arithmetic on global 'n' (a nil value)
false
--]]
```

## Lua 调试（Debug）

Lua 提供了 debug 库用于提供创建自定义调试器的功能。

Lua 中 debug 库包含以下函数：

1. debug():
   - 进入一个用户交互模式，运行用户输入的每个字符串。 使用简单的命令以及其它调试设置，用户可以检阅全局变量和局部变量， 改变变量的值，计算一些表达式，等等。
   - 输入一行仅包含 cont 的字符串将结束这个函数， 这样调用者就可以继续向下运行。

2. getfenv(object):
   - 返回对象的环境变量。

3. gethook(optional thread):
   - 返回三个表示线程钩子设置的值： 当前钩子函数，当前钩子掩码，当前钩子计数。

4. getinfo ([thread,] f [, what]):
   - 返回关于一个函数信息的表。 你可以直接提供该函数， 也可以用一个数字 f 表示该函数。
   - 数字 f 表示运行在指定线程的调用栈对应层次上的函数： 0 层表示当前函数（getinfo 自身）； 1 层表示调用 getinfo 的函数 （除非是尾调用，这种情况不计入栈）；等等。
   - 如果 f 是一个比活动函数数量还大的数字， getinfo 返回 nil。

5. debug.getlocal ([thread,] f, local):
   - 此函数返回在栈的 f 层处函数的索引为 local 的局部变量 的名字和值。 这个函数不仅用于访问显式定义的局部变量，也包括形参、临时变量等。

6. getmetatable(value):
   - 把给定索引指向的值的元表压入堆栈。如果索引无效，或是这个值没有元表，函数将返回 0 并且不会向栈上压任何东西。

7. getregistry():
   - 返回注册表表，这是一个预定义出来的表， 可以用来保存任何 C 代码想保存的 Lua 值。

8. getupvalue (f, up)
   - 此函数返回函数 f 的第 up 个上值的名字和值。 如果该函数没有那个上值，返回 nil 。
   - 以 '(' （开括号）打头的变量名表示没有名字的变量 （去除了调试信息的代码块）。

9. sethook ([thread,] hook, mask [, count]):
   - 将一个函数作为钩子函数设入。 字符串 mask 以及数字 count 决定了钩子将在何时调用。 掩码是由下列字符组合成的字符串，每个字符有其含义：
     - 'c': 每当 Lua 调用一个函数时，调用钩子；
     - 'r': 每当 Lua 从一个函数内返回时，调用钩子；
     - 'l': 每当 Lua 进入新的一行时，调用钩子。

10. setlocal ([thread,] level, local, value):
    - 这个函数将 value 赋给 栈上第 level 层函数的第 local 个局部变量。 如果没有那个变量，函数返回 nil 。 如果 level 越界，抛出一个错误。

11. setmetatable (value, table):
    - 将 value 的元表设为 table （可以是 nil）。 返回 value。

12. setupvalue (f, up, value):
    - 这个函数将 value 设为函数 f 的第 up 个上值。 如果函数没有那个上值，返回 nil 否则，返回该上值的名字。

13. traceback ([thread,] [message [, level]]):
    - 如果 message 有，且不是字符串或 nil， 函数不做任何处理直接返回 message。 否则，它返回调用栈的栈回溯信息。 字符串可选项 message 被添加在栈回溯信息的开头。 数字可选项 level 指明从栈的哪一层开始回溯 （默认为 1 ，即调用 traceback 的那里）。

### 调试类型

- 命令行调试
- 图形界面调试

命令行调试器有：RemDebug、clidebugger、ctrace、xdbLua、LuaInterface - Debugger、Rldb、ModDebug。

图形界调试器有：SciTE、Decoda、ZeroBrane Studio、akdebugger、luaedit。

## Lua 垃圾回收

Lua 采用了自动内存管理。 这意味着你不用操心新创建的对象需要的内存如何分配出来， 也不用考虑在对象不再被使用后怎样释放它们所占用的内存。

Lua 运行了一个垃圾收集器来收集所有死对象 （即在 Lua 中不可能再访问到的对象）来完成自动内存管理的工作。 Lua 中所有用到的内存，如：字符串、表、用户数据、函数、线程、 内部结构等，都服从自动管理。

Lua 实现了一个增量标记-扫描收集器。 它使用这两个数字来控制垃圾收集循环： 垃圾收集器间歇率和垃圾收集器步进倍率。 这两个数字都使用百分数为单位 （例如：值 100 在内部表示 1 ）。

垃圾收集器间歇率控制着收集器需要在开启新的循环前要等待多久。 增大这个值会减少收集器的积极性。 当这个值比 100 小的时候，收集器在开启新的循环前不会有等待。 设置这个值为 200 就会让收集器等到总内存使用量达到 之前的两倍时才开始新的循环。

垃圾收集器步进倍率控制着收集器运作速度相对于内存分配速度的倍率。 增大这个值不仅会让收集器更加积极，还会增加每个增量步骤的长度。 不要把这个值设得小于 100 ， 那样的话收集器就工作的太慢了以至于永远都干不完一个循环。 默认值是 200 ，这表示收集器以内存分配的"两倍"速工作。

如果你把步进倍率设为一个非常大的数字 （比你的程序可能用到的字节数还大 10% ）， 收集器的行为就像一个 stop-the-world 收集器。 接着你若把间歇率设为 200 ， 收集器的行为就和过去的 Lua 版本一样了： 每次 Lua 使用的内存翻倍时，就做一次完整的收集。

### 垃圾回收器函数

Lua 提供了以下函数collectgarbage ([opt [, arg]])用来控制自动内存管理:

- `collectgarbage("collect")`: 做一次完整的垃圾收集循环。通过参数 opt 它提供了一组不同的功能：

- `collectgarbage("count")`: 以 K 字节数为单位返回 Lua 使用的总内存数。 这个值有小数部分，所以只需要乘上 1024 就能得到 Lua 使用的准确字节数（除非溢出）。

- `collectgarbage("restart")`: 重启垃圾收集器的自动运行。

- `collectgarbage("setpause")`: 将 arg 设为收集器的 间歇率。 返回 间歇率 的前一个值。

- `collectgarbage("setstepmul")`: 返回 步进倍率 的前一个值。

- `collectgarbage("step")`: 单步运行垃圾收集器。 步长"大小"由 arg 控制。 传入 0 时，收集器步进（不可分割的）一步。 传入非 0 值， 收集器收集相当于 Lua 分配这些多（K 字节）内存的工作。 如果收集器结束一个循环将返回 true 。

- `collectgarbage("stop")`: 停止垃圾收集器的运行。 在调用重启前，收集器只会因显式的调用运行。

## Lua 面向对象

### 面向对象特征

- 封装：将数据和方法捆绑在一起，隐藏实现细节，只暴露必要的接口，提高安全性和可维护性。

- 继承：通过派生新类复用和扩展现有代码，减少重复编码，提高开发效率和可扩展性。

- 多态：同一操作作用于不同对象时表现不同，支持统一接口调用，增强灵活性和扩展性。

- 抽象：简化复杂问题，定义核心类和接口，隐藏不必要的细节，便于管理复杂性。

### Lua 类

Lua 中的类可以通过 `table + function` 模拟出来。

至于继承，可以通过 `metetable` 模拟出来（不推荐用，只模拟最基本的对象大部分实现够用了）。

在 Lua 中，表（table）可以视为对象的一种变体。和对象一样，表具有状态（成员变量），并且可以代表独立的实体。  

``` lua
-- 定义矩形类
-- 定义矩形类
Rectangle = {area = 0, length = 0, breadth = 0}

-- 创建矩形对象的构造函数
function Rectangle:new(o, length, breadth)
  o = o or {}  -- 如果未传入对象，创建一个新的空表
  setmetatable(o, self)  -- 设置元表，使其继承 Rectangle 的方法
  self.__index = self  -- 确保在访问时能找到方法和属性
  o.length = length or 0  -- 设置长度，默认为 0
  o.breadth = breadth or 0  -- 设置宽度，默认为 0
  o.area = o.length * o.breadth  -- 计算面积
  return o
end

-- 打印矩形的面积
function Rectangle:printArea()
  print("矩形面积为 ", self.area)
end

-- 运行实例：
local rect1 = Rectangle:new(nil, 5, 10)  -- 创建一个长为 5，宽为 10 的矩形
rect1:printArea()  -- 输出 "矩形面积为 50"

local rect2 = Rectangle:new(nil, 7, 3)  -- 创建一个长为 7，宽为 3 的矩形
rect2:printArea()  -- 输出 "矩形面积为 21"
```

### Lua 继承

继承是指一个对象直接使用另一对象的属性和方法，可用于扩展基础类的属性和方法。

Lua 中的继承通过设置子类的元表来实现。

我们可以创建一个新表，并将其元表设置为父类。

``` lua
-- 定义矩形类
Rectangle = {area = 0, length = 0, breadth = 0}

-- 创建矩形对象的构造函数
function Rectangle:new(o, length, breadth)
  o = o or {}  -- 如果未传入对象，创建一个新的空表
  setmetatable(o, self)  -- 设置元表，使其继承 Rectangle 的方法
  self.__index = self  -- 确保在访问时能找到方法和属性
  o.length = length or 0  -- 设置长度，默认为 0
  o.breadth = breadth or 0  -- 设置宽度，默认为 0
  o.area = o.length * o.breadth  -- 计算面积
  return o
end

-- 打印矩形的面积
function Rectangle:printArea()
  print("矩形面积为 ", self.area)
end

-- 定义正方形类，继承自矩形类
Square = Rectangle:new()  -- Square 继承 Rectangle 类

-- 重写构造函数（正方形的边长相等）
function Square:new(o, side)
  o = o or {}  -- 如果未传入对象，创建一个新的空表
  setmetatable(o, self)  -- 设置元表，使其继承 Rectangle 的方法
  self.__index = self  -- 确保在访问时能找到方法和属性
  o.length = side or 0  -- 设置边长
  o.breadth = side or 0  -- 正方形的宽度和长度相等
  o.area = o.length * o.breadth  -- 计算面积
  return o
end

-- 运行实例：
local rect = Rectangle:new(nil, 5, 10)  -- 创建一个长为 5，宽为 10 的矩形
rect:printArea()  -- 输出 "矩形面积为 50"

local square = Square:new(nil, 4)  -- 创建一个边长为 4 的正方形
square:printArea()  -- 输出 "矩形面积为 16"
```

Rectangle 类：依然是矩形的基本类，拥有 length、breadth 和 area 属性，以及计算和打印面积的方法。

Square 类继承自 Rectangle：Square 类通过 Rectangle:new() 来继承 Rectangle 类的方法和属性。由于正方形的长度和宽度相等，我们在 Square:new 方法中重写了构造函数，将 length 和 breadth 设置为相同的值（即 side）。

重写构造函数：Square:new(o, side) 方法创建正方形对象时，使用传入的边长 side 初始化 length 和 breadth 属性，并计算面积。

### 函数重写

在 Lua 中，函数重写（也称为方法重写）指的是在继承过程中，子类对父类中已有方法的重新定义或替换。

子类可以根据需要修改或扩展父类的方法行为。

``` lua
-- 定义动物类（Animal）
Animal = {name = "Unknown"}

-- Animal 类的构造函数
function Animal:new(o, name)
  o = o or {}  -- 如果没有传入对象，则创建一个新的空表
  setmetatable(o, self)  -- 设置元表，使其继承 Animal 的方法
  self.__index = self  -- 让对象可以访问 Animal 的方法
  o.name = name or "Unknown"  -- 设置名称，默认为 "Unknown"
  return o
end

-- Animal 类的方法：叫声
function Animal:speak()
  print(self.name .. " makes a sound.")
end


-- 定义狗类（Dog），继承自 Animal
Dog = Animal:new()  -- Dog 继承 Animal 类

-- 重写狗类的构造函数
function Dog:new(o, name, breed)
  o = o or {}  -- 如果没有传入对象，则创建一个新的空表
  setmetatable(o, self)  -- 设置元表，使其继承 Dog 和 Animal 的方法
  self.__index = self  -- 让对象可以访问 Dog 的方法
  o.name = name or "Unknown"
  o.breed = breed or "Unknown"
  return o
end

-- 重写狗类的叫声方法（重写 Animal 的 speak 方法）
function Dog:speak()
  print(self.name .. " barks.")
end


-- 创建 Animal 对象
local animal = Animal:new(nil, "Generic Animal")
animal:speak()  -- 输出 "Generic Animal makes a sound."

-- 创建 Dog 对象
local dog = Dog:new(nil, "Buddy", "Golden Retriever")
dog:speak()  -- 输出 "Buddy barks."
```

Animal 类：定义了一个基础类 Animal，具有 name 属性和 speak 方法。speak 方法是一个默认的实现，输出"某个动物发出声音"。

Dog 类继承 Animal：Dog 类继承自 Animal，并通过 Dog:new() 方法创建自己的实例。

重写 speak 方法：在 Dog 类中，重写了 speak 方法，将其行为从父类的"发出声音"改为"狗狗叫"。这就是方法重写的体现，子类（Dog）改变了父类（Animal）方法的行为。

### 多态

Lua 的多态性通过元表和方法重写实现。当不同类型的对象调用相同的方法时，Lua 会根据对象的实际类型执行不同的方法。

``` lua
-- 定义一个"类"（实际上是一个表）
Person = {}

-- 为"类"添加一个构造函数
function Person:new(name, age)
    local obj = {}  -- 创建一个新的表作为对象
    setmetatable(obj, self)  -- 设置元表，表示它是Person类的实例
    self.__index = self  -- 设置索引元方法，指向Person
    obj.name = name
    obj.age = age
    return obj
end

-- 添加方法
function Person:greet()
    print("Hello, my name is " .. self.name)
end

-- 定义一个子类 Student 继承自 Person
Student = Person:new()

-- 子类重写父类的方法
function Student:greet()
    print("Hi, I'm a student and my name is " .. self.name)
end

local person2 = Person:new("Charlie", 25)
local student2 = Student:new("David", 18)

-- 多态：不同类型的对象调用相同的方法
person2:greet()  -- 输出 "Hello, my name is Charlie"
student2:greet()  -- 输出 "Hi, I'm a student and my name is David"
```

### 封装

封装通常通过将数据和方法封装在一个表中实现。我们可以通过控制表的访问权限来模拟封装，例如使用 metamethods 来限制外部访问。

``` lua
-- 定义一个"类"（实际上是一个表）
Person = {}

-- 添加封装：隐藏属性
function Person:new(name, age)
    local obj = {}
    setmetatable(obj, self)
    self.__index = self
    obj.name = name
    obj.age = age
    return obj
end

function Person:setName(name)
    self.name = name  -- 提供方法来修改 name
end

function Person:getName()
    return self.name  -- 提供方法来获取 name
end
```

### 抽象

抽象指的是简化复杂的事物，将不需要的细节隐藏。虽然 Lua 本身没有类的概念，但我们可以通过封装来达到抽象的目的。

``` lua
-- 只暴露接口，不暴露实现细节
function Person:showInfo()
    print("Name: " .. self.name)
    print("Age: " .. self.age)
end
```

## Lua 数据库访问

介绍 Lua 数据库的操作库：LuaSQL。他是开源的，支持的数据库有：ODBC, ADO, Oracle, MySQL, SQLite 和 PostgreSQL。

LuaSQL 可以使用 LuaRocks 来安装可以根据需要安装你需要的数据库驱动。

LuaRocks 安装方法：

``` sh
$ wget http://luarocks.org/releases/luarocks-2.2.1.tar.gz
$ tar zxpf luarocks-2.2.1.tar.gz
$ cd luarocks-2.2.1
$ ./configure; sudo make bootstrap
$ sudo luarocks install luasocket
$ lua
Lua 5.3.0 Copyright (C) 1994-2015 Lua.org, PUC-Rio
> require "socket"
```

安装不同的数据库驱动：

``` sh
luarocks install luasql-sqlite3
luarocks install luasql-postgres
luarocks install luasql-mysql
luarocks install luasql-sqlite
luarocks install luasql-odbc
```

Lua 连接 MySQL 数据库

``` lua
require "luasql.mysql"

--创建环境对象
env = luasql.mysql()

--连接数据库
conn = env:connect("数据库名","用户名","密码","IP地址",端口)

--设置数据库的编码格式
conn:execute"SET NAMES UTF8"

--执行数据库操作
cur = conn:execute("select * from role")

row = cur:fetch({},"a")

--文件对象的创建
file = io.open("role.txt","w+");

while row do
    var = string.format("%d %s\n", row.id, row.name)

    print(var)

    file:write(var)

    row = cur:fetch(row,"a")
end


file:close()  --关闭文件对象
conn:close()  --关闭数据库连接
env:close()   --关闭数据库环境
```
