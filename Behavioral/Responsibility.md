## 责任链模式
责任链是什么->核心结构->SpringMVC拦截器关联->实战应用->与策略/观察者对比->面试高频题
### 责任链是什么
责任链模式：多个处理器组成链，请求沿着链传递，每个处理器决定自己处理还是下一个。
```
// 类比：请假审批链
// 员工请假 → 组长（≤3天批）→ 经理（≤7天批）→ 总监（全部批）
// 组长不处理就传经理，经理不处理传总监

// 核心：解耦"请求"和"处理者"，链上动态传递
```
解决什么问题
```
// ❌ 多层 if 判断谁处理：
public void approve(LeaveRequest request) {
    if (request.getDays() <= 3) {
        leader.approve(request);
    } else if (request.getDays() <= 7) {
        manager.approve(request);
    } else {
        director.approve(request);
    }
}
// 新增审批人 → 改 if（违反开闭）

// ✅ 责任链：每个处理器自己决定（谁处理谁拦截）
```
### 核心结构
```
// ① 处理接口（抽象处理者）
public abstract class Approver {
    // 下一个处理器
    protected Approver next;

    public void setNext(Approver next) {
        this.next = next;
    }

    // 处理请求（自己处理 or 传给下一个）
    public abstract void approve(LeaveRequest request);
}

// ② 具体处理器
public class LeaderApprover extends Approver {
    @Override
    public void approve(LeaveRequest request) {
        if (request.getDays() <= 3) {
            System.out.println("组长批准：" + request.getDays() + " 天");
        } else if (next != null) {
            next.approve(request);  // 传给下一个
        }
    }
}

public class ManagerApprover extends Approver {
    @Override
    public void approve(LeaveRequest request) {
        if (request.getDays() <= 7) {
            System.out.println("经理批准：" + request.getDays() + " 天");
        } else if (next != null) {
            next.approve(request);
        }
    }
}

// ③ 组装链
public class ChainBuilder {
    public Approver build() {
        Approver leader = new LeaderApprover();
        Approver manager = new ManagerApprover();
        Approver director = new DirectorApprover();

        leader.setNext(manager);      // 组长 → 经理
        manager.setNext(director);    // 经理 → 总监
        return leader;
    }
}

// ④ 使用
Approver chain = new ChainBuilder().build();
chain.approve(new LeaveRequest(5));   // 组长不批 → 经理批
```
关键点
```
// ① 每个处理者持有"下一个"引用
// ② 处理不了 → 传给下一个（动态）
// ③ 新增处理者 → 链上插入（改组装处，不改处理者）
// ④ 链的顺序可以灵活调整
```
### SpringMVC拦截器关联
```
// Spring MVC 的拦截器链就是责任链！

// ① 拦截器接口
public interface HandlerInterceptor {
    boolean preHandle(...);     // 前处理（false 中断）
    void postHandle(...);       // 后处理
    void afterCompletion(...);  // 完成
}

// ② 多个拦截器按顺序组成链
registry.addInterceptor(authInterceptor);    // 鉴权
registry.addInterceptor(rateLimitInterceptor);  // 限流
registry.addInterceptor(logInterceptor);      // 日志

// ③ 执行顺序（责任链）：
// preHandle1 → preHandle2 → preHandle3 → Controller
// → postHandle3 → postHandle2 → postHandle1
// → afterCompletion3 → 2 → 1

// 特点：
// preHandle 返回 false → 中断（不再往下）
// 类似责任链"处理不了/拦截"就停
```
其他责任链应用
```
// ① Servlet 过滤器链（FilterChain）
// ② AOP 通知链（MethodInterceptor.proceed）
// ③ MyBatis 拦截器（Plugin）
// ④ Netty 的 ChannelPipeline
// ⑤ Gateway 的过滤器链
// 都是责任链模式
```
### 实战应用
```
// 判题请求的校验链：
public abstract class JudgeFilter {
    protected JudgeFilter next;

    public void setNext(JudgeFilter next) {
        this.next = next;
    }

    public void doFilter(Submission submission) {
        // 子类处理，处理完调用 next.doFilter
        if (next != null) {
            next.doFilter(submission);
        }
    }
}

// 处理器1：参数校验
public class ValidateFilter extends JudgeFilter {
    @Override
    public void doFilter(Submission s) {
        if (s.getCode() == null || s.getCode().isEmpty()) {
            throw new IllegalArgumentException("代码为空");
        }
        super.doFilter(s);  // 传给下一个
    }
}

// 处理器2：语言检查
public class LanguageFilter extends JudgeFilter {
    @Override
    public void doFilter(Submission s) {
        if (!SUPPORTED.contains(s.getLanguage())) {
            throw new UnsupportedOperationException("不支持的语言");
        }
        super.doFilter(s);
    }
}

// 处理器3：频率限制
public class RateLimitFilter extends JudgeFilter {
    @Override
    public void doFilter(Submission s) {
        if (limitService.isOverLimit(s.getUserId())) {
            throw new RateLimitException("提交过于频繁");
        }
        super.doFilter(s);
    }
}

// 组装
// validate → language → rateLimit → 判题
```
### 与策略/观察者对比
| 对比     | 责任链    | 策略   | 观察者   |
| ------ | ------ | ---- | ----- |
| **关系** | 链式传递   | 可选替换 | 一对多通知 |
| **方向** | 逐个往下   | 选择其一 | 广播所有  |
| **核心** | 谁处理谁拦截 | 换算法  | 通知变化  |
| **类比** | 审批链    | 出行方式 | 公众号订阅 |
责任链vs策略
```
// 策略：选"一个"处理（互斥）
// 责任链：可以"多个"依次处理（串联）

// 例：
// 判题语言选择 → 策略（选一种）
// 提交校验（参数→语言→限流）→ 责任链（依次过）
```
### 高频面试题
#### 题目1：责任链模式是什么
```
// 多个处理者串成链
// 每个决定处理还是传下一个
```
#### 题目2：和策略区别
```
// 策略：选一个
// 责任链：依次过多个
```
#### 题目3：Spring MVC哪里用责任链
```
// 拦截器链（HandlerInterceptor）
// 过滤器链、AOP 通知链
```
#### 题目4：怎么中断链
```
// 处理者返回 false/抛异常 → 停止传递
// preHandle 返回 false 中断
```
#### 题目5：责任链优点
```
// 解耦（请求和处理者分离）
// 开闭（加处理者不改原有）
// 灵活（链顺序可调）
```
#### 题目6：责任链缺点
```
// 链长影响性能
// 请求可能没人处理（要兜底）
// 调试难（链上哪一步出问题）
```
