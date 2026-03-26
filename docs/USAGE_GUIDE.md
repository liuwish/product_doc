# Product Doc Skill 使用指南

> JSON+MD 混合文档管理实践指南

---

## 快速开始

### 创建新文档

**步骤1：选择模板**
```bash
cd ~/.openclaw/workspace/skills/product_doc
cp templates/prd_template.json docs/requirements/my_product.json
cp templates/prd_template.md docs/requirements/my_product.md
```

**步骤2：编辑 JSON（结构化数据）**
```json
{
  "meta": {
    "id": "PRD-002",
    "title": "我的产品需求文档",
    "version": "1.0",
    "status": "draft",
    "author": "你的名字"
  },
  "product": {
    "name": "产品名称",
    "positioning": "产品定位"
  },
  "features": [
    {
      "id": "F001",
      "name": "功能名称",
      "priority": "high",
      "status": "todo"
    }
  ]
}
```

**步骤3：编辑 MD（详细内容）**
```markdown
# 我的产品需求文档

## 1. 产品概述
{详细描述...}

## 2. 核心功能
### 2.1 功能1
{使用场景、用户故事...}
```

**步骤4：提交到 Git**
```bash
git add docs/requirements/my_product.*
git commit -m "添加产品文档：我的产品"
git push
```

---

## 核心理念

### JSON 负责什么？

✅ **结构化元数据：**
- 文档基本信息（ID、版本、状态）
- 功能列表（ID、优先级、状态、负责人）
- 时间线和里程碑
- KPI 指标
- 关联关系

❌ **不要在 JSON 中写：**
- 长篇文字描述
- 用户故事细节
- 设计讨论过程
- 示例和截图

### Markdown 负责什么？

✅ **可读性内容：**
- 详细功能描述
- 用户故事和场景
- 设计思路和决策过程
- 图表、截图、流程图
- 讨论记录

❌ **不要在 MD 中写：**
- 重复 JSON 已有的数据（如功能列表表格）
- 可以用引用替代

---

## 最佳实践

### 1. 保持同步

**方法1：引用 JSON 字段（推荐）**
```markdown
# 功能列表

> 详细功能数据见：[my_product.json](./my_product.json#features)

## 功能1：用户登录（F001）
{详细描述...}
```

**方法2：自动化同步脚本**
```bash
# 将来实现
npm run sync
```

### 2. 单一数据源原则

**规则：JSON 是唯一权威数据源**
- 修改功能状态？→ 更新 JSON
- 修改负责人？→ 更新 JSON
- 修改优先级？→ 更新 JSON
- 修改详细说明？→ 更新 MD

### 3. 命名约定

**配对文件：**
```
2048_game_prd.json
2048_game_prd.md
```

**JSON 字段命名：**
- 小驼峰：`createdAt`, `targetUsers`
- 避免缩写：`description` 而非 `desc`
- 保持一致：所有 ID 字段都叫 `id`

### 4. 版本控制

**提交时同时包含两个文件：**
```bash
git add docs/requirements/my_product.json docs/requirements/my_product.md
git commit -m "更新产品文档：添加新功能 F005"
```

**使用语义化版本：**
- `v1.0.0`：重大版本
- `v1.1.0`：新增功能
- `v1.1.1`：Bug 修复

---

## 自动化工具

### 校验 JSON

```bash
# 使用 jq 验证 JSON 格式
jq empty docs/requirements/*.json

# 使用 JSON Schema 验证结构
# (将来实现)
npm run validate
```

### 查询数据

```bash
# 查询所有高优先级功能
jq '.features[] | select(.priority=="high")' docs/requirements/my_product.json

# 统计待办功能数量
jq '[.features[] | select(.status=="todo")] | length' docs/requirements/*.json
```

### 生成报告

```bash
# 将来实现
npm run generate:dashboard  # 生成项目仪表板
npm run generate:matrix     # 生成需求追踪矩阵
```

---

## 常见场景

### 场景1：更新功能状态

