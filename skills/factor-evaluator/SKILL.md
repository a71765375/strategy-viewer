---
name: "factor-evaluator"
description: "从已生成的策略 MD 文档中提取因子信息，更新 blogsummary/factor/ 因子索引库。由 strategy-summarizer 在完成文档生成后自动调用。"
---

# Factor Evaluator — 因子索引更新器

## 触发条件

- 由 `strategy-summarizer` 在 Step 10 自动调用
- 参数：新生成的 MD 文档路径（相对于项目根）

## 工作流程

### Step 0：查重

检查 `blogsummary/factor/_index.md` 中原帖 URL 是否已存在：

```python
import os
md_path = "{传入的参数}"  # 如 blogsummary/strategy/docs/xxx.md
with open(md_path) as f:
    content = f.read()

# 从文档中提取原帖 URL（格式：📎 原始帖子：{url}）
import re
url_match = re.search(r'原始帖子[：:]\s*(https?://\S+)', content)
post_url = url_match.group(1) if url_match else None

# 检查入口表是否已有该 URL
if post_url:
    with open('blogsummary/factor/_index.md') as f:
        if post_url in f.read():
            print(f"⚠️ 帖子已处理过，跳过: {post_url}")
            return
```

- **已存在** → 跳过，不做任何操作
- **不存在** → 继续后续步骤

### Step 1：提取元信息

从策略 MD 文档中提取：

| 字段 | 提取方式 |
|------|---------|
| 策略名称 | 文档 `# ` 标题 |
| 策略概要 | 从文档首段或策略概述中提炼（选股+择时+年化+Calmar） |
| 年化收益率 | 搜索如 `36.18%` 的年化数值 |
| 收益回撤比（Calmar） | 搜索 `Calmar` 或 `收益回撤比` 后的数值 |
| 原帖 URL | 搜索 `📎 原始帖子：{url}` |
| 因子清单 | 从 `二、因子说明` 和 `三、策略配置` 中提取（名称、方向、ICIR、权重） |
| 择时方式 | 从策略描述中提取 |
| 特殊标注 | 如含未来函数等警告 |

### Step 2：定主因子

按优先级确定：

1. **策略名称中明确包含的因子** → 名称即主因子
   - 例："小市值反转" → 主因子 = 反转(Ret)
2. **复合型策略** → 选 ICIR 或权重最高的因子
   - 例：若未命名则选因子中 ICIR 最高的
3. **择时类策略** → 归入择时类，择时方法本身当主因子
   - 例："布林带择时策略" → 主因子 = 布林带择时

其余因子列为副因子，用 `;` 分隔。

### Step 3：更新入口表

在 `blogsummary/factor/_index.md` 末尾追加一行：

```markdown
| {主因子} | {副因子} | {年化} | {Calmar} | {概要} | [查看](../strategy/docs/{文件名}.md) | [原帖]({url}) |
```

### Step 4：更新各因子档案

遍历每个涉及因子（包括主因子和副因子）：

**4a. 更新 `archive/_index.md`**

在对应子目录的 archive 表格末尾追加一行（格式与入口表一致）。

**4b. 更新 `作为副因子.md`**

在文件末尾追加该策略中本因子的使用记录：

```markdown
## {策略名称}
- 参数方向：{方向}，ICIR {值}，权重 {值}
- 同策略其他因子：{列表}
```

**4c. 更新 `常见搭配.md`**

在文件末尾分析该因子与本次策略中其他因子的组合关系。

**4d. 创建 `README.md`（仅首次）**

如果因子目录不存在，先创建目录和 README.md，包括：定义、经济含义、用法、代码实现。

#### 因子目录分类规则

| 因子特征 | 归入大类 |
|---------|---------|
| 成交量、成交额、换手率、市值、波动率、振幅 | 量价类 |
| Ret、反转、动量、Alpha类因子 | 动量反转类 |
| 基本面财务指标 | 基本面类 |
| 择时方法 | 择时类（只有 README + archive） |

#### 代码实现模板

如果策略配置中有明确的因子代码，在 README.md 中附加实现：

```python
def add_factor(df, param=None, **kwargs):
    col_name = kwargs['col_name']
    # ... 因子计算逻辑
    return df[[col_name]], {col_name: 'last'}
```

### Step 5：更新择时档案

如果策略使用了择时方式，在 `blogsummary/factor/择时类/` 对应子目录下：

**5a. archive/_index.md**：追加帖子行。

**5b. README.md（仅首次）**：创建说明文档，含定义、参数、效果、代码实现。

### Step 6：提交

```bash
git add blogsummary/factor/
git commit -m "feat(factor): add {策略名称} to factor index"
git push
```

**提交策略**：
- 单篇处理 → 立即 commit + push
- 批量处理 → 每篇先 commit，全部完成后统一 push
- 注：`blogsummary/strategy/` 下的文件由 strategy-summarizer 管理，factor-evaluator 不碰

---

## 文件结构参考

```
blogsummary/factor/
├── _index.md                  # 全量入口表
├── 量价类/
│   ├── _index.md             # 本类概览
│   └── {因子名}/
│       ├── README.md         # 因子说明 + 代码实现
│       ├── 作为副因子.md      # 哪些策略用了、怎么用的
│       ├── 常见搭配.md        # 常和谁组合
│       └── archive/_index.md # 帖子清单表格
├── 动量反转类/ (同上)
├── 基本面类/  (同上)
└── 择时类/   (只有 README + archive)
```
