---
source:
  - "[[从 JSON 字符串到 Java 对象：Fastjson 1_2_83 全程解析｜得物技术]]"
create: 2025-10-17
---
## 1. 核心设计哲学与宏观架构

Fastjson 的设计哲学是 **“极致性能下的分层与协作”**。它将复杂的 JSON 转换问题拆解为一系列专注、高效且可扩展的组件，其架构设计处处体现着对性能的极致追求，但也因此在某些设计上（如 AutoType）引入了复杂性和安全风险。

### 1.1. 核心设计思想

- **门面模式 (Facade)** 
    用户仅与 `JSON.java` 的静态方法交互，如 `JSON.toJSONString()` 和 `JSON.parseObject()`。这极大地简化了 API，隐藏了内部由 `JSONSerializer`、`SerializeConfig`、`SerializeWriter` 等众多组件构成的复杂协作体系。
- **中央调度 + 策略模式 (Controller + Strategy):**
    - **调度器 (Controller)** 
        `JSONSerializer` (序列化) 和 `DefaultJSONParser` (反序列化) 是有状态的中央控制器。它们不仅驱动流程，还管理着当前的执行上下文（如对象图路径 `SerialContext`、解析位置 `ParseContext`），为循环引用检测、错误报告等功能提供支持。
    - **策略 (Strategy)** 
        `ObjectSerializer` 和 `ObjectDeserializer` 接口是策略的抽象。Fastjson 为每一种 Java 类型（如 `JavaBean`、`List`、`Map`、`Date`、`Enum` 等）都提供了具体的策略实现。调度器根据待处理对象的类型，动态选择并委托给相应的策略执行者。
- **配置与缓存 (Configuration & Cache):**
    - `SerializeConfig` 和 `ParserConfig` 是框架的“大脑”和配置中心。它们的核心是一个**线程安全的缓存**（通常是 `ConcurrentHashMap` 或类似结构），存储着 `Class` 到其对应（反）序列化器的映射。
    - **首次访问**一个类时，配置中心会负责查找或创建（可能通过 ASM 动态生成）其处理器，然后放入缓存。**后续访问**则直接从缓存中获取，避免了重复创建和查找的开销，这是保证高性能的关键一环。它们也是所有自定义扩展（如注册 `Module`）的入口。
- **性能至上 (Performance First):**
    - **ASM 字节码生成:** 这是 Fastjson 的“核武器”。它绕过了 Java 反射的性能瓶颈（方法查找、安全检查、参数装箱等），在运行时直接生成操作字节码，编译成与手写 Java 代码性能相近的直接方法调用和字段访问。
    - **极致 I/O 优化:**
        - **`SerializeWriter`**
            一个高度优化的字符写入器。它内部使用 `ThreadLocal` 缓存的 `char[]` 数组 (`bufLocal`) 作为缓冲区，避免了 `String` 拼接带来的大量临时对象和 GC 压力。缓冲区大小会动态扩展，并在使用完毕后，若大小未超过阈值 (`BUFFER_THRESHOLD`)，则会回收到 `ThreadLocal` 中供下次复用。
        - **`JSONLexer`**
            一个精巧的词法分析状态机。它直接在 `char[]` 数组上通过指针 (`bp`) 移动进行扫描和解析，并设计了大量 `scanFieldXXX` 这样的“快速路径”方法，在匹配字段名和解析基础类型值时，最大限度地避免创建 `String` 等中间对象。

### 1.2. 核心组件与职责 (详细版)

