## 装饰器模式
装饰器是什么->核心结构->JavaIO关联->实战应用->对比->面试高频题
### 装饰器模式是什么
装饰器模式：动态地给对象增加功能，通过包装一层层叠加，不改原有类。装饰器和被装饰者实现同一接口。
```
// 类比：咖啡加料
// 咖啡（原对象）→ 加奶（装饰器1）→ 加糖（装饰器2）→ 加冰（装饰器3）
// 每加一层料，咖啡"功能增强"（口味变丰富）
// 咖啡本身没变（还是咖啡），只是被一层层包起来

// 类比：蛋糕加装饰
// 蛋糕胚 → 抹奶油 → 摆水果 → 写名字
// 一层层叠加，每次加一种装饰
```
解决什么问题
```
// ❌ 继承爆炸（每组合一个类）：
public class Coffee {}
public class MilkCoffee extends Coffee {}
public class SugarCoffee extends Coffee {}
public class MilkSugarCoffee extends Coffee {}   // 组合要新类
public class MilkSugarIceCoffee extends Coffee {} // 组合爆炸！
// n 种配料 = 2^n 个类（不可维护）

// ✅ 装饰器：配料是装饰器，随意组合
Coffee coffee = new IceDecorator(new SugarDecorator(new MilkDecorator(new Coffee())));
// 任意组合，不用新建类
```
### 核心结构
```
// ① 组件接口（咖啡）
public interface Coffee {
    double cost();
    String description();
}

// ② 具体组件（基础咖啡）
public class BasicCoffee implements Coffee {
    @Override
    public double cost() {
        return 10.0;
    }

    @Override
    public String description() {
        return "咖啡";
    }
}

// ③ 抽象装饰器（持有被装饰者，转发调用）
public abstract class CoffeeDecorator implements Coffee {
    protected final Coffee coffee;   // 持有被装饰者（组合）

    public CoffeeDecorator(Coffee coffee) {
        this.coffee = coffee;
    }

    @Override
    public double cost() {
        return coffee.cost();   // 转发给被装饰者
    }

    @Override
    public String description() {
        return coffee.description();
    }
}

// ④ 具体装饰器（加奶）
public class MilkDecorator extends CoffeeDecorator {
    public MilkDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public double cost() {
        return super.cost() + 3.0;   // 原价 + 加奶价
    }

    @Override
    public String description() {
        return super.description() + " + 奶";
    }
}

// 具体装饰器（加糖）
public class SugarDecorator extends CoffeeDecorator {
    public SugarDecorator(Coffee coffee) {
        super(coffee);
    }

    @Override
    public double cost() {
        return super.cost() + 2.0;
    }

    @Override
    public String description() {
        return super.description() + " + 糖";
    }
}

// ⑤ 使用（层层包装，任意组合）
Coffee coffee = new SugarDecorator(          // 最外层：加糖
    new MilkDecorator(                        // 中层：加奶
        new BasicCoffee()));                  // 内层：基础咖啡

System.out.println(coffee.description());  // 咖啡 + 奶 + 糖
System.out.println(coffee.cost());         // 15.0
```
核心思想
```
// ① 装饰器实现同一接口（接口不变）
// ② 装饰器持有被装饰者（组合）
// ③ 装饰器转发 + 增强（叠加功能）
// ④ 任意组合（不用建类）
// ⑤ 符合开闭（加装饰不改原有）
```
### 和JavaIO流的关联
```
// Java IO 就是装饰器模式的教科书

// 基础流（具体组件）：
// FileInputStream（读文件）

// 装饰器（叠加功能）：
// BufferedInputStream（加缓冲，性能增强）
// DataInputStream（加基本类型读取）
// ObjectInputStream（加对象反序列化）

// 层层包装：
InputStream in = new BufferedInputStream(
    new FileInputStream("test.txt"));
// 文件流（基础）→ 缓冲（性能增强）

// 经典组合：
ObjectInputStream ois = new ObjectInputStream(
    new BufferedInputStream(
        new FileInputStream("obj.dat")));
// 文件 → 缓冲 → 对象读取（三层装饰）

// 为什么是装饰器：
// ① 都是 InputStream 子类（同一接口）
// ② 每个包装类持有 InputStream（组合）
// ③ 叠加功能（缓冲/类型/对象）
// ④ 任意组合（想加几层加几层）
```
其他装饰器应用
```
// ① Servlet 的 Response 包装（HttpServletResponseWrapper）
// ② Collections 的同步/不可变包装（synchronizedList 是装饰）
// ③ Spring 的 HttpMessageConverter（装饰消息处理）
// ④ Java NIO 的 ChannelWrapper
```
### 实战应用
```
// 判题结果增强（叠加统计/缓存/通知）
public interface JudgeResultProvider {
    JudgeResult getResult(Long taskId);
}

// 基础实现
public class BasicResultProvider implements JudgeResultProvider {
    @Override
    public JudgeResult getResult(Long taskId) {
        return judgeService.getById(taskId);
    }
}

// 装饰器1：加缓存
public class CacheResultDecorator implements JudgeResultProvider {
    private final JudgeResultProvider delegate;
    private final Cache cache;

    public CacheResultDecorator(JudgeResultProvider delegate, Cache cache) {
        this.delegate = delegate;
        this.cache = cache;
    }

    @Override
    public JudgeResult getResult(Long taskId) {
        // 先查缓存（增强）
        JudgeResult cached = cache.get(taskId);
        if (cached != null) return cached;
        JudgeResult result = delegate.getResult(taskId);  // 转发
        cache.put(taskId, result);
        return result;
    }
}

// 装饰器2：加日志
public class LogResultDecorator implements JudgeResultProvider {
    // 类似，包装加日志

}

// 使用（任意组合）
JudgeResultProvider provider = new LogResultDecorator(   // 日志
    new CacheResultDecorator(                            // 缓存
        new BasicResultProvider(), cache));
```
### 与适配器/代理对比
| 对比     | 装饰器   | 适配器   | 代理   |
| ------ | ----- | ----- | ---- |
| **接口** | 不变    | 换成新接口 | 不变   |
| **目的** | 增强功能  | 接口转换  | 控制访问 |
| **方向** | 叠加功能  | 转换适配  | 间接访问 |
| **多层** | ✅ 可多层 | 单层    | 单层   |
装饰器 vs 继承
```
// 继承：静态组合（编译期定死）
// 装饰器：动态组合（运行期叠加）

// 例子：
// 继承：MilkSugarCoffee（写死组合）
// 装饰：Sugar(Milk(Coffee))（任意组合）

// 结论：需要灵活组合时用装饰器（避免类爆炸）
```
### 面试高频题
#### 题目1：装饰器模式是什么
```
// 动态叠加功能（包装）
// 接口不变，一层层增强
```
#### 题目2：Java IO为什么是装饰器
```
// FileInputStream 基础流
// BufferedInputStream 等包装类叠加功能
// 同一接口 + 组合 + 任意组合
```
#### 题目3：和适配器区别
```
// 装饰器：接口不变增强
// 适配器：接口转换
```
#### 题目4：和继承区别
```
// 继承：静态组合（类爆炸）
// 装饰器：动态组合（灵活）
```
#### 题目5：装饰器优点
```
// ① 灵活组合（不用类爆炸）
// ② 符合开闭（加装饰不改原类）
// ③ 功能可叠加可移除
```
#### 题目6：缺点
```
// ① 类数量多（每功能一装饰器）
// ② 多层包装调试难
// ③ 顺序影响结果（依赖顺序的注意）
```
