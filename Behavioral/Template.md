## 模版方法模式
模版方法是什么->核心结构->JdbcTemplate关联->与策略对比->实战应用->面试高频题
### 模版方法是什么
模版方法：父类定义算法的固定步骤，把可变步骤交给子类实现。固定步骤不变，细节子类定制。
```
// 类比：做菜流程（模板）
// 洗菜 → 切菜 → 炒菜 → 装盘（骨架固定）
// 做什么菜（子类定制）：番茄炒蛋 / 宫保鸡丁

// 核心：父类"模板方法" + 子类"实现钩子"
```
解决什么问题
```
// ❌ 重复代码：
public class JavaJudge {
    public void execute() {
        validate();      // 固定
        compile();       // Java 特有
        run();           // Java 特有
        saveResult();    // 固定
    }
}
public class PythonJudge {
    public void execute() {
        validate();      // 重复！
        run();           // Python 特有
        saveResult();    // 重复！
    }
}
// validate/saveResult 重复

// ✅ 模板方法：固定逻辑放父类，可变逻辑子类实现
```
### 核心结构
```
// ① 抽象父类（模板类）
public abstract class BaseJudge {

    // 模板方法（final：子类不能改骨架）
    public final JudgeResult execute(Submission submission) {
        validate(submission);          // 步骤 1：固定（父类实现）
        String output = run(submission); // 步骤 2：可变（子类实现）
        JudgeResult result = check(submission, output); // 步骤 3：可变（子类实现）
        saveResult(result);            // 步骤 4：固定（父类实现）
        return result;
    }

    // 固定步骤（父类实现）
    private void validate(Submission submission) {
        // 通用校验：代码非空、长度限制
    }

    // 可变步骤（子类实现）—— 抽象方法
    protected abstract String run(Submission submission);

    // 可变步骤（子类实现）
    protected abstract JudgeResult check(Submission submission, String output);

    // 固定步骤
    private void saveResult(JudgeResult result) {
        // 保存判题结果
    }

    // 钩子方法（可选，子类按需覆写）
    protected boolean needSpecialCheck() {
        return false;  // 默认不需要
    }
}

// ② 具体子类
public class JavaJudge extends BaseJudge {
    @Override
    protected String run(Submission submission) {
        // Java 编译 + 运行
        return compileAndRun(submission);
    }

    @Override
    protected JudgeResult check(Submission submission, String output) {
        // Java 判分逻辑
        return new JudgeResult("AC", 200);
    }
}

// ③ 使用
JudgeResult result = new JavaJudge().execute(submission);
// 骨架固定：校验 → 运行 → 判分 → 保存
// Java 只定制了"运行"和"判分"
```
模版方法的特征
```
// ① 模板方法 final：骨架不能被子类改
// ② 抽象方法：子类必须实现（可变步骤）
// ③ 钩子方法：子类可选覆写（扩展点）
// ④ 固定方法 private/普通：父类实现
```
### JdbcTemplate关联
```
// JdbcTemplate 就是模板方法 + 回调的经典
// 骨架：获取连接 → 执行 SQL → 处理结果 → 关闭

// 简化版：
public class JdbcTemplate {
    // 模板方法：固定数据库操作骨架
    public <T> T execute(StatementCallback<T> action) {
        // ① 获取连接（固定）
        Connection con = getConnection();

        // ② 创建 Statement（固定）
        Statement stmt = con.createStatement();

        // ③ 执行业务（可变 → 回调）
        T result = action.doInStatement(stmt);

        // ④ 关闭资源（固定）
        close(stmt);
        close(con);
        return result;
    }
}

// 使用（只写业务 SQL 部分）
String name = jdbcTemplate.execute(stmt -> {
    ResultSet rs = stmt.executeQuery("SELECT name FROM user WHERE id=1");
    rs.next();
    return rs.getString("name");
});
// 连接管理、异常处理、资源关闭都不用管（骨架做）
```
Spring里的模版
```
// JdbcTemplate：数据库操作骨架
// RestTemplate：HTTP 调用骨架
// RedisTemplate：Redis 操作骨架
// MongoTemplate：MongoDB 操作骨架
// 都是"模板方法"思想
```
### 与策略模式对比
| 对比       | 模板方法     | 策略模式      |
| -------- | -------- | --------- |
| **关系**   | 继承（父类骨架） | 组合（接口注入）  |
| **可变粒度** | 算法内某几步可变 | 整个算法替换    |
| **骨架**   | 固定（父类）   | 无骨架（各自实现） |
| **复用**   | 复用骨架代码   | 复用策略接口    |
| **适用**   | 步骤固定细节不同 | 算法整体多变    |
### 实战应用
```
// 模板方法（固定判题流程）+ 策略（语言实现）组合

// ① 模板方法：判题骨架
public abstract class BaseJudge {
    public final JudgeResult execute(Submission s) {
        validate(s);                                   // 固定
        JudgeStrategy strategy = getStrategy();        // 策略钩子
        return strategy.judge(s);                      // 语言策略
    }
    protected abstract JudgeStrategy getStrategy();
}

// ② 子类 + 策略：
public class JavaJudge extends BaseJudge {
    protected JudgeStrategy getStrategy() {
        return new JavaJudgeStrategy();  // 返回语言策略
    }
}

// 效果：
// 骨架复用（校验等固定逻辑）
// 语言差异用策略（可扩展）
// 模板 + 策略 组合是常见最佳实践
```
### 面试高频题
#### 题目1：模版方法是什么
```
// 父类定骨架，子类实现细节
// 骨架 final 不变
```
#### 题目2：与策略区别
```
// 模板：继承 + 骨架固定 + 部分可变
// 策略：组合 + 整个算法替换
```
#### 题目3：JdbcTemplate是什么模式
```
// 模板方法（骨架）+ 回调（可变部分）
```
#### 题目4：钩子方法是什么
```
// 可选覆写的方法（默认实现）
// 给子类提供扩展点
```
#### 题目5：模版方法为什么final
```
// 骨架不能被破坏
// 保证算法结构一致
```