| 组件类 | 角色 | 核心职责与内部机制 |
| :--- | :--- | :--- |
| `JSON` | **门面 (Facade)** | 提供简单易用的静态 API，是所有操作的统一入口。 |
| `JSONSerializer` | **序列化调度器** | 1. 驱动序列化流程；2. 维护 `SerialContext` 链表，记录对象图路径，用于循环引用检测；3. 持有 `SerializeWriter` 实例，将序列化结果写入其中。 |
| `DefaultJSONParser` | **反序列化调度器** | 1. 驱动反序列化流程；2. 持有 `JSONLexer` 实例，消费 Token 流；3. 维护 `ParseContext`，用于处理 `$ref` 引用和错误定位；4. 管理 `ResolveTask` 列表，用于在解析完成后处理引用关系。 |
| `SerializeConfig` | **序列化配置中心** | 1. 缓存 `Class` -> `ObjectSerializer` 的映射；2. 扩展点：提供 `register(Module)` 方法；3. 内部通过 SPI 和 Module 机制发现和加载自定义序列化器。 |
| `ParserConfig` | **反序列化配置中心** | 1. 缓存 `Class` -> `ObjectDeserializer` 的映射；2. 核心安全组件：`checkAutoType` 方法在此实现；3. 扩展点：提供 `register(Module)` 和 `addAccept/addDeny` 等方法。 |
| `JavaBeanSerializer` | **JavaBean 序列化器** | 1. 遍历目标对象的 `FieldSerializer` 列表；2. 对每个字段值进行递归的 `JSONSerializer.write()` 调用；3. 处理循环引用 (`writeReference`)。 |
| `JavaBeanDeserializer` | **JavaBean 反序列化器** | 1. 创建目标类的空实例；2. 进入主循环，通过 `JSONLexer` 消费 Token；3. 采用“快速/慢速路径”智能匹配 JSON key 与 Java 字段；4. 递归调用 `DefaultJSONParser` 解析字段值；5. 最终将解析出的值填充到对象实例中。 |
| `SerializeWriter` | **高效写入工具** | 内部维护 `ThreadLocal<char[]>` 缓冲区，通过 `count` 指针进行写入，自动扩容，用完回收。是 Fastjson 写入性能的基石。 |
| `JSONLexer` | **词法分析工具** | 内部是一个大型状态机，通过 `bp` (buffer position), `ch` (current char), `token` 等状态变量，将 JSON 字符流高效地切割成结构化的 Token。 |

## 2. 核心工作流程 (可视化深度版)

### 2.1. 序列化流程 (Java 对象 -> JSON)