**步骤：**
1. 编辑 JSON 文件，修改 `features[].status`
2. 提交 Git：`git commit -m "更新功能状态：F001 已完成"`
3. MD 文件无需改动（状态数据在 JSON 中）

### 场景2：添加新功能

**步骤：**
1. 在 JSON 中添加新 feature 对象
2. 在 MD 中添加详细描述章节
3. 提交：`git commit -m "添加新功能：F007 推送通知"`

### 场景3：更新时间线

**步骤：**
1. 编辑 JSON 的 `timeline.milestones`
2. 提交：`git commit -m "调整里程碑：MVP 延期至 5月15日"`
3. MD 可选择性更新相关说明

### 场景4：生成 API 文档

**步骤：**
1. 编写 OpenAPI 规范 JSON（`api_spec.json`）
2. 使用工具生成 MD：
   ```bash
   # 使用 widdershins 或类似工具
   widdershins api_spec.json -o api_spec.md
   ```
3. 手动补充使用示例和注意事项

---

## 目录组织

### 推荐结构

```
docs/
├── requirements/
│   ├── README.md              # 需求文档索引
│   ├── 2048_game_prd.json
│   ├── 2048_game_prd.md
│   ├── feature_backlog.json   # 功能待办池
│   └── user_stories.md        # 用户故事集
├── design/
│   ├── README.md
│   ├── system_architecture.json
│   ├── system_architecture.md
│   ├── ui_spec.json           # UI 规范
│   └── ui_spec.md
├── api/
│   ├── README.md
│   ├── openapi.json           # OpenAPI 3.0 规范
│   └── api_guide.md           # API 使用指南
└── archive/
    └── deprecated_features.md
```

### 索引文件示例

**requirements/README.md：**
```markdown
# 需求文档索引

## 产品需求文档（PRD）
- [2048 游戏 PRD](./2048_game_prd.md) ([JSON](./2048_game_prd.json))

## 功能待办
- [Feature Backlog](./feature_backlog.json)

## 用户故事
- [User Stories](./user_stories.md)
```

---

## 工具与插件

### VS Code 插件推荐

- **JSON Schema Validator**：校验 JSON 结构
- **Markdown All in One**：Markdown 编辑增强
- **TODO Highlight**：高亮待办事项

### 命令行工具

- `jq`：JSON 查询和处理
- `yq`：YAML/JSON 转换
- `pandoc`：文档格式转换

---

## FAQ

### Q1：为什么要用 JSON+MD，而不是纯 MD？

**A：** 纯 MD 适合小型项目，但大型项目需要：
- 程序化处理（自动生成报告、仪表板）
- 数据一致性（状态、优先级、负责人）
- 工具集成（导入到任务管理系统）

JSON 提供结构化数据，MD 提供可读性，两者互补。

### Q2：JSON 和 MD 不一致怎么办?

**A：** 遵循"JSON 为准"原则：
- JSON 是单一数据源（SSOT）
- MD 仅提供详细说明
- 使用校验脚本检测不一致

### Q3：是否所有文档都需要 JSON？

**A：** 不是，根据需求：
- ✅ 需要 JSON：PRD、API 文档、系统架构
- ❌ 不需要 JSON：会议记录、讨论文档、教程

### Q4：如何避免维护双份文件的负担？

**A：** 
1. 使用模板快速创建
2. 明确职责划分（JSON 管数据，MD 管说明）
3. 开发自动化工具（JSON → MD 生成）
4. 使用引用而非复制

---

## 示例项目

查看 `docs/requirements/2048_game_prd.*` 作为完整示例：
- JSON：完整的元数据、功能列表、时间线、KPI
- MD：详细的功能描述、竞品分析、开发计划

---

## 贡献指南

欢迎改进文档规范和工具！

1. Fork 项目
2. 创建功能分支：`git checkout -b feature/doc-template`
3. 提交更改：`git commit -m "添加新模板"`
4. 推送分支：`git push origin feature/doc-template`
5. 创建 Pull Request

---

**最后更新：** 2026-03-26  
**维护者：** 刘伟世 (yach067659)
