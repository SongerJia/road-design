## 享元模式
享元是什么->内部/外部状态->核心结构->池化缓存关联->实战应用->面试高频题
### 享元模式是什么
享元模式：大量相似对象共享同一个实例，通过复用减少对象数量，节省内存。核心是区分内部状态和外部状态。
```
// 类比：图书馆借书
// 书（享元对象）：一本书只有一份，大家共享
// 借书人（外部状态）：谁借的、借多久（不共享）

// 类比：共享单车
// 单车（享元对象）：复用同一批车
// 使用者（外部状态）：谁在用（独立）
```
解决什么问题
```
// ❌ 大量重复对象（内存爆炸）：
public class Char {
    private final char c;
    public Char(char c) { this.c = c; }
}

// 渲染一篇文章（1 万个 'a'）
for (int i = 0; i < 10000; i++) {
    list.add(new Char('a'));  // 1 万个相同对象！
}
// 'a' 只应该有 1 个，却创建了 1 万个

// ✅ 享元：共享 'a' 这一个对象
Char a = factory.get('a');  // 每次都返回同一个
```
### 内部状态 vs 外部状态
```
// 内部状态（共享）：对象固有的、不会变的
// 例：字符 'a'、颜色 红色

// 外部状态（独立）：使用场景相关的、会变的
// 例：位置 x,y、字体大小

// 设计：
// 内部状态放对象里（共享）
// 外部状态由客户端传入（不共享）

// 例：字符渲染
public class CharFlyweight {
    private final char c;      // 内部状态（共享：字符本身）

    public CharFlyweight(char c) {
        this.c = c;
    }

    // 外部状态作为参数传入（不存对象里）
    public void render(int x, int y, int size) {
        System.out.println("字符 '" + c + "' 在 (" + x + "," + y + ") 大小 " + size);
    }
}
// 'a' 对象只有 1 个，位置/大小由调用时传入（外部）
```
### 核心结构
```
// ① 享元工厂（管理共享对象）
public class CharFactory {
    // 缓存池（享元池）
    private static final Map<Character, CharFlyweight> POOL = new HashMap<>();

    // 获取享元对象（没有则创建，有则复用）
    public static CharFlyweight get(char c) {
        // 从池中取（共享）
        return POOL.computeIfAbsent(c, CharFlyweight::new);
    }

    public static int poolSize() {
        return POOL.size();
    }
}

// ② 享元对象（共享）
public class CharFlyweight {
    private final char c;   // 内部状态（不可变）

    public CharFlyweight(char c) {
        this.c = c;
    }

    public void render(int x, int y) {  // 外部状态传入
        System.out.println(c + " at (" + x + "," + y + ")");
    }
}

// ③ 使用（大量渲染，只有少量对象）
for (int i = 0; i < 10000; i++) {
    // 反复获取同一个对象（不创建新的）
    CharFlyweight a = CharFactory.get('a');
    a.render(i % 10, i / 10);   // 外部状态传入
}
// 只创建了 1 个 'a' 对象，渲染了 1 万次 ✅
// POOL 里只有用到的字符数（26 个字母 ≈ 26 个对象）
```
核心思想
```
// ① 内部状态共享（对象复用）
// ② 外部状态传入（不存对象）
// ③ 工厂管理池（有则复用，无则创建）
// ④ 内存节省：n 个使用 → 1 个对象
```
### 和Java的关联
①字符串常量池
```
// String 就是享元模式
// "abc" 字面量只创建一次，复用同一对象

String s1 = "abc";
String s2 = "abc";
System.out.println(s1 == s2);  // true！同一个对象（共享）
// 常量池 = 享元池

// 但 new String("abc") 不同（堆新对象）：
String s3 = new String("abc");
System.out.println(s1 == s3);  // false
```
②Integer缓存池
```
// Integer.valueOf 享元缓存（-128~127）
Integer i1 = 100;
Integer i2 = 100;
System.out.println(i1 == i2);  // true（缓存共享）

Integer i3 = 200;
Integer i4 = 200;
System.out.println(i3 == i4);  // false（超出缓存）
```
③线程池/连接池（池化思想）
```
// 线程池：复用线程（不频繁创建）
// 连接池：复用数据库连接
// 对象池：复用对象
// 都是享元思想（复用减少开销）

// 区别：
// 享元：对象共享（同一实例）
// 池：对象复用（借出还回，可多个不同实例）
// 但本质都是"复用省资源"
```
### 实战应用
```
// 判题"规则模板"共享（大量题目共用模板）
public class RuleTemplate {
    // 内部状态（共享）：规则模板本身
    private final String language;
    private final String checkScript;

    public RuleTemplate(String language, String checkScript) {
        this.language = language;
        this.checkScript = checkScript;
    }

    // 外部状态（调用传入）：具体题目
    public JudgeResult apply(Long questionId, String code) {
        // 用模板 + 题目代码执行
        return execute(checkScript, code);
    }
}

// 工厂（模板池）
public class TemplateFactory {
    private static final Map<String, RuleTemplate> POOL = new HashMap<>();

    public static RuleTemplate get(String language) {
        // 有则复用（同语言共用模板）
        return POOL.computeIfAbsent(language, lang -> loadTemplate(lang));
    }
}

// 使用：1 万个 Java 题共用 1 个 Java 模板
RuleTemplate javaTemplate = TemplateFactory.get("java");
javaTemplate.apply(1001L, userCode1);
javaTemplate.apply(1002L, userCode2);
// 只创建 1 个 Java 模板（共享），不用每道题一个
```
### 面试高频题
#### 题目1：享元模式是什么
```
// 共享复用对象，节省内存
// 内部状态共享 + 外部状态传入
```
#### 题目2：内部/外部状态
```
// 内部：固有不变（共享）
// 外部：场景相关（传入）
```
#### 题目3：Java哪里用了
```
// 字符串常量池、Integer 缓存池、线程池
```
#### 题目4：和对象池区别
```
// 享元：同一实例共享
// 池：借出还回（可不同实例）
// 本质都是复用
```
#### 题目5：享元模式的好处
```
// ① 大幅节省内存（n→1）
// ② 减少对象创建（性能）
```
#### 题目6：享元模式需要注意什么
```
// ① 内部状态必须不可变（共享安全）
// ② 外部状态不能放对象里（否则共享混乱）
// ③ 池的并发安全（HashMap → ConcurrentHashMap）
```
