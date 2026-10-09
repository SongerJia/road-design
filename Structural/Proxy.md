## 代理模式
代理是什么->静态代理->JDK动态代理->CGLIB->对比->应用场景->面试高频题
### 代理是什么
代理模式：不直接访问目标对象，通过代理对象访问。代理可以在调用前后附加逻辑。
```
客户端 → 代理对象（增强逻辑）→ 目标对象
```
为什么需要代理
```
// ① 增强：加日志、鉴权、事务（不改目标代码）
// ② 解耦：目标不知道代理存在
// ③ 延迟：代理延迟加载目标（懒加载）
// ④ 远程：代理代表远程对象（RPC）

// 类比：明星的经纪人（代理）
// 你找明星 → 先找经纪人 → 经纪人安排（筛选、收费）
```
### 静态代理
```
// 代理类手动编写（编译期存在）

// ① 接口
public interface UserService {
    void getUser();
}

// ② 目标类
public class UserServiceImpl implements UserService {
    public void getUser() {
        System.out.println("获取用户");
    }
}

// ③ 代理类（手动写）
public class UserServiceProxy implements UserService {
    private final UserService target;

    public UserServiceProxy(UserService target) {
        this.target = target;
    }

    @Override
    public void getUser() {
        System.out.println("前置：日志");
        target.getUser();          // 调用目标
        System.out.println("后置：统计");
    }
}

// 使用
UserService service = new UserServiceProxy(new UserServiceImpl());
service.getUser();

// 缺点：
// ① 一个接口一个代理类（代码膨胀）
// ② 代理类要手写（维护成本）
// ③ 目标类改动 → 代理也要改
// 解决：动态代理（运行时生成）
```
### JDK动态代理
```
// 运行时动态生成代理类（基于接口）

// ① 实现 InvocationHandler
public class LogHandler implements InvocationHandler {
    private final Object target;

    public LogHandler(Object target) {
        this.target = target;
    }

    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
        // 前置增强
        System.out.println("前置：" + method.getName());

        // 调用目标方法
        Object result = method.invoke(target, args);

        // 后置增强
        System.out.println("后置：" + method.getName());
        return result;
    }
}

// ② 创建代理
public class ProxyFactory {
    public static Object createProxy(Object target) {
        return Proxy.newProxyInstance(
            target.getClass().getClassLoader(),   // 类加载器
            target.getClass().getInterfaces(),     // 接口列表
            new LogHandler(target)                 // 调用处理器
        );
    }
}

// 使用
UserService proxy = (UserService) ProxyFactory.createProxy(new UserServiceImpl());
proxy.getUser();
// 输出：前置：getUser / 获取用户 / 后置：getUser

// 关键限制：目标类必须有接口！
// 没有接口 → JDK 代理无法创建（要 CGLIB）
```
原理
```
// Proxy.newProxyInstance：
// ① 根据接口动态生成一个代理类（字节码）
// ② 代理类实现所有接口
// ③ 方法调用 → InvocationHandler.invoke()
// ④ invoke 里可以增强 + 反射调目标

// 代理类和目标类实现同一接口
// 不是继承目标类
```
### CGLIB代理
```
// 运行时生成目标类的"子类"（基于继承）
// 目标类不需要接口

// ① 实现 MethodInterceptor   回调接口，是对于代理类的方法做什么
public class LogInterceptor implements MethodInterceptor {
    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy)
            throws Throwable {
        // 前置增强
        System.out.println("前置：" + method.getName());

        // 调用目标（注意：用 proxy.invokeSuper，不是 method.invoke）
        Object result = proxy.invokeSuper(obj, args);

        // 后置增强
        System.out.println("后置：" + method.getName());
        return result;
    }
}

// ② 创建代理
public class CglibProxyFactory {
    public static Object createProxy(Class<?> targetClass) {
        Enhancer enhancer = new Enhancer();
        enhancer.setSuperclass(targetClass);      // 父类 = 目标类
        enhancer.setCallback(new LogInterceptor());
        return enhancer.create();                 // 生成子类实例
    }
}

// 使用（不需要接口！）
UserServiceImpl proxy = (UserServiceImpl) CglibProxyFactory.createProxy(UserServiceImpl.class);
proxy.getUser();

// 限制：
// ① 目标类不能 final（不能继承）
// ② 方法不能 final（不能覆写）
// ③ 私有方法不能代理
```
CGLIB原理
```
// Enhancer 生成目标类的子类
// 覆写目标方法
// 调用 → MethodInterceptor.intercept()
// 内部用 MethodProxy.invokeSuper 调用父类（目标）

// 注意：
// 用 invokeSuper 而不是 method.invoke(obj)
// （invoke 会再走拦截器，死循环！）
```
### JDK vs CGLIB
| 对比              | JDK 动态代理               | CGLIB                  |
| --------------- | ---------------------- | ---------------------- |
| **原理**          | 接口 + InvocationHandler | 继承 + 子类                |
| **接口要求**        | ✅ 必须有接口                | ❌ 不需要                  |
| **目标 final**    | 无所谓                    | ❌ 不能 final             |
| **方法 final**    | 无所谓                    | ❌ 不能 final             |
| **创建速度**        | 快                      | 慢（字节码生成）               |
| **调用速度**        | JDK8+ 已优化              | 快                      |
| **Spring Boot** | ❌ 默认                   | ✅ 默认（proxyTargetClass） |
### 应用场景
①SpringAOP
```
// AOP = 动态代理 + 切面逻辑
// 有接口 → JDK 代理
// 无接口 → CGLIB

// 应用：
// @Transactional（事务）
// @Async（异步）
// @Cacheable（缓存）
// 自定义切面（日志、限流）
```
②Mybatis Mapper（JDK代理）
```
// Mapper 接口没有实现类
// MyBatis 用 JDK 代理生成实现
// 调用方法 → 代理 → 执行 SQL → 返回结果
```
③懒加载（Hibernate）
```
// 代理对象替代真实实体
// 访问属性时才真正加载（延迟）
```
④RPC\远程调用
```
// Feign/MyBatis 代理：
// 方法调用 → 网络请求 → 返回
// 客户端无感（像本地调用）
```
### 面试高频题
#### 题目1：代理模式是什么
```
// 通过代理间接访问目标
// 代理附加增强逻辑
```
#### 题目2：JDK和CGLIB的区别
```
// JDK：接口 + InvocationHandler
// CGLIB：继承 + 子类
// JDK 要接口，CGLIB 类不能 final
```
#### 题目3：Spring使用哪种
```
// 有接口 JDK，没接口 CGLIB
// Boot 2+ 默认 CGLIB
```
#### 题目4：静态代理 vs 动态代理
```
// 静态：手写（编译期）
// 动态：运行时生成（JDK/CGLIB）
```
#### 题目5：CGLIB为什么不能代理final
```
// 基于继承生成子类
// final 类不能继承、final 方法不能覆写
```
#### 题目6：AOP和代理的关系
```
// AOP 的实现就是动态代理
// 切面逻辑织入代理
```
