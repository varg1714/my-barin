---
source: https://mp.weixin.qq.com/s/24UZVu-yrXwIt_YWn9txTg
create: 2024-06-20 15:28
read: false
---

# Go-Job 让你的任务调度不再繁琐｜得物技术

## 1. Go-Job 让你的任务调度不再繁琐｜得物技术

![](https://mmbiz.qpic.cn/mmbiz_gif/AAQtmjCc74DZeqm2Rc4qc7ocVLZVd8FOASKicbMfKsaziasqIDXGPt8yR8anxPO3NCF4a4DkYCACam4oNAOBmSbA/640?wx_fmt=gif)

**目录**

一、背景

二、架构设计

    1. 整体架构

    2. 服务端设计

        2.1 服务组成

        2.2 任务设计

            2.2.1 任务生成

            2.2.2 任务匹配

            2.2.3 任务分片

            2.2.4 任务生命周期

            2.2.5 任务与执行器

    3. SDK 设计

        3.1 任务执行

        3.2 连接管理

            3.2.1 建立连接

            3.2.2 断线重连

三、实战指南

    1. 代码开发

    2. 触发器创建

    3. 任务查看

四、成果与展望

**一**

**背景**

在选择任务调度平台时，团队遇到了一些实际的问题。现有的开源项目如 XXL-Job、Elastic-Job，虽然功能强大，但主要是围绕 Java 设计，而我们团队主要使用 Go 语言进行开发。这使得我们在集成和使用这些工具时遇到了诸多不顺。经过深入的调研和讨论，决定开发一个适合 Go 语言的任务调度框架，以满足我们的特定业务需求。于是，Go-Job 应运而生。

**为了让大家有个全面的了解，接下来主要探讨它的架构设计和功能特性。**

**本文的另一亮点是借助 GPT 生成了一些精美的章节彩色插图。看看大家和 GPT 是否是 “同款” 理解！**

**二**

**架构设计**

Go-Job 的目标是充分利用 Go 语言的优势，提供高性能、易扩展的分布式任务调度解决方案，满足不同业务团队的复杂需求和快速变化的技术环境。就像量脚定制鞋一样，专为 Go 项目量身打造！

**整体架构**

首先，让我们先认识一些关键术语：

**namespace**：命名空间，用于资源隔离。

**handler**：任务处理类，用户自定义实现。

**worker**：运行 handler 的业务服务，通过 SDK 接入，与 Go-Job 服务端通信。

**trigger**：调度平台上的触发器，包含调度规则、模式、超时配置等。

**task**：根据调度规则生成的任务信息。

**runInstance：**任务执行的最小单元。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdgZrVbLhoRx7MsqspykXbgN9xue84J8MmpDy0Y3eticzBs7UPQhjEdGg/640?wx_fmt=png&from=appmsg)

从上图可以看出，整个架构可分为三部分：

*   Web 控制台：负责任务配置管理，包括任务创建 / 编辑、历史任务查看、权限控制等；  
    
*   调度服务：负责触发器管理、任务生成、任务调度匹配等；  
    
*   执行器：接收并执行任务，同时将自身状态上报给调度服务。  

**任务调度流程如下**:  

*   通过 SDK 注册实现脚本逻辑的函数；  
    
*   通过在 Web 控制台创建定时任务；  
    
*   通过调度服务负责任务的定时创建，并下发给匹配的执行器；  
    
*   执行器执行接收到的任务，并上报任务状态。

**让我们用一个生动的比喻来理解这套架构**：

想象一下，你在一个大型积木乐园工作，任务就是搭建各种酷炫的模型。你有一个控制台（**Web 控制台**），可以配置不同的积木任务。在这里可以创建新的模型任务、编辑已有的任务、查看模型的历史记录，还能控制谁有权参与搭建。

接着，你有一位超级智能的朋友（**调度服务**），他会根据你设定的规则，定时提醒你什么时候该搭建什么样的模型。每当时间到了，他就会生成一个新的任务，并指派给最适合的搭建员（**执行器**）。

这些搭建员会认真执行每一个任务，拼接积木块（**Task**）并汇报他们的进展。这样一来，你就可以轻松管理和监督整个搭建过程，而每个模型也会在指定的时间内完成。

