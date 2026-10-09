## 桥接模式
桥接是什么->解决继承爆炸->核心结构->JDBC关联->实战应用->面试高频题
### 桥接模式是什么
桥接模式：把抽象部分和实现部分分离，让它们可以独立变化。用组合关系代理继承关系，避免继承爆炸。
```
// 类比：手机与系统
// 手机品牌（抽象）：小米、华为、苹果
// 操作系统（实现）：Android、iOS、鸿蒙
// 两个维度独立扩展：
// 小米 + Android、小米 + 鸿蒙、华为 + 鸿蒙...
// 组合比继承灵活得多

// 类比：画笔与颜色
// 画笔（类型）：圆珠笔、钢笔、铅笔
// 颜色（实现）：红色、蓝色、黑色
// 任意组合
```
### 解决继承爆炸
```
// ❌ 继承实现（两个维度交叉 = 类爆炸）：
// 维度1：设备（电视、空调、音响）
// 维度2：遥控器（基础、语音、触屏）

// 继承写法：
// 电视基础遥控、电视语音遥控、电视触屏遥控
// 空调基础遥控、空调语音遥控、空调触屏遥控
// 音响基础遥控、音响语音遥控、音响触屏遥控
// 3 设备 × 3 遥控 = 9 个类！
// n × m = n*m 个类（爆炸）

// ✅ 桥接：设备（抽象）持有遥控（实现）
// 设备 3 个类 + 遥控 3 个类 = 6 个类
// 组合任意：new 电视(new 语音遥控())
// 扩展：加 1 个设备 = 加 1 类（不用 3 个）
```
什么时候用桥接
```
// ① 两个维度独立变化
// ② 继承会导致类爆炸（n×m）
// ③ 抽象和实现都要扩展
```
### 核心结构
```
// ① 实现接口（维度2：遥控器）
public interface RemoteControl {
    void powerOn();
    void powerOff();
    void setVolume(int volume);
}

// ② 具体实现（各种遥控器）
public class BasicRemote implements RemoteControl {
    @Override
    public void powerOn() { System.out.println("基础遥控：开机"); }
    @Override
    public void powerOff() { System.out.println("基础遥控：关机"); }
    @Override
    public void setVolume(int volume) { System.out.println("音量：" + volume); }
}

public class VoiceRemote implements RemoteControl {
    @Override
    public void powerOn() { System.out.println("语音遥控：开机"); }
    // ...
}

// ③ 抽象类（维度1：设备，持有实现）
public abstract class Device {
    protected final RemoteControl remote;  // 桥（组合实现）

    public Device(RemoteControl remote) {
        this.remote = remote;   // 桥接实现
    }

    // 抽象方法（子类扩展）
    public abstract void play();

    // 通用操作（委托给实现）
    public void powerOn() { remote.powerOn(); }
    public void powerOff() { remote.powerOff(); }
}

// ④ 具体抽象（各种设备）
public class TV extends Device {
    public TV(RemoteControl remote) {
        super(remote);
    }

    @Override
    public void play() {
        System.out.println("电视播放节目");
    }
}

public class Speaker extends Device {
    public Speaker(RemoteControl remote) {
        super(remote);
    }

    @Override
    public void play() {
        System.out.println("音响播放音乐");
    }
}

// ⑤ 使用（任意组合）
Device tv = new TV(new BasicRemote());       // 电视 + 基础遥控
Device speaker = new Speaker(new VoiceRemote());  // 音响 + 语音遥控

tv.powerOn();      // 委托给遥控实现
tv.play();         // 设备自己的行为
tv.powerOff();
```
核心思想
```
// ① 抽象（设备）持有实现（遥控）—— 桥
// ② 两个维度独立扩展（加设备/加遥控只加一个类）
// ③ 组合代替继承（避免 n×m 爆炸）
// ④ 调用：抽象方法 → 委托实现（桥传递）
```
### 和JDBC的关联
```
// JDBC 就是桥接模式的经典应用！

// 抽象部分：JDBC API（DriverManager、Connection）
// 实现部分：各数据库驱动（MySQL、Oracle、PostgreSQL）

// 桥：DriverManager 持有具体驱动
Class.forName("com.mysql.cj.jdbc.Driver");   // 加载实现
Connection conn = DriverManager.getConnection(url, user, pass);  // 桥接

// 为什么是桥接：
// ① 抽象（JDBC API）不依赖具体数据库
// ② 实现（驱动）独立扩展（加数据库=加驱动）
// ③ 客户端代码不变，换驱动换数据库
// ④ JDBC API 持有驱动实现（桥）

// 其他桥接应用：
// ① SLF4J（日志门面）+ log4j/logback（实现）
// ② Java 集合：List 接口 + ArrayList/LinkedList 实现
// ③ 图形库：Shape + 不同渲染器
```
### 实战应用
```
// 两个维度：
// ① 判题引擎类型（抽象）：本地引擎、云引擎、第三方引擎
// ② 判题模式（实现）：标准判题、性能判题、安全判题

// 如果继承：2×3 = 6 个类（爆炸）
// 桥接：引擎 3 类 + 模式 2 类 = 5 类（组合）

// 实现接口（模式）
public interface JudgeMode {
    JudgeResult execute(Submission s);
}

// 具体实现（模式）
public class StandardMode implements JudgeMode {
    @Override
    public JudgeResult execute(Submission s) {
        return judge(s);   // 标准判题
    }
}
public class PerformanceMode implements JudgeMode {
    // 性能判题（记录耗时、内存）
}

// 抽象（引擎）
public abstract class JudgeEngine {
    protected final JudgeMode mode;   // 桥

    public JudgeEngine(JudgeMode mode) {
        this.mode = mode;
    }

    public abstract JudgeResult judge(Submission s);
}

// 具体抽象（引擎）
public class LocalJudgeEngine extends JudgeEngine {
    public LocalJudgeEngine(JudgeMode mode) { super(mode); }
    @Override
    public JudgeResult judge(Submission s) {
        return mode.execute(s);   // 委托模式实现
    }
}

// 使用（任意组合）
JudgeEngine local = new LocalJudgeEngine(new StandardMode());
JudgeEngine cloud = new CloudJudgeEngine(new PerformanceMode());
```
### 高频面试题
#### 题目1：桥接模式是什么
```
// 抽象和实现分离，独立扩展
// 组合代替继承
```
#### 题目2：解决什么问题
```
// 两个维度独立变化时
// 继承会 n×m 类爆炸
```
#### 题目3：JDBC为什么是桥接
```
// JDBC API（抽象）+ 数据库驱动（实现）
// 换驱动换数据库，代码不变
```
#### 题目4：桥接模式和适配器模式区别
```
// 桥接：设计时分离维度（主动设计）
// 适配器：运行时适配不兼容（补救）
```
#### 题目5：桥接模式优缺点
```
优点
// ① 避免类爆炸（n+m vs n×m）
// ② 两个维度独立扩展
// ③ 符合开闭

缺点
// ① 设计复杂（要识别维度）
// ② 增加抽象层（理解成本）
```
