

下面从多个角度对比说明👇

------

### 🧩 一、底层数据结构

| 项目         | HashMap                            | TreeMap                            |
| ------------ | ---------------------------------- | ---------------------------------- |
| 底层结构     | 数组 + 链表 + 红黑树（JDK 1.8 起） | 红黑树                             |
| 键的存储顺序 | 无序（哈希分布）                   | 有序（按键的自然顺序或指定比较器） |

------

### ⚙️ 二、键的排序规则

| 项目 | HashMap        | TreeMap                                 |
| ---- | -------------- | --------------------------------------- |
| 顺序 | 不保证顺序     | 自动按键排序（自然顺序或 `Comparator`） |
| 示例 | 输出顺序不固定 | 输出按升序排列（或自定义规则）          |

```java
Map<String, Integer> hashMap = new HashMap<>();
hashMap.put("c", 3);
hashMap.put("a", 1);
hashMap.put("b", 2);
System.out.println(hashMap); // 输出顺序不确定

Map<String, Integer> treeMap = new TreeMap<>();
treeMap.put("c", 3);
treeMap.put("a", 1);
treeMap.put("b", 2);
System.out.println(treeMap); // {a=1, b=2, c=3}
```

------

### 🧠 三、键的要求

| 项目            | HashMap                     | TreeMap                                                 |
| --------------- | --------------------------- | ------------------------------------------------------- |
| 键是否可为 null | ✅ 可以一个 null 键          | ❌ 不允许 null 键（会抛出 `NullPointerException`）       |
| 是否要求可比较  | ❌ 不要求键实现 `Comparable` | ✅ 键必须可比较（实现 `Comparable` 或传入 `Comparator`） |

------

### 🚀 四、性能对比

| 操作               | HashMap 平均时间复杂度 | TreeMap 时间复杂度                                      |
| ------------------ | ---------------------- | ------------------------------------------------------- |
| 插入 / 删除 / 查找 | O(1)（哈希分布良好）   | O(log n)（红黑树查找）                                  |
| 遍历有序性         | 无序                   | 有序，支持范围查询（如 `subMap`、`headMap`、`tailMap`） |

------

### 🧾 五、使用场景对比

| 场景                     | 推荐使用    |
| ------------------------ | ----------- |
| 需要快速查找、插入、删除 | ✅ `HashMap` |
| 需要有序存储或范围查询   | ✅ `TreeMap` |
| 需要根据 key 排序输出    | ✅ `TreeMap` |
| 不关心顺序，仅存取性能   | ✅ `HashMap` |

------

### 🧮 六、示例对比总结

```java
// HashMap：性能优先，无序
Map<Integer, String> hashMap = new HashMap<>();
hashMap.put(3, "C");
hashMap.put(1, "A");
hashMap.put(2, "B");
System.out.println(hashMap); // 输出无序

// TreeMap：有序存储
Map<Integer, String> treeMap = new TreeMap<>();
treeMap.put(3, "C");
treeMap.put(1, "A");
treeMap.put(2, "B");
System.out.println(treeMap); // 输出 {1=A, 2=B, 3=C}
```

------

### 📊 七、总结对比表

| 对比项           | HashMap              | TreeMap                |
| ---------------- | -------------------- | ---------------------- |
| 底层结构         | 数组 + 链表 + 红黑树 | 红黑树                 |
| 是否有序         | 否                   | 是（按键排序）         |
| 是否允许 null 键 | 允许一个             | 不允许                 |
| 时间复杂度       | O(1)                 | O(log n)               |
| 键要求           | 不要求可比较         | 必须可比较或传入比较器 |
| 典型场景         | 快速存取             | 有序/范围操作          |

------

是否希望我帮你画一个**HashMap vs TreeMap 内部结构对比图（带图解红黑树、哈希桶等）**？