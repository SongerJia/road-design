## 适配器模式
适配器是什么->两种实现->类适配 vs 对象适配->Spring HandlerAdapter->应用场景->面试高频题
### 适配器模式是什么
适配器模式：把一个类的接口转换为客户端期望的另一个接口，让原本不兼容的类能一起工作。
```
// 类比：充电头转换器
// 国外插座（110V）→ 转换器（适配器）→ 手机（220V 设备）
// 不改插座、不改手机，加个转换器就能用

// 类比：USB 转 Type-C 转接头
// 两端接口不匹配 → 加转接头适配
```
解决什么问题
```
// ❌ 接口不兼容（无法直接使用）：
public interface ChineseSocket {
    void connect220V();
}
public class ChinesePlug implements ChineseSocket {
    public void connect220V() { ... }
}

// 但是你的设备只支持：
public interface USInterface {
    void connect110V();
}
// 设备无法直接用 ChinesePlug（接口不匹配）

// ✅ 适配器：把 220V 适配成 110V
USInterface adapter = new SocketAdapter(chinesePlug);  // 能用了
```
### 核心结构 （两种实现）
①类适配器（继承）
```
// 用"继承"实现适配（适配器继承被适配者）

// 被适配者（已有的类）
public class Adaptee {
    public void specificRequest() {
        System.out.println("被适配者的方法");
    }
}

// 目标接口（客户端期望的）
public interface Target {
    void request();
}

// 类适配器（继承被适配者 + 实现目标接口）
public class ClassAdapter extends Adaptee implements Target {
    @Override
    public void request() {
        specificRequest();  // 直接调用继承的方法
    }
}

// 使用
Target target = new ClassAdapter();
target.request();  // 客户端只面对 Target 接口
```
②对象适配器（组合）
```
// 用"组合"实现适配（适配器持有被适配者）
public class ObjectAdapter implements Target {
    private final Adaptee adaptee;   // 组合

    public ObjectAdapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
        adaptee.specificRequest();  // 委托给被适配者
    }
}

// 使用
Target target = new ObjectAdapter(new Adaptee());
target.request();
```

|对比|类适配器|对象适配器|
|---|---|---|
|**实现**|继承|组合|
|**耦合**|强（继承）|弱（组合）|
|**灵活性**|低|高（可换被适配者）|
|**推荐**|少用|✅ 推荐|
|**多重适配**|可多重继承（Java 不行）|组合多个|
### 和Spring的关联
HandlerAdapter就是适配器
```
// Spring MVC 的 HandlerAdapter 是典型适配器
// 目的：让 DispatcherServlet 统一调用"不同类型的 Handler"

// 问题：
// ① @RequestMapping 方法（HandlerMethod）
// ② HttpRequestHandler 接口
// ③ Controller 接口（旧式）
// 三种 Handler 调用方式不同！

// 解决：每种 Handler 一个适配器
// ① RequestMappingHandlerAdapter → 适配 HandlerMethod
// ② HttpRequestHandlerAdapter → 适配 HttpRequestHandler
// ③ SimpleControllerHandlerAdapter → 适配 Controller

// DispatcherServlet 只面对统一接口：
public interface HandlerAdapter {
    boolean supports(Object handler);        // 是否支持该 Handler
    ModelAndView handle(HttpServletRequest request,
                        HttpServletResponse response,
                        Object handler);     // 统一调用
}

// 效果：
// DispatcherServlet 不关心 Handler 类型
// 每种类型由对应适配器处理（适配差异）
```
其他适配器应用
```
// ① LoggerAdapter（日志门面适配：SLF4J 适配 log4j/logback）
// ② TypeAdapter（Gson 类型适配器）
// ③ DriverAdapter（JDBC 驱动适配不同数据库）
// ④ MyBatis CacheAdapter（缓存适配）
```
### 实战应用
```
// 适配不同"判题源"
public interface JudgeProvider {
    JudgeResult submit(Submission s);
}

// 已有系统（不兼容）：第三方判题 API
public class ThirdPartyJudgeApi {
    public String judge(String code, String lang) {
        // 第三方接口（参数格式不同）
        return "PASS:200";
    }
}

// 适配器：把第三方 API 适配成统一接口
public class ThirdPartyJudgeAdapter implements JudgeProvider {
    private final ThirdPartyJudgeApi api;

    public ThirdPartyJudgeAdapter(ThirdPartyJudgeApi api) {
        this.api = api;
    }

    @Override
    public JudgeResult submit(Submission s) {
        // 转换参数格式 + 调用第三方 + 转换返回
        String result = api.judge(s.getCode(), s.getLanguage());
        return parseResult(result);
    }
}

// 使用（统一接口）
JudgeProvider provider = new ThirdPartyJudgeAdapter(new ThirdPartyJudgeApi());
JudgeResult result = provider.submit(submission);
// 换判题源 → 换适配器（不改业务代码）
```
### 与适配器/代理对比
| 对比     | 适配器      | 装饰器   | 代理      |
| ------ | -------- | ----- | ------- |
| **目的** | 接口转换     | 功能增强  | 访问控制/增强 |
| **接口** | 换成新接口    | 保持原接口 | 保持原接口   |
| **关系** | 不兼容 → 兼容 | 包装加功能 | 间接访问    |
| **类比** | 转换头      | 加装饰   | 经纪人     |
### 高频面试题
#### 题目1：适配器模式是什么
```
// 接口转换，让不兼容的类能一起工作
```
#### 题目2：类适配器和对象适配器
```
// 类：继承（耦合强，少用）
// 对象：组合（推荐）
```
#### 题目3：Spring哪里用了
```
// HandlerAdapter（适配不同 Handler）
// DispatcherServlet 统一调用
```
#### 题目4：和装饰者区别
```
// 适配器：接口换成新的（转换）
// 装饰器：接口不变加功能（增强）
```
#### 题目5：和代理区别
```
// 适配器：解决不兼容
// 代理：控制访问
```
#### 题目6：适配器优点/缺点
```
// 优点：解耦、复用已有类
// 缺点：适配器多了增加复杂度
```


