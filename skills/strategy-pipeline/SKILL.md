---
name: "strategy-pipeline"
description: "量化小论坛帖子全流程编排器。单入口：URL → 自动调度 xbrowser → 策略处理 → 因子入库 → git，防止断链漏步"
---

# Strategy Pipeline — 量化帖子全流程编排

## 触发条件

用户给出一个量化小论坛帖子 URL，要求「处理」「跑 pipeline」「总结」「入库」「分析策略」等。

**单入口，读完本 SKILL.md 即开始，不需要先读其他 skill 的 SKILL.md。**

---

## 前置条件

- `xbrowser` skill 已安装就绪
- `strategy-viewer` 项目已 clone（`~/disk/workspace/strategy-viewer/`）
- 以下 skill 供本 pipeline 按需调用（本 pipeline 会引用它们的方法，但不需要预先读取，流程中会指引）：
  - `html-strategy-processor` — 文档生成
  - `factor-evaluator` — 因子索引入库

---

## Pipeline 标准流程

所有步骤**必须严格按照以下顺序执行**。每完成一步必须继续下一步，不允许提前结束。

### Step 0：确认入口

解析用户输入：
- **URL**：量化小论坛帖子链接（`https://bbs.quantclass.cn/thread/xxx` 或 `https://bbs.quantclass.cn/topic/index?id=xxx`）
- **要求**：用户是否有额外指示（如"只保存不分析"、"重点看代码"等）
- **无 URL** → 告知用户需要提供 URL

### Step 1：浏览器抓取

使用 `xbrowser` skill 的 CFT Chrome 打开帖子：

```bash
# 1. 初始化
xb init

# 2. 打开帖子
xb run --browser default open <URL>

# 3. 等 SPA 渲染完
xb run --browser default wait --load networkidle

# 4. 快照确认登录态
xb run --browser default snapshot -i
```

登录态确认后（有用户头像/名字/退出登录），通过 CDP 获取完整内容：

- 从 `~/.qclaw/tools/xbrowser/profiles/cft/DevToolsActivePort` 读取 CFT Chrome 的 CDP 端口
- 连 CDP WebSocket，用 `Runtime.evaluate` 取 `document.body.innerText`
- 用 `DOM.getDocument` + `DOM.getOuterHTML` 取完整 HTML
- 保存到 `blogsummary/strategy/htmls/`
- 关闭：`xb cleanup`

> ⚠️ **为什么不直接用 snapshot 的文本？** snapshot 只暴露交互元素（按钮/链接/heading），不暴露段落正文和代码块。必须走 CDP 的 `innerText`。

### Step 2：提取策略信息

从 Step 1 保存的文本/HTML 中提取：

**必取字段：**
- **策略名称** — 从标题或 h1 获取
- **作者** — 帖子用户
- **来源 URL**
- **因子清单** — 名称、原理、逻辑、参数、方向（升/降序）
- **过滤规则** — 如有
- **回测指标** — 年化、最大回撤、夏普、胜率、Calmar 等
- **代码附件** — 文件名称、大小
- **正文核心逻辑** — 策略思路简述

### Step 3：生成策略文档

使用 `/tmp/fix_extraction_and_regenerate.py` 中的提取逻辑，从 HTML 生成策略文档。
文档包含以下章节：

1. **# 标题** — HTML 文件名（不含任务标记和收益数字）
2. **作者** — 从 `【任务】【作者】` 或 `【小组任务】【作者】` 提取
3. **## 策略思路** — HTML 正文中提取的策略核心逻辑
   - 从策略描述/逻辑章节（一、策略思路/策略逻辑/策略描述/选股策略等）开始提取
   - 遇到以下内容即停止：
     - 二、资金曲线、三、策略评价、历年收益、一、策略简介
     - 三、策略代码、config代码
     - 打赏/评论/上一页/下一页/最热讨论
   - 跳过：页面导航、版主信息、目录、评论、附件列表
   - 传入 config 参数行（持仓周期、选股数量、因子列表等）需要跳过
   - 章节头（二、策略描述、三、config、三、策略回测、二、参数等）跳行但不截停
   - 配置值行（单引号值、日期至今、xx万、板块名/板块名）跳过
   - 配置代码 `'name': 'value'` 只在有 >0 行内容后触发截停，避免误杀短帖
4. **## 选股因子** — 从 HTML 提取 `factor_list[]` 解析为表格
5. **## 过滤条件** — 从 HTML 提取 `filter_list[]`
6. **## 回测表现** — 标题中的指标（年化_回撤_回撤比）或正文指标
7. **## 可取之处** — AI 分析：年化评级、Calmar 评级、因子类型分析、策略结构简评

### ⚠️ Step 3 完成后 → 立即跳到 Step 4，不允许结束

### Step 4：因子索引入库

参照 `factor-evaluator` skill 的方法：

#### Step 4a：确定主因子

