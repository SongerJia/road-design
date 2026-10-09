## 工厂模式
工厂是什么->简单工厂->工厂方法->抽象工厂->三工厂对比->Spring关联->面试高频题
### 工厂是什么
工厂模式：把创建对象的逻辑封装到工厂类中，客户端不直接new，而是通过工厂方法获取对象。解耦使用和创建。
为什么需要
```
// ❌ 客户端直接 new：
public class OrderService {
    public void create() {
        // 直接依赖具体类（耦合）
        PayService payService = new AlipayService();  // 换支付方式要改代码！
    }
}

// ✅ 工厂创建：
PayService payService = PayFactory.create("alipay");
// 换支付方式：改参数（工厂内部判断）
// 客户端只依赖接口 PayService（解耦）
```
### 简单工厂
```
// 一个工厂类，根据参数返回不同的产品
public class PayFactory {

    public static PayService create(String type) {
        switch (type) {
            case "alipay":
                return new AlipayService();
            case "wechat":
                return new WechatService();
            case "card":
                return new CardService();
            default:
                throw new IllegalArgumentException("未知支付类型");
        }
    }
}

// 使用
PayService pay = PayFactory.create("alipay");

// 优点：客户端不接触具体类（简单解耦）
// 缺点：工厂里 switch 会越来越大（违反开闭原则）
// 新增类型要改工厂（不满足开闭）
```
应用举例
```
// JDK 里到处都是简单工厂：
// Integer.valueOf(int) → 按值返回（缓存池）
// Calendar.getInstance() → 按地区返回不同实现
// 线程池 Executors.newFixedThreadPool() → 创建不同线程池
```
### 工厂方法
```
// 把"创建逻辑"下沉到子类
// 每个工厂子类创建一种产品
// 父类定义"创建"接口（抽象方法）

// ① 产品接口
public interface PayService {
    void pay(double amount);
}

// ② 具体产品
public class AlipayService implements PayService {
    public void pay(double amount) {
        System.out.println("支付宝支付 " + amount);
    }
}
public class WechatService implements PayService {
    public void pay(double amount) {
        System.out.println("微信支付 " + amount);
    }
}

// ③ 抽象工厂（定义创建方法）
public abstract class PayFactory {
    // 工厂方法：子类实现
    public abstract PayService create();

    // 模板逻辑：公共处理
    public void pay(double amount) {
        PayService service = create();  // 子类决定创建哪个
        service.pay(amount);
    }
}

// ④ 具体工厂（各自创建自己的产品）
public class AlipayFactory extends PayFactory {
    @Override
    public PayService create() {
        return new AlipayService();
    }
}
public class WechatFactory extends PayFactory {
    @Override
    public PayService create() {
        return new WechatService();
    }
}

// 使用
PayFactory factory = new AlipayFactory();
factory.pay(100);  // 支付宝支付 100

// 优点：新增产品 = 新增工厂（满足开闭原则）
// 缺点：类数量多（一个产品一个工厂）
```
### 抽象工厂
```
// 创建"一族"相关的产品（不止一个产品）
// 工厂接口定义多个创建方法

// 场景：一家"支付平台"同时提供 支付 + 退款 + 对账

// ① 抽象产品族
public interface PayService { void pay(double amount); }
public interface RefundService { void refund(double amount); }

// ② 具体产品族（阿里系）
public class AlipayService implements PayService {
    public void pay(double amount) { ... }
}
public class AlipayRefundService implements RefundService {
    public void refund(double amount) { ... }
}

// ③ 抽象工厂（创建一整个产品族）
public interface PayFactory {
    PayService createPayService();
    RefundService createRefundService();
}

// ④ 具体工厂（阿里的产品族）
public class AlipayFactory implements PayFactory {
    public PayService createPayService() { return new AlipayService(); }
    public RefundService createRefundService() { return new AlipayRefundService(); }
}

// 使用：拿到阿里的整套服务
PayFactory factory = new AlipayFactory();
PayService pay = factory.createPayService();
RefundService refund = factory.createRefundService();

// 特点：保证"一族产品"是配套的（不会混搭）
// 适用：系统需要成套产品（跨平台 UI 组件等）
```
### 三种工厂对比
| 对比       | 简单工厂         | 工厂方法        | 抽象工厂     |
| -------- | ------------ | ----------- | -------- |
| **创建方式** | 一个工厂 + 参数判断  | 多个工厂（每产品一个） | 工厂创建一族产品 |
| **开闭原则** | ❌ 违反（加类型改工厂） | ✅ 符合        | ✅ 符合     |
| **产品数量** | 多个产品         | 一个产品        | 一族产品     |
| **复杂度**  | 低            | 中           | 高        |
| **适用**   | 简单场景         | 标准          | 成套产品     |
### Spring中的工厂
```
// ① BeanFactory：就是工厂模式
// 根据 BeanDefinition 创建不同 Bean

Object bean = beanFactory.getBean("userService");

// ② FactoryBean：复杂对象的工厂
// 创建需要特殊逻辑的对象

public class UserFactoryBean implements FactoryBean<User> {
    public User getObject() {
        // 自定义创建逻辑
        return new User("张三", 25);
    }
}

// ③ 静态工厂：
// <bean factory-method="createInstance"/>

// 面试："Spring 用了什么设计模式？"
// 回答：工厂模式（BeanFactory 管理 Bean 创建）
```
### 面试高频题
#### 题目1：三种工厂的区别
```
// 简单：一个工厂判断参数
// 方法：子类工厂各建一个产品
// 抽象：工厂创建一族产品
```
#### 题目2：简单工厂的缺点
```
// switch 膨胀、违反开闭
// 新增类型要改工厂
```
#### 题目3：工厂方法怎么满足开闭
```
// 新增产品 = 新增工厂类
// 不用改已有代码
```
#### 题目4：抽象工厂解决什么
```
// 一族产品的配套一致性
// 不会混搭（阿里的一族）
```
#### 题目5：Spring哪里用了工厂
```
// BeanFactory、FactoryBean
```
#### 题目6：工厂和单例结合
```
// 工厂创建 + 单例缓存（实例只创建一次）
// Spring 就是"工厂 + 单例"组合
```