虽然这个架构看起来复杂，但每一个组件就像拼接积木一样简单易懂。通过控制台、智能调度和执行器的配合，可以高效、灵活地完成各种任务。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJd1lWH62EoY5X88lJgriarEkW4zXMQrsPicfOpMvZy2S9cH5wLzcwia5mNw/640?wx_fmt=png&from=appmsg)

(GPT 理解的架构)

**服务端设计**

说完整体架构和任务调度流程，再来看看服务端设计。Go-Job 的内部运作就像积木拼接。每一块积木都有自己的角色和作用，拼搭在一起形成一个高效的任务调度系统。

**服务组成**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdqJ2nIPypkbJVto1kggtO6e8ic2s08FcuYaCdAoOuD3p5WJwqKtxNaQg/640?wx_fmt=png&from=appmsg)

从上图可以看出，Go-Job 的服务层由三部分组成，它们共同构成了系统的基础：

*   **Controller**：服务集群的统一入口，提供 RESTful API 和 RPC 接口服务，所有任务的增删改查以及客户端 SDK 的连接操作都由 Controller 处理，并将相应的操作转发到其它模块。
*   提供 Web 控制台管理的 REST API 接口（包括命名空间管理、任务管理、数据查询等操作）。
    
*   提供 SDK 连接管理能力，负责维护客户端通过 SDK 接入的长连接状态。
    
*   作为请求控制入口，所有的请求都需要经过 Controller 来实现转发。
*   **Trigger**：调度服务的核心模块，通过监听触发器状态，定时创建任务，并推送到 Matching 服务用于任务匹配。
*   提供命名空间、触发器等数据的增删改查能力。
    
*   监听所有触发器，负责在规定时间到达时创建任务。
    
*   将生成的任务推送到 Matching 服务。
*   **Matching**：负责分配调度任务到接入客户端的节点上，通过接收 Trigger 推送的待匹配任务，基于匹配策略将其与接入的客户端节点进行匹配，并将匹配成功的任务和客户端进行绑定操作，推送完整的任务信息给 SDK。  

通过这些模块的协作，系统能够高效地处理和调度任务，确保每个任务都能找到最适合的执行器来完成。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdHWHYBFvppY62nMw9geEDkZ5Umh0UvO03kiakLZELVcts7vZ4zXxdT6g/640?wx_fmt=png&from=appmsg)

(GPT 理解的服务关系)

**任务设计**

紧接着，我们来深入探讨任务设计的细节。任务设计是确保系统高效运作的关键部分，包括任务的生成、匹配、状态变化以及执行过程中的各个环节。

*   **任务生成**  

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdeSsuMiaU36kulicCkXy5rRAshjRqBTMPTytKmQerD4UuryXichPQiajCcw/640?wx_fmt=png&from=appmsg)

当用户创建触发器时，系统首先根据调度规则计算最近一次需要执行的时间，生成一条触发事件，并插入到任务队列中。任务管理模块异步获取到期的触发事件，负责生成相应的 Task 任务，并将任务推送到 Matching 服务中。

Matching 服务接收到任务后，负责将其与合适的客户端进行匹配，并推送任务给客户端执行。客户端执行完任务后，将结果上报给任务管理模块。任务管理模块更新本次任务的结果后，重新计算下一次需要执行的时间，并再次将触发事件推送到任务队列，进入下一轮调度循环。

**一些问题思考**:

*   **时效性保证**：整个任务触发队列的消费者支持水平扩容，通过监控生产 / 消费速率，能够灵活的通过增加消费节点的形式来增加消费者实例从而提高处理并发度，减少排队时间。
    
*   **数据一致性**: 单次触发事件的消费，保证在一个事务下进行，基于持久化的特性，支持失败重试，确保任务最终被触发一次。
    
*   **健康检查**: 定时检查已注册未停用的触发器清单，通过计算它们的调度规则结合它们最近一次触发的任务信息，综合确认触发器任务生成的健康状况，如果监测到未正常创建任务的触发器，则生产异常报告，并推送给运维人员。
*   **任务匹配**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdib5LnTFCpe7NrdcKQuHIdR98IcemGxA4mv0Yf98VcRCJqbAnlby0v5w/640?wx_fmt=png&from=appmsg)

在任务调度整个系统中，任务匹配是确保任务能够被正确执行的关键部分。如上图所示，任务匹配模块设计分为三个主要部分：任务队列、客户端请求缓存、匹配模块。

