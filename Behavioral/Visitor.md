## 访问者模式
访问者是什么->核心结构->双分派->应用场景->面试高频题
### 访问者模式是什么
访问者模式：在不修改数据结构的情况下，增加新的操作。把操作从数据结构中分离出来，通过访问者对象执行。
```
// 类比：体检（访问者）
// 医院里有各种检查设备（操作）
// 每个科室（访问者）对不同的人做不同检查
// 病人结构不变（人还是那个人），检查项目可以不断新增
```
解决什么问题
```
// ❌ 数据结构里塞操作（每加操作改结构）：
public class Order {
    public void exportExcel() { ... }    // 操作 1
    public void exportPdf() { ... }      // 操作 2
    public void sendToLog() { ... }      // 操作 3
    // 每加一个操作 → 改 Order 类（违反开闭）
}

// ✅ 访问者：操作放访问者里（结构不改）
Order order = new Order();
ExcelExporter visitor = new ExcelExporter();  // 新增操作
order.accept(visitor);  // 结构不变，操作可扩展
```
### 核心结构
```
// ① 元素接口（数据结构）
public interface Element {
    void accept(Visitor visitor);  // 接受访问者
}

// ② 具体元素（结构稳定）
public class Order implements Element {
    private Long id;
    private Double amount;

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);  // 把自己交给访问者
    }

    public Long getId() { return id; }
    public Double getAmount() { return amount; }
}

public class User implements Element {
    private String name;

    @Override
    public void accept(Visitor visitor) {
        visitor.visit(this);
    }
    // ...
}

// ③ 访问者接口（操作）
public interface Visitor {
    // 每种元素对应一个 visit 方法（重载）
    void visit(Order order);
    void visit(User user);
}

// ④ 具体访问者（一种操作）
public class ExcelExporter implements Visitor {
    @Override
    public void visit(Order order) {
        System.out.println("导出订单到 Excel：" + order.getId());
    }

    @Override
    public void visit(User user) {
        System.out.println("导出用户到 Excel：" + user.getName());
    }
}

// ⑤ 新增操作（不改任何元素！）
public class PdfExporter implements Visitor {
    @Override
    public void visit(Order order) {
        System.out.println("导出订单 PDF");
    }
    @Override
    public void visit(User user) {
        System.out.println("导出用户 PDF");
    }
}

// 使用
List<Element> elements = List.of(new Order(1L, 100.0), new User("张三"));
// 操作 1：Excel 导出
for (Element e : elements) e.accept(new ExcelExporter());
// 操作 2：PDF 导出（新增，不用改 Order/User）
for (Element e : elements) e.accept(new PdfExporter());
```
### 双分派
什么是双分派
```
// 普通调用：单分派（按对象类型决定方法）
order.accept(visitor);
// ① 第一次分派：按"元素类型"调用 accept（Order/User 各自 accept）
// ② 第二次分派：visit 重载按"具体元素类型"选择方法
//    visitor.visit(this) 中 this 是 Order → visit(Order)
//    this 是 User → visit(User)

// 效果：类型信息传递到了访问者（不用 instanceof 判断）
```
对比 if-else
```
// ❌ 访问者里用 instanceof 判断（不优雅）
public void export(Object element) {
    if (element instanceof Order) {
        // 导出订单
    } else if (element instanceof User) {
        // 导出用户
    }
}

// ✅ 双分派：visit 重载自动分发
visitor.visit(this);  // 编译器按 this 类型选方法
```
### 应用场景
①编译器AST遍历
```
// 语法树节点（元素）：表达式、语句、声明...
// 操作（访问者）：类型检查、代码生成、优化
// 结构稳定（语法固定），操作频繁扩展（优化器迭代）

// Java ASTVisitor 就是访问者模式
```
②报表/统计
```
// 数据对象（元素）：订单、用户、商品...
// 操作（访问者）：求和、分组、导出
// 新增统计口径 → 加访问者
```
③文件系统
```
// 文件/目录（元素）
// 操作（访问者）：计算大小、压缩、扫描病毒
// 结构稳定，操作扩展
```
④Spring中类似思想
```
// BeanDefinitionVisitor（处理 Bean 定义）
// 不是完全访问者，但思路类似（遍历对象做操作）
```
### 面试高频题
#### 题目1：访问者模式是什么
```
// 不改数据结构，增加操作
// 操作封装成访问者
```
#### 题目2：核心机制
```
// 双分派：元素 accept → 访问者 visit 重载
// 类型信息自动分发
```
#### 题目3：和迭代器区别
```
// 迭代器：遍历元素（取数据）
// 访问者：对每个元素做操作（处理数据）
```
#### 题目4：什么时候用
```
// ① 数据结构稳定
// ② 操作频繁扩展
// ③ 需要对多种类型做不同操作
```
#### 题目5：访问者模式优点
```
// ① 符合开闭（加操作不改结构）
// ② 操作集中（一个访问者一处逻辑）
// ③ 双分派类型安全
```
#### 题目6：访问者模式缺点
```
// ① 元素新增要改所有访问者（visitor 加方法）
// ② 元素要暴露 accept（轻微侵入）
// ③ 只适合结构稳定的场景
```
