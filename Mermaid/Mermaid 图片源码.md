
## 1. Mermaid 图片源码

### 1.1. 数据库

#### 1.1.1. 主从复制方式对比

```txt
graph LR
    A[事务提交请求] --> B{复制模式}
    
    B -->|异步复制| C[写入redo log prepare]
    B -->|半同步复制| D[写入redo log prepare]
    B -->|全同步复制| E[分布式事务协调]
    
    C --> F[写入binlog]
    D --> G[写入binlog]
    E --> H[所有节点协调]
    
    F --> I[立即提交redo log]
    G --> J[等待从库ACK]
    H --> K[所有节点确认]
    
    I --> L[返回客户端]
    J --> M{收到ACK?}
    K --> N[返回客户端]
    
    M -->|是| O[提交redo log]
    M -->|超时| P[降级异步提交]
    
    O --> Q[返回客户端]
    P --> Q
    
    style C fill:#c8e6c9
    style D fill:#fff3e0
    style E fill:#ffebee
```

#### 1.1.2. 半同步流程

```txt
sequenceDiagram
    participant Client as 客户端
    participant Master as 主库
    participant Slave as 从库
    
    Note over Client,Slave: 半同步复制详细流程
    
    Client->>Master: 执行INSERT/UPDATE/DELETE
    Master->>Master: 开始事务
    
    rect rgb(255, 243, 224)
        Note over Master: 两阶段提交-准备阶段
        Master->>Master: 写入redo log prepare
        Master->>Master: 写入binlog到磁盘
    end
    
    rect rgb(255, 235, 238)
        Note over Master,Slave: 半同步复制阶段
        Master->>Slave: 发送binlog事件
        
        alt 从库正常响应
            Slave->>Slave: 接收binlog事件
            Slave->>Slave: 写入relay log
            Slave->>Master: 发送ACK确认
            Master->>Master: 收到ACK确认
        else 从库超时无响应
            Master->>Master: 等待超时
            Master->>Master: 自动降级为异步
        end
    end
    
    rect rgb(232, 245, 233)
        Note over Master: 两阶段提交-提交阶段
        Master->>Master: 提交redo log commit
    end
    
    Master->>Client: 返回事务提交成功
    
    rect rgb(245, 245, 245)
        Note over Slave: 异步执行阶段
        Slave->>Slave: SQL线程执行语句
        Slave->>Slave: 更新从库数据
    end
```