---
description: Lua 的元表、元方法等的介绍。
tags:
  - lua
---

# Lua 元表和元方法

## 元表

Lua 中的每个值都可以有一个元表（`metatable`）。  
元素是普通的 Lua 表，它定义了原始值在某些事件下的行为。  
开发者可以通过在值的元表中**设置特定字段来覆盖其原有的行为**。  
例如，当非数字值执行加法时，Lua 会检查该值的元表的字段 `__add` 中的函数。如果找到一个，Lua 会调用这个函数来执行加法。  
*(有点类似于 Java 中重载运算符的函数。)*  

### 查询元表

`getmetatable` 是一个 Lua 函数，它可以用来获取给定值的元表。元表是一个特殊的表，它定义了对原始值进行某些操作的行为（例如算术运算、比较、字符串连接等）。

示例

``` lua
local t = {}
local mt = { __index = function(t, k) return 'default value' end }
setmetatable(t, mt)

print(getmetatable(t))  -- 输出元表 mt 的引用
```

在这个例子中，我们为表 t 设置了一个元表 mt，然后使用 `getmetatable` 来获取并打印这个元表。

### 替换元素

可以使用 `setmetatable` 函数来替换表（table）的元表（metatable）。  
然而，对于其他类型的数据（例如数字、字符串、函数等），你不能直接从 Lua 代码中更改它们的元表，除非借助于调试库（debug library）。

### 值与元表

#### 表和用户数据

表（table） 和 完整用户数据（full userdata） 每个实例都可以有自己的元表。  
（可以为每个表或用户数据单独设置一个元表，也可以让多个表或用户数据共享同一个元表。）

##### 设置元表示例

为每个表设置一个元表。

```lua
local t1 = {}
local t2 = {}

local mt1 = { __index = function() return "t1's metatable" end }
local mt2 = { __index = function() return "t2's metatable" end }

setmetatable(t1, mt1)
setmetatable(t2, mt2)

print(getmetatable(t1)) -- 输出 mt1
print(getmetatable(t2)) -- 输出 mt2
```

为多个表设置共享同一个元表

``` lua
local mt = { __index = function() return "shared metatable" end }

local t1 = setmetatable({}, mt)
local t2 = setmetatable({}, mt)

print(getmetatable(t1) == getmetatable(t2)) -- 输出 true
```

#### 其他类型

对于其他类型（如数字、字符串、布尔值、函数等），Lua 不允许为单个值设置独立的元表。  
每种类型的所有值共享一个全局的元表。
如果你想为某种类型设置元表，通常需要通过调试库（debug.setmetatable）来实现。

## 元方法

元表中每个事件的键都是一个字符串，事件名称以两个下划线作为前缀，相应的值名称为 `metavalue`。  
对于大多数事件，元表值必须是一个函数，被称为元方法。  

除非另有说明，元方法（metamethod）实际上可以是任何可调用的值，这包括一个函数或者一个带有 `__call` 元方法的值。  

- 可调用的值：指的是可以在 Lua 中通过类似函数调用语法使用的值。最常见的可调用值就是函数。
- 带有 `__call` 元方法的值：在 Lua 中，如果一个值（例如表）设置了 `__call` 元方法，那么这个值也可以像函数一样被调用。这意味着你可以定义一个表，并为其设置 `__call` 元方法，然后就可以使用类似于调用函数的语法来“调用”这个表。

### 元方法示例

直接使用函数作为元方法

``` lua
local mt = {}
mt.__index = function(t, k)
    print("Accessing key:", k)
    return rawget(t, k)
end

local tbl = setmetatable({}, mt)

tbl.a = 1
print(tbl.a)  -- 输出 "Accessing key: a" 然后是 "1"
```

使用带有 `__call` 元方法的值作为元方法

``` lua
local callableTable = {}
callableTable.__call = function(t, ...)
    print("Called with arguments:", ...)
end

local mt = {}
mt.__add = callableTable  -- 将元方法 __add 设置为 callableTable

local tbl = setmetatable({}, mt)

-- 使用 + 运算符调用 __add 元方法，相当于调用了 callableTable 的 __call 方法
tbl + 5  -- 输出 "Called with arguments: 5"
```

