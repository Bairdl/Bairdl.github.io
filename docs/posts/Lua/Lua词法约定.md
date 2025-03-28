---
description: 详细介绍 Lua 的词法规约，包括标识符、关键字、字符串、数字常量、注释等。
tags:
  - lua
---

# Lua 词法约定（Lexical Conventions）

Lua 是一种自由格式的语言，具有灵活的词法规则。

---

## 1. 标识符（Identifiers）

### 定义

标识符是用于命名变量、表字段和标签的名称。在 Lua 中，标识符可以由以下字符组成：

- 拉丁字母（A-Z 和 a-z）
- 阿拉伯数字（0-9）
- 下划线（`_`）

**规则：**

- 标识符不能以数字开头。
- 标识符不能是 Lua 的保留字（见下一节）。
- Lua 是区分大小写的语言，例如 `and` 是一个保留字，而 `And` 和 `AND` 是两个不同的有效标识符。

### 命名约定

虽然 Lua 不强制要求，但为了代码可读性和避免冲突，建议遵循以下约定：

- **避免使用以下划线开头并跟随大写字母的名称**（如 `_VERSION`），因为这种命名方式通常被 Lua 内部使用。

### 标识符示例

```lua
local myVariable = 42        -- 有效的标识符
local _myVar = "Hello"       -- 有效的标识符
local _MYCONSTANT = 3.14     -- 有效的标识符，但不推荐
local 123var = 10            -- 错误：标识符不能以数字开头
```

---

## 2. 关键字（Keywords）

Lua 的关键字是保留的，不能用作标识符。以下是 Lua 的所有关键字：

```plaintext
and       break     do        else      elseif    end
false     for       function  goto      if        in
local     nil       not       or        repeat    return
then      true      until     while
```

### 关键字示例

```lua
if true then
    print("This is valid")
end

-- local function = 42  -- 错误：function 是关键字，不能用作标识符
```

---

## 3. 字符串（Strings）

Lua 支持两种类型的字符串：短字符串和长字符串。

### 短字符串

短字符串由单引号（`'`）或双引号（`"`）包围，支持 C 风格的转义序列。

#### 转义序列

以下是一些常用的转义序列：

- `\a`：响铃
- `\b`：退格
- `\f`：换页
- `\n`：换行
- `\r`：回车
- `\t`：水平制表符
- `\v`：垂直制表符
- `\\`：反斜杠
- `\"`：双引号
- `\'`：单引号

#### 特殊规则

- 反斜杠后跟换行符会生成一个换行符。
- `\z` 会跳过随后的一段空白字符（包括换行符），适合拆分长字符串而不引入额外的换行和空格。

#### 短字符串示例

```lua
local str1 = 'Hello\nWorld'         -- 包含换行符的字符串
local str2 = "Path with backslash:\\folder"
local str3 = '\u{4e2d}\u{6587}'    -- Unicode 字符：中文
local str4 = '\x48\x65\x6C\x6C\x6F' -- 十六进制 ASCII 编码的字符串 "Hello"

-- 使用 \z 跳过多余的空白
local longStr = "This is a very long string that we want to split \
\z
    without adding extra newlines."
print(longStr)
```

### 长字符串

长字符串由长括号（`[[...]]` 或 `[=[...]=]` 等）定义，可以跨越多行且不解释任何转义序列。

#### 层次定义

- 开始长括号：`[=` 加上若干等号 `=` 再加上 `[`，例如 `[[` 表示第 0 层，`[=[` 表示第 1 层。
- 结束长括号：与开始长括号匹配，例如 `]]` 或 `]=]`。

#### 长字符串示例

```lua
local level0 = [[
这是一个
第一层长字符串
]]

local level1 = [=[
这是
第二层长字符串
]=]

-- 注意：开长括号后紧跟换行符时，换行符不会包含在字符串中
local noNewline = [[
Hello, World!
]]
print(noNewline)  -- 输出没有多余的换行符
```

---

## 4. 数字常量（Numeric Constants）

Lua 支持十进制和十六进制的数字常量，分为整数和浮点数。

### 十进制常量

- 整数：无小数点或指数部分。
- 浮点数：有小数点或指数部分。

#### 十进制常量示例

```lua
local int1 = 42               -- 整数
local float1 = 3.14           -- 浮点数
local float2 = 314.16e-2      -- 科学计数法表示的浮点数
```

### 十六进制常量

- 以 `0x` 或 `0X` 开头。
- 支持小数部分和二进制指数（以 `p` 或 `P` 标记）。

#### 十六进制常量示例

```lua
local hexInt = 0xFF            -- 十六进制整数
local hexFloat = 0x1.fp10      -- 十六进制浮点数，等于 1984
```

---

## 5. 注释（Comments）

Lua 支持两种类型的注释：短注释和长注释。

### 短注释

以双连字符（`--`）开头，持续到行尾。

#### 短注释示例

```lua
-- 这是一个短注释
local x = 10  -- 变量声明后的注释
```

### 长注释

以 `--[=[` 开头，以 `]=]` 结束，可以跨越多行。

#### 长注释示例

```lua
--[[
这是一个
多行长注释
]]
```

---

## 6. 其他标记（Other Tokens）

Lua 中还有一些特殊符号，用于语法结构。以下是一些常见的标记：

```plaintext
+     -     *     /     %     ^     #
&     ~     |     <<    >>    //
==    ~=    <=    >=    <     >     =
(     )     {     }     [     ]     ::
;     :     ,     .     ..    ...
```

### 其他标记示例

```lua
local a = 10 + 5
local b = a == 15 and true or false
local c = { key = "value" }
```

### 算术运算符

- `+`：加法运算符。
- `-`：减法运算符。
- `*`：乘法运算符。
- `/`：除法运算符。
- `%`：取模运算符（求余数）。
- `^`：幂运算符。

#### 示例

```lua
local sum = 5 + 3       -- 8
local difference = 5 - 2 -- 3
local product = 4 * 2   -- 8
local quotient = 10 / 2 -- 5.0
local remainder = 9 % 4 -- 1
local power = 2 ^ 3     -- 8 (2 的 3 次方)
```

### 位运算符

- `&`：按位与。
- `~`：按位异或。
- `|`：按位或。
- `<<`：左移。
- `>>`：右移。

#### 示例

```lua
local bitwiseAnd = 5 & 3    -- 1
local bitwiseXor = 5 ~ 3    -- 6
local bitwiseOr = 5 | 3     -- 7
local leftShift = 5 << 1    -- 10 (相当于 5 * 2)
local rightShift = 8 >> 1   -- 4 (相当于 8 / 2，但结果是整数)
```

### 关系运算符

- `==`：等于。
- `~=`：不等于。
- `<=`：小于等于。
- `>=`：大于等于。
- `<`：小于。
- `>`：大于。

#### 示例

```lua
local isEqual = 5 == 5      -- true
local isNotEqual = 5 ~= 3   -- true
local isLessThanOrEqual = 5 <= 3 -- false
local isGreaterThanOrEqual = 5 >= 3 -- true
local isLessThan = 5 < 3    -- false
local isGreaterThan = 5 > 3 -- true
```

### 赋值运算符

- `=`：赋值运算符。

#### 示例

```lua
local x = 10               -- 将数值 10 赋给变量 x
x = x + 1                 -- 增加 x 的值
```

### 分组符号

- `(` 和 `)`：用于改变运算优先级或函数调用。

#### 示例

```lua
local result = (5 + 3) * 2 -- 16
print(math.max(1, 2, 3))   -- 函数调用
```

### 表构造符号

- `{` 和 `}`：用于创建表（table），Lua 中的数据结构。

#### 示例

```lua
local t = { key = "value", 10, 20 } -- 创建一个包含字段 'key' 和两个元素的表
```

### 索引符号

- `[` 和 `]`：用于访问表中的元素。

#### 示例

```lua
local t = { a = 1, b = 2 }
print(t["a"])              -- 输出 1
print(t.a)                 -- 同样输出 1
```

### 名称空间分隔符

- `::`：通常用于模块中定义全局变量时，但在现代 Lua 编程实践中较少使用。

### 语句终止符

- `;`：可选的语句结束符，在一行中有多个语句时使用。

#### 示例

```lua
local x = 1; local y = 2  -- 在同一行定义多个局部变量
```

### 字段分隔符

- `,`：用于分隔列表中的元素或参数列表中的参数。
- `.`：用于访问对象的属性或方法。

#### 示例

```lua
local t = { x = 1, y = 2 } -- 使用逗号分隔表字段
print(string.format("%d %d", t.x, t.y)) -- 使用点运算符访问表字段
```

### 连接运算符

- `..`：字符串连接运算符。

#### 示例

```lua
local str = "Hello" .. " World!" -- "Hello World!"
```

### 可变参数

- `...`：表示函数中的可变参数。

#### 示例

```lua
function concat(...)
    return table.concat({...})
end
print(concat("Hello", " ", "World")) -- Hello World
```
