# 2048 游戏数据流设计文档

**文档版本：** v1.0  
**创建日期：** 2026-03-26  
**关联文档：** [2048_game_prd.md](../requirements/2048_game_prd.md)

---

## 1. 整体数据流架构

```mermaid
graph TB
    subgraph "用户层"
        A[用户输入] --> B{输入类型}
        B -->|滑动| C[方向事件]
        B -->|点击| D[按钮事件]
    end
    
    subgraph "控制层"
        C --> E[游戏控制器]
        D --> E
        E --> F{事件处理}
        F -->|移动| G[移动处理器]
        F -->|重新开始| H[初始化处理器]
        F -->|撤销| I[撤销处理器]
    end
    
    subgraph "业务逻辑层"
        G --> J[棋盘状态管理]
        H --> J
        I --> J
        J --> K[合并计算]
        K --> L[得分计算]
        L --> M[胜负判断]
    end
    
    subgraph "数据存储层"
        M --> N[游戏状态]
        N --> O[本地存储]
        N --> P[内存状态]
    end
    
    subgraph "视图层"
        P --> Q[状态订阅]
        Q --> R[UI 渲染]
        R --> S[动画效果]
        S --> T[用户反馈]
    end
    
    T -.反馈循环.-> A
    
    style A fill:#e1f5ff
    style R fill:#fff4e1
    style N fill:#e8f5e9
    style M fill:#ffebee
```

---

## 2. 核心数据流详解

### 2.1 游戏初始化流程

```mermaid
sequenceDiagram
    participant U as 用户
    participant C as 控制器
    participant G as 游戏引擎
    participant S as 状态管理
    participant V as 视图层
    
    U->>C: 启动游戏
    C->>G: 初始化游戏
    G->>S: 创建空棋盘(4x4)
    G->>S: 生成2个初始方块
    Note over S: 70% 概率为2<br/>30% 概率为4
    S->>V: 发送初始状态
    V->>U: 渲染棋盘
    V->>U: 显示得分=0
```

### 2.2 方块移动流程

```mermaid
flowchart TD
    A[用户滑动] --> B{检测方向}
    B -->|上| C[向上移动]
    B -->|下| D[向下移动]
    B -->|左| E[向左移动]
    B -->|右| F[向右移动]
    
    C --> G[遍历每列]
    D --> G
    E --> H[遍历每行]
    F --> H
    
    G --> I[压缩方块]
    H --> I
    I --> J[合并相同方块]
    J --> K{是否有移动?}
    
    K -->|是| L[生成新方块]
    K -->|否| M[无效移动]
    
    L --> N[更新得分]
    N --> O[检查胜负]
    O --> P{游戏结束?}
    
    P -->|胜利| Q[显示胜利]
    P -->|失败| R[显示失败]
    P -->|继续| S[渲染动画]
    
    M --> T[提示用户]
    S --> U[等待下一步]
    T --> U
    
    style L fill:#c8e6c9
    style Q fill:#81c784
    style R fill:#e57373
```

### 2.3 方块合并算法

```mermaid
graph LR
    A[原始行: 2,2,4,0] --> B[压缩: 2,2,4]
    B --> C{相邻相同?}
    C -->|2==2| D[合并: 4,4]
    C -->|4≠空| E[保持: 4]
    D --> F[结果: 4,4,0,0]
    E --> F
    F --> G[更新得分+4]
    
    style D fill:#ffeb3b
    style G fill:#4caf50
```

### 2.4 新方块生成流程

```mermaid
flowchart TD
    A[需要生成新方块] --> B[查找空位]
    B --> C{是否有空位?}
    C -->|否| D[棋盘已满]
    C -->|是| E[随机选择空位]
    E --> F[生成随机数]
    F --> G{概率判断}
    G -->|<0.7| H[生成方块2]
    G -->|≥0.7| I[生成方块4]
    H --> J[插入棋盘]
    I --> J
    J --> K[触发出现动画]
    K --> L[更新状态]
    
    style H fill:#e3f2fd
    style I fill:#fff9c4
```

---

## 3. 状态管理数据流

### 3.1 状态树结构

