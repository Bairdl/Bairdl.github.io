---
description: Lua 的基础信息的介绍，包括值和基本类型，错误处理等。
tags:
  - lua
---

# Lua 基础

## 1. 简介

Lua 是一种脚本语言，特点为高效、轻量级、可嵌入。  

它支持面向过程编程、面向函数编程、函数式编程、数据驱动编程和数据描述。

Lua 是**动态类型**的，使用基于寄存器的虚拟机**解释字节码运行**，并且具有自动内存管理和分代垃圾回收功能。

Lua 是作为一个库来实现的，使用 C 进行编写，是 C 和 C++ 的一个子集。

作为一种扩展语言，**Lua没有 `main` 程序的概念**：它嵌入在主机客户端中工作，称为嵌入程序或者简称主机。

### 高效

1. 虚拟机（VM）与即时编译（JIT）：
   - Lua使用基于寄存器的虚拟机，这比基于栈的虚拟机更接近于真实的硬件架构，使得执行效率更高。
   - LuaJIT是一个非常著名的Lua扩展，它提供了即时编译技术，能够将Lua代码实时编译成本地机器码，显著提升了性能，特别是对于计算密集型任务。
2. 紧凑的数据结构：
   - Lua中的table是其唯一的数据结构，它被精心设计以支持数组、集合、记录等多种数据组织方式，并且内存占用小，访问速度快。
   - Lua还提供了一些优化机制，比如字符串内部化，确保相同的字符串在内存中只存储一份副本，减少内存消耗并加快字符串比较速度。
3. 增量式垃圾回收：
   - Lua采用了增量式的垃圾回收算法，该算法可以在程序运行过程中逐步清理不再使用的对象，避免了长时间的停顿，提高了整体执行效率。

### 轻量级

1. 简洁的核心库：
   - Lua的标准库非常小巧，仅包含了最基本的功能，如基本输入输出、数学运算等。这不仅减少了Lua本身的体积，也使其更加灵活，可以根据具体需求添加或移除模块。
2. 低资源消耗：
   - Lua的设计目标之一就是低资源消耗，它的解释器可以运行在内存和处理能力都非常有限的环境中。例如，在嵌入式系统中，Lua解释器可以很容易地集成进去，而不会对系统的性能造成太大影响。
3. 易于学习与使用：
   - Lua语法简单直观，容易上手，降低了开发者的入门门槛，同时也意味着编写出来的代码通常也比较简洁，便于维护和优化。

### 可嵌入性

1. C API：
   - Lua拥有一个强大且易用的C API，允许开发者轻松地将Lua嵌入到其他应用程序中。通过这个API，不仅可以调用Lua脚本，还能让Lua脚本访问宿主应用的数据结构和函数。
2. 灵活性：
   - Lua的设计考虑到了高度的灵活性，允许用户根据需要定制和扩展Lua的行为。无论是修改默认的内存管理策略，还是增加新的数据类型，都可以通过适当的接口实现。
3. 跨平台兼容性：
   - Lua本身是完全用ANSI C编写的，这意味着它可以几乎无改动地移植到任何支持C语言的平台上，极大地增强了其可嵌入性和适用范围。

## 2. 基本概念

### 2.1 值和类型

Lua 是一种**动态类型语言**，所以其变量没有类型。在该语言中没有类型定义，所以值都有自己的类型。

Lua 中有八种基本数据类型：

1. `nil`：
   1. 通常表示缺少有用的值。
   2. （nil 在条件判断中会使条件为false）
2. `boolean`：
   1. false 和 true 。
   2. *（false 和 nil 都会使条件为 false 所以称为 false 值，任何其他值都使条件为 true。）*
3. `number`：表示整数和实数（浮点）数。
   1. `integer`：64 位整数
   2. `float`：双精度（64位）浮点数。
4. `string`：表示不可变的字节序列。
   1. 可以包含任何 8 比特的值，即使是 `\0`。(通常被视为字符串结束标志的零字符也可以存在于字符串内部。)
   2. **Lua 字符串也是与编码无关的，不对字符串的内容做出任何假设。**（但是字符串的长度必须要能够用 Lua 整数来表示。）
5. `function`：
   1. Lua 可以调用和操作用 Lua 编写的函数和用 C 编写的函数。二者都使用 `function` 类型表示。
6. `userdata`：
   1. `userdata` 类型用来允许任意的 C 数据被存储到Lua 变量中。一个 `userdata` 值表示一个原始内存块。
   2. `full userdata`：一个由 Lua 管理的内存块的对象。
   3. `light userdata`：一个 C 指针值。
   4. **`userdata` 值不能在 Lua 中创建或修改，只能通过 C API 进行。**（可以通过 C++ API吗）？
