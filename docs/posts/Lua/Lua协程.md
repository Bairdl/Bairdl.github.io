---
description: Lua 协程介绍
tags:
  - lua
---

# Lua 协程（Coroutines）

Lua 支持协程，也称为协作多线程。  
Lua 中的协程表示独立的执行线程。  
**与多线程系统中的线程不同，协程仅支持通过显式调用 `yield` 函数来暂停其执行。**  

## 一、创建协程  

### create 函数

通过调用 `coroutine.create` 函数来创建协程。  
该函数的唯一参数为一个 function，其为该协程的主函数。  
该函数会创建一个新的协程并返回一个指向它的句柄（handle）（一个 thread 类型的对象）。**它不会启动协程。**  

### wrap 函数

通过调用 `coroutine.wrap` 函数来创建线程，但它并不直接返回协程对象，而是返回一个函数。  

函数作用：

- 每次调用这个函数时，都会恢复协程的执行。（类似于 `coroutine.resume`，其内部实际上也是调用了 `coroutine.resume` 来恢复协程执行。）
- 传递给这个函数的参数都会作为额外参数传递给 `coroutine.resume` 。
- `coroutine.wrap` 返回的是 `coroutine.resume` 的所有返回值，除了第一个布尔值。也就是说，`coroutine.wrap` 直接返回协程的输出结果。

`coroutine.wrap`不会捕获错误，而是将错误直接传播给调用者，导致程序抛出异常。此外，在这种情况下，协程会被自动关闭（类似于调用 `coroutine.close`）。

## 二、执行协程

通过调用 `coroutine.resume` 函数来执行协程。  
当首次调用 `coroutine.resume` 时，将 `coroutine.create` 返回的线程对象作为其第一个参数传递。协程会通过调用其 `main` 函数来开始执行。  
**传递给 `coroutine.resume` 的额外参数将作为参数传递给主函数。**  
**协程开始运行后，会持续运行，直到终止或 `yield` 暂停执行。**  

## 三、终止协程

终止协程的执行通常有两种方式：

1. 其主函数返回，返回 `true` + 主函数返回的任何值。
2. 出现无保护的错误，返回 `false` + error 对象。

## 四、暂停协程

通过调用 `coroutine.yield` 函数来暂停协程。  
当协程 yield 时，会将控制权返回给调用了 `coroutine.resume` 的代码。  

当一个协程通过调用 `coroutine.yield` 暂停时，对应的 `coroutine.resume` 调用会立即返回，即使 `coroutine.yield` 是在被主函数直接或间接调用的嵌套函数内部发生的。如果是因为 yield 导致返回，`coroutine.resume` 也会返回 true，以及传递给 `coroutine.yield` 的任何值。  

调用 `coroutine.yield` 后，为了使协程继续执行，你需要再次调用 `coroutine.resum`e。下一次当你对同一个协程调用 `coroutine.resume` 时，它会从上次 `coroutine.yield` 被调用的地方继续执行，而 `coroutine.yield` 调用则会返回传给 `coroutine.resume` 的任何额外参数（新参数）。  

## 五、协程工作示例代码

``` lua
     function foo (a)
       print("foo", a)
       return coroutine.yield(2*a)
     end
     
     co = coroutine.create(function (a,b)
           print("co-body", a, b)
           local r = foo(a+1)
           print("co-body", r)
           local r, s = coroutine.yield(a+b, a-b)
           print("co-body", r, s)
           return b, "end"
     end)
     
     print("main", coroutine.resume(co, 1, 10))
     print("main", coroutine.resume(co, "r"))
     print("main", coroutine.resume(co, "x", "y"))
     print("main", coroutine.resume(co, "x", "y"))
```

输出结果：

``` plaintext
     co-body 1       10
     foo     2
     main    true    4
     co-body r
     main    true    11      -9
     co-body x       y
     main    true    10      end
     main    false   cannot resume dead coroutine
```

## 六、通过 C API来创建和操作协程

- `lua_newthread`
- `lua_resume`
- `lua_yield`