**1. 任务队列 (taskQueue)**  

任务队列是待调度任务的存储单元，所有待调度的任务都会维护在任务队列中。具体细分如下：

*   **queueManage**：
*   **职责**：负责管理不同脚本的任务队列状态，包括队列的创建、删除等操作。通过管理不同的 taskQueue 来组织任务，根据任务的命名空间和脚本 ID 生成唯一索引，以确保不同脚本队列的唯一性。
*   **taskQueue**：
*   **职责**：存储等待调度的任务。调度器通过 queueManage 获取对应的 taskQueue，从中获取当前脚本等待调度的任务。
    
*   **内部子队列**：

1.  **activeQ**：基于触发时间排序的优先队列，触发时间与当前时间越接近的任务会优先被取出。
    
2.  **deadlineQ**：已匹配或已超时的任务会移入此队列，并最终归档。

**2. 客户端请求缓存 (clientCache)**  

客户端请求缓存是所有通过 SDK 接入的客户端节点的集合，这些节点的任务获取请求都会统一维护在该缓存中。

*   **clientCache**：
*   **职责**：维护所有客户端节点的请求状态。
    
*   **功能**：每个客户端的任务请求都会经历以下四个主要步骤：

1.  **入队 (inQueue)**：客户端请求进入等待队列。
    
2.  **等待 (wait)**：客户端在队列中等待任务分配。
    
3.  **超时 / 获取任务 (timeout/matched)**：在等待过程中，如果等待超时则进入超时处理状态，如果获取到任务则进入任务绑定状态。
    
4.  **返回 (return)**：任务匹配完毕后，客户端返回。

**3. 匹配模块 (scheduler)**

匹配模块是任务调度的核心部分，负责在有新任务生成或新节点接入时触发匹配动作。每次匹配尝试包括过滤和匹配两个主要步骤：

*   **过滤 (filter)**：
*   **职责**：剔除状态为已超时、已匹配、已关闭等不可调度的任务，确保进入匹配阶段的任务都是可执行的。
*   **匹配 (match)**：
*   **职责**：将经过过滤的任务和客户端进行匹配，基于标签 (label) 的匹配机制，只有双方标签一致的情况下，任务才能被分配给客户端。此机制目前用于支持染色环境、蓝绿部署等高级功能。

**任务匹配流程**：  

*   **任务生成**：新的任务生成后，进入对应的任务队列 (taskQueue) 中等待调度。
    
*   **触发调度**：当检测到新任务或新客户端接入时，触发一次匹配动作。
    
*   **过滤和匹配**：
*   调度器首先对任务进行过滤，剔除不可调度的任务。  
    
*   过滤后的任务和客户端进入匹配阶段，基于标签 (label) 进行匹配。

**4. 任务下发**：完成匹配后的任务和客户端进行双向绑定，并将任务下发到客户端执行。

通过实现脚本纬度的队列管理确保了不同类型的任务互不干预，同时通过对过期 / 超时资源的异步处理，提高了任务处理的时效性和系统资源的利用率。调度过程中的 filter&match 模块作为核心中间流程，支持灵活的扩展和变更，为未来实现更复杂的调度策略提供扩展基础。

*   **任务分片**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJd7JkylHDCiaYHECN9HUPNp8OTPgw2jYwib38uM8u2tPwlsz1ewf6Tz6Tg/640?wx_fmt=png&from=appmsg)

在某些业务场景中，单个任务的执行时间过长，为了解决这一问题，我们采用 “分而治之” 的策略。在创建触发器时，通过设置分片数量，可以将一次任务调度拆分成多个子任务，并发推送到不同的节点执行，同时会携带分片信息。这样不仅能提升任务执行的效率同时也能提升节点资源利用率。

**分片机制示例**

如上图所示，有五台集群机器，当前有两个任务 Task A 和 Task B。假设它们的总处理时间都是 8 分钟：

*   **Task A**：未设置分片，由单台 pod 执行，最终耗时为 8 分钟。
    
*   **Task B：**设置了 4 个分片，因此每个分片处理 1/4 的业务数据，每个子任务耗时 2 分钟。由于分片任务并发执行，最终耗时为 2 分钟。

通过分片机制，Task B 的执行时间缩短为 Task A 的 1/4，资源利用率提高了 4 倍。  

