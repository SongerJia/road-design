## 状态模式
状态是什么->核心结构->订单状态机应用->与策略对比->实战应用->面试高频题
### 状态是什么
状态模式：对象的行为随内部变化而变化。把每个状态封装成类，状态切换时对象自动改变行为。
```
// 类比：人的状态（清醒/睡觉/疲惫）
// 清醒时：效率高
// 睡觉时：不干活
// 疲惫时：效率低
// 行为随状态变，不用 if 判断"现在什么状态"
```
解决什么问题
```
// ❌ if-else 状态判断（订单流转）
public void pay(Order order) {
    if ("PENDING".equals(order.getStatus())) {
        // 可以支付
    } else if ("PAID".equals(order.getStatus())) {
        throw new IllegalStateException("已支付");
    }
    // 每加一个状态/操作 → 改代码
}

// ✅ 状态模式：状态类自己定义"能做什么"
```
### 核心结构
```
// ① 状态接口
public interface OrderState {
    // 每种状态定义：能执行的操作
    void pay(OrderContext ctx);
    void ship(OrderContext ctx);
    void complete(OrderContext ctx);
}

// ② 具体状态：待支付
public class PendingState implements OrderState {
    @Override
    public void pay(OrderContext ctx) {
        System.out.println("支付成功");
        ctx.setState(new PaidState());   // 切换状态
    }

    @Override
    public void ship(OrderContext ctx) {
        throw new IllegalStateException("未支付不能发货");
    }

    @Override
    public void complete(OrderContext ctx) {
        throw new IllegalStateException("未支付不能完成");
    }
}

// ③ 具体状态：已支付
public class PaidState implements OrderState {
    @Override
    public void pay(OrderContext ctx) {
        throw new IllegalStateException("已支付，不能重复支付");
    }

    @Override
    public void ship(OrderContext ctx) {
        System.out.println("发货成功");
        ctx.setState(new ShippedState());
    }

    @Override
    public void complete(OrderContext ctx) {
        throw new IllegalStateException("未发货不能完成");
    }
}

// ④ 上下文（持有状态）
public class OrderContext {
    private OrderState state;

    public OrderContext() {
        this.state = new PendingState();  // 初始状态
    }

    public void setState(OrderState state) {
        this.state = state;
    }

    // 操作转发给状态（行为随状态变化）
    public void pay() { state.pay(this); }
    public void ship() { state.ship(this); }
    public void complete() { state.complete(this); }
}

// ⑤ 使用
OrderContext order = new OrderContext();  // 待支付
order.pay();       // 支付成功 → 变已支付
order.ship();      // 发货成功 → 变已发货
order.pay();       // ❌ 已支付，不能重复支付（状态自己拦）
```
核心思想
```
// ① 状态封装成类（每个状态一个类）
// ② 状态类自己定义"能做什么、不能做什么"
// ③ 操作 → 转发给当前状态 → 状态可能切换
// ④ 非法操作 → 状态自己抛异常（不用 if 判断）
```
### 订单状态机
```
// 订单状态流转（状态机）：
// 待支付 → 已支付 → 已发货 → 已完成
//    ↘ 已取消（可随时取消？按规则）

public enum OrderStatus {
    PENDING, PAID, SHIPPED, COMPLETED, CANCELLED
}

// 状态机定义：每个状态允许的转移
public class OrderStateMachine {

    // 状态转移表（允许的流转）
    private static final Map<OrderStatus, Set<OrderStatus>> TRANSITIONS = Map.of(
        PENDING, Set.of(PAID, CANCELLED),       // 待支付 → 支付/取消
        PAID, Set.of(SHIPPED, CANCELLED),       // 已支付 → 发货/取消
        SHIPPED, Set.of(COMPLETED),             // 已发货 → 完成
        COMPLETED, Set.of(),                    // 完成：不可再流转
        CANCELLED, Set.of()                     // 取消：不可再流转
    );

    public static void transition(Order order, OrderStatus target) {
        OrderStatus current = order.getStatus();
        Set<OrderStatus> allowed = TRANSITIONS.get(current);

        if (!allowed.contains(target)) {
            throw new IllegalStateException(
                "非法流转：" + current + " → " + target);
        }
        order.setStatus(target);
    }
}

// 使用
transition(order, PAID);    // ✅ 待支付→已支付
transition(order, SHIPPED); // ✅ 已支付→已发货
transition(order, PENDING); // ❌ 已发货不能回待支付（抛异常）
```
状态机 vs 状态模式
```
// 状态机：定义"状态转移规则"（表/配置）
// 状态模式：状态封装为类（行为随状态变）

// 可结合：
// 状态模式实现行为切换 + 状态机表校验流转合法性
```
### 与策略模式对比
| 对比      | 状态模式        | 策略模式   |
| ------- | ----------- | ------ |
| **核心**  | 行为随状态变化     | 算法可替换  |
| **切换者** | 对象内部（状态自己切） | 客户端主动换 |
| **方向**  | 状态转移（流程）    | 算法选择   |
| **类比**  | 订单状态流转      | 出行方式选择 |
| **关系**  | 状态之间有关联（转移） | 策略互相独立 |
关键区别
```
// 状态：状态之间"会切换"（有流程关系）
// 策略：策略之间"独立"（不会互相切换）

// 例：
// 订单 PENDING → PAID → SHIPPED（状态流转）→ 状态模式
// 判题 java/python/js（选一个算法）→ 策略模式
```
### 实战应用
```
// 判题任务状态：
// 待判 → 判题中 → 成功 / 失败 / 超时

// ① 状态接口
public interface TaskState {
    void start(JudgeContext ctx);
    void success(JudgeContext ctx);
    void fail(JudgeContext ctx);
}

// ② 待判状态
public class PendingState implements TaskState {
    public void start(JudgeContext ctx) {
        ctx.setState(new JudgingState());  // 开始判题
    }
    // ...
}

// ③ 判题中
public class JudgingState implements TaskState {
    public void success(JudgeContext ctx) {
        ctx.setState(new SuccessState());  // 判题成功
    }
    public void fail(JudgeContext ctx) {
        ctx.setState(new FailState());
    }
    // start 非法（已在判题）
}

// 使用
JudgeContext task = new JudgeContext();  // 待判
task.start();        // → 判题中
task.success();      // → 成功（或 fail → 失败）
task.start();        // ❌ 判题中不能重新开始（状态自己拦）
```
### 面试高频题
#### 题目1：状态模式是什么
```
// 行为随状态变化
// 状态封装为类，自动切换
```
#### 题目2：和策略区别
```
// 状态：状态间会流转（流程）
// 策略：算法独立选择
```
#### 题目3：订单状态怎么管理
```
// 状态模式 或 状态机表
// 状态机表定义允许的流转（非法流转抛异常）
```
#### 题目4：状态模式好处
```
// 去掉 if-else 状态判断
// 非法操作状态自己拦
// 扩展新状态（加类+改转移表）
```
#### 题目5：状态模式缺点
```
// 类数量多（每状态一个类）
// 状态转移逻辑分散（需要状态机表辅助）
```
#### 题目6：和状态机关系
```
// 状态模式：行为封装
// 状态机：转移规则
// 可结合使用
```