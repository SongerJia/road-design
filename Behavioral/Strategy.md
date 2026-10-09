## 策略模式
策略是什么->核心结构->实际应用->与if-else对比->与状态/模版对比->面试高频题
### 策略模式是什么
策略模式：定义一组算法，把它封装起来，运行时可以互相替换。客户端不关心具体算法，只依赖策略接口。
```
// 类比：出行方式（策略）
// 打车 / 地铁 / 骑行 → 都是"去目的地"的策略
// 换策略不用改代码（换参数/换对象）
```
解决什么问题
```
// ❌ 大量 if-else 判断算法
public Result judge(Submission submission) {
    if ("java".equals(submission.getLanguage())) {
        return judgeJava(submission);     // Java 判题逻辑
    } else if ("python".equals(submission.getLanguage())) {
        return judgePython(submission);   // Python 判题逻辑
    } else if ("js".equals(submission.getLanguage())) {
        return judgeJs(submission);
    }
    // 新增语言 → 加 else → 违反开闭原则
}

// ✅ 策略：每种语言一个策略类
JudgeStrategy strategy = strategyMap.get(submission.getLanguage());
return strategy.judge(submission);
// 新增语言 → 加策略类 + 注册 → 不改原代码（开闭 ✅）
```
### 核心结构
三大角色
```
// ① 策略接口（Strategy）
public interface JudgeStrategy {
    JudgeResult judge(Submission submission);
}

// ② 具体策略（ConcreteStrategy）
@Component("javaStrategy")
public class JavaJudgeStrategy implements JudgeStrategy {
    @Override
    public JudgeResult judge(Submission submission) {
        // Java 编译 + 运行 + 判题
        return new JudgeResult("PASS", 200);
    }
}

@Component("pythonStrategy")
public class PythonJudgeStrategy implements JudgeStrategy {
    @Override
    public JudgeResult judge(Submission submission) {
        // Python 判题
        return new JudgeResult("PASS", 180);
    }
}

// ③ 上下文（Context）—— 持有策略，调用策略
@Service
public class JudgeContext {
    // 策略注册表（Spring 注入所有策略）
    @Autowired
    private Map<String, JudgeStrategy> strategyMap;  // key = bean 名

    public JudgeResult execute(String language, Submission submission) {
        JudgeStrategy strategy = strategyMap.get(language + "Strategy");
        if (strategy == null) {
            throw new UnsupportedOperationException("不支持的语言: " + language);
        }
        return strategy.judge(submission);  // 调用策略（多态）
    }
}

// 使用
JudgeResult result = judgeContext.execute("java", submission);
```
为什么Map注入是策略的完美确认
```
// Spring 会把所有 JudgeStrategy 实现注入到 Map
// key = Bean 名，value = 策略实例

// 新增语言：加一个策略类（@Component）→ 自动注册
// 不用改任何代码 ✅（开闭原则完美）
```
### 与if-else对比
| 对比       | if-else | 策略模式      |
| -------- | ------- | --------- |
| **扩展**   | 加分支改原代码 | 加策略类（不改）  |
| **开闭原则** | ❌ 违反    | ✅ 符合      |
| **职责**   | 判断逻辑集中  | 每个策略独立    |
| **测试**   | 难       | 易（单测每个策略） |
| **复杂度**  | 低       | 中（类变多）    |
什么时候用策略
```
// ① 多个算法/行为，运行时切换
// ② 大量 if-else 判断"做什么"
// ③ 新增行为频繁（扩展性要求高）

// 什么时候不用
// ① 分支少且稳定（3 个以内）
// ② 一次性判断（不频繁变更）
```
### 策略+工厂/枚举组合
策略+枚举（简化注册）
```
// 枚举自带策略（前面 JUC 讲过"常量特定方法"）
public enum JudgeStrategy {
    JAVA {
        public JudgeResult judge(Submission s) { ... }
    },
    PYTHON {
        public JudgeResult judge(Submission s) { ... }
    };

    public abstract JudgeResult judge(Submission s);
}

// 使用
JudgeStrategy.valueOf("JAVA").judge(submission);
```
策略+工厂
```
// 工厂负责"选择策略"
public class StrategyFactory {
    public static JudgeStrategy create(String language) {
        switch (language) {
            case "java": return new JavaJudgeStrategy();
            case "python": return new PythonJudgeStrategy();
            default: throw new IllegalArgumentException();
        }
    }
}
// 客户端：StrategyFactory.create("java") 拿到策略
```
和策略模式对比其他模式
```
// 策略 vs 状态模式：
// 策略：客户端主动换算法（外部选择）
// 状态：对象内部自动切换（状态驱动）

// 策略 vs 模板方法：
// 策略：整个算法替换（接口多实现）
// 模板：算法骨架固定，细节子类实现（继承）
```
### 实战应用
```
// 需求：判题支持多种语言

// ① 策略接口
public interface JudgeStrategy {
    JudgeResult judge(Submission submission);
}

// ② Java 策略
@Component("java")
public class JavaJudgeStrategy implements JudgeStrategy {
    @Override
    public JudgeResult judge(Submission submission) {
        // 1. 编译 .java
        // 2. 运行 + 输入测试用例
        // 3. 比对输出
        return JudgeResult.of(200, "AC");
    }
}

// ③ Python 策略
@Component("python")
public class PythonJudgeStrategy implements JudgeStrategy {
    @Override
    public JudgeResult judge(Submission submission) {
        // 1. 运行 .py
        // 2. 比对输出
        return JudgeResult.of(200, "AC");
    }
}

// ④ 上下文（策略分发）
@Service
public class JudgeService {
    @Autowired
    private Map<String, JudgeStrategy> strategies;

    public JudgeResult judge(Submission submission) {
        JudgeStrategy strategy = strategies.get(submission.getLanguage());
        return strategy.judge(submission);
    }
}

// ⑤ 新增语言（Go）→ 加一个策略类即可
@Component("go")
public class GoJudgeStrategy implements JudgeStrategy { ... }
// 无需修改任何已有代码 ✅
```
### 面试高频题
#### 题目1：策略模式是什么
```
// 算法封装 + 运行时替换
// 客户端依赖接口
```
#### 题目2：和if-else区别
```
// 策略：开闭原则、独立测试
// if-else：改原代码、难测试
```
#### 题目3：策略模式三大角色
```
// 策略接口、具体策略、上下文（策略分发）
```
#### 题目4：Spring怎么用策略
```
// Map 注入（Bean 名做 key）
// 按条件取策略
```
#### 题目5：策略和状态模式区别
```
// 策略：外部换算法
// 状态：内部状态驱动
```
#### 题目6：策略模式的缺点
```
// 类数量变多
// 客户端要知道有哪些策略
```