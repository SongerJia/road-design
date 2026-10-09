## 备忘录模式
备忘录是什么->核心结构->编辑器撤销->undo log关联->应用场景->高频面试题
### 备忘录模式是什么
备忘录模式：在不破坏封装的前提下，报错对象的内部状态快照。需要时可恢复到之前的状态。
```
// 类比：游戏存档
// 打游戏 → 存档（保存状态快照）
// 挂了 → 读档（恢复到存档时的状态）

// 类比：Ctrl+Z
// 编辑文档 → 每一步自动存档
// 撤销 → 恢复到上一步
```
解决什么问题
```
// ❌ 直接暴露状态（破坏封装）：
public class Editor {
    public String text;   // 外部直接改/存 → 封装被破坏
}
// 要保存状态 → 要么暴露字段，要么复制整个对象

// ✅ 备忘录：快照对象封装状态，不暴露内部实现
```
### 核心结构
三大角色
```
// ① 备忘录（Memento）：状态快照（不可变）
public class Memento {
    private final String state;   // 快照内容（final 不可变）

    public Memento(String state) {
        this.state = state;
    }

    // 只有发起者能读取状态
    String getState() {
        return state;
    }
}

// ② 发起者（Originator）：需要被保存/恢复的对象
public class Editor {
    private String text;      // 内部状态

    public void write(String text) {
        this.text = text;
    }

    public String getText() {
        return text;
    }

    // 保存快照（创建一个备忘录）
    public Memento save() {
        return new Memento(text);
    }

    // 恢复快照（从备忘录取状态）
    public void restore(Memento memento) {
        this.text = memento.getState();
    }
}

// ③ 管理者（Caretaker）：管理备忘录（存/取，不修改内容）
public class History {
    private final Deque<Memento> stack = new ArrayDeque<>();

    public void push(Memento memento) {
        stack.push(memento);
    }

    public Memento pop() {
        return stack.pop();
    }
}

// 使用
Editor editor = new Editor();
History history = new History();

editor.write("第一版");
history.push(editor.save());    // 存档

editor.write("第二版");
history.push(editor.save());    // 存档

editor.write("第三版");

editor.restore(history.pop());  // 撤销 → 第二版
System.out.println(editor.getText());  // "第二版"

editor.restore(history.pop());  // 撤销 → 第一版
System.out.println(editor.getText());  // "第一版"
```
核心思想
```
// ① 备忘录封装状态快照（不可变，保护封装）
// ② 发起者自己保存/恢复（知道怎么还原）
// ③ 管理者只管"存/取"（不改内容）
// ④ 只有发起者能读取备忘录状态（封装边界）
```
### 和命令模式配合
```
// 命令模式 + 备忘录模式 = 完美撤销
// 命令记录"操作"，备忘录记录"操作前状态"

public class EditCommand implements Command {
    private final Editor editor;
    private Memento backup;     // 操作前快照（备忘录）

    @Override
    public void execute() {
        backup = editor.save();  // 先存档
        editor.write("新内容");
    }

    @Override
    public void undo() {
        editor.restore(backup);  // 恢复快照（备忘录）
    }
}

// 使用
CommandManager manager = new CommandManager();
manager.execute(new EditCommand(editor));  // 编辑
manager.undo();   // 撤销（恢复备忘录快照）
manager.redo();   // 重做
```
### 和数据库undo log关联
```
// 数据库事务的回滚 = 备忘录思想！

// ① undo log：
// 每次修改前，把"旧值"记录到 undo log（快照）
// 回滚时：根据 undo log 恢复旧值

// 类比：
// undo log = 备忘录集合
// 事务 = 管理者
// 数据行 = 发起者

// ② 快照恢复（备份恢复）：
// MySQL 备份 = 全量快照
// binlog 重放 = 恢复到某时间点

// ③ Redis RDB/AOF：
// RDB = 快照（备忘录）
// 重启 = 恢复快照
```
其他应用
```
// ① 编辑器撤销（Ctrl+Z / Ctrl+Y）
// ② 游戏存档/读档
// ③ 数据库事务回滚（undo log）
// ④ 表单重置（保存初始值，恢复）
// ⑤ 配置回滚（版本快照）
```
### 实战应用
```
// 判题配置的"试运行 → 回滚"
public class JudgeConfig {
    private int timeLimit;
    private int memoryLimit;

    public Memento save() {
        return new Memento(timeLimit, memoryLimit);
    }

    public void restore(Memento memento) {
        this.timeLimit = memento.getTimeLimit();
        this.memoryLimit = memento.getMemoryLimit();
    }
}

// 试配置
JudgeConfig config = new JudgeConfig();
Memento backup = config.save();      // 备份当前配置
config.setTimeLimit(5000);           // 试新配置
config.setMemoryLimit(512);
// 压测效果不好
config.restore(backup);              // 回滚旧配置
```
### 面试高频题
#### 题目1：备忘录模式是什么
```
// 保存状态快照，支持回滚
// 不破坏封装
```
#### 题目2：三大角色
```
// 发起者（保存/恢复）、备忘录（快照）、管理者（存取）
```
#### 题目3：和命名模式关系
```
// 命令记录操作 + 备忘录记录状态
// 配合实现撤销
```
#### 题目4：和undo log关系
```
// undo log 就是备忘录思想
// 修改前记旧值，回滚恢复
```
#### 题目5：备忘录模式优点
```
// ① 不破坏封装（快照对象隐藏状态）
// ② 状态可恢复（回滚）
// ③ 职责分离（发起者/备忘录/管理者）
```
#### 题目6：备忘录模式缺点
```
// ① 快照占用内存（状态多时大）
// ② 频繁存档性能开销
// ③ 只适合状态可完整保存的场景
```