7. `thread`：
   1. `thread` 类型表示独立的执行线程，它用于实现协程。
   2. Lua 线程与操作系统线程无关。它支持所有系统上协程，即使是那些本身不支持线程的协程。
   3. **协程的切换是由程序自身控制的，通过显式调用来实现。本质上是串行执行的。**
8. `table`
   1. `table` 实现了关联熟手，即不仅可以将数字作为数组的索引，还可以将**除 `nil` 和 `Nan` 之外的任何 Lua 值作为索引** 。
   2. `tables` 是 Lua 中唯一的数据结构机制：可用于表示普通数组、列表、符号表、集合、记录、图、树等。
   3. 为了表示记录，Lua 使用字段名来作为索引，即提供 `a.name`（语法糖） 来表示 `a["name"]`。
   4. 表字段也可以是任何类型，包括函数。

**`table`、`function`、`thread`和 `full userdata` 是对象，变量实际上存储的是对其的引用。**

### 2.2 环境和全局环境

在 Lua 中，任何对自由变量（在当前作用域内没有通过声明绑定到任何值的名字）`var` 的使用，都语法上都会转换为 `_ENV.var`。

用作 `_ENV` 值的任何表都被成为环境。  

Lua 保留了一个全局环境，该值保存在 C 注册表中的特殊索引处。  
在 Lua 中，全局变量 `_G` 使用相同的值进行初始化。

当 Lua 加载一个块时，其 `_ENV` 变量的默认值是全局环境。  
因此默认情况下，Lua 代码中的 free 名称指向的是全局环境中的条目，也就是全局变量。  

### 2.3 错误处理

Lua 代码可以通过调用 `error` 函数显式抛出一个错误。并且，`error` 函数一旦被调用，就永远不会返回。(`error` 会终止程序的执行，除非有 `pcall` 或 `xpcall` 来捕获错误并进行处理。)

要捕获 Lua 中的错误，可以使用  `pcall` （或 `xpcall`） 来进行受保护的调用。  
函数 `pcall` 在保护模式下调用给定函数。运行函数中的任何错误都会停止其执行，并且 control 会立即返回到 `pcall` 返回状态代码。

#### 错误对象

当 Lua 发生错误时，会生成一个**错误对象**，并将其传播给调用者。这个错误对象包含了关于错误的信息。  
例如：  

``` lua
local x = nil
print(x.foo)  -- 尝试访问 nil 的字段，触发错误
```

输出：  

``` plaintext
lua: test.lua:2: attempt to index a nil value (local 'x')
stack traceback:
    test.lua:2: in main chunk
    ...
```

- 错误对象可以是**任何类型的值**：字符串、数字、表（table）、函数等。
- 默认情况下，Lua 自己生成的错误对象是一个字符串（例如 "attempt to index a nil value"），但程序可以生成任意类型的错误对象。(开发者可以通过 error 函数生成任意类型的错误对象。)

``` lua
error("This is a string error")  -- 字符串错误对象
error(42)                        -- 数字错误对象
error({code = 500, msg = "Internal Error"})  -- 表作为错误对象
```

#### 错误对象的处理

错误对象的处理由 Lua 程序或其宿主环境（host environment）负责。这意味着：

- 如果使用 `pcall` 或 `xpcall` 捕获错误，捕获到的错误对象可以被进一步分析和处理。
- 宿主环境（比如嵌入 Lua 的 C 程序）也可以根据错误对象的类型执行特定的操作。

例如，使用 `pcall` 捕获错误并处理非字符串类型的错误对象：

``` lua
local status, err = pcall(function()
    error({code = 500, msg = "Something went wrong!"})
end)

if not status then
    if type(err) == "table" then
        print("Error code:", err.code)
        print("Error message:", err.msg)
    else
        print("Unknown error:", err)
    end
end
```

输出：  

``` plaintext
Error code: 500
Error message: Something went wrong!
```

#### 消息处理器

`xpcall` 函数：这是 Lua 提供的一个内置函数，允许你以保护模式调用另一个函数，并且可以指定一个消息处理器来处理任何发生的错误。与 `pcall` 不同，`pcall` 只能捕获错误并返回原始的错误对象，而 `xpcall` 允许你在错误发生后通过消息处理器对错误对象进行进一步处理。

示例：

``` lua
-- 定义一个消息处理器
local function my_message_handler(err)
    -- 返回带有堆栈跟踪信息的新错误对象
    return debug.traceback("Error: " .. tostring(err), 2)
end

-- 使用 xpcall 调用可能抛出错误的函数，并使用自定义的消息处理器
local status, result = xpcall(function()
    error("Something went wrong!")
end, my_message_handler)

if not status then
    print(result)  -- 打印经过消息处理器处理后的错误信息
end
```
