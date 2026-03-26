# 产品文档库

> 版本管理仓库，用于存储和管理产品相关文档

## 目录结构

```
product_doc/
├── README.md           # 本文件
├── docs/              # 文档存放目录
│   ├── requirements/  # 需求文档
│   ├── design/        # 设计文档
│   ├── api/           # API 文档
│   └── archive/       # 归档文档
└── templates/         # 文档模板
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

- 使用 Markdown 格式编写
- 文件命名：小写字母+下划线（如：user_guide.md）
- 提交信息：清晰描述变更内容
- 重要版本打 tag：`git tag v1.0.0`

## 维护者

- 刘伟世 (yach067659)
- 培优课堂智能助手（自动化管理）

---

**创建时间：** 2026-03-26  
**最后更新：** 2026-03-26
