---
name: "strategy-summarizer"
description: "量化论坛帖子策略总结器。总入口：用户提供 URL + 提示词，自动完成抓取→解析→生成md→发微信（发文件附件）。触发场景：用户给出帖子 URL，要求总结策略、提取因子、分析代码等。"
---


> **PROJECT_ROOT**: `/Volumes/SN770/workspace/quant/frame/strategy-digest`
> 以下所有相对路径（`strategy/htmls/`、`strategy/docs/`、`notes/`）均相对于此目录。

# Strategy Summarizer — 策略总结总入口

一条消息搞定：URL → HTML → 策略文档 → 发微信。

## 触发条件

- 用户给出帖子 URL，要求"总结""分析""提取""整理"
- 用户说"帮我看看这个帖子"
- 用户给出多个 URL 要求批量处理

## 工作流程

### Step 0：解析输入

从用户消息中解析出：
- **URL**：从消息中提取以 `http` 开头的链接
- **提示词**：用户对总结的具体要求（如"重点提取因子逻辑和代码配置"）
- 若用户只给 URL 无提示词，默认按 `html-strategy-processor` 的标准结构生成（策略思路 → 因子说明 → 代码 → 总结）

### Step 1：连接浏览器

Chrome 已通过 QClaw 常驻在 28800 端口，直接连：

```python
import sys
sys.path.insert(0, '/Users/davidli/Library/Application Support/QClaw/openclaw/config/skills/browser-cdp/scripts')
from cdp_client import CDPClient
from browser_actions import BrowserActions
import time

CDP_URL = "http://127.0.0.1:28800"
client = CDPClient(CDP_URL)
client.connect()
actions = BrowserActions(client, None)
```

**异常处理**：若连接失败（Connection refused），告知用户「Chrome 未开启，请先手动打开 Chrome」，不要继续。

### Step 2：打开 URL

```python
tabs = client.list_tabs()
target_tab = None
for t in tabs:
    if url in t.get('url', ''):
        target_tab = t
        break

if target_tab:
    client.attach(target_tab['id'])
else:
    tab = client.create_tab(url)
    client.attach(tab['id'])

actions.wait_for_load()
time.sleep(3)
```

### Step 3：提取正文内容

用 CDP evaluate 执行 JavaScript，提取帖子正文（策略名、因子、代码块等）：

```javascript
(function() {
    var selectors = [
        '.thread-content', '.post-content', '.article-content',
        '.vditor-reset', '[class*=content]', 'article', '.markdown-body'
    ];
    for (var s of selectors) {
        var el = document.querySelector(s);
        if (el && el.innerText.length > 200) return el.innerText;
    }
    var divs = document.querySelectorAll('div, article, section');
    var best = null, bestLen = 0;
    for (var d of divs) {
        var text = d.innerText;
        if (text.length > bestLen && text.length < 50000) {
            bestLen = text.length; best = d;
        }
    }
    return best ? best.innerText : 'NOT_FOUND';
})()
```

**异常处理**：若返回 `NOT_FOUND` 或内容过少（< 500字），告知用户「页面内容无法提取，可能是登录墙或动态加载失败，请手动确认」。

### Step 4：提取页面标题

```python
title = actions.evaluate("document.title") or "untitled"
```

### Step 5：提取完整 HTML（用于后续文件保存）

```python
doc_result = client.send("DOM.getDocument", {"depth": -1, "pierce": True})
node_id = doc_result["root"]["nodeId"]
html_result = client.send("DOM.getOuterHTML", {"nodeId": node_id})
full_html = html_result["outerHTML"]
```

### Step 6：生成策略文档

根据正文内容和用户提示词，生成 markdown 文档，写入：

```
{strategy-digest}/strategy/docs/{清理后的标题}.md
```

**文档结构**（默认，可根据提示词调整侧重点）：
```markdown
# {策略名}

📎 原始帖子：{url}

## 策略概述
- 来源、核心思路（一两句话）

## 附件信息
- 有附件：文件列表
- 无附件：标注"无"

## 一、策略思路
（按提示词决定详略，若提示词侧重思路则重点写）

## 二、因子说明
（每个因子：原理 / 逻辑 / 参数 / ICIR）

## 三、策略配置
（config.py 完整内容）

## 四、策略代码
（关键因子文件代码）

## 五、总结
```

### Step 7：保存 HTML 并重命名

