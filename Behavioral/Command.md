## 命令模式
命令是什么->核心结构->撤销重做->Runnable关联->实战应用->面试高频题
### 命令模式是什么
命令模式：把操作或请求封装为命令对象。请求的发送者和执行者解耦，命令可以排队、记录、撤销。
```
// 类比：点餐
// 顾客（发送者）→ 服务员收单（命令）→ 厨师执行（接收者）
// 顾客不用认识厨师（解耦）
// 菜单可以排队、可以退单（撤销）
```
解决什么问题
```
// ❌ 直接调用（耦合）：
public class Editor {
    public void copy() { ... }
    public void paste() { ... }
    // 每加一个操作 → 界面代码要改
}

// ✅ 命令封装：
// 按钮 → 命令对象（封装操作）→ 编辑器
// 加操作 = 加命令类（不改界面）
```
### 核心结构
四大角色
```
// ① 命令接口
public interface Command {
    void execute();       // 执行
    void undo();          // 撤销（可选）
}

// ② 接收者（执行实际操作的对象）
public class Editor {
    private String text;

    public void copy() {
        System.out.println("复制文本");
    }
    public void paste() {
        System.out.println("粘贴文本");
    }
}

// ③ 具体命令（封装"复制"操作）
public class CopyCommand implements Command {
    private final Editor editor;   // 持接收者

    public CopyCommand(Editor editor) {
        this.editor = editor;
    }

    @Override
    public void execute() {
        editor.copy();   // 转发给接收者
    }

    @Override
    public void undo() {
        // 复制没有 undo（可选实现）
    }
}

// ④ 调用者（持有命令，触发执行）
public class Button {
    private Command command;

    public void setCommand(Command command) {
        this.command = command;
    }

    public void onClick() {
        command.execute();  // 触发命令（不用知道具体操作）
    }
}

// 使用
Editor editor = new Editor();
Button button = new Button();
button.setCommand(new CopyCommand(editor));  // 绑定命令
button.onClick();   // 执行复制
```
核心思想
```
// ① 操作封装成对象（命令）
// ② 调用者只依赖"命令接口"（不依赖具体操作）
// ③ 命令持"接收者"（转发执行）
// ④ 加操作 = 加命令类（开闭原则 ✅）
```
### 撤销与重做
```
// 命令模式最大的价值：支持撤销/重做

// ① 命令带 undo
public class PasteCommand implements Command {
    private final Editor editor;
    private String backupText;   // 记录操作前状态

    public PasteCommand(Editor editor) {
        this.editor = editor;
    }

    @Override
    public void execute() {
        backupText = editor.getText();  // 先备份
        editor.paste();
    }

    @Override
    public void undo() {
        editor.setText(backupText);     // 恢复备份
    }
}

// ② 撤销栈
public class CommandManager {
    private final Deque<Command> undoStack = new ArrayDeque<>();
    private final Deque<Command> redoStack = new ArrayDeque<>();

    public void execute(Command command) {
        command.execute();
        undoStack.push(command);   // 入撤销栈
        redoStack.clear();         // 新命令清空重做栈
    }

    public void undo() {
        Command command = undoStack.pop();
        command.undo();
        redoStack.push(command);   // 入重做栈
    }

    public void redo() {
        Command command = redoStack.pop();
        command.execute();
        undoStack.push(command);
    }
}

// 使用
CommandManager manager = new CommandManager();
manager.execute(new PasteCommand(editor));  // 粘贴
manager.undo();   // Ctrl+Z 撤销
manager.redo();   // Ctrl+Y 重做
```
为什么能撤销
```
// ① 命令记录了"操作前状态"（备份）
// ② undo() 恢复备份
// ③ 撤销栈记录执行顺序
// ④ 类似"操作日志 + 快照"的组合
```
### 与Java关联
Runnable/Callable就是命令
```
// Runnable 是"无返回值命令"
public interface Runnable {
    void run();   // 类似 execute()
}

// 应用：
// ① 线程池任务（execute/submit）
// ② 定时任务（TimerTask）
// ③ CompletableFuture 任务

// 线程池把 Runnable（命令）排队执行：
executor.execute(() -> process());  // 命令对象
// 就是命令模式：任务封装 → 队列 → 执行
```
事务/日志应用
```
// ① 数据库事务：每个 SQL 是命令，支持回滚（undo）
// ② 操作日志：记录命令序列，可重放
// ③ 消息队列：消息体就是命令（发给执行者）
```
### 实战应用
```
// 判题任务 = 命令对象
public class JudgeCommand implements Runnable {
    private final Submission submission;
    private final JudgeService judgeService;

    public JudgeCommand(Submission submission, JudgeService judgeService) {
        this.submission = submission;
        this.judgeService = judgeService;
    }

    @Override
    public void run() {   // 命令执行
        judgeService.judge(submission);
    }
}

// 提交到线程池（命令排队）
executor.execute(new JudgeCommand(submission, judgeService));
// 线程池就是"调用者"：接收命令 → 排队 → 执行

// 好处：
// ① 任务和提交解耦
// ② 排队控制并发
// ③ 可以记录、统计任务（命令可查询）
```
### 高频面试题
#### 题目1：命令模式是什么
```
// 操作封装成对象
// 发送者与执行者解耦
```
#### 题目2：怎么实现撤销
```
// 命令记录操作前状态（备份）
// undo() 恢复 + 撤销栈
```
#### 题目3：Runnable是命令吗
```
// 是！Runnable 就是无返回值命令
// 线程池接收 Runnable 排队执行
```
#### 题目4：和策略模式区别
```
// 策略：算法选择（行为不同）
// 命令：操作封装（行为执行，可撤销）
// 策略关注"怎么做"，命令关注"把做封装"
```
#### 题目5：命令的好处
```
// 解耦（发送者不认识执行者）
// 可排队（命令对象可存储）
// 可撤销（undo）
// 可记录（日志/重放）
```
#### 题目6：命令的缺点
```
// 类数量膨胀（一操作一命令）
// 命令和执行者耦合在命令里
```
