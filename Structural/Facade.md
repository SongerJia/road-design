## 外观模式
外观是什么->核心结构->Service关联->实战应用->与中介者对比->面试高频题
### 外观模式是什么
外观模式：为复杂的子系统提供一个统一的门面接口，客户端只面对门面，不用接触子系统内部的复杂组件。
```
// 类比：前台接待
// 客户（客户端）→ 前台（外观）→ 各部门（子系统）
// 客户不用知道找哪个部门、走什么流程
// 前台统一处理（门面）

// 类比：遥控器
// 遥控器（外观）→ 电视/音响/空调（子系统）
// 一键操作，不用分别操作每个设备
```
解决什么问题
```
// ❌ 客户端直接操作所有子系统（复杂耦合）：
public void createOrder(Order order) {
    // 客户端要知道所有内部组件
    orderDao.insert(order);           // 数据层
    inventoryService.deduct(...);     // 库存
    messageService.send(...);         // 通知
    logService.save(...);             // 日志
    statsService.update(...);         // 统计
    // 客户端耦合所有子系统！改内部 → 客户端全改
}

// ✅ 外观：客户端只调一个方法
orderFacade.createOrder(order);
// 内部怎么协作由外观处理（客户端无感）
```
### 核心结构
```
// ① 子系统（复杂组件，客户端不需要知道）
public class OrderDao {
    public void insert(Order order) { ... }
}
public class InventoryService {
    public void deduct(Order order) { ... }
}
public class MessageService {
    public void send(Order order) { ... }
}

// ② 外观类（统一门面）
public class OrderFacade {
    private final OrderDao orderDao = new OrderDao();
    private final InventoryService inventoryService = new InventoryService();
    private final MessageService messageService = new MessageService();

    // 统一入口：客户端只调这个
    public void createOrder(Order order) {
        // 内部编排所有子系统（外观内部处理）
        orderDao.insert(order);            // 1. 存数据
        inventoryService.deduct(order);    // 2. 扣库存
        messageService.send(order);        // 3. 发通知
    }
}

// ③ 客户端（只依赖外观）
public class OrderController {
    private final OrderFacade orderFacade;   // 只注入外观

    public void create(Order order) {
        orderFacade.createOrder(order);   // 一个方法搞定
    }
}

// 好处：
// ① 客户端不耦合子系统
// ② 子系统内部随便改（客户端无感）
// ③ 客户端代码简洁
```
核心思想
```
// ① 外观封装子系统（一个门面）
// ② 客户端只依赖外观（解耦）
// ③ 子系统内部自由演进（不影响客户端）
// ④ 客户端调用简化（一个方法完成复杂流程）
```
### 和service层的关联
```
// 实际开发中的"Service 层"就是外观模式！

// Controller（客户端）只调 Service（外观）
// Service 内部编排 Mapper/Dao/其他服务（子系统）

@RestController
public class OrderController {
    @Autowired
    private OrderService orderService;   // 只依赖外观（Service）

    @PostMapping("/order")
    public Result create(@RequestBody Order order) {
        return orderService.createOrder(order);  // 一个方法
    }
}

@Service
public class OrderService {   // 外观（门面）
    @Autowired
    private OrderMapper orderMapper;          // 子系统
    @Autowired
    private StockService stockService;        // 子系统
    @Autowired
    private MessageService messageService;    // 子系统

    @Transactional
    public Result createOrder(Order order) {  // 统一编排
        orderMapper.insert(order);
        stockService.deduct(order.getSkuId(), order.getQty());
        messageService.sendNotice(order);
        return Result.success();
    }
}

// 就是外观模式：
// Controller 不碰 Mapper/Stock/Message
// 所有子系统编排在 Service（外观）里
```
### 实战应用
```
// 判题外观：提交一次判题的完整流程
@Service
public class JudgeFacade {
    @Autowired
    private SubmissionService submissionService;  // 子系统
    @Autowired
    private JudgeQueueService queueService;        // 子系统
    @Autowired
    private ResultService resultService;           // 子系统

    // 统一入口（客户端只调这个）
    public JudgeResult submitAndJudge(Submission submission) {
        // 内部编排（外观处理）
        Long taskId = submissionService.save(submission);  // 保存
        queueService.enqueue(taskId);                      // 入队
        JudgeResult result = judge(taskId);                // 判题
        resultService.save(result);                        // 保存结果
        return result;
    }
}
// 客户端（Controller）只依赖 JudgeFacade
// 内部换组件（如换判题引擎）客户端无感
```
### 与中介者对比
| 对比     | 外观模式       | 中介者模式      |
| ------ | ---------- | ---------- |
| **方向** | 对外简化（门面）   | 对内协调（枢纽）   |
| **调用** | 单向（客户端→外观） | 双向（对象↔中介者） |
| **目的** | 隐藏复杂度      | 降低对象间耦合    |
| **类比** | 前台接待       | 聊天室        |
| **关系** | 简化入口       | 协调交互       |
外观 vs 单例
```
// 外观类通常做成单例（全局一个门面）
// Spring 的 Service 默认单例 ✅
```
### 面试高频题
#### 题目1：外观模式是什么
```
// 统一门面封装子系统
// 客户端只面对外观
```
#### 题目2：和中介者区别
```
// 外观：对外简化（单向）
// 中介者：对内协调（双向）
```
#### 题目3：Service层是外观
```
// 是！Service 编排 Mapper/其他服务
// Controller 只依赖 Service（门面）
```
#### 题目4：外观模式的好处
```
// ① 客户端解耦（不碰子系统）
// ② 子系统内部自由改
// ③ 调用简化
```
#### 题目5：外观模式的缺点
```
// ① 外观可能很臃肿（编排逻辑多）
// ② 过度使用 → 子系统不可直接访问（限制）
```
#### 题目6：外观模式什么时候用
```
// ① 系统复杂、组件多
// ② 客户端需要简单入口
// ③ 分层架构（Controller→Service）
```