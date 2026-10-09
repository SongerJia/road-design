## 中介者模式
中介者是什么->网状vs星状->核心结构->MQ关联->应用场景->面试高频题
### 中介者模式是什么
中介者模式：对象之间不直接通信，通过中介者协调交互。把对象间的网状依赖变为星状依赖，降低耦合。
```
// 类比：聊天室
// 没有聊天室：每个人要认识其他人（网状，10 人 45 条线）
// 有聊天室（中介）：每人只连服务器（星状，10 人 10 条线）

// 类比：房产中介
// 买家卖家不直接谈，通过中介协调（减少互相寻找的复杂度）
```
解决什么问题
```
// ❌ 网状依赖（对象间互相引用）
public class LoginDialog {
    private TextBox username;    // 互相引用
    private TextBox password;
    private Button loginBtn;
    private Label errorLabel;

    public void init() {
        username.setOnChange(...);   // 要改 password/button 的状态
        password.setOnChange(...);
        loginBtn.setOnClick(...);
    }
    // 每加一个组件 → 所有组件都要改（爆炸式耦合）
}

// ✅ 中介者：组件只依赖中介者（不互相引用）
```
### 网状 vs 星状
网状依赖（问题）
```
A ── B
A ── C
A ── D
B ── C
B ── D
C ── D
// 4 个对象：6 条线
// 10 个对象：45 条线
// n 个对象：n(n-1)/2 条线（O(n²) 爆炸）
// 改一个 → 牵连所有
```
星状依赖（解决）
```
	        A
	        │
    B ── Mediator ── C
	        │
	        D
// 所有对象只连中介者
// n 个对象：n 条线（O(n)）
// 改一个 → 只改中介者
```
### 核心结构
```
// ① 中介者接口
public interface Mediator {
    void notify(Object sender, String event);
}

// ② 具体中介者（协调所有对象）
public class DialogMediator implements Mediator {
    private TextBox username;
    private TextBox password;
    private Button loginBtn;
    private Label errorLabel;

    // 注册所有组件
    public void setUsername(TextBox username) { this.username = username; }
    public void setPassword(TextBox password) { this.password = password; }
    public void setLoginBtn(Button loginBtn) { this.loginBtn = loginBtn; }

    // 核心：接收组件通知，协调其他组件
    @Override
    public void notify(Object sender, String event) {
        if (sender == username) {
            // 用户名变化 → 更新登录按钮状态
            loginBtn.setEnabled(!username.getText().isEmpty()
                && !password.getText().isEmpty());
        } else if (sender == password) {
            loginBtn.setEnabled(...);
        }
    }
}

// ③ 组件（只依赖中介者，不互相引用）
public class TextBox {
    private Mediator mediator;  // 持有中介者

    public void onChange() {
        // 通知中介者（不直接改别人）
        mediator.notify(this, "change");
    }
}

// 使用
DialogMediator mediator = new DialogMediator();
TextBox username = new TextBox(mediator);
Button loginBtn = new Button(mediator);
mediator.setUsername(username);
mediator.setLoginBtn(loginBtn);

username.setText("张三");  // → 中介者协调 → 按钮变可用
```
核心思想
```
// ① 对象不互相引用（只依赖中介者）
// ② 交互逻辑集中在中介者（一处管理）
// ③ 加新组件 → 只改中介者（不是所有组件）
// ④ 对象职责单一（只管自己 + 通知中介者）
```
### 和MQ的关联
```
// 消息中间件（MQ）就是典型的中介者！

// 没有 MQ（网状）：
// 订单服务 → 库存服务（直接调）
// 订单服务 → 支付服务（直接调）
// 订单服务 → 物流服务（直接调）
// 每加一个下游 → 订单服务要改

// 有 MQ（星状）：
// 订单服务 → MQ（只发消息）
// 库存/支付/物流 → MQ（各自订阅）
// 订单服务不认识任何下游 ✅

// 对比：
// 中介者模式 = MQ 的"思想原型"
// MQ 是中介者的分布式实现
```
其他中介者应用
```
// ① 注册中心（Nacos）：服务之间不直连，通过注册中心找
// ② 消息队列：生产者消费者通过 MQ 通信
// ③ GUI 框架：组件交互通过控制器协调
// ④ 机场塔台：飞机不互相沟通，通过塔台协调
```
### 实战应用
```
// 判题流程协调（多个模块通过中介者协作）
public interface JudgeMediator {
    void notifyModule(Object sender, JudgeEvent event);
}

public class JudgeMediatorImpl implements JudgeMediator {
    private TaskService taskService;
    private JudgeService judgeService;
    private ResultService resultService;

    @Override
    public void notifyModule(Object sender, JudgeEvent event) {
        // 根据事件协调各模块
        if (event == JudgeEvent.SUBMITTED) {
            judgeService.startJudge(event.getTask());
        } else if (event == JudgeEvent.COMPLETED) {
            resultService.save(event.getResult());
            taskService.updateStatus(event.getTask());
        }
        // 协调逻辑集中一处
    }
}
// 各模块只通知中介者，不互相依赖
```
### 面试高频题
#### 题目1：中介者模式是什么
```
// 对象间不直接通信，通过中介者协调
// 网状 → 星状
```
#### 题目2：和观察者/责任链区别
```
// 观察者：一对多通知（广播）
// 责任链：链式传递（逐个）
// 中介者：集中协调（星状）
```
#### 题目3：MQ是中介者吗
```
// 是！MQ 是中介者的分布式实现
// 生产者消费者通过 MQ 解耦
```
#### 题目4：中介者的好处
```
// ① 解耦（对象不互相引用）
// ② 交互逻辑集中（一处管理）
// ③ 加对象只改中介者
```
#### 题目5：中介者缺点
```
// ① 中介者可能很臃肿（集中所有逻辑）
// ② 中介者挂了 → 全部瘫痪（单点）
```
#### 题目6：和代理/外观区别
```
// 外观：封装复杂子系统（对外简化）
// 中介者：协调对象交互（对内协调）
```