**代码示例：处理分片任务**

在业务代码层面，我们可以通过 GetTaskInfo 方法区分不同的子任务。以下是一个示例代码：

```
func (w *HelloHandler) Do(ctx job.Context) error {
    info := job.GetTaskInfo(ctx)
    fmt.Printf("完整参数 :%s \n", info.GetParam())
    fmt.Printf("分片参数 :%s \n", info.GetShardParam())
    fmt.Printf("当前执行id :%s \n", info.GetRunInstanceId())
    fmt.Printf("分片id :%d \n", info.GetShardNum())
    fmt.Printf("任务id :%v \n", info.GetTaskId())
    ...
    return nil
}
```

通过上述代码，我们能够获取并处理任务的分片参数和分片 ID，从而避免业务层不知道应执行哪部分任务的问题。

*   **任务生命周期**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdHB1quHHAqe0JOnSg6IH2ia8f199qoCz5rIib9GCPKq1qnopkZhhaLfOA/640?wx_fmt=png&from=appmsg)

在系统中，每个任务都有一个完整的生命周期。从任务创建到执行完成或失败，任务会经历多个状态转换。下面我们通过图示详细说明任务的生命周期及其各个状态的变化过程。

从图中可以看出，任务的生命周期可以分为以下几种状态：

*   **创建 (Created)**：触发器被触发后，会创建一个新的任务, 此时任务的初始状态即为 Created 状态。在此阶段，系统会创建任务并初始化相关数据, 并推送到 Matching 服务等待动态。
    
*   **等待执行 (WaitingToRun)**：任务在此状态表示已经和执行器进行了匹配，等待被执行器接收并开始执行。执行器通过拉取请求获得任务并创建 runInstance。
    
*   **执行中 (Running)**：任务正在被执行器处理。执行器上报任务开始事件，任务状态更新为 “执行中”。
    
*   **取消 (Canceled)**：任务在执行过程中可能因人为干预或系统策略被取消。任务被取消后，状态更新为 “取消”。
    
*   **完成 (RunToCompletion)**：任务成功执行并完成。执行器上报任务完成事件，任务状态更新为 “完成”，并结束生命周期。
    
*   **失败 (Faulted)**：任务执行过程中可能会失败。失败的任务状态更新为 “失败”，系统可能会触发重试策略。
*   **任务与执行器**

在实际业务场景中，不同脚本的任务量级差异较大。为了保证不同脚本之间任务处理的时效性互不影响，我们在任务匹配的设计上采用了命名空间 + 脚本 ID 组合生成唯一索引的方法。这个索引用于将不同脚本的任务维护在独立的待匹配队列中。

通过以下图片进行设计说明（其中 A、B、C 表示不同的业务脚本，client 表示业务侧的执行器，svc 则是调度服务端）。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJd42b0ZDH5YMnvY1hqDyt4o1JyNy2u29Q5DfSUAvlS9OKe4fuEyAibR0A/640?wx_fmt=png&from=appmsg)

从图片可以看出，不同的触发器，只要绑定的是同一个业务脚本，那么其创建的任务都会进入到同一个队列中。当多个执行器来获取任务时，会依次从对应脚本的任务队列中获得匹配的任务。

**优势**：

*   **独立队列**：不同脚本的任务队列相互独立，保证任务互不干预。
    
*   **动态扩展**：执行器能够根据任务数量动态扩展或缩减节点数量，提高系统的灵活性和适应性。

下面针对脚本的任务数量变化，详细介绍下执行器的处理逻辑。

*   任务数大于执行器数

假设有 3 个执行器，且脚本 A 的任务数超过 3 个，此时脚本 A 的任务并发处理速率就是 3，即同时可以并发处理 3 个任务。执行器可以基于其配置的调度策略和处理速度，动态进行节点扩展，以保证任务的处理速率。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdlGAibObOy1yFjrPia1EH1pRFZ68UVszOa2bE1X0WBAtKglBQiaq5a8DpA/640?wx_fmt=png&from=appmsg)

*   任务数小于执行器数

假设有 3 个执行器，但脚本 A 的任务数少于 3 个，此时第三个执行器将处于等待任务的状态，直到有新的任务生成并匹配成功后才能继续执行任务。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdubicdRYuJCofUiaWmgK6QYxXZCEhnJUVvJeA4kBkJkeEJXqn5U5RciaCA/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdUtUic2VFVLhR9YUibGp14ddFKVsPu7McJw93VrLib2Pc6x3Gia4TFUKGQQ/640?wx_fmt=png&from=appmsg)

