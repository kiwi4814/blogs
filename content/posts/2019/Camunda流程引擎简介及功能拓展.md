+++
title = "Camunda流程引擎简介及功能拓展"
date = 2019-08-25T20:35:47+08:00
slug = "/camunda"
tags = ["流程引擎","Camunda"]
categories = ["技术"]
+++


## 一、流程引擎以及BPMN2.0规范

---

### Camunda和BPMN

Camunda BPM （BPM，Business Process Manager，业务流程管理）是一个灵活的工作流和过程自动化框架，它的核心是一个在Java虚拟机内部运行的原生BPMN 2.0流程引擎，因此它可以嵌入到任何Java应用程序或运行时容器中。Camunda BPM与Java EE 6集成，并可以与Spring Framework完美匹配。 Camunda BPM附带了用于创建工作流和决策模型，在生产中操作已部署模型以及允许用户执行分配给他们的工作流任务的工具。

![流程引擎架构图（来源：官方文档）](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/architecture-overview.png)

流程引擎架构图（来源：官方文档）

**下面我们来看下维基百科对于BPMN的定义：**

> 业务流程模型和标记法（BPMN，Business Process Model and Notation）是对象管理组织（OMG，Object Management Group）维护的关于业务流程建模的行业性标准。它创建在与UML的活动图非常相似的流程图法（flowcharting）基础上，为“业务流程图”（BPD, Business Process Diagram）中的特定业务流程提供一套图形化标记法。BPMN的目标是，通过提供一套既匹配业务人员直观又能表现复杂流程语义的标记法，同时为技术人员和业务人员从事业务流程管理提供支持。BPMN规范还提供从标记法的图到执行语言基础构造的映射，尤其是业务流程执行语言（BPEL）。
> 

BPMN文件的底层数据格式是xml，定义了一些标签的标准含义和图形表示。一方面通过图形方便所有人理解流程，另一方面限制实现方必须按着流程的要求来实现。

Camunda BPM官方提供了用于建模BPMN工作流和DMN决策的桌面应用程序Camunda Modeler。

---

### [Camunda Modeler](https://camunda.com/products/modeler/)

下面为使用Camunda Moleler设计的一张只含有基本元素的流程定义：

![一个普通的流程图示例](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/1559570847866.png)

一个普通的流程图示例

对应的BPMN源码为：

```xml
......
  <bpmn:process id="Process_1em8emw" name="项目材料审核" isExecutable="true">
    <bpmn:startEvent id="StartEvent_1" name="开始节点">
      <bpmn:outgoing>SequenceFlow_1pagkpa</bpmn:outgoing>
    </bpmn:startEvent>
    <bpmn:sequenceFlow id="SequenceFlow_1pagkpa" sourceRef="StartEvent_1" targetRef="Task_0hn604q" />
    <bpmn:userTask id="Task_0hn604q" name="填写材料">
      <bpmn:incoming>SequenceFlow_1pagkpa</bpmn:incoming>
      <bpmn:incoming>SequenceFlow_058xsmq</bpmn:incoming>
      <bpmn:outgoing>SequenceFlow_0rs76c2</bpmn:outgoing>
      <bpmn:multiInstanceLoopCharacteristics />
    </bpmn:userTask>
      ......
    <bpmn:exclusiveGateway id="ExclusiveGateway_0ol3vgn" name="是否需要合伙人批阅">
      <bpmn:incoming>SequenceFlow_0icnkq2</bpmn:incoming>
      <bpmn:outgoing>SequenceFlow_1hoelow</bpmn:outgoing>
      <bpmn:outgoing>SequenceFlow_160qlaz</bpmn:outgoing>
    </bpmn:exclusiveGateway>
    <bpmn:endEvent id="EndEvent_1qn3z3o" name="结束">
      <bpmn:incoming>SequenceFlow_0ymsxgc</bpmn:incoming>
      <bpmn:incoming>SequenceFlow_160qlaz</bpmn:incoming>
    </bpmn:endEvent>
  </bpmn:process>
......
```

