# 产品文档库

> 版本管理仓库，采用 **JSON+Markdown 混合管理模式**，兼顾机器可读性和人类友好性

## 核心理念

**JSON+MD 双格式管理：**
- 📊 **JSON**：结构化元数据，支持自动化处理和程序解析
- 📝 **Markdown**：可读性文档，便于人类编辑和协作
- 🔗 **关联引用**：通过文件名约定保持同步

## 目录结构

```
product_doc/
├── README.md              # 本文件
├── docs/                  # 文档存放目录
│   ├── requirements/      # 需求文档
│   │   ├── *.json         # 结构化需求数据
│   │   └── *.md           # 可读需求描述
│   ├── design/            # 设计文档
│   │   ├── *.json         # 设计元数据
│   │   └── *.md           # 设计说明
│   ├── api/               # API 文档
│   │   ├── *.json         # API 规范（OpenAPI）
│   │   └── *.md           # API 使用说明
│   └── archive/           # 归档文档
├── templates/             # 文档模板
│   ├── prd_template.json
│   ├── prd_template.md
│   ├── api_template.json
│   └── api_template.md
└── scripts/               # 自动化脚本
    ├── validate.js        # JSON 校验
    ├── sync.js            # JSON ↔ MD 同步
    └── generate.js        # 文档生成
```

## 使用说明

### 基础操作

**查看状态：**
```bash
cd ~/.openclaw/workspace/skills/product_doc
git status
```

**提交文档：**
```bash
git add .
git commit -m "描述本次更新内容"
```

**查看历史：**
```bash
git log --oneline
```

### 分支管理

**创建功能分支：**
```bash
git checkout -b feature/new-doc
```

**合并到主分支：**
```bash
git checkout master
git merge feature/new-doc
```

## 文档规范

### 双格式约定

**1. 文件命名配对**
- JSON 文件：`{name}.json`
- MD 文件：`{name}.md`
- 示例：`2048_game_prd.json` + `2048_game_prd.md`

**2. JSON 职责**
- 产品元数据（ID、状态、优先级、负责人）
- 结构化数据（需求列表、API 端点、类关系）
- 可量化指标（KPI、时间线、资源分配）
- 关联关系（依赖、引用）

**3. Markdown 职责**
- 详细描述和说明
- 用例场景和示例
- 图表和截图
- 讨论和决策记录

**4. 数据同步原则**
- JSON 为单一数据源（Single Source of Truth）
- MD 通过引用 JSON 字段避免重复
- 修改元数据时优先更新 JSON
- 使用 `scripts/sync.js` 自动同步

### 命名规范

- 文件名：小写字母+下划线（如：`user_guide.md`）
- JSON 字段：小驼峰（如：`createdAt`, `assignedTo`）
- Git 提交：清晰描述变更内容
- 版本标签：语义化版本（如：`v1.0.0`）

### JSON Schema 规范

**PRD 文档结构：**
```json
{
  "meta": {
    "id": "string",
    "title": "string",
    "version": "string",
    "status": "draft|review|approved|archived",
    "createdAt": "ISO8601",
    "updatedAt": "ISO8601",
    "author": "string",
    "reviewers": ["string"]
  },
  "product": {
    "name": "string",
    "positioning": "string",
    "targetUsers": ["string"],
    "goals": ["string"]
  },
  "features": [
    {
      "id": "string",
      "name": "string",
      "description": "string",
      "priority": "high|medium|low",
      "status": "todo|in_progress|done",
      "assignee": "string",
      "estimatedDays": "number",
      "dependencies": ["string"]
    }
  ],
  "timeline": {
    "start": "YYYY-MM-DD",
    "milestones": [
      {"date": "YYYY-MM-DD", "event": "string"}
    ]
  },
  "kpis": [
    {"metric": "string", "target": "string"}
  ]
}
```

**API 文档结构（遵循 OpenAPI 3.0）：**
```json
{
  "openapi": "3.0.0",
  "info": {
    "title": "string",
    "version": "string"
  },
  "paths": {
    "/endpoint": {
      "get": {
        "summary": "string",
        "parameters": [],
        "responses": {}
      }
    }
  }
}
```

## 维护者

- 刘伟世 (yach067659)
- 培优课堂智能助手（自动化管理）

---

**创建时间：** 2026-03-26  
**最后更新：** 2026-03-26