```mermaid
graph TB
    A[GameState] --> B[board: Tile[][]]
    A --> C[score: number]
    A --> D[bestScore: number]
    A --> E[gameStatus: Status]
    A --> F[history: State[]]
    A --> G[undoCount: number]
    
    B --> H[Tile: {value, id, merged}]
    E --> I[Status: playing/won/lost]
    F --> J[用于撤销功能]
    
    style A fill:#bbdefb
    style E fill:#ffccbc
    style F fill:#c5cae9
```

### 3.2 状态更新流程

```mermaid
sequenceDiagram
    participant A as Action
    participant R as Reducer
    participant S as State
    participant M as Middleware
    participant V as View
    
    A->>R: dispatch(MOVE_UP)
    R->>S: 获取当前状态
    R->>R: 计算新状态
    Note over R: 移动、合并、生成
    R->>M: 新状态
    M->>M: 校验合法性
    M->>M: 记录历史
    M->>S: 更新状态
    S->>V: 通知订阅者
    V->>V: 重新渲染
```

---

## 4. 数据持久化流程

### 4.1 本地存储流程

```mermaid
flowchart LR
    A[游戏状态变化] --> B{需要保存?}
    B -->|是| C[序列化状态]
    B -->|否| D[仅内存更新]
    
    C --> E[生成JSON]
    E --> F[LocalStorage.setItem]
    F --> G[持久化完成]
    
    H[页面刷新] --> I[LocalStorage.getItem]
    I --> J{数据存在?}
    J -->|是| K[反序列化]
    J -->|否| L[创建新游戏]
    K --> M[恢复游戏状态]
    
    style F fill:#a5d6a7
    style M fill:#90caf9
```

### 4.2 存储数据结构

```mermaid
erDiagram
    GAME_STATE {
        number[][] board
        number score
        number bestScore
        string gameStatus
        timestamp lastPlayed
        number moves
    }
    
    USER_PROFILE {
        string userId
        number totalGames
        number wins
        number[] scoreHistory
    }
    
    SETTINGS {
        boolean soundEnabled
        boolean animationEnabled
        string theme
    }
    
    GAME_STATE ||--o{ USER_PROFILE : belongs_to
    USER_PROFILE ||--|| SETTINGS : has
```

---

## 5. 实时交互数据流

### 5.1 用户输入处理

```mermaid
stateDiagram-v2
    [*] --> Idle: 游戏就绪
    Idle --> Processing: 用户输入
    
    Processing --> Validating: 验证移动
    Validating --> Invalid: 无效移动
    Validating --> Computing: 有效移动
    
    Computing --> Merging: 计算合并
    Merging --> Spawning: 生成新方块
    Spawning --> Checking: 检查状态
    
    Checking --> Won: 达到2048
    Checking --> Lost: 无法移动
    Checking --> Animating: 继续游戏
    
    Animating --> Idle: 动画完成
    Invalid --> Idle: 提示用户
    
    Won --> [*]
    Lost --> [*]
```

### 5.2 动画队列流程

```mermaid
flowchart TD
    A[触发移动] --> B[收集动画任务]
    B --> C[移动动画队列]
    B --> D[合并动画队列]
    B --> E[出现动画队列]
    
    C --> F[按顺序执行]
    D --> F
    E --> F
    
    F --> G{队列为空?}
    G -->|否| H[执行下一个]
    G -->|是| I[动画完成]
    H --> F
    I --> J[解锁用户输入]
    
    style F fill:#fff59d
    style I fill:#a5d6a7
```

---

## 6. 网络同步数据流（可选功能）

### 6.1 云端同步架构

```mermaid
graph TB
    subgraph "客户端"
        A[本地状态] --> B[同步触发]
        B --> C[数据加密]
        C --> D[HTTP Request]
    end
    
    subgraph "网络层"
        D --> E[API Gateway]
        E --> F[负载均衡]
    end
    
    subgraph "服务端"
        F --> G[认证服务]
        G --> H[数据校验]
        H --> I[数据库写入]
        I --> J[Redis 缓存]
    end
    
    subgraph "响应"
        J --> K[返回确认]
        K --> L[客户端更新]
    end
    
    style D fill:#e1bee7
    style I fill:#c5e1a5
    style K fill:#b2dfdb
```