- 开始事件 **StartEvent**
  
    原型细线标记，标明一个流程的开始事件，分为空启动、异常启动、定时启动等。
    
- 结束事件 **EndEvent**
  
    原型粗线标记，标明一个流程的结束事件，分为空结束、异常结束、取消结束等。
    
- 顺序流 **SequenceFlow**
  
    顺序流表明两个模型之间的连接线，分为标准顺序流和条件顺序流。
    
- 任务 **Task**
  
    最重要的业务模型，一般用一个矩形表示，根据矩形左上角小图标的不同可以分为用户任务、Service任务、脚本任务、邮件任务等等。
    
- 网关 **Gateway**
  
    网关是用于控制流程走向的执行令牌，根据功能的不同主要分为排他网关、并行网关、包容网关、事件网关。
    
- 子流程 **Subprocess**
  
    把一切需要处理的任务归结到一起作为作为一个大流程的一部分，因为子流程嵌入在主流程中，所有也叫“嵌入式子流程”
    
- 边界事件 **Boundary Event**
- 中间事件 **Intermediate Event**
- 监听器 **Listener**

---

### UEL表达式

Camunda BPM支持统一表达式语言（EL），它是JSP 2.1标准（JSR-245）的一部分，使用开源JUEL（Java Unified Expression Language，EL的Java实现）实现。在Camunda BPM的绝大多数地方都可以使用UEL表达式，主要支持两个UEL表达式：UEL-value 和 UEL-method。

---

### 流程引擎的生命周期

![流程引擎的生命周期](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/Camunda%E6%B5%81%E7%A8%8B%E5%BC%95%E6%93%8E%E7%AE%80%E4%BB%8B%20a4fe0f2279754a49b750f583f42fa27b.png)

流程引擎的生命周期

生命周期

- **定义**：工作流的生命周期从定义开始，此阶段的主要任务是收集业务需求并且转化为流程定义。
- **发布**：由开发人员打包各种资源，在流程引擎平台发布流程定义。在具体的流程引擎中包括流程定义文件（.bpmn结尾）、自定义表单、任务监听类等等。
- **执行**：具体的流程引擎按照事先定义好的流程处理路线以任务驱动的方式执行业务流程。
- **监控**：根据任务（Task）的结果做出相应的处理。
- **优化**：不满足业务需求的流程或者需求变更的流程进行重新设计和发布。

## 二、流程引擎基础概念介绍

---

### 多租户 Multitenancy

总的来说，多租户是一个软件为多个不同组织提供服务的概念。其核心是数据是隔离的，一个组织不能看到其他组织的数据。在这个语境中，一个这样的组织（或部门、团队……）被称为一个*租户（tenant）*。

在Camunda流程引擎中部署流程定义时，可以传递一个*租户标识符（tenant identifier）*，当然为了实际使用流程数据上的租户标识符，所有查询API都可以通过租户过滤。

---

### 流程定义 Process Definitions

一个流程定义规定了整个流程的结构，Camunda BPM使用BPMN 2.0规范作为它的主要设计语言，并在此基础上扩展了新的元素和属性。

主要属性：tenant，processDefinitionKey，version，processDefinitionId

---

### 流程实例 Process Instances

流程实例是一个流程定义（Process Definitions）与业务对象联系的入口，也就是说是流程定义的实际应用。

流程实例与流程定义的关系与面向对象编程中的对象和类之间的关系相同。

关键属性：processInstanceId、instanceRemark

---

### 流程执行对象 Execution

Execution的含义就是一个流程实例（Process Instances）具体要执行的过程对象。一个流程实例中，流程实例本身也是一条Execution，作为所有Execution的根节点一起形成一个树状结构。

---

### 活动实例 Activity Instances

