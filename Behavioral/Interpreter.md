## 解释器模式
解释器是什么->核心结构->表达式分析->规则引擎关联->应用场景->面试高频题
### 解释器模式是什么
解释器模式：定义一个语言的语法规则，用解释器对象解析并执行符合语法的表达式。
```
// 类比：计算器
// 输入 "3 + 5 * 2"（表达式）
// 计算器按语法规则解析 → 得出 13
// 语法规则固定，可以解释不同输入

// 类比：翻译机
// 定义"语言规则"（语法）
// 输入一句话 → 按规则解析 → 翻译执行
```
解决什么问题
```
// ❌ 硬编码判断表达式：
public int eval(String exp) {
    if (exp.contains("+")) {
        // 解析加法
    } else if (exp.contains("*")) {
        // 解析乘法
    }
    // 每加一种运算 → 改代码（违反开闭）
}

// ✅ 解释器：每个语法规则一个类，可组合
// "3 + 5 * 2" → 表达式树 → 节点解释执行
```
### 核心结构
语法与表达式树
```
// 表达式 "3 + 5 * 2" 的语法树：
//        +
//       / \
//      3   *
//         / \
//        5   2

// 每个节点是一个"表达式"（解释器节点）
// 叶子：数字（终结符）
// 内部：运算符（非终结符，组合子表达式）
```
代码实现
```
// ① 抽象表达式（所有节点的接口）
public interface Expression {
    int interpret();   // 解释执行
}

// ② 终结符表达式（数字）
public class NumberExpression implements Expression {
    private final int value;

    public NumberExpression(int value) {
        this.value = value;
    }

    @Override
    public int interpret() {
        return value;   // 数字直接返回
    }
}

// ③ 非终结符表达式（运算符，组合两个子表达式）
public class AddExpression implements Expression {
    private final Expression left;    // 左子表达式
    private final Expression right;   // 右子表达式

    public AddExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() + right.interpret();  // 组合解释
    }
}

public class MultiplyExpression implements Expression {
    private final Expression left;
    private final Expression right;

    public MultiplyExpression(Expression left, Expression right) {
        this.left = left;
        this.right = right;
    }

    @Override
    public int interpret() {
        return left.interpret() * right.interpret();
    }
}

// ④ 使用（手动构建语法树）
Expression expression = new AddExpression(
    new NumberExpression(3),
    new MultiplyExpression(
        new NumberExpression(5),
        new NumberExpression(2)
    )
);
System.out.println(expression.interpret());  // 13
```
核心思想
```
// ① 每个语法规则一个类（终结符/非终结符）
// ② 表达式构建成"树"（组合）
// ③ 解释 = 递归调用 interpret()
// ④ 加语法规则 = 加类（开闭 ✅）
```
### 结合规则引擎
```
// 规则引擎（如 Drools）内部就用了解释器思想

// 场景：判题平台的"判题规则"
// 规则："分数 >= 60 AND 无编译错误"
// 解析成规则树 → 解释执行 → 判断通过

// 简化实现：
// "score > 60 && errors == 0"
// 表达式树：
//       &&
//      /  \
//   >60   ==0
//   /      \
// score   errors
```
应用场景
```
// ① 规则引擎（业务规则解释执行）
// ② SQL 解析器（语句 → 语法树 → 执行）
// ③ 正则表达式（模式 → 匹配）
// ④ 模板解析（占位符替换）
// ⑤ 配置文件解析
```
### 优缺点
优点
```
// ① 语法可扩展（加规则 = 加类）
// ② 符合开闭原则
// ③ 易于实现简单语言
// ④ 表达式树可复用/可遍历
```
缺点
```
// ① 类数量爆炸（规则多时）
// ② 解释效率低（递归解释）
// ③ 语法复杂时难维护
// ④ 实际用现成引擎（Drools/ANTLR）更多
```
### 面试高频题
#### 题目1：解释器模式是什么
```
// 定义语法规则，解释执行
// 表达式树 + 递归解释
```
#### 题目2：核心结构
```
// 抽象表达式（interpret）
// 终结符（数字）+ 非终结符（运算符组合）
```
#### 题目3：应用场景
```
// 规则引擎、SQL 解析、正则、模板解析
```