(GPT 理解的任务设计)

**SDK 设计**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdzOIibtZCX5MZ4kC2H4TYcSE8rvQDxeVUibqW65p7iaaz4F0Jv3r2QPjLA/640?wx_fmt=png&from=appmsg)

在 Go-Job 系统中，业务服务通过 SDK 接入自定义的脚本，使任务处理更加灵活和高效。整个实现包含两个部分：脚本部分和 SDK 部分。

**脚本部分**

脚本部分是用户自定义的核心，通过实现 Do 接口来声明一个自定义的脚本。每个脚本都可以根据具体的业务需求进行编写，从而实现个性化的任务处理逻辑。

**SDK 部分**

SDK 部分是系统与脚本交互的桥梁，它包含了连接管理、数据收发、任务执行、健康检查和配置管理等模块。

*   **连接管理**: 负责建立和维护与 gRPC 服务器的连接。该模块确保在连接断开时，能够及时检测并重新连接。
    
*   **数据收发:** 负责通过 gRPC Stream 双向流发送和接收数据，包括接收任务和将结果返回给服务端。
    
*   **任务执行**：负责异步执行任务对应 Do 函数方法。
    
*   **健康检查**: 通过定期发送心跳包来维持连接的活跃状态，并监控连接的健康状况。在检测到故障时，触发相应的处理机制，如重启模块或重新建立连接。
    
*   **配置管理:** 负责管理 SDK 依赖的各种配置信息，包括环境变量、label 信息、服务端地址等。这些配置可以通过外部配置文件或动态更新。

**任务执行**

SDK 中的 worker 模块负责具体的任务处理逻辑，包括接收任务、初始化任务上下文、执行任务和上报执行结果。

具体步骤包括：

*   **接收任务 (recvTask)**：从数据收发模块接收任务数据。
    
*   **初始化任务上下文 (initTaskCtx)**：初始化任务处理所需的上下文信息 (函数初始化、context 初始化、超时设置等)。
    
*   **执行任务 (Process Do)**：执行对应任务函数逻辑并返回执行结果。
    
*   **任务完成 (finishTask)**：任务执行结束后的清理工作，包括对象销毁，结果上报等。

**连接管理**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdcpN02ukWeiaxC2NCTu0u6JQwAicuTzL5MvcKN9HxgSKBd1etJ3z5zdKg/640?wx_fmt=png&from=appmsg)

SDK 内部通过 gRPC 的双向 stream 流来实现与调度平台的交互操作，上图展示了如何保证 gRPC 双向流的稳定建立过程。整个模块分为两个主要部分：建立连接和断线重连。

*   **建立连接**

建立连接过程确保客户端和服务端能够成功建立 gRPC 连接，并进行数据交互。具体步骤如下：

*   **未连接状态**：
*   **客户端 (client)** 和**服务端 (service)** 初始处于未连接状态。
*   **发起连接请求**：
*   客户端向服务端发起 gRPC stream 连接请求。
    
*   服务端接收连接请求。
*   **连接成功通知**：
*   服务端成功接收连接请求后，发送连接成功通知给客户端。
    
*   客户端接收到连接成功通知后，触发 OnConnected 事件。
*   **客户端初始化**：
*   在 OnConnected 事件中，客户端通过该连接注册相关的脚本信息。
*   **服务端初始化**：
*   服务端在连接建立后，开始监测心跳状态。
    
*   服务端注册 client 节点信息，准备处理任务。
*   **数据交互**：
*   双方完成连接初始化后，基于该链接开始进行数据交互，包含任务请求、下发、状态上报等操作。
*   **断线重连**

当客户端和服务端之间的连接意外断开时，断线重连过程确保连接能够自动恢复，并继续任务处理。具体步骤如下：

*   **连接异常断开**：
*   客户端和服务端检测到连接异常断开。
*   **触发重连事件**：
*   客户端触发 Reconnecting 事件，并开始尝试重新建立连接。
    
*   定时发起一次连接请求，直到连接成功。
*   **服务端处理断开事件**：
*   服务端检测到连接断开，触发 OnDisconnected 事件。
    