Activity Instances的概念与Execution比较像，不同的是Exectuion是把Activity Instances串起来的线的唯一标识。Activity Instances也是树状结构的，可能是Task、Subprocess、MuiltiInstance等等。

---

### 用户任务 UserTask

顾名思义，表示需要人员去审批的任务。

根据任务的处理方式的不同，可以分为单人审批，多人审批（并行），多人审批（串行）等。

任务可以设置的主要属性有审批人，任务到期日，任务优先级等。其中关于审批人的设置，除了直接填写用户之外，还可以填写UEL表达式来支持从流程变量中获取审批人或者组以及从JavaBean中获取审批人。

---

### 任务和任务定义 Jobs and Job Definitions

---

### 流程变量 Process Variables

流程变量是指在流程运行状态的时候（ProcessInstance而不是ProcessDefinition）存储的数据，它的数据结构本质为`Map<String, Object>`，可以通过流程引擎提供的API进行数据的修改。流程变量是范围性的，也就是说可以为某个任务节点单独设置流程变量，这个值仅仅存在于当前的任务节点。也可以为整个流程设置全局性的流程变量，在整个流程的运行过程中都是可以查看并使用的。

## 三、表结构及接口设计

---

### 流程引擎表结构

![流程引擎的表结构（来源：官方文档）](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/erd_710_bpmn.svg)

流程引擎的表结构（来源：官方文档）

根据前缀的不同，这些表可以分为以下几类：

- **ACT_GE_**：General，存储通用的配置
- **ACT_ID_**：Identity，存储身份相关的数据
- **ACT_RE_**：Repository，存储静态信息，如流程定义和流程资源等
- **ACT_HI_**：History，这些表中保存的都是历史数据，比如执行过的流程实例、变量、任务，等等。
- **ACT_RU_**：只保存流程实例在执行过程中的运行时数据，并且当流程结束后会立即移除这些数据，这是为了保证运行时表尽量的小并运行的足够快。

---

### 拓展表表结构

自定义拓展表的表结构以GC开头，主要用于存储流程引擎拓展功能的数据、投资管理系统特殊的业务逻辑以及前端在线画图功能的数据等。