![序列化.svg](https://r2.129870.xyz/img/2025/e3e2669d710244763ffd213159e5ae2b.svg)

### 2.2. 反序列化流程 (JSON -> Java 对象)

![反序列化.svg](https://r2.129870.xyz/img/2025/e9fbc242901fa8355b20a3f6eacf5859.svg)

## 3. 关键特性与机制深度剖析

### 3.1. ASM 性能优化

- **对比反射**
    Java 反射 (`Method.invoke`) 慢，因为它包含：
    1.  运行时方法签名查找
    2. 访问权限检查
    3. 参数数组创建和装箱/拆箱
    4. 无法被 JIT 编译器深度内联优化。
- **ASM 的魔法**
    `ASMSerializerFactory` 在首次处理一个类时，会动态生成一个新的 `...ASMSerializer` 类。这个类的 `write` 方法内部没有反射，而是像手写代码一样的直接调用：

  ```java
  // 伪代码，示意 ASM 生成的逻辑
  public void write(JSONSerializer serializer, Object object, ...) {
      Person bean = (Person) object;
      SerializeWriter writer = serializer.out;
      writer.writeFieldName("name");
      writer.writeString(bean.getName());
      writer.writeFieldName("age");
      writer.writeInt(bean.getAge());
      // ...
  }
  ```

  这样的代码对 JIT 编译器极其友好，可以被完全内联和优化，性能逼近原生 Java 代码。反序列化同理。

### 3.2. 扩展性设计：SPI 与 Module

- **SPI (Service Provider Interface)**:
    - **机制**
        Java 标准的服务发现机制，通过在 classpath 下的 `META-INF/services/` 目录中放置配置文件来实现。
    - **Fastjson 实现**
        定义了 `AutowiredObjectSerializer` 接口，它比 `ObjectSerializer` 多一个 `getAutowiredFor()` 方法，用于声明其负责处理的 Java 类型。`SerializeConfig` 在初始化或需要时，通过 `ServiceLoader.load(...)` 扫描并加载这些“插件”。
    - **优点**
        无侵入式、自动发现。
    - **缺点**
        零散，不便于统一管理和配置一组相关的序列化器。
- **Module (模块)**:
    - **机制**
        一个更聚合、更强大的扩展方式。`Module` 接口定义了 `createSerializer` 和 `createDeserializer` 方法，可以作为一个“工厂”同时提供一整套相关类型的（反）序列化器。
    - **使用**
        必须通过 `config.register(new MyModule())` **手动编码注册**。
    - **优点**
        组织性强（如 `JodaModule` 统一处理所有 Joda-Time 类型），便于统一配置（可在模块构造函数中传入配置），注册效率高（无 classpath 扫描），注册顺序可控。

### 3.3. AutoType 机制与安全风险

- **功能价值 (The Good)**
    解决了 JSON 在表示多态对象时的天生缺陷。例如，一个 `List<Animal>` 序列化后，反序列化时无法知道里面的对象应该是 `Dog` 还是 `Cat`。`@type` 字段 `{"@type":"com.my.Dog", "name":"wangcai"}` 精确地记录了实现类，使得多态得以完美还原。
- **安全漏洞 (The Bad)**
    `@type` 字段的解析和实例化过程构成了一个典型的“反序列化漏洞”攻击面。攻击者可以指定任意存在于服务端 classpath 中的类。如果这个类在其构造函数、setter 或 getter（在某些情况下）中执行了危险操作（如文件操作、网络请求、命令执行），攻击就会成功。
- **经典 Gadget**
    `com.sun.rowset.JdbcRowSetImpl`。其 `setDataSourceName` 方法会触发 JNDI 远程查询，攻击者可搭建恶意的 LDAP/RMI 服务器，让受害者服务器加载并执行任意恶意代码，造成 RCE (远程代码执行)。
- **猫鼠游戏 (The Ugly)**
    Fastjson 早期采用黑名单策略，但攻击者通过各种手段绕过，如：
    - **L-notation**: `Lcom.sun.rowset.JdbcRowSetImpl;` (JNI 类型描述符)
    - **双重反序列化**: 利用某些类的 `toString` 或 `hashCode` 方法触发二次反序列化。
    - **`Throwable` 子类**: 利用 `Throwable` 的 `getStackTrace` 过程中的反序列化。
- **当前建议**:
    1. **绝对不要**使用 `JSON.parse(text)` 或 `JSON.parseObject(text)`，它们会默认尝试开启 AutoType。
    2. **始终**使用 `JSON.parseObject(text, YourClass.class)` 指定目标类型。
    3. 如果必须使用多态，**强烈建议升级到 Fastjson2**，它默认关闭 AutoType 并提供了更安全的 `seeAlso` 白名单机制。
    4. 若无法升级，在 Fastjson 1.x 中应**全局开启 `safeMode`**，并配置严格的白名单。

### 3.4. 循环引用处理

- **序列化**
    `JSONSerializer` 内部有一个 `IdentityHashMap<Object, SerialContext> references`。`IdentityHashMap` 使用 == 而非 ` equals()` 来比较 key，确保是同一个对象实例。
    - 首次遇到对象 `A`，将其放入 `references`，并正常序列化。
    - 当再次遇到对象 `A` 时，`references.containsKey(A)` 为 `true`。此时调用 `writeReference`，它会从 `SerialContext` 中获取 `A` 的路径（如 `$.data.items[0]`），并写入 `{"$ref":"$.data.items[0]"}`。
- **反序列化**: 这是一个**两阶段**过程。
    - **阶段一 (解析)**
        当 `JavaBeanDeserializer` 遇到 `$ref` 字段时，它不会立即解析。而是创建一个 `ResolveTask` 对象，该对象包含了需要被赋值的对象和引用的路径，并将其存入 `DefaultJSONParser` 的任务列表中。
    - **阶段二 (处理)**
        在整个 JSON 文本解析完毕后，`DefaultJSONParser` 会调用 `handleResovleTask` 方法。该方法遍历所有 `ResolveTask`，根据路径在已经构建好的对象图中找到被引用的对象，然后完成最终的赋值操作。

## 4. 总结与行动指南

- **Fastjson 1.x 评价**: 一个为性能而生的、设计精良但也存在历史包袱的库。它的 ASM 优化、I/O 处理和扩展机制都堪称典范，但 AutoType 的安全问题是其无法回避的硬伤。
- **行动指南**:
    - **新项目**: **首选 Jackson** (生态成熟，默认安全) 或 **Fastjson2** (极致性能，设计更现代，默认安全)。
    - **现有 Fastjson 1.x 项目**:
        1. **立即升级**: 确保使用的是官方修复了已知漏洞的最新版本 (如 1.2.83)。
        2. **代码审计**: 全局搜索 `parse(`, `parseObject(String` 等不带 `Class` 参数的方法，全部替换为指定具体类型的重载方法。
        3. **关闭 AutoType**: 如果业务不需要多态反序列化，通过 `ParserConfig.getGlobalInstance().setAutoTypeSupport(false);` 全局禁用。
        4. **启用安全模式**: 如果必须使用 AutoType，务必开启 `safeMode` (`ParserConfig.getGlobalInstance().setSafeMode(true);`)，并配置尽可能小的白名单 (`addAccept`)。
        5. **迁移计划**: 制定计划，逐步将项目从 Fastjson 1.x 迁移到 Jackson 或 Fastjson2。