*   移除客户端注册的信息，清理相关资源。
*   **重新建立连接**：
*   客户端再次向服务端发起 gRPC stream 连接请求。
    
*   服务端接收连接请求并处理。
*   **连接恢复**：
*   服务端成功接收连接请求后，发送连接成功通知给客户端。
    
*   客户端接收到连接成功通知后，触发 Reconnected 事件。
*   **重新初始化**：
*   客户端在 Reconnected 事件中，重新注册相关的脚本。
    
*   服务端重新注册 client 节点信息，继续处理任务。
*   **数据交互恢复**：
*   链接重新建立后，基于该链接开始进行数据交互，包含任务请求、下发、状态上报等操作。

通过上述建立连接和断线重连的详细步骤，SDK 和任务调度平台的连接管理模块确保了客户端和服务端之间的稳定通信。即使在网络不稳定的情况下，也能通过自动重连机制恢复连接，保证任务调度和处理的连续性和可靠性。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdaIX8LY0tuibHHhlklAguibIDFIMBZWRmLjpL0PCJFqgqT6SOgUTS8lqg/640?wx_fmt=png&from=appmsg)

(GPT 理解的 SDK)

**三**

**实战指南**

要实现一个自定义的脚本，并接入到任务调度平台只需要简单的 5 个步骤。

**代码开发**

在这段旅程中，将通过简单的 5 个步骤来实现一个自定义脚本，轻松接入调度平台。

**第一步: 在项目中引入 SDK 包**

go-job-sdk 

**第二步: 在项目的 config 结构体中引入相关 config**

```
import "go-job-sdk/config"

Config struct {
     JobConfig config.Config `yaml:"jobConfig"`
}
```

**第三步: 创建一个执行器结构体，并实现 Do 接口**

在 Do 接口内，实现业务逻辑

```
import (
   "go-job-sdk/job"
)

type HelloHandler struct{}

func (w *HelloHandler) Do(ctx job.Context) error {
    // 可以通过GetTaskInfo方法可以从上下文中获取任务的基本信息
   info := job.GetTaskInfo(ctx)
   fmt.Printf("参数 :%s \n", info.Param)
   fmt.Printf("当前执行id :%s \n", info.RunInstanceId)
   fmt.Printf("分片id :%d \n", info.ShardNum)
   fmt.Printf("执行超时时间 :%v \n", info.RunTimeout)
   fmt.Printf("任务id :%v \n", info.TaskId)
   return nil
}
```

‍**第四步: 接入 Go-Job 平台**

```
import (
   ...
   "go-job-sdk/config"
   "go-job-sdk/job"
   "go-job-sdk/worker"
)

func main() {
   group := worker.NewWorkerGroup(context.Background(), cfg.JobConfig)
   group.Add("hello-world1", &HelloHandler{}) // key 服务内唯一
   group.Add("hello-world2", &HelloHandler{}) // key 服务内唯一

    // 启动: 连接并注册脚本到go-job
   if err := group.Start(); err != nil {
      fmt.Println(err)
      return
   }

    // 该方法会block在这里. 若不需要block, 可以不调用wait方法
   if err := group.Wait(); err != nil {
      fmt.Println(err)
      return
   }
}
```

**第五步: 配置 Config**

```
jobConfig:
 app: "demo" # 脚本所在服务名称
 disable: true # 是否停止job注册 默认:false
 jobCenterService:
 rpcAddress: "xx" # 调度平台服务地址
 namespace: "test" # 命名空间
 handlers:
 hello-world1: # [脚本名称]关闭指定名称的脚本
 disable: true
```

 截止到这里，自定义执行器就已经创建好了，接下来可以去调度平台创建触发器。

**触发器创建**

支持丰富配置，包括调度策略、重试次数、分片设置、执行超时、任务超时、上下线设置、参数设置、染色环境、告警配置等。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdf53fzAVfelllbAUjhmYR7PUEpD6DKuiak0hgfjwKVsNIXzhGeW0UXZQ/640?wx_fmt=png&from=appmsg)

目前触发器的创建支持丰富的配置设置，包括:  

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdLQjhOTXh0UKkLqpY5tc9GnPiaib7FCPqOw3v1Bn3tnicQm1AraffxJnLg/640?wx_fmt=png&from=appmsg)

**任务查看**

