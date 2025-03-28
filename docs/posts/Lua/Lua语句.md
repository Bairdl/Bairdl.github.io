---
description: Lua 支持常规的语句，语句包括块、赋值、控制结构、函数调用和变量声明等。
tags:
  - lua
---

# Lua 语句（Statements）

Lua 支持一组常规的语句，类似于其他编程语言。这组语句包括块、赋值、控制结构、函数调用和变量声明等。

---

## 1. 块（Blocks）

### 定义

块是一系列语句列表，这些语句按顺序执行：

```lua
block ::= {stat}
```

在 Lua 中，空语句允许你通过分号来分隔语句，也可以以分号开始一个块或连续写两个分号：

```lua
stat ::= ‘;’
```

#### 示例

```lua
do
    print("First statement")
    ;(print or io.write)("Second statement") -- 使用分号避免语法歧义
end
```

### 显式块

显式块可以用来明确界定一个代码块，以便控制变量声明的作用域：

```lua
stat ::= do block end
```

#### 示例

```lua
do
    local x = 10
    print(x) -- 输出 10
end
-- print(x) -- 这里会报错，因为 x 在显式块外不可见
```

---

## 2. 代码块（Chunks）

### 定义

Lua 的编译单元称为代码块（chunk）。从语法上讲，一个代码块就是一个块：

```lua
chunk ::= block
```

Lua 将一个代码块视为匿名函数的主体，它可以定义局部变量、接收参数并返回值。匿名函数是在外部局部变量 `_ENV` 的作用域内编译的。

#### 示例

```lua
local function chunk()
    local x = "Hello, World!"
    print(x)
end
chunk() -- 调用代码块
```

---

## 3. 赋值（Assignment）

### 定义

Lua 支持多重赋值。因此，赋值语句定义了左侧的变量列表和右侧的表达式列表：

```lua
stat ::= varlist ‘=’ explist
varlist ::= var {‘,’ var}
explist ::= exp {‘,’ exp}
```

在赋值之前，值列表会根据变量的数量进行调整。

#### 示例

```lua
local i = 3
i, a[i] = i + 1, 20 -- 设置 a[3] 为 20，不影响 a[4]
```

### 全局变量赋值

全局变量赋值 `x = val` 等价于 `_ENV.x = val`。

#### 示例

```lua
_ENV.myGlobalVar = "This is a global variable"
print(myGlobalVar) -- 输出 "This is a global variable"
```

---

## 4. 控制结构（Control Structures）

Lua 提供了常见的控制结构如 `if`、`while` 和 `repeat`，它们具有通常的意义和熟悉的语法：

```lua
stat ::= while exp do block end
stat ::= repeat block until exp
stat ::= if exp then block {elseif exp then block} [else block] end
```

### 条件表达式

条件表达式可以返回任何值。只有 `false` 和 `nil` 被认为是假；所有其他值都被认为是真的，包括数字 `0` 和空字符串。

#### 示例

```lua
local condition = true
if condition then
    print("Condition is true")
else
    print("Condition is false")
end
```

### 循环控制

- `break`：终止最近的循环。
- `return`：用于从函数或代码块中返回值。

#### 示例

```lua
function example()
    for i = 1, 5 do
        if i == 3 then break end
        print(i)
    end
    return "Finished"
end
print(example()) -- 输出 1, 2, Finished
```

---

## 5. For 循环（For Statement）

Lua 的 `for` 循环有两种形式：数值型和泛型。

### 数值型 For 循环

```lua
stat ::= for Name ‘=’ exp ‘,’ exp [‘,’ exp] do block end
```

#### 示例

```lua
for i = 1, 5, 2 do
    print(i) -- 输出 1, 3, 5
end
```

### 泛型 For 循环

```lua
stat ::= for namelist in explist do block end
namelist ::= Name {‘,’ Name}
```

#### 示例

```lua
for k, v in pairs({a = 1, b = 2}) do
    print(k, v) -- 输出键值对 a 1 和 b 2
end
```

---

## 6. 函数调用作为语句（Function Calls as Statements）

为了允许可能的副作用，函数调用可以作为语句执行：

```lua
stat ::= functioncall
```

在这种情况下，所有返回值都会被丢弃。

### 示例

```lua
function sayHello()
    print("Hello!")
end
sayHello() -- 函数调用作为语句，无返回值处理
```

---

## 7. 局部声明（Local Declarations）

局部变量可以在块内的任何地方声明，并且声明时可以包含初始化：

```lua
stat ::= local attnamelist [‘=’ explist]
attnamelist ::=  Name attrib {‘,’ Name attrib}
```

如果存在初始化，则其语义与多重赋值相同。否则，所有变量都初始化为 `nil`。

### 示例

```lua
local x, y = 10, 20
print(x, y) -- 输出 10, 20
```

### 属性

每个变量名后面可以跟一个属性（位于尖括号之间），有两个可能的属性：

- `const`：声明一个常量变量，即不能在其初始化后再次赋值的变量。
- `close`：声明一个待关闭变量。

#### 示例

```lua
local closeMe <close> = setmetatable({}, {__close = function() print("Closing!") end})
do return end -- 触发 closeMe 的 __close 方法
```

---

## 8. 待关闭变量（To-be-closed Variables）

待关闭变量的行为类似于常量局部变量，但在变量超出作用域时会自动调用其 `__close` 元方法。

### 示例

```lua
local resource <close> = setmetatable({}, {__close = function(self) print("Resource closed.") end})
-- 当资源超出作用域时，__close 方法会被自动调用
```
