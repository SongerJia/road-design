## 单例模式
单例是什么->六种写法->各写法对比->反射/序列化破坏->面试高频题
### 单例是什么
单例模式：保证一个类只有一个实例，并提供一个全局访问点。
为什么需要
```
// ① 共享资源：配置类、线程池、连接池（全局一份）
// ② 节省资源：创建代价大的对象只创建一次
// ③ 统一状态：计数器、全局 ID 生成
// ④ Spring Bean 默认单例
```
单例三要素
```
// ① 私有构造器（外部不能 new）
// ② 静态实例（类级别一份）
// ③ 全局访问点（getInstance()）
```
### 六种写法
①饿汉式
```
public class Singleton {
    // 类加载时就创建（线程安全）
    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

// 优点：线程安全（类加载时初始化）、简单
// 缺点：不管用不用都创建（浪费）；类加载时就占用
```
②懒汉式
```
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        if (instance == null) {       // 线程 A 和 B 可能同时进来
            instance = new Singleton();  // 创建两次！❌
        }
        return instance;
    }
}

// 问题：多线程下可能创建多个实例（线程不安全）
// 不推荐
```
③懒汉式+Synchronized
```
public class Singleton {
    private static Singleton instance;

    private Singleton() {}

    // 方法加锁：安全但每次都要抢锁（性能差）
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }
}

// 优点：线程安全
// 缺点：每次调用都加锁（即使已初始化）→ 性能差
```
④DCL 双重检查锁
```
public class Singleton {
    // volatile：防止指令重排（关键！）
    private static volatile Singleton instance;

    private Singleton() {}

    public static Singleton getInstance() {
        // 第一次检查（无锁）
        if (instance == null) {
            // 加锁后第二次检查
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}

// 为什么 volatile（核心面试点）：
// new Singleton() 三步：分配内存 → 初始化 → 赋值引用
// 可能重排为：分配内存 → 赋值引用 → 初始化
// 其他线程看到"引用不为 null"但"对象未初始化"→ 拿半成品
// volatile 禁止重排 ✅

// 优点：线程安全 + 性能好（只在第一次竞争锁）
// 推荐：懒加载 + 高性能
```
⑤静态内部类
```
public class Singleton {
    private Singleton() {}

    // 静态内部类（懒加载 + 线程安全）
    private static class Holder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}

// 原理：
// ① 外部类加载时不初始化 Holder（懒加载）
// ② 第一次 getInstance() 才加载 Holder（触发创建）
// ③ 类加载天然线程安全

// 优点：懒加载 + 线程安全 + 高性能（无锁）
// 推荐（比 DCL 更优雅）
```
⑥枚举
```
public enum Singleton {
    INSTANCE;  // 唯一的实例

    public void doSomething() {
        System.out.println("单例方法");
    }
}

// 使用
Singleton.INSTANCE.doSomething();

// 优点：
// ① JVM 保证线程安全
// ② 序列化安全（JVM 处理）
// ③ 反射无法破坏（JVM 禁止反射创建枚举）
// ④ 代码最简洁

// 推荐：《Effective Java》作者推荐
```
### 六种写法对比
| 写法         | 懒加载 | 线程安全 | 性能  | 推荐    |
| ---------- | --- | ---- | --- | ----- |
| 饿汉式        | ❌   | ✅    | 高   | 简单场景  |
| 懒汉式（不安全）   | ✅   | ❌    | 高   | ❌     |
| 懒汉式 + sync | ✅   | ✅    | 低   | ❌     |
| **双重检查锁**  | ✅   | ✅    | 高   | ✅     |
| **静态内部类**  | ✅   | ✅    | 高   | ✅     |
| **枚举**     | ✅   | ✅    | 高   | ✅ 最推荐 |
**Q:** "单例模式有哪几种写法？推荐哪种？" 
**A:** "六种：饿汉、懒汉（不安全）、懒汉加锁、双重检查锁（DCL）、静态内部类、枚举。推荐：① 一般场景用 DCL（volatile 防重排）或静态内部类；② 最推荐枚举（JVM 保证线程安全、序列化安全、反射无法破坏，代码最简洁）。"
### 破坏单例
①反射破坏
```
// 反射可以调用 private 构造器 → 创建第二个实例！

Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
constructor.setAccessible(true);  // 绕过 private
Singleton s2 = constructor.newInstance();
// 成功创建第二个实例 → 单例被破坏！

// 防御：
public class Singleton {
    private static boolean flag = false;

    private Singleton() {
        synchronized (Singleton.class) {
            if (flag) {
                throw new RuntimeException("禁止反射创建实例");
            }
            flag = true;
        }
    }
}

// 枚举天然防反射：
// JVM 禁止反射创建枚举实例（抛 IllegalArgumentException）
```
②序列化破坏
```
// 序列化后再反序列化 → 得到新实例！

// 防御：加 readResolve()
public class Singleton implements Serializable {
    // 反序列化时调用，返回同一个实例
    protected Object readResolve() {
        return INSTANCE;  // 返回已存在的实例
    }
}

// 枚举天然防序列化：
// JVM 反序列化枚举时直接返回已有实例
```
③克隆破坏
```
// 实现 Cloneable 后 clone() 可创建新实例
// 防御：重写 clone() 返回单例，或直接抛异常
```
### 单例 vs Spring 单例
```
// 标准单例（类本身保证）：
// private 构造器 + 静态实例

// Spring 单例（容器保证）：
// Bean 默认 singleton，但构造器不一定 private
// 容器管理（getBean 返回同一个）

// 区别：
// ① 标准单例：类自己保证
// ② Spring 单例：容器保证（可以多个构造器）
// ③ 标准单例：全局一个
// ④ Spring 单例：每个容器一个（多容器多实例）
```
注意
```
// 单例 + 有状态字段 → 线程不安全！
// 单例尽量"无状态"（方法内局部变量）

// 有状态单例的并发问题：
public class Counter {
    private int count;  // ❌ 单例共享，多线程不安全
    public void increment() { count++; }
}
// 解决：无状态 / ThreadLocal / AtomicInteger
```
### 面试高频题
#### 题目1：单例的六种写法
```
// 饿汉、懒汉、sync 懒汉、DCL、静态内部类、枚举
```
#### 题目2：DCL为什么加volatile
```
// new 三步可能重排（引用赋值在初始化前）
// volatile 禁止重排，防止拿到半成品
```
#### 题目3：静态内部类为什么懒加载
```
// 外部类加载不初始化 Holder
// 第一次 getInstance 才加载（类加载线程安全）
```
#### 题目4：为什么枚举最推荐
```
// 线程安全、序列化安全、反射安全、简洁
```
#### 题目5：怎么破坏单例
```
// 反射、序列化、克隆
// 防御：标志/readResolve/clone 抛异常/枚举
```
#### 题目6：单例注意什么
```
// 无状态（线程安全）
// Spring 单例 vs 标准单例区别
```
