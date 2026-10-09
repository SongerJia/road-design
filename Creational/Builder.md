## 建造者模式
建造者是什么->核心结构->与工厂对比->Lombok @Builder->实际应用->面试高频题
### 建造者是什么
建造者模式：把复杂对象的构造过程和表示分离。通过一步步设置参数，最后build产品对象。
适用场景
```
// ① 参数多（5+ 个）
// ② 部分参数可选
// ③ 对象不可变（final 字段）
// ④ 构造器太长/太乱
```
对比直接构建
```
// ❌ 长构造器（分不清参数）
User user = new User("张三", 25, "北京", "13800000000", "男", "Java", true, false);

// ✅ Builder 链式（清晰）
User user = User.builder()
    .name("张三")
    .age(25)
    .city("北京")
    .build();
```
### 核心结构
```
public class User {
    // ① 字段 final（不可变）
    private final String name;
    private final Integer age;
    private final String city;

    // ② 私有构造器（只能 Builder 创建）
    private User(UserBuilder builder) {
        this.name = builder.name;
        this.age = builder.age;
        this.city = builder.city;
    }

    // ③ 静态 Builder
    public static UserBuilder builder() {
        return new UserBuilder();
    }

    // ④ Builder 类
    public static class UserBuilder {
        // 和 User 相同的字段（可变）
        private String name;
        private Integer age;
        private String city;

        // 链式方法（返回 this）
        public UserBuilder name(String name) {
            this.name = name;
            return this;
        }
        public UserBuilder age(Integer age) {
            this.age = age;
            return this;
        }
        public UserBuilder city(String city) {
            this.city = city;
            return this;
        }

        // ⑤ build() 校验 + 创建
        public User build() {
            // 可选校验（必填项检查）
            if (name == null) {
                throw new IllegalStateException("name 必填");
            }
            return new User(this);
        }
    }
}

// 使用
User user = User.builder()
    .name("张三")
    .age(25)
    .city("北京")
    .build();
```
为什么字段不可变
```
// ① 线程安全（不可变对象天然安全）
// ② 防止中途修改
// ③ 和 setter 式（可变）对比：Builder 产出不可变对象
```
### 与工厂模式对比
| 对比      | 工厂模式       | 建造者模式        |
| ------- | ---------- | ------------ |
| **关注点** | 创建哪个产品     | 怎么一步步构建      |
| **参数**  | 简单（一个参数决定） | 复杂（多个参数）     |
| **步骤**  | 一步创建       | 多步设置 + build |
| **链式**  | 无          | ✅            |
| **适合**  | 产品类型选择     | 复杂对象组装       |
### Lombok @Builder
```
// Lombok 一行生成 Builder
@Data
@Builder          // 生成 Builder
public class User {
    private String name;
    private Integer age;
    private String city;
}

// 使用
User user = User.builder()
    .name("张三")
    .age(25)
    .city("北京")
    .build();
```
做了什么
```
// 自动生成：
// ① 静态 builder() 方法
// ② UserBuilder 内部类
// ③ 链式方法（name/age/city）
// ④ build() 方法

// 注意：@Builder 生成的是"全参构造器"路径
// 配合 @NoArgsConstructor/@AllArgsConstructor 使用，当自定义了构造方法时，必须加全参构造器
```
Spring中的Builder
```
// ① BeanDefinitionBuilder（Bean 定义）
// ② UriComponentsBuilder（URL 构建）
// ③ HttpHeaders / RequestBuilder
// ④ RestTemplate 相关

// 应用：组装请求参数
```
### 实战应用
```
// 判题任务的复杂构建
@Builder
public class JudgeTask {
    private Long taskId;
    private String language;      // Java/Python/...
    private String code;          // 提交代码
    private List<String> testCases;  // 测试用例
    private Integer timeLimit;    // 时间限制
    private Integer memoryLimit;  // 内存限制
    private Boolean isSpecial;    // 是否特判
}

// 使用
JudgeTask task = JudgeTask.builder()
    .taskId(1001L)
    .language("java")
    .code(sourceCode)
    .testCases(cases)
    .timeLimit(2000)
    .memoryLimit(256)
    .build();
// 参数清晰，不用记构造器顺序
```
### 面试高频题
#### 题目1：建造者模式是什么
```
// 链式设置参数 + build 产出
// 适合参数多的不可变对象
```
#### 题目2：和工厂的区别
```
// 工厂：选类型（一步）
// Builder：组装参数（多步）
```
#### 题目3：为什么字段final
```
// 不可变（线程安全、防止修改）
```
#### 题目4：Lombok @Builder的作用
```
// 自动生成 Builder
// 一行搞定
```
#### 题目5：build里能做什么
```
// 参数校验、默认值、构造对象
```