### 常见的元方法及其作用

| 元方法名称 | 元方法作用 | 备注 |
| --- | --- | --- |
| `__add` | 加法 (+) 操作。如果任何加法的操作数不是数字，Lua 将尝试调用元方法。它首先检查第一个操作数（即使它是一个数字）；如果该操作数没有为 `__add` 定义元方法，则 Lua 将检查第二个操作数。如果找到元方法，则以两个操作数为参数调用它，调用的结果（调整为一个值）就是操作的结果。否则，如果未找到元方法，Lua 会引发错误。 | 行为类似于其他算术操作 |
| `__sub` | 减法 (-) 操作。行为类似于加法操作。 | - |
| `__mul` | 乘法 (*) 运算。行为类似于加法操作。 | - |
| `__div` | 除法 (/) 操作。行为类似于加法操作。 | - |
| `__mod` | 模 (%) 运算。行为类似于加法操作。 | - |
| `__pow` | 幂 (^) 运算。行为类似于加法操作。 | - |
| `__unm` | 否定（一元 -）操作。行为类似于加法操作。 | - |
| `__idiv` | 地板除 (//) 操作。行为类似于加法操作。 | - |
| `__band` | 按位 AND (&) 操作。行为类似于加法操作，不同之处在于，如果任何操作数既不是整数也不是可强制转换为整数的浮点数，则 Lua 将尝试元方法。 | - |
| `__bor` | 按位 OR ($&#124$) 操作。行为类似于按位 AND 操作。 | - |
| `__bxor` | 按位异或（binary ~）运算。行为类似于按位 AND 操作。 | - |
| `__bnot` | 按位 NOT（一元 ~）运算。行为类似于按位 AND 操作。 | - |
| `__shl` | 按位左移 (<<) 运算。行为类似于按位 AND 操作。 | - |
| `__shr` | 按位右移 (>>) 运算。行为类似于按位 AND 操作。 | - |
| `__concat` | 串联 (..) 操作。行为类似于加法操作，不同之处在于如果任何操作数既不是字符串也不是数字（它总是可以强制转换为字符串），Lua 将尝试元方法。 | - |
| `__len` | 长度 (#) 操作。如果对象不是字符串，Lua 将尝试其元方法。如果有元方法，Lua 以对象为参数调用它，调用的结果（总是调整为一个值）就是操作的结果。如果没有元方法，但对象是表，则 Lua 使用表长度操作。否则，Lua 会引发错误。 | - |
| `__eq` | 等于 (==) 运算。行为类似于加法操作，不同之处在于 Lua 只会在被比较的值是两个表或两个完整的 userdata 并且它们在原始上不相等时才会尝试元方法。调用的结果始终转换为布尔值。 | - |
| `__lt` | 小于 (<) 操作。行为类似于加法操作，不同之处在于 Lua 仅在被比较的值既不是两个数字也不是两个字符串时才会尝试元方法。此外，调用的结果始终转换为布尔值。 | - |
| `__le` | 小于等于 (<=) 操作。行为类似于小于操作。 | - |
| `__index` | 索引访问操作 table[key]。当 table 不是表或 table 中不存在 key 时，会发生此事件。在 table 的元表中查找 metavalue。 | 此事件的元值可以是函数、表或具有 __index 元值的任何值。 |
| `__newindex` | 索引分配 table[key] = value。与 index 事件一样，当 table 不是表或 table 中不存在 key 时，会发生此事件。在 table 的元表中查找 metavalue。 | 此事件的元值可以是函数、表或具有 __newindex 元值的任何值。 |
| `__call` | 调用操作 func(args)。当 Lua 尝试调用非函数值（即 func 不是一个函数）时，会发生此事件。在 func 中查找 metamethod。如果存在，则调用 metamethod，将 func 作为其第一个参数，后跟原始调用的参数（args）。所有结果都是操作的结果。这是唯一允许多个结果的 metamethod。 | - |
| `__gc` | 垃圾回收处理。 | - |
| `__close` | 关闭操作。 | - |
| `__mode` | 控制弱引用的行为。 | - |
| `__name` | 当它包含一个字符串时，可以由 tostring 和错误消息使用。 | - |