```python
def sanitize_filename(name: str) -> str:
    import re
    name = re.sub(r'[\\/:*?"<>|]', '', name)
    name = re.sub(r'【.*?】', '', name)   # 去掉【】标记
    name = re.sub(r'\[.*?\]', '', name)
    # 去掉年化/回撤等数字参数（太长）
    name = re.sub(r'[\d.]+[%年化回撤Calmar夏普]', '', name)
    name = name.strip('_-.')
    if len(name) > 80: name = name[:80]
    return name or "untitled"

safe_name = sanitize_filename(title)
import os
html_path = f"/Volumes/SN770/workspace/quant/frame/strategy-digest/strategy/htmls/{safe_name}.html"
with open(html_path, "w", encoding="utf-8") as f:
    f.write(full_html)
print(f"HTML已保存: {html_path}")
```

### Step 8：更新索引（必须全部执行）

**必须依次完成以下全部操作，缺一不可：**

**8.1 更新 `strategy/htmls/README.md`**
- 在表格末尾追加一行：策略名 | 文档路径 | HTML路径 | 主要因子 | 日期
- 如果表格不存在则创建

**8.2 更新 `notes/good_factor.md`**
- 在末尾追加新策略段落（因子聚焦格式）
- 段落末尾附上原始帖子 URL：`📎 原始帖子：{url}`
- 追加格式参考文档末尾已有策略的格式

**8.3 更新 `strategy/docs/README.md`（如果存在）**
- 在表格末尾追加一行

**8.4 完成后自检清单（必须逐项核对）**
完成 Step 8 后，必须执行以下自检，确认全部完成再进入 Step 9：

```
✅ 1. strategy/htmls/ 目录下有对应 HTML 文件？
✅ 2. strategy/docs/ 目录下有对应 md 文档？
✅ 3. notes/good_factor.md 末尾已追加新策略段落？
✅ 4. notes/good_factor.md 段落末尾包含原始帖子 URL？
✅ 5. strategy/htmls/README.md 已更新索引表格？
```

如果有任一项为❌，立即修复后再继续。**不要跳过任何一项，也不要假设已经完成。**

### Step 9：发送文档到微信（最重要）

**通过 `message` 工具发送 md 文件作为附件**，同时附简短说明：

- `action`: `send`
- `channel`: `openclaw-weixin`
- `message`: `📄 {策略名} 已生成完整文档，详见附件`
- `filePath`: 填入生成的 md 文件的绝对路径

```python
import os
# 读取生成的文档路径
import glob
docs = sorted(glob.glob(f"/Volumes/SN770/workspace/quant/frame/strategy-digest/strategy/docs/*.md"))
latest_md = docs[-1]  # 最新生成的文档

msg = f"📄 {strategy_name} 已生成完整文档，详见附件"
print(f"文件路径: {latest_md}")
print(f"消息内容: {msg}")
# message 工具使用:
# action=send, channel=openclaw-weixin, message=msg, filePath=latest_md
```

> ⚠️ **必须发送文件作为附件**，不要发送文字内容。微信消息限制约 2000 字，文档内容远超此限制，发文件才是正确方式。

---

## ⚠️ 最终自检（发微信前必须逐项确认）

在发送微信前，再次核对以下所有文件操作已全部完成：

```
📁 文件生成检查：
✅ HTML文件存在？  → ls strategy/htmls/{sanitized_name}.html
✅ MD文档存在？    → ls strategy/docs/{sanitized_name}.md

📝 索引更新检查：
✅ good_factor.md 已追加？ → tail -5 notes/good_factor.md
✅ README.md 已更新？      → grep "{strategy_name}" strategy/htmls/README.md

🔗 URL追踪检查：
✅ good_factor.md 段落末尾有 URL？ → grep "bbs.quantclass.cn" notes/good_factor.md | tail -1
```

**任一项不存在 → 立即修复 → 重新确认 → 再发微信**

---

## 批量处理多个 URL

用户一次给多个 URL 时，逐个执行 Step 2~9，每完成一个发一次微信，最终发一条汇总。

```python
urls = [...]  # 从消息中提取的所有URL
for i, url in enumerate(urls):
    print(f"[{i+1}/{len(urls)}] 正在处理：{url}")
    # 执行 Step 2~9
    ...
print("全部完成")
```

---

## 异常处理总览

| 异常情况 | 处理方式 |
|---------|---------|
| Chrome 未启动（Connection refused） | 微信告知「Chrome 未开启，请先手动打开」 |
| 页面内容 < 500字 | 微信告知「页面内容无法提取，可能是登录墙」 |
| HTML 保存失败 | 打印错误，继续尝试生成 md（正文已提取） |
| md 生成失败 | 微信告知具体哪一步出错 |
| message 发送失败 | 打印错误，不阻塞流程 |

---

## 文件路径速查

```
strategy-digest/
├── strategy/
│   ├── docs/          # 生成的策略文档（.md）
│   └── htmls/         # 抓取的HTML源文件
└── notes/
    └── good_factor.md # 因子聚焦笔记
```
