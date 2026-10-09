## 观察者模式
观察者是什么->核心结构->Spring 事件关联->实战应用->同步vs异步->面试高频题
### 观察者是什么
观察者模式：定义对象间一对多依赖。被观察者状态发生变化时，自动通知所有观察者。
```
// 类比：公众号订阅
// 公众号发文章（被观察者）→ 所有订阅者（观察者）自动收到
// 订阅者不用天天来问"发了吗"（解耦）

// 也叫：发布-订阅（Publish-Subscribe）
```
解决什么问题
```
// ❌ 订单创建后，手动通知所有模块
public void createOrder() {
    orderDao.insert();
    smsService.send();      // 耦合！
    logService.save();      // 加模块改这里
    statsService.update();  // 改一次加一行
}
// 新增"发优惠券" → 改 createOrder（违反开闭）

// ✅ 观察者：发布事件，订阅者自己处理
publisher.publishEvent(new OrderCreatedEvent(order));
// 谁关心谁订阅（不改发布者）
```
### 核心结构
```
// ① 观察者接口
public interface OrderListener {
    void onOrderCreated(Order order);
}

// ② 具体观察者（订阅者）
@Component
public class SmsListener implements OrderListener {
    @Override
    public void onOrderCreated(Order order) {
        // 发短信
        smsService.send(order.getPhone(), "下单成功");
    }
}

@Component
public class StatsListener implements OrderListener {
    @Override
    public void onOrderCreated(Order order) {
        // 更新统计
        statsService.update(order);
    }
}

// ③ 被观察者（发布者）
@Service
public class OrderService {
    // 持有所有观察者
    @Autowired
    private List<OrderListener> listeners;  // Spring 注入所有实现

    public void createOrder(Order order) {
        // 核心业务
        orderDao.insert(order);

        // 通知所有观察者（自动）
        for (OrderListener listener : listeners) {
            listener.onOrderCreated(order);
        }
        // 新增观察者 → 加一个类 → 自动通知（不改 OrderService）
    }
}
```
核心思想
```
// ① 发布者不知道观察者是谁（解耦）
// ② 观察者自己注册（Spring 自动收集）
// ③ 新增观察者 → 不改发布者（开闭 ✅）
// ④ 一对多：一个事件通知所有
```
### Spring事件机制
```
// Spring 内置观察者模式 = 事件机制

// ① 定义事件
public class OrderCreatedEvent extends ApplicationEvent {
    private final Order order;

    public OrderCreatedEvent(Object source, Order order) {
        super(source);
        this.order = order;
    }
    public Order getOrder() { return order; }
}

// ② 发布事件（被观察者）
@Service
public class OrderService {
    @Autowired
    private ApplicationEventPublisher publisher;

    public void createOrder(Order order) {
        orderDao.insert(order);
        // 发布事件（通知所有监听者）
        publisher.publishEvent(new OrderCreatedEvent(this, order));
    }
}

// ③ 监听事件（观察者）
@Component
public class SmsListener {
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        // 发短信（自动被调用）
        smsService.send(event.getOrder().getPhone(), "下单成功");
    }
}

@Component
public class StatsListener {
    @EventListener
    public void onOrderCreated(OrderCreatedEvent event) {
        // 更新统计（自动被调用）
    }
}
```
Spring事件的优点
```
// ① 一行 @EventListener 注册观察者（不用实现接口）
// ② 新增观察者不用改发布者（开闭）
// ③ 支持异步（@Async + @EventListener）
// ④ 支持事件顺序（@Order）
```
### 同步 vs 异步
同步观察者
```
// 发布事件 → 立即执行所有监听器（阻塞）
// 特点：顺序执行、结果可控、慢

// 场景：必须立即完成的（扣库存）
publisher.publishEvent(event);  // 等所有监听器执行完才返回
```
异步观察者
```
// 发布事件 → 监听器异步执行（不阻塞）
// 特点：快、不影响主流程、顺序不保证

// 用法：
@EventListener
@Async  // 异步执行
public void onOrderCreated(OrderCreatedEvent event) {
    // 发短信（不阻塞下单流程）
}
```

|对比|同步|异步|
|---|---|---|
|**执行**|发布时立即|后台线程|
|**阻塞**|阻塞发布者|不阻塞|
|**顺序**|保证|不保证|
|**适用**|核心逻辑|通知/统计/日志|
|**失败影响**|影响主流程|不影响|
实战
```
// 下单：
// 同步监听：扣库存（必须成功）
// 异步监听：发短信、统计（不影响下单）

@EventListener
public void deductStock(OrderCreatedEvent e) {
    // 同步：扣库存（核心）
}

@EventListener
@Async
public void sendSms(OrderCreatedEvent e) {
    // 异步：发短信（次要）
}
```
### 实战应用
```
// 判题完成事件 → 通知多个模块
public class JudgeCompletedEvent extends ApplicationEvent {
    private final JudgeResult result;
    // ...
}

// 发布（判题服务）
publisher.publishEvent(new JudgeCompletedEvent(this, result));

// 观察者 1：更新排行榜（同步）
@EventListener
public void updateRank(JudgeCompletedEvent e) { ... }

// 观察者 2：通知用户（异步）
@EventListener
@Async
public void notifyUser(JudgeCompletedEvent e) { ... }

// 观察者 3：记录统计（异步）
@EventListener
@Async
public void updateStats(JudgeCompletedEvent e) { ... }

// 新增"推送学习建议" → 加一个 @EventListener → 不改判题服务 ✅
```
### 面试高频题
#### 题目1：观察者模式是什么
```
// 一对多通知
// 状态变化自动通知所有观察者
```
#### 题目2：和发布订阅关系
```
// 观察者模式 ≈ 发布订阅
// 发布者 + 订阅者（解耦）
```
#### 题目3：Spring事件是什么
```
// 观察者模式的实现
// publishEvent + @EventListener
```
#### 题目4：同步和异步观察者
```
// 同步：立即执行（核心逻辑）
// 异步：后台执行（通知统计）
```
#### 题目5：观察者有什么好处
```
// 解耦（发布者不知道谁监听）
// 开闭（新增监听不改发布者）
```
#### 题目6：有什么注意点
```
// ① 同步监听异常可能影响主流程
// ② 异步监听顺序不保证
// ③ 监听器过多影响性能
```
