---
description: Lua 中有三种类型的变量：全局变量、局部变量和表字段。
tags:
  - lua
---

# Lua 变量（Variables）

变量是用于存储值的容器。Lua 中有三种类型的变量：**全局变量**、**局部变量** 和 **表字段**。

---

## 全局变量与局部变量

### 定义

- **全局变量**：未明确声明为局部变量的变量默认为全局变量。
- **局部变量**：通过 `local` 关键字显式声明，具有词法作用域（lexical scoping）。

变量名由标识符表示，其语法形式如下：

```plaintext
var ::= Name
```

#### 示例

```lua
x = 10                  -- 全局变量
local y = 20            -- 局部变量
function test()
    print(x)            -- 访问全局变量
    print(y)            -- 访问局部变量
end
test()
```

### 变量的作用域

- **全局变量**：可以在程序的任何地方访问，除非被覆盖或删除。
- **局部变量**：仅在其定义的作用域内可见。局部变量可以被定义在它们作用域内的函数自由访问。

#### 示例

```lua
local a = 5             -- 局部变量
function outer()
    local b = 10        -- 局部变量，属于 outer 的作用域
    function inner()
        print(a, b)     -- inner 可以访问 outer 的局部变量 b 和外部的局部变量 a
    end
    inner()
end
outer()
```

### 初始值

在第一次赋值之前，变量的值默认为 `nil`。

#### 示例

```lua
print(x)                -- 输出 nil，因为 x 尚未赋值
x = 42
print(x)                -- 输出 42
```

---

## 表字段（Table Fields）

表字段是 Lua 中一种特殊的变量，用于存储表中的值。表字段可以通过两种方式访问：

1. 使用方括号 `[ ]` 进行索引。
2. 使用点运算符 `.` 访问。

### 方括号索引

方括号用于动态访问表字段，其语法如下：

```plaintext
var ::= prefixexp ‘[’ exp ‘]’
```

#### 示例

```lua
local t = { key1 = "value1", key2 = "value2" }
print(t["key1"])        -- 输出 value1
local key = "key2"
print(t[key])           -- 动态访问，输出 value2
```

### 点运算符访问

点运算符是方括号索引的语法糖，适用于静态字段名。其语法如下：

```plaintext
var ::= prefixexp ‘.’ Name
```

#### 示例

```lua
local t = { name = "Alice", age = 30 }
print(t.name)           -- 输出 Alice
print(t.age)            -- 输出 30
```

### 元表对表字段访问的影响

表字段的访问行为可以通过元表（metatable）进行修改。例如，使用 `__index` 元方法可以定义当字段不存在时的行为。

#### 示例

```lua
local t = {}
local mt = {
    __index = function(table, key)
        return "Default value for " .. key
    end
}
setmetatable(t, mt)

print(t.nonExistentKey) -- 输出 Default value for nonExistentKey
```

---

## 全局变量与 `_ENV`

在 Lua 中，全局变量实际上是通过一个特殊的表 `_ENV` 来实现的。对全局变量 `x` 的访问等价于 `_ENV.x`。

由于 Lua 的编译机制，`_ENV` 本身并不是全局变量，而是每个代码块（chunk）的一个隐式局部变量。

### 示例

```lua
_ENV = { x = 42 }       -- 替换当前的环境表
print(x)                -- 输出 42

-- 恢复默认环境
_ENV = _G
print(x)                -- 输出 nil，因为 x 不再存在于默认环境中
```

### `_G` 表

`_G` 是 Lua 的全局环境表，默认情况下所有全局变量都存储在 `_G` 中。

#### 示例

```lua
_G.myGlobal = "Hello, World!"
print(myGlobal)         -- 输出 Hello, World!
print(_G.myGlobal)      -- 同样输出 Hello, World!
```

---

## 总结

Lua 的变量分为三类：全局变量、局部变量和表字段。每种变量都有其特定的用途和作用域规则：

- **全局变量** 默认在程序的任何地方可见，但可能会导致命名冲突。
- **局部变量** 提供了更好的封装性和性能，推荐优先使用。
- **表字段** 是 Lua 数据结构的核心，支持动态访问和元表扩展。