控制台支持查看触发器的调度历史及任务执行详情。

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdLUZb2l4cRXeS6j4tFxLtAgLCibT8BvMgyicGnLArj9pURNESyR2N85Sg/640?wx_fmt=png&from=appmsg)

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdQjqCtYqXYyr3SaQzaelpEEqE1Zy6TO0tGxicN1pb4d2WiaEzkABmweIA/640?wx_fmt=png&from=appmsg)

控制台支持查看触发器的调度历史，以及每个任务对应的执行详细，包括客户端 IP、trace 等信息。

**四**

**成果与展望**

自项目上线以来，项目已成功接入公司内部多个业务团队，通过持续迭代功能，基本满足了公司不同业务需求。未来，将进一步增强平台能力，主要包含以下几个方面：

*   **任务监控与告警**
*   **可视化监控**：开发实时监控仪表盘，提供任务执行状态、资源使用情况等关键信息的可视化展示。
    
*   **高级告警配置**：支持更多维度的告警配置，如任务执行时间过长、资源超限等，并提供多渠道通知方式（如邮件、短信、电话等）。
*   **丰富的调度策略**
*   **多层级调度**：支持基于业务优先级的多层级调度策略，确保关键任务优先执行。
    
*   **节假日和工作日调度**：增加如节假日、工作日等场景的调度策略，满足不同场景的调度需求。
*   **安全与合规**
*   **权限管理**：加强用户权限管理，细化权限控制，确保系统数据安全。

**以****上是文章的完整内容，感谢大家抽出宝贵的时间阅读。**

**最后，让 GPT 为大家呈现一张全局架构插图，感谢观看！**

![](https://mmbiz.qpic.cn/mmbiz_png/AAQtmjCc74BGftqyJhcdlVicbVIicttTJd5U81m5ArYTQRMUe6sq7tD6k9Qg08uTKRD1ib3wATHPFqqoSQicluGxYw/640?wx_fmt=png&from=appmsg)

**往期回顾**

1. [可视化流量录制规则探索和实践｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525938&idx=1&sn=f72b5d3399c0abfbefc4a66367b849a9&chksm=c161336df616ba7bb5131cd3c3cbf2ffbd8b544e56c14cac9ad841f5b9054062c1f95c35b782&scene=21#wechat_redirect)  
2. [在得物的小程序生态实践](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525886&idx=1&sn=2dbea387abfb6c9e77dcd17a75cc5623&chksm=c16130a1f616b9b7d9afb7d55e9ff1d88f40d1dfd9b3bc3be8e0dcbb0b053021b279271f3e49&scene=21#wechat_redirect)  
3. [客服测试流水线编排设计思路和准入准出应用｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525842&idx=1&sn=8ef60b41b54efe9db929266934a7905d&chksm=c161308df616b99ba004dadd99239b94f163d7f51d5082fae34ca429ea68da56b35bc60260bb&scene=21#wechat_redirect)  
4. [深入剖析时序 Prophet 模型：工作原理与源码解析｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525652&idx=1&sn=9ad2289419f427b572cef7623dc11ab3&chksm=c161304bf616b95d181e3c40e29a05ee7491f6f8f92fbc3365eb137d838a1d9d2c4eda836860&scene=21#wechat_redirect)  
5. [深入理解 Babel - 项目管理工具 lerna 解析｜得物技术](http://mp.weixin.qq.com/s?__biz=MzkxNTE3ODU0NA==&mid=2247525627&idx=1&sn=0df103e2e7cfa5dabdacadfaeb503ddb&chksm=c16131a4f616b8b2609532df41fd6010ffdf05f47e25b9d3544149c9556410081887f1e91282&scene=21#wechat_redirect)  

文 / fred

关注得物技术，每周一、三、五更新技术干货

要是觉得文章对你有帮助的话，欢迎评论转发点赞～

未经得物技术许可严禁转载，否则依法追究法律责任。

“

**扫码添加小助手微信**

如有任何疑问，或想要了解更多技术资讯，请添加小助手微信：

![](https://mmbiz.qpic.cn/mmbiz_jpg/AAQtmjCc74BGftqyJhcdlVicbVIicttTJdZS9zWP6lzKGtXOVzmiaU0PKt1sFYLX5icrcfkWj5yB2MrkNibvvAesaMw/640?wx_fmt=jpeg&from=appmsg)