[拓展表部分表结构展示](https://www.notion.so/a28f6f1ec71f42f6874f176ef24a77a8)

---

### 流程引擎API

![流程引擎API（来源：官方文档）](https://kiwi4814-1256211473.cos.ap-nanjing.myqcloud.com/img/api.services.png)

流程引擎API（来源：官方文档）

- **RepositoryService**提供了管理和控制发布包和流程定义的操作，发布一个流程定义意味着把它上传到引擎中，所有流程都会在保存进数据库之前分析解析好。
    - 查询引擎中的流程定义和部署信息。
    - 暂停或激活流程定义，对应全部和特定流程定义。
    - 获得流程定义的文件
- **IdentityService**可以管理（创建，更新，删除，查询…）群组和用户。
- **TaskService**主要涉及关于任务的操作。
    - 查询分配给用户或组的任务
    - 创建独立运行任务
    - 手工设置任务的审批人
    - 认领并完成一个任务，对任务进行多样化的操作
- **HistoryService**提供了流程引擎的所有历史数据的查询，比如历史流程实例、历史任务实例、历史流程变量等。

## 四、流程功能设计

---

### 选人

- **设置审批人**
  
    在流程设计的时候预先选好节点审批人，可以选用户、角色、部门、团队、团队-角色等等，也可自己实现接口用于选人。
    
- **动态选人（自选审批人）**
  
    在流程运行的时候动态的选择下一节点的审批人。
    

---

### 监听

流程引擎本身提供了在流程运行过程中各个节点的监听的实现，可以在捕捉到这些事件之后执行Java代码或者计算表达式。这些事件包括流程开始和结束，网关，节点，顺序流，任务的创建和完成等等。可以执行的代码包括UEL表达式、实现了对应监听接口的Java Class等。

---

### 会签

在多人审批的情况下，会存在会签的情况，比如投决会审批，当投委会成员投票数大于等于2/3时，流程就可以通过并且自动跳入下一个节点。针对这种情况，流程引擎本身提供了任务模型（Task）上的完成条件`completionCondition`以及顺序流（SequenceFlow）上的条件表达式`conditionExpression`。这两者都支持UEL表达式的条件判断，其中UEL表达式中可以使用流程变量。在Task上还可以使用流程引擎内置的额外四个变量：

- nrOfInstances 节点的所有的实例数量
- nrOfCompletedInstances 已经完成的实例数量
- nrOfActiveInstances 当前激活的实例数量，当`isSequential=true`时， nrOfActiveInstances的值永远为1
- loopCounter 实例循环的次数，依次递减

---

### 代理

任务可交由代理人来代为审批

---

### 打回

流程审批人将流程打回至流程发起人那里，等待重新发起

---

### 撤回

流程发起人撤回发起的流程

---

### 跳转

流程在执行中可以任意跳转到其他节点

---

### 加签

- 前加签
- 后加签
- 中间加签

## 五、提供的接口

[通用的参数名称说明](https://www.notion.so/9ba42812297746d492be6bc27f356f5a)

下面只列几个常用的接口，其他详见接口文档。

---

### 1. 查询流程定义列表

功能：用于在发起流程时候选择合适的流程去发起，比如过阶段审批、现金流审批、会议室申请等等。

接口地址：`/api/definition/findProcessDefinitionWithType.json`

主要查询条件：租户ID（`tenantId`），流程类型（`processType`）【表示类型】

---

### 2. 查询流程实例列表

功能：用于客户端查看和管理已发起的流程

接口地址：`/api/instance/findHisInstanceList.json`

主要查询条件：详见`ProcessSearch`类

---

### 3. 查询单个流程实例详情

功能：查询单个流程实例的详情，包括流程定义的xml字符串、流程实例的属性、流程实例已经生成的Activity instance列表等。

接口地址：`/api/instance/findInstanceInfo.json`

主要查询条件：流程实例ID（`processInstanceId`）

---

### 4. 发起一个流程实例

功能：根据流程的定义发起一个流程实例，是客户端发起流程的入口。

接口地址：`/api/instance/startProcessInstance.json`

参数：租户ID（`tenantId`），流程唯一Key（`processDefinitionKey`）

---

### 5. 查询待办列表

功能：查询待办列表

接口地址：`/api/task/findTasks.json`

主要查询条件：租户ID（`tenantId`），流程实例ID（`instanceId`），审批人（`userId`），任务定义Key（`taskDefinitionKey`）

---

### 6. 查询单个待办详情

功能：用于审批人登录系统审批的时候查看审批的详细情况以及表单等等。

接口地址：`/api/task/selectTaskInfoById.json`

主要查询条件：待办ID（`taskId`）

---

### 7. 待办审批

功能：客户端审批待办的入口

接口地址：`/api/task/audit.json`

参数：审批结果，审批意见

---

### 8. 查询操作日志

功能：查询流程的操作日志

接口地址：`/api/process/findProcessOperHistoryList.json`

主要查询条件：租户ID（`tenantId`），流程实例ID（`processInstanceId`），任务定义Key（`taskDefinitionKey`）

## **参考文档**

***Camunda BPM版本：7.10***

[【Camunda Docs】](https://docs.camunda.org/manual/7.10/)

[【Camunda BPM Javadocs】](https://docs.camunda.org/javadoc/camunda-bpm-platform/7.10/org/camunda/bpm/engine/)

[【BPMN 2.0规范】](https://docs.camunda.org/javadoc/camunda-bpm-platform/7.10/org/camunda/bpm/engine/)

[【UEL Docs】](https://docs.oracle.com/javaee/5/tutorial/doc/bnahq.html)

【Activiti相关博文】自行百度