### 6.2 排行榜数据流

```mermaid
sequenceDiagram
    participant C as 客户端
    participant A as API
    participant R as Redis
    participant D as 数据库
    
    C->>A: 提交得分
    A->>A: 验证合法性
    A->>D: 保存游戏记录
    A->>R: 更新排行榜(ZADD)
    R->>A: 返回当前排名
    A->>C: 确认提交
    
    C->>A: 请求排行榜
    A->>R: ZREVRANGE top100
    R->>A: 返回数据
    A->>C: 排行榜数据
```

---

## 7. 性能优化数据流

### 7.1 渲染优化

```mermaid
flowchart TD
    A[状态变化] --> B{虚拟DOM Diff}
    B -->|变化小| C[局部更新]
    B -->|变化大| D[全量更新]
    
    C --> E[最小DOM操作]
    D --> F[批量DOM操作]
    
    E --> G[requestAnimationFrame]
    F --> G
    
    G --> H[浏览器渲染]
    H --> I[硬件加速]
    I --> J[60fps 流畅动画]
    
    style G fill:#80deea
    style J fill:#81c784
```

### 7.2 缓存策略

```mermaid
graph LR
    A[计算请求] --> B{缓存命中?}
    B -->|是| C[返回缓存]
    B -->|否| D[执行计算]
    D --> E[存入缓存]
    E --> F[返回结果]
    C --> G[快速响应]
    F --> G
    
    style C fill:#c5e1a5
    style D fill:#ffcc80
```

---

## 8. 错误处理数据流

```mermaid
flowchart TD
    A[操作执行] --> B{是否出错?}
    B -->|否| C[正常流程]
    B -->|是| D[捕获错误]
    
    D --> E{错误类型}
    E -->|用户输入错误| F[提示用户]
    E -->|数据损坏| G[恢复默认状态]
    E -->|网络错误| H[离线模式]
    E -->|未知错误| I[日志记录]
    
    F --> J[用户重试]
    G --> K[新游戏]
    H --> L[本地存储]
    I --> M[上报监控]
    
    style D fill:#ef9a9a
    style M fill:#ffab91
```

---

## 9. 数据结构定义

### 9.1 核心类型

```typescript
// 棋盘方块
interface Tile {
  id: string;          // 唯一标识
  value: number;       // 数值 (2, 4, 8, ...)
  position: Position;  // 位置
  merged?: boolean;    // 是否刚合并
}

// 位置
interface Position {
  row: number;
  col: number;
}

// 游戏状态
interface GameState {
  board: (Tile | null)[][];  // 4x4 棋盘
  score: number;              // 当前得分
  bestScore: number;          // 最高分
  gameStatus: 'playing' | 'won' | 'lost';
  history: GameState[];       // 历史状态(撤销)
  undoCount: number;          // 剩余撤销次数
}

// 移动方向
type Direction = 'up' | 'down' | 'left' | 'right';

// 动画任务
interface AnimationTask {
  type: 'move' | 'merge' | 'spawn';
  tile: Tile;
  from?: Position;
  to?: Position;
  duration: number;
}
```

---

## 10. 数据流总结

### 关键路径
1. **用户输入** → 事件处理 → 状态更新 → 视图渲染
2. **状态变化** → 本地存储 → 数据持久化
3. **游戏逻辑** → 合并计算 → 得分更新 → 胜负判断

### 性能关键点
- 🚀 使用虚拟 DOM 减少渲染开销
- 🎯 动画队列管理避免卡顿
- 💾 本地存储缓存减少计算
- ⚡ requestAnimationFrame 保证 60fps

### 扩展性考虑
- 📊 预留网络同步接口
- 🏆 支持排行榜和社交功能
- 🎨 可配置主题和动画
- 📱 响应式设计适配多端

---

**文档版本：** v1.0  
**最后更新：** 2026-03-26  
**维护者：** 刘伟世

> 💡 **提示：** 所有流程图使用 Mermaid 格式，可直接在 GitHub/Markdown 编辑器中渲染。
