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
