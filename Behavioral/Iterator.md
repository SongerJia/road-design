## 迭代器模式
迭代器是什么->核心结构->ArrayList源码->fail-fast关联->应用场景->高频面试题
### 迭代器模式是什么
迭代器模式：提供统一的方式遍历集合元素，不暴露集合的底层结构。使用时只依赖iterator接口，不考虑底层是数组、链表还是树。
```
// 类比：图书管理员（迭代器）
// 不管书放在哪（书架/箱子/仓库），给你一个"翻页器"
// 你只需要：有没有下一本？给我下一本！

// 核心：遍历逻辑从集合中抽离，统一接口
```
解决什么问题
```
// ❌ 每种集合一种遍历方式：
// ArrayList：for (int i = 0; i < list.size(); i++)
// LinkedList：node.next()（不知道内部结构没法遍历）
// HashSet：完全不知道内部怎么存

// ✅ 统一迭代器：
Iterator<String> it = collection.iterator();
while (it.hasNext()) {
    String s = it.next();
}
// ArrayList/LinkedList/HashSet 都能用同一套代码
```
### 核心结构
两大角色
```
// ① 迭代器接口（提供遍历方法）
public interface Iterator<E> {
    boolean hasNext();   // 是否还有下一个
    E next();            // 返回下一个并移动游标

    // 默认方法（可选）
    default void remove() {
        throw new UnsupportedOperationException();
    }
}

// ② 具体迭代器（每个集合有自己的实现）
public class ArrayList<E> {
    // ...
    public Iterator<E> iterator() {
        return new Itr();  // 返回自己的迭代器
    }
}
```
核心：迭代器持有集合内部状态
```
public class ArrayList<E> {
    // 集合内部数据
    private Object[] elementData;
    private int size;

    // 具体迭代器（内部类：能访问集合私有成员）
    private class Itr implements Iterator<E> {
        int cursor = 0;                 // 游标：下一个要返回的位置
        int expectedModCount = modCount; // fail-fast 快照

        @Override
        public boolean hasNext() {
            return cursor != size;      // 游标没到末尾
        }

        @Override
        public E next() {
            checkForComodification();   // fail-fast 检查
            int i = cursor;
            // 越界检查...
            cursor = i + 1;             // 游标前进
            return (E) elementData[i];
        }
    }
}
```
关键点
```
// ① 迭代器是"内部类"：能直接访问集合的私有字段
// ② 迭代器持有"游标"（遍历位置）
// ③ 迭代器持有"modCount 快照"（fail-fast）
// ④ 每个集合有自己迭代器（但接口统一）
```
### 与Java关联
①for-each就是迭代器语法糖
```
// for-each：
for (String s : list) {
    System.out.println(s);
}

// 编译后等价于：
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    System.out.println(s);
}

// 所以：
// ① for-each 遍历中不能调用 list.remove()（fail-fast）
// ② for-each 拿不到下标（用传统 for 才有）
// ③ for-each 不能修改元素（s = "x" 只改局部变量）
```
②ListIterator 增强版
```
// List 专有的迭代器（双向）
List<String> list = Arrays.asList("A", "B", "C");

ListIterator<String> it = list.listIterator();
it.hasNext();       // 是否有下一个
it.next();          // 下一个
it.hasPrevious();   // 是否有前一个（双向）
it.previous();      // 前一个
it.set("X");        // 修改当前元素（迭代器方式安全）
it.add("D");        // 插入（迭代器方式安全）
it.remove();        // 删除（安全）
```
③fail-fast
```
// 迭代器创建时记录 expectedModCount = modCount
// 每次 next() 前检查：
// modCount != expectedModCount → ConcurrentModificationException

// 什么时候 modCount 变：
// list.add/remove/clear（结构性修改）
// list.set 不变（不算结构性修改）

// 迭代器的 remove 为什么安全：
// 内部会同步 expectedModCount = modCount
// 所以下次检查能通过 ✅
```
### 迭代器remove安全 vs 集合remove
```
// ❌ 集合直接 remove（fail-fast） 这个是语法糖，在遍历里对原集合进行结构性改变会异常
List<String> list = new ArrayList<>(Arrays.asList("A", "B", "C"));
for (String s : list) {
    if ("B".equals(s)) {
        list.remove(s);  // ❌ modCount 变了，下次 next 抛异常
    }
}

// ✅ 迭代器 remove（安全）   使用的是迭代器去删除
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if ("B".equals(s)) {
        it.remove();  // ✅ 同步 expectedModCount，安全
    }
}

// ✅ removeIf（Java 8+，底层就是迭代器）
list.removeIf("B"::equals);
```
为什么迭代器remove安全
```
// ArrayList.Itr.remove() 源码：
public void remove() {
    // ① 调用 ArrayList 的 remove（modCount++）
    // ② 同步 expectedModCount = modCount（关键！）
    expectedModCount = modCount;
    // 所以下次 checkForComodification() 能通过
}
```
### 应用场景
①集合框架统一遍历
```
// Collection.iterator() 是所有集合的统一入口
// ArrayList（数组）、LinkedList（链表）、HashSet（哈希）
// 遍历代码不用管底层结构
```
②数据库游标（ResultSet）
```
// ResultSet 就像迭代器：
// next() → 有没有下一行
// getString() → 取当前行数据
// 客户端不用关心数据库内部存储
```
③树/图遍历（自定义迭代器）
```
// 二叉树的中序/前序迭代器
// 把递归遍历封装成迭代器
// 客户端可以用 for-each 遍历树
```
④文件/流读取（FileReader等）
```
// 按行读取文件的迭代器
// 屏蔽底层 IO 细节
```
### 高频面试题
#### 题目1：迭代器模式是什么
```
// 统一遍历接口，屏蔽底层结构
// hasNext + next
```
#### 题目2：for-each和迭代器关系
```
// for-each 编译后就是迭代器循环
// 所以 for-each 里不能 list.remove()
```
#### 题目3：迭代器的remove为什么安全
```
// 内部同步 expectedModCount = modCount
// 集合直接 remove 不同步 → fail-fast
```
#### 题目4：fail-fast怎么实现
```
// 迭代器记录 modCount 快照
// next 前检查，变了抛 ConcurrentModificationException
```
#### 题目5：ListIterator和Iterator区别
```
// ListIterator：双向、支持 set/add、List 专用
// Iterator：单向、只能 remove
```
#### 题目6：迭代器优点
```
// ① 统一接口（解耦集合和遍历）
// ② 安全删除
// ③ 不暴露内部结构
```

