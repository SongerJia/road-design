## 原型模式
原型是什么->核心实现->浅拷贝vs深拷贝->应用场景->面试高频题
### 原型是什么
原型模式：通过复制已有对象来创建新对象，而不是通过new创建对象。避免重复初始化开销。
```
// 类比：复印机
// 不需要重新排版文档，直接复印一份

// 适用场景：
// ① 对象创建成本高（复杂初始化、数据库查询、IO）
// ② 需要大量相似对象（只改部分字段）
// ③ 对象初始化耗时长
```
### 核心实现
```
// Java 的 clone() 就是原型模式
public class User implements Cloneable {  // 必须实现 Cloneable（标记接口）
    private String name;
    private Integer age;
    private Address address;

    // 重写 clone（扩大访问权限）
    @Override
    public User clone() {
        try {
            return (User) super.clone();  // Object.clone() 浅拷贝
        } catch (CloneNotSupportedException e) {
            throw new AssertionError();
        }
    }
}

// 使用（复制对象）
User user1 = new User("张三", 25, new Address("北京"));
User user2 = user1.clone();   // 复制，不用 new + set 一堆
user2.setName("李四");
```
为什么必须实现Clonable
```
// Cloneable 是"标记接口"（没有方法）
// 告诉 Object.clone()：这个类允许克隆
// 不实现 → 调用 clone() 抛 CloneNotSupportedException

// 为什么这么设计？
// ① 防御：默认不允许随意克隆
// ② 显式声明：开发者确认可以克隆
```
### 浅拷贝 vs 深拷贝
浅拷贝 Object.clone默认
```java
// 复制基本类型 + 引用类型只复制"引用"

// 结果：两个对象共享同一个 Address
User user1 = new User("张三", 25, new Address("北京"));
User user2 = user1.clone();

user2.getAddress().setCity("上海");
System.out.println(user1.getAddress().getCity());  // "上海"！
// user1 也被改了（共享同一个 Address 对象）
```
深拷贝 手动实现
```java
// ① 重写 clone：引用类型也 clone
@Override
public User clone() {
    try {
        User cloned = (User) super.clone();
        cloned.address = this.address.clone();  // Address 也要 Cloneable + clone
        return cloned;
    } catch (CloneNotSupportedException e) {
        throw new AssertionError();
    }
}

// ② 拷贝构造器
public User(User other) {
    this.name = other.name;
    this.address = new Address(other.address);  // 新建
}

// ③ 序列化深拷贝（最彻底）
public User deepClone() {
    ByteArrayOutputStream bos = new ByteArrayOutputStream();
    try (ObjectOutputStream oos = new ObjectOutputStream(bos)) {
        oos.writeObject(this);
        try (ObjectInputStream ois = new ObjectInputStream(new ByteArrayInputStream(bos.toByteArray()))) {
            return (User) ois.readObject();
        }
    }
    // 要求：实现 Serializable
}
```
对比

|拷贝|基本类型|引用类型|说明|
|---|---|---|---|
|**浅拷贝**|复制值|共享引用|简单，可能互相影响|
|**深拷贝**|复制值|复制新对象|独立，开销大|
### 应用场景
①Spring bean的原型作用域
```
// Spring 的 prototype 每次 getBean 都返回新实例
// 类似"原型模式"（复制模板）
@Scope("prototype")
public class UserContext { ... }

// 每次获取都是新对象（隔离状态）
```
②数据库行复制/对象复制
```
// 编辑"副本"而不是直接改原对象：
// 表单编辑 → 复制原对象 → 改副本 → 提交/回滚
```
③大量相似对象
```
// 创建一批原型对象缓存
// 需要时 clone（比 new 快，跳过初始化）

// 例：游戏中的子弹、缓存对象
```
④不可变对象的修改
```
// 不可变对象要"修改"→ 复制一份再改
// LocalDateTime.plusDays() 就是返回新对象（原型思想）
```
### 面试高频题
#### 题目1：原型模式是什么
```
// 通过复制已有对象创建新对象
// clone() 实现
```
#### 题目2：浅拷贝和深拷贝
```
// 浅：引用共享
// 深：引用也复制（独立）
```
#### 题目3：怎么实现拷贝
```
// 重写 clone 手动复制引用
// 拷贝构造器
// 序列化
```
#### 题目4：clonable是干嘛的
```
// 标记接口，允许 clone
// 不实现会抛异常
```
#### 题目5：原型 vs 工厂/建造者
```
// 原型：复制已有对象（快）
// 工厂：按类型创建
// 建造者：链式组装参数
```
#### 题目6：clone()的缺点
```
// 浅拷贝问题
// 不调用构造器（可能绕过初始化逻辑）
// 推荐拷贝构造器/工厂方法替代
```
