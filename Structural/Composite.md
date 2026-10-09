## 组合模式
组合是什么->核心结构->菜单树应用->实战应用->与迭代器配合->面试高频题
### 组合模式是什么
组合模式：把对象组合成树形结构，让客户端对单个对象和组合对象的使用保持一致。
```
// 类比：公司组织架构
// 公司（节点）
// ├── 研发部（节点）
// │   ├── 张三（叶子）
// │   └── 李四（叶子）
// └── 市场部（节点）
//     └── 王五（叶子）

// 无论"部门"还是"员工"，都能统一操作（统计人数、发通知）
```
解决什么问题
```
// ❌ 区分叶子/节点处理（客户端要判断）：
public void count(Dept dept) {
    if (dept.isLeaf()) {  // 客户端要区分！
        count++;
    } else {
        for (Dept child : dept.getChildren()) {
            count(child);  // 递归
        }
    }
}

// ✅ 组合模式：叶子/节点统一接口
// 客户端不用区分，直接调用（树自己处理递归）
```
### 核心结构
```
// ① 组件接口（叶子/节点统一）
public interface Component {
    void show();            // 统一操作
    double getPrice();      // 统一属性
}

// ② 叶子（没有子节点）
public class Leaf implements Component {
    private final String name;
    private final double price;

    public Leaf(String name, double price) {
        this.name = name;
        this.price = price;
    }

    @Override
    public void show() {
        System.out.println("商品：" + name + "，价格：" + price);
    }

    @Override
    public double getPrice() {
        return price;
    }
}

// ③ 组合节点（有子节点）
public class Composite implements Component {
    private final String name;
    private final List<Component> children = new ArrayList<>();

    public Composite(String name) {
        this.name = name;
    }

    public void add(Component component) {
        children.add(component);   // 添加子节点/叶子
    }

    public void remove(Component component) {
        children.remove(component);
    }

    @Override
    public void show() {
        System.out.println("组合：" + name);
        for (Component child : children) {
            child.show();   // 递归展示（统一处理）
        }
    }

    @Override
    public double getPrice() {
        // 组合的价格 = 所有子项之和（递归）
        return children.stream().mapToDouble(Component::getPrice).sum();
    }
}

// ④ 使用（客户端不用区分叶子/节点）
Component box = new Composite("礼盒");           // 节点
box.add(new Leaf("苹果", 5.0));                  // 叶子
Component inner = new Composite("内盒");          // 嵌套节点
inner.add(new Leaf("糖果", 3.0));
inner.add(new Leaf("贺卡", 2.0));
box.add(inner);                                   // 节点加节点

box.show();       // 统一展示（树自己递归）
box.getPrice();   // 10.0（统一汇总）
// 客户端调同一方法，不用管是叶子还是组合
```
核心思想
```
// ① 叶子/节点实现同一接口（统一处理）
// ② 节点持有子节点列表（树结构）
// ③ 节点方法递归调用子节点
// ④ 客户端不用区分（一致对待）
```
### 菜单树应用
```
// 菜单树是组合模式经典应用
public interface MenuItem {
    void display();
}

// 叶子：具体菜单项
public class MenuLeaf implements MenuItem {
    private final String name;
    private final String url;

    public MenuLeaf(String name, String url) {
        this.name = name;
        this.url = url;
    }

    @Override
    public void display() {
        System.out.println("菜单项：" + name + " → " + url);
    }
}

// 节点：菜单分组
public class MenuGroup implements MenuItem {
    private final String name;
    private final List<MenuItem> items = new ArrayList<>();

    public MenuGroup(String name) {
        this.name = name;
    }

    public void add(MenuItem item) {
        items.add(item);
    }

    @Override
    public void display() {
        System.out.println("菜单组：" + name);
        for (MenuItem item : items) {
            item.display();   // 递归
        }
    }
}

// 构建树
MenuGroup root = new MenuGroup("系统管理");
MenuGroup userMenu = new MenuGroup("用户管理");
userMenu.add(new MenuLeaf("用户列表", "/user/list"));
userMenu.add(new MenuLeaf("用户新增", "/user/add"));
root.add(userMenu);
root.add(new MenuLeaf("角色管理", "/role/list"));

root.display();  // 统一遍历整棵树
```
其他组合应用
```
// ① 文件系统（目录=节点，文件=叶子）
// ② 菜单树（分组=节点，菜单项=叶子）
// ③ 组织架构（部门=节点，员工=叶子）
// ④ 购物车（商品=叶子，套餐=节点）
// ⑤ HTML DOM（元素=节点，文本=叶子）
// ⑥ HashMap 内部（TreeNode 树）
```
### 实战应用
```
// 判题"套餐"（组合题 = 多个题目）
public interface QuestionComponent {
    double getScore();
}

// 叶子：单题
public class SingleQuestion implements QuestionComponent {
    private final String title;
    private final double score;

    public SingleQuestion(String title, double score) {
        this.title = title;
        this.score = score;
    }

    @Override
    public double getScore() {
        return score;
    }
}

// 节点：题组（多题组合）
public class QuestionGroup implements QuestionComponent {
    private final String name;
    private final List<QuestionComponent> questions = new ArrayList<>();

    public void add(QuestionComponent q) {
        questions.add(q);
    }

    @Override
    public double getScore() {
        // 组内总分 = 子题之和（递归）
        return questions.stream().mapToDouble(QuestionComponent::getScore).sum();
    }
}

// 使用：单题 + 题组统一计算总分
QuestionGroup exam = new QuestionGroup("考试");
exam.add(new SingleQuestion("选择题", 10.0));
QuestionGroup module = new QuestionGroup("编程题模块");
module.add(new SingleQuestion("Java 题", 30.0));
module.add(new SingleQuestion("算法题", 40.0));
exam.add(module);

double total = exam.getScore();  // 80.0（统一处理）
```
### 与迭代器配合
```
// 组合模式 + 迭代器：树形遍历统一
// 组合提供"树结构"，迭代器提供"遍历方式"

// 组合树的遍历：
// ① 深度优先（先子后兄弟）
// ② 层次遍历（按层）
// ③ 客户端可以用迭代器模式屏蔽遍历细节
```
优缺点
```
// 优点：
// ① 叶子/节点统一处理（客户端简单）
// ② 树结构天然适合（递归）
// ③ 新增类型容易（加叶子类）

// 缺点：
// ① 叶子/节点接口统一后，叶子也要有 add（空实现）
// ② 类型安全弱（编译期不区分叶子节点）
```
### 面试高频题
#### 题目1：组合模式是什么
```
// 树形结构，叶子/节点统一处理
// 客户端一致对待
```
#### 题目2：组合模式的核心是什么
```
// 统一接口 + 节点递归
// 叶子实现、节点持子列表递归
```
#### 题目3：组合模式的应用
```
// 菜单树、文件系统、组织架构、DOM
```
#### 题目4：组合模式和递归关系
```
// 组合的"遍历/汇总"天然用递归
// 节点方法调用子节点
```
