---
name: "html-page-saver"
description: "通过浏览器打开 URL，完整渲染 JS 后保存 HTML 到 strategy/htmls/，并重命名文件。触发场景：用户给出一个帖子/帖子列表的 URL，要求抓取并保存。"
---

# HTML Page Saver

用浏览器（browser-cdp）打开 URL，等待 JS 完全渲染后，将完整 HTML 保存到 `strategy/htmls/`，并重命名为简洁文件名。

## 依赖

> browser-cdp 已全局注册，无需额外安装依赖。

## 触发条件

- 用户给出 URL，要求"抓取""保存""爬取""下载"帖子页面
- 用户给出多个 URL，要求批量处理

## 工作流程

### Step 1：准备工作

**必须先确认项目路径**（按优先级）：
1. 若当前 cwd 是 `strategy-digest/` 项目，直接使用
2. 若 cwd 不在项目内，从命令行参数或用户消息中解析出项目路径
3. 若无法确定，询问用户

```python
import os
# 确认项目根目录（strategy-digest/）
PROJECT_ROOT = "/Volumes/SN770/workspace/quant/frame/strategy-digest"
HTMLS_DIR = os.path.join(PROJECT_ROOT, "strategy", "htmls")
os.makedirs(HTMLS_DIR, exist_ok=True)
```

### Step 2：连接浏览器

```python
import sys
sys.path.insert(0, '/Users/davidli/Library/Application Support/QClaw/openclaw/config/skills/browser-cdp/scripts')

from browser_launcher import BrowserLauncher, BrowserNeedsCDPError
from cdp_client import CDPClient
from browser_actions import BrowserActions
from page_snapshot import PageSnapshot

launcher = BrowserLauncher()
try:
    cdp_url = launcher.launch(browser='chrome')
except BrowserNeedsCDPError as e:
    print(f"⚠️ {e}")
    sys.exit(1)

client = CDPClient(cdp_url)
client.connect()
actions = BrowserActions(client, None)
snapshot = PageSnapshot(client)
```

### Step 3：打开页面

```python
# 先检查是否有已有标签页
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

# 等待 JS 渲染完成
actions.wait_for_load()
import time; time.sleep(3)
```

### Step 4：登录检测

页面加载后，检测是否同时存在"登录"和"注册"按钮。如果两个都有，说明处于未登录状态，立即终止当前 skill 并通过微信通知用户。

```python
has_login_and_register = actions.evaluate('''
    var buttons = [...document.querySelectorAll('button span, a span, span')]
        .map(s => s.textContent.trim())
    var hasLogin = buttons.some(t => t === '登录')
    var hasRegister = buttons.some(t => t === '注册')
    return hasLogin && hasRegister
''')

if has_login_and_register:
    print("⚠️ 检测到未登录状态（登录+注册按钮同时存在），skill 终止")
    # 通过 message 工具通知用户：
    # action=send, channel=openclaw-weixin
    # message="⚠️ 检测到页面需要登录，请在浏览器中手动登录后重新发送 URL。"
    # to=<当前用户id>, accountId=<当前accountId>
    return None
```

> ⚠️ 检测到未登录后**直接结束 skill**，不轮询等待。用户手动登录后重新发送 URL 即可。

### Step 5：滚动触发加载

用 Ctrl+End 滚到页面底部，触发懒加载内容。

```python
import time

# Ctrl+End 滚到底部
actions.send_keys(["Control", "End"])
time.sleep(2)  # 等待内容加载
```

### Step 6：获取页面标题（用于命名）

```python
title = actions.evaluate("document.title") or url
```

### Step 7：获取完整 HTML

用 CDP 的 `DOM.getDocument` + `DOM.getOuterHTML`：

```python
# 通过 CDP 原始命令获取完整渲染后的 HTML
try:
    doc_result = client.send("DOM.getDocument", {"depth": -1, "pierce": True})
    node_id = doc_result.get("root", {}).get("nodeId")
    html_result = client.send("DOM.getOuterHTML", {"nodeId": node_id})
    full_html = html_result.get("outerHTML", "")
except Exception:
    # 降级：用 page_snapshot 的大文本模式
    full_html = snapshot.get_text()

if not full_html or len(full_html) < 500:
    print("⚠️ HTML 内容过少，页面可能未加载完成或需要登录")
    return None
```

### Step 8：保存 HTML 文件

```python
def sanitize_filename(name: str) -> str:
    """清理文件名，只保留合法字符"""
    import re
    # 移除非法字符
    name = re.sub(r'[\\/:*?"<>|]', '', name)
    name = name.strip()
    # 截断过长文件名
    if len(name) > 100:
        name = name[:100]
    return name or "untitled"

safe_title = sanitize_filename(title)
save_path = os.path.join(HTMLS_DIR, f"{safe_title}.html")

with open(save_path, "w", encoding="utf-8") as f:
    f.write(full_html)

print(f"✅ HTML 已保存：{save_path}")
print(f"   文件大小：{len(full_html):,} bytes")
```

### Step 9：更新索引（可选）

若 `strategy/htmls/README.md` 存在，在末尾追加一行：

```
| {标题} | {url} | {当前日期} |
```

---

## 文件命名规范

| 原标题 | 清理后 |
|--------|--------|
| `【任务】【 SinTimg】【年化30%】小市值策略_回撤5%` | `小市值策略_回撤5.html` |
| `【新手向】均线择时策略-精华帖第3期` | `均线择时策略-精华帖第3期.html` |

清理规则：
1. 移除所有 `【】` 及其内容
2. 移除 URL 参数和多余空格
3. 只保留中文、字母、数字、连字符、下划线

---

## 批量处理多个 URL

当用户给出多个 URL 时，**逐个处理**，每个 URL 重复 Step 3~9。

```python
urls = [
    "https://example.com/post1",
    "https://example.com/post2",
    # ...
]

for i, url in enumerate(urls):
    print(f"\n[{i+1}/{len(urls)}] 处理：{url}")
    # 执行 Step 3~7
    ...
```

---

## ⚠️ 重要注意事项

1. **遇到登录页立即停下** — 告知用户手动登录，不要尝试绕过
2. **等待 JS 渲染** — `wait_for_load()` 后再加 `sleep(3)`，确保动态内容完整
3. **CDP 获取 HTML** — 优先用 `DOM.getOuterHTML`，失败才降级到 `page_snapshot`
4. **检查 HTML 有效性** — 长度 < 500 bytes 视为异常，打印警告并跳过
5. **复用已有标签页** — 先 `list_tabs()`，有匹配的直接切换，不要新建
6. **任务结束后不要 `close()` / `stop()`** — 保持连接活跃
7. **单个 URL 操作最多重试 2 次**，仍失败则记录并跳过