按优先级：
1. **策略名称中明确出现且排在第一位的因子** → 名称即主因子
   - 例："基于总市值 × 成交额10日STD，回归动量构建" → 主因子 = 小市值（从"总市值"来）
   - 例："RSV+均线趋势+动量组合择时" → 主因子 = 均线趋势（复合中选最独特的）
2. 复合型策略 → 取 ICIR 或权重最高的因子
3. 择时类 → 择时方法本身当主因子

副因子：标题中其他因子 + 核心逻辑中剩下的因子，用 `; ` 分隔。

#### Step 4b：更新 `_index.md`

追加一行到 `factor/_index.md`：

```markdown
| {主因子} | {副因子} | {年化} | {Calmar} | {概要} | [查看](../strategy/docs/{文件名}.md) | [原帖]({URL}) |
```

#### Step 4c：遍历每个因子，更新对应目录

对**主因子和每个副因子**（不重复）分别执行：

1. **查找已有因子目录**：`factor/量价类/`、`factor/动量反转类/`、`factor/基本面类/`、`factor/择时类/`
2. 若无匹配目录 → 判断归类后新建
3. 更新 `archive/_index.md`（追加帖子行）
4. 更新 `作为副因子.md`（追加该因子的参数方向、ICIR、权重、同策略其他因子）
5. 更新 `常见搭配.md`（追加该因子与本策略其他因子的组合逻辑）

**因子归类参照表：**

| 因子特征 | 归入大类 |
|---------|---------|
| 成交量、成交额、换手率、市值、波动率、振幅 | 量价类 |
| Ret、反转、动量、Alpha类因子 | 动量反转类 |
| 基本面财务指标 | 基本面类 |
| 择时方法 | 择时类（只有 README + archive） |

> ⚠️ 如果策略名称为复合（如"总市值 × 成交额STD"），主因子和副因子都要入库，不要只入库主因子。

### ✅ Step 4 完成后 → 立即跳到 Step 5

### Step 5：Git 推送（批次分支）

推送时**不推 main**，推批次分支。批次名格式固定为 `batch/{YYYY-MM-DD}`（当天日期，Asia/Shanghai 时区）。

**首次推该批次：**
```bash
BRANCH="batch/$(TZ=Asia/Shanghai date +%Y-%m-%d)"

cd ~/disk/workspace/strategy-viewer
# 检查批次分支是否存在远端
if ! git ls-remote --exit-code --heads origin "$BRANCH" 2>/dev/null; then
  git checkout -b "$BRANCH"
else
  # 远端已有 → 拉下来作为本地分支
  git fetch origin "$BRANCH"
  git checkout -b "$BRANCH" "origin/$BRANCH"
fi

git add blogsummary/strategy blogsummary/factor blogsummary/notes
git commit -m "feat: {策略名}"

# 走代理推
export http_proxy=http://127.0.0.1:7897 https_proxy=http://127.0.0.1:7897
git push origin "$BRANCH"

# 切回 main 保持干净
git checkout main
```

**后续同批次帖子（branch 已存在）：**
```bash
BRANCH="batch/$(TZ=Asia/Shanghai date +%Y-%m-%d)"

git fetch origin "$BRANCH"
git checkout "$BRANCH"
git pull origin "$BRANCH"

# 工作和 commit ...

git push origin "$BRANCH"
git checkout main
```

### Pipeline 完成

输出完成卡片给用户：

```
✅ Pipeline 完成：{策略名}
- 主因子：{因子名}
- 副因子：{因子名}
- 年化：{数值} / 最大回撤：{数值}
- 文档：docs/{文件名}.md
- 因子入库：{量价类/动量反转类/...}
- 推送分支：batch/2026-07-27
```

---

## 分支流程

### 只保存不分析
用户说"先存下来" / "只抓取" — 只执行 Step 1，通知完成。

### 非因子帖子（纯讨论/干货分享）
下载的帖子不含选股因子/回测 — 跳过 Step 4，其余正常。

### 帖子无回测指标但有策略代码
按已有数据提炼因子，年化/回撤填 `—`，正常入库。

### 已有帖子重复处理
Step 1 之前先检查 `htmls/README.md` 和 `good_factor.md`，如已有记录 → 告知用户"该帖子已处理过"并停止。

---

## 已知问题记录

- **xbrowser snapshot 拿不到段落正文**：已解决，通过 CDP 直连取 innerText 可以绕过
- **主因子分类有灰色地带**：标题复合结构（A×B）时，主因子取更体现策略独特性的那个

---

## 执行链上下文（本 skill 在各 pipeline 中的位置）

| 项目 | 内容 |
|------|------|
| 角色 | 编排器，单入口 |
| 上游 | 无（用户提供 URL 即启动） |
| 本 pipeline 调用的子 skill | xbrowser → html-strategy-processor（方法） → factor-evaluator（方法） |
| 完成产物 | htmls/ + docs/ + factor/ + notes/ 全链路更新，git 已推批次分支 |
