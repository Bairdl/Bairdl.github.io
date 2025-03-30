---
description: StringBuilder 的 insert 函数与 append 函数。
tags:
  - java
---

# StringBuilder 的 insert 和 append

`StringBuilder` 是 Java 中用于高效操作字符串的一个可变类。它的底层实现基于字符数组（`char[]`），通过动态扩展数组容量来支持高效的字符串拼接和插入操作。

## 1. **`append` 方法**

### **底层逻辑**

- **核心思想**：`append` 方法将新的内容追加到当前 `StringBuilder` 对象的末尾。
- **具体实现**：
  1. 检查当前字符数组是否还有足够的空间容纳新内容。如果没有，则调用 `ensureCapacityInternal` 方法扩展数组容量。
     - 扩容策略：如果需要扩容，新容量通常是原容量的两倍再加2（即 `newCapacity = (oldCapacity << 1) + 2`）。
  2. 将新内容复制到字符数组的末尾，并更新内部的 `count` 变量（表示当前字符数组中有效字符的数量）。

### **代码片段示意**

```java
@Override
public StringBuilder append(String str) {
    if (str == null) {
        return appendNull();
    }
    int len = str.length();
    ensureCapacityInternal(count + len); // 确保有足够的容量
    str.getChars(0, len, value, count);  // 将字符串复制到字符数组中
    count += len;                        // 更新有效字符数
    return this;
}
```

### **性能分析**

- **时间复杂度**：
  - 如果不需要扩容，`append` 的时间复杂度为 **O(n)**，其中 `n` 是追加内容的长度。
  - 如果需要扩容，涉及数组复制操作，时间复杂度为 **O(N)**，其中 `N` 是现有字符数组的长度。
- **优点**：
  - 动态扩容机制避免了频繁创建新对象，比直接使用 `String` 拼接效率更高。
  - 追加操作只需简单地在数组末尾写入数据，非常高效。
- **适用场景**：
  - 需要频繁追加字符串时，`StringBuilder` 是首选。

---

## 2. **`insert` 方法**

### **底层逻辑**

- **核心思想**：`insert` 方法将指定的内容插入到 `StringBuilder` 对象的指定位置。
- **具体实现**：
  1. 检查当前字符数组是否有足够的空间容纳新内容。如果没有，则调用 `ensureCapacityInternal` 方法扩展数组容量。
  2. 将插入点之后的所有字符向后移动（腾出空间），然后将新内容复制到指定位置。
  3. 更新内部的 `count` 变量。

### **代码片段示意**

```java
@Override
public StringBuilder insert(int offset, String str) {
    if ((offset < 0) || (offset > length()))
        throw new StringIndexOutOfBoundsException(offset);
    if (str == null)
        str = "null";
    int len = str.length();
    ensureCapacityInternal(count + len); // 确保有足够的容量
    System.arraycopy(value, offset, value, offset + len, count - offset); // 移动字符
    str.getChars(0, len, value, offset); // 插入新内容
    count += len;                        // 更新有效字符数
    return this;
}
```

### **性能分析**

- **时间复杂度**：
  - 如果不需要扩容，`insert` 的时间复杂度主要取决于 `System.arraycopy` 操作，其复杂度为 **O(N)**，其中 `N` 是从插入点开始需要移动的字符数量。
  - 如果需要扩容，涉及数组复制操作，时间复杂度为 **O(N)**，其中 `N` 是现有字符数组的长度。
- **缺点**：
  - 插入操作需要移动字符，因此性能通常比 `append` 差。
  - 插入点越靠前，需要移动的字符越多，性能开销越大。
- **适用场景**：
  - 需要在字符串中间插入内容时使用。

---

## 3. **性能对比：`append` vs `insert`**

| **方法**       | **操作类型**         | **时间复杂度**       | **性能特点**                                                                 |
|----------------|--------------------|---------------------|------------------------------------------------------------------------------|
| `append`       | 追加到末尾          | O(n) 或 O(N)（扩容时） | 高效，只需要简单地将新内容写入数组末尾，适合频繁追加操作。                                      |
| `insert`       | 插入到指定位置       | O(N)                | 较低效，需要移动插入点后的所有字符，尤其是插入点靠前时性能较差。                                 |

---

## 4. **优化建议**

1. **预估容量**：
   - 如果可以提前预估最终字符串的长度，可以通过构造函数或 `ensureCapacity` 方法设置初始容量，减少扩容次数。

   ```java
   StringBuilder sb = new StringBuilder(100); // 初始容量为100
   ```

2. **尽量避免频繁插入**：
   - 如果需要频繁插入，考虑先将内容追加到 `StringBuilder` 中，最后再排序或调整顺序。

3. **批量操作**：
   - 如果需要多次拼接或插入，优先将内容组织好后再一次性操作，减少不必要的移动和扩容。

---

## 5. **总结**

- **`append` 方法**：适合高频追加操作，性能较高，底层实现简单直接。
- **`insert` 方法**：适合在字符串中间插入内容，但由于需要移动字符，性能相对较低。
- **选择依据**：根据实际需求选择合适的方法。如果可能，优先使用 `append`，并通过预估容量进一步优化性能。
