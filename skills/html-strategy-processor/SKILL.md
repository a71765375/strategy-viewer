---
name: "html-strategy-processor"
description: "Processes HTML strategy documents from quant forum. Invoke when user adds new HTML files to strategy/htmls/ directory or asks to process strategy HTML files."
---


> **PROJECT_ROOT**: `/Volumes/SN770/workspace/quant/frame/strategy-digest`
> 以下所有相对路径（`strategy/htmls/`、`strategy/docs/`、`notes/`）均相对于此目录。

# HTML Strategy Document Processor

This skill processes HTML strategy documents from quantitative trading forums and generates simplified factor-focused documentation.

## When to Invoke

**Invoke this skill when:**
- User adds new HTML files to `strategy/htmls/` directory
- User asks to process strategy HTML files
- User mentions "处理HTML策略" or "生成策略文档"
- User wants to update strategy documentation from HTML sources

---

## ⚠️ CRITICAL RULE - READ THIS FIRST

### 🚫 ABSOLUTELY NO EXTERNAL URL ACCESS

**THIS IS THE MOST IMPORTANT RULE:**

1. **NEVER access any external URLs** from HTML files
2. **NEVER use WebFetch or WebSearch tools** to access links found in HTML
3. **NEVER follow any links** in the HTML content, including:
   - Image URLs
   - CSS/JS resources
   - External references
   - Any hyperlinks
4. **ONLY process local HTML file content** - nothing else
5. **If you need to access any URL not explicitly provided by the user, STOP and ask first**

**Why this matters:**
- HTML files may contain links to external resources
- Accessing these URLs could violate security policies
- All necessary information is already in the local HTML file
- You do NOT need to fetch anything from the internet

**VIOLATION OF THIS RULE IS UNACCEPTABLE**

---

## Processing Workflow

### Step 1: Identify New HTML Files

1. Read `strategy/htmls/README.md` to check existing indexed files
2. List all HTML files in `strategy/htmls/` directory
3. Compare to identify new unprocessed HTML files
4. If no new files, inform user and exit

### Step 2: Extract Strategy Information

For each new HTML file, extract:

**Basic Information:**
- Strategy name (from title or h1 tag)
- Author name
- Source URL (from HTML comments or meta tags)

**Factor Information (CRITICAL):**
- Factor names
- Factor principles (因子原理)
- Factor logic (因子逻辑)
- Factor parameters
- How factors work together

**Strategy Overview:**
- Brief strategy description (1-2 sentences)
- Core strategy concept

**Attachment Information:**
- Look for attachment section in HTML (usually in `<div class="container-attachment">`)
- Extract file names and sizes
- **What counts as attachment:**
  - Code files: .py, .zip, .rar, .7z, .xlsx, .xls, .csv, .json, .txt, .doc, .pdf
- **What does NOT count as attachment:**
  - Image files: .png, .jpg, .jpeg, .gif, .svg, .webp
- Record: file count, file names, file sizes

### Step 3: Generate Documentation

Create structured markdown file in `strategy/docs/`:

**File naming:** Use strategy name, clean and concise
- Example: `G-alpha-70_小市值成交额缩波策略.md`

**Document structure (SIMPLIFIED):**
```markdown
# [Strategy Name]

## 策略概述
- Strategy name, author, source

## 附件信息
- If has attachments:
  - 文件个数：X个
  - 文件列表：
    1. filename1.py
    2. filename2.py
- If no attachments:
  - 无

## 一、策略思路
- Core strategy concept (brief)

## 二、策略描述
- Detailed strategy logic
- Factor explanations

## 三、策略代码
- config.py configuration
- Factor code

## 四、总结
- Strategy insights
```

### Step 4: Rename HTML File

Rename HTML file to clean, descriptive name:
- Remove task markers like 【任务】【选股策略再择时-2期-2026】
- Remove author name prefix like 【SinTimg】【kunki】
- Remove performance numbers (年化、回撤等)
- Keep core strategy information
- Example: `成交额STD_市值_均线择时策略.html`

### Step 5: Update Index File

Update `strategy/htmls/README.md`:

1. Add new entry to index table with:
   - Strategy name
   - Document path
   - HTML source file
   - Main factors (list only)
   - Creation date

2. Keep it simple - no performance data in index

### Step 6: Update Notes (MOST IMPORTANT)

Update `notes/good_factor.md`:

**Format (SIMPLIFIED - Factor Focused):**
```markdown
## 策略N：[Strategy Name]

### 策略简介
Brief strategy description (1-2 sentences)

### 附件信息
- If has attachments:
  - 文件个数：X个
  - 文件列表：filename1.py, filename2.py
- If no attachments:
  - 无

### 因子说明

**1. [Factor Name]**
- **原理**: Factor principle
- **逻辑**: Factor logic
- **参数**: Parameter (if any)

**2. [Factor Name]**
- **原理**: Factor principle
- **逻辑**: Factor logic
- **参数**: Parameter (if any)

### 文件位置
- 详细文档: strategy/docs/[strategy-name].md
- HTML源文件: strategy/htmls/[strategy-name].html
```

**IMPORTANT:**
- NO strategy parameters table
- NO performance data
- NO risk warnings
- NO improvement suggestions
- ONLY factor principles, logic, and brief strategy intro

## Important Notes

### ⚠️ HTML Parsing Guidelines (CRITICAL)

1. **🚫 ABSOLUTELY NO EXTERNAL URL ACCESS** - This is the #1 rule
   - Do NOT use WebFetch tool
   - Do NOT use WebSearch tool
   - Do NOT access any links found in HTML
   - Do NOT fetch images, CSS, JS, or any external resources
   - ONLY read and process the local HTML file content
   
2. **Only extract content** from the HTML file itself
3. **Look for content in**:
   - `<div class="thread-content">` or similar containers
   - `<article>` tags
   - `<h1>`, `<h2>`, `<h3>` headers
   - `<pre><code>` for code blocks
   - `<p>` paragraphs

### Content Extraction

1. **Strategy name**: Usually in `<title>` or first `<h1>`
2. **Author**: Look for author name in header or meta tags
3. **Content**: Extract from main content area, ignore navigation/sidebar
4. **Code**: Extract from `<pre><code>` blocks, preserve formatting
5. **Factors**: Look for factor names in strategy description and config
6. **Attachments**: 
   - Look for `<div class="container-attachment">` or similar attachment sections
   - Extract file names from `<span class="file-name">` or similar elements
   - Extract file sizes from `<span class="size">` or similar elements
   - Filter out image files (.png, .jpg, .jpeg, .gif, .svg, .webp)
   - Only count code files (.py, .zip, .rar, .7z, .xlsx, .xls, .csv, .json, .txt, .doc, .pdf)

### Factor Information Extraction

**Key sections to look for:**
- 策略思路
- 策略描述
- 因子说明
- config.py中的factor_list

**Extract for each factor:**
- Factor name
- Factor principle (why this factor works)
- Factor logic (how this factor is used)
- Factor parameters (if mentioned)

### File Management

1. **Always check** if file already exists before creating
2. **Use consistent naming** conventions
3. **Preserve original HTML** files (just rename them)
4. **Update index** after each new strategy
5. **Maintain notes** in chronological order

### Error Handling

1. If HTML parsing fails, inform user and ask for guidance
2. If required information is missing, use placeholders
3. If file already exists, ask user before overwriting
4. If multiple new files, process them one by one

## Example Usage

**User adds new HTML file:**
```
User: I added a new HTML file to strategy/htmls/
```

**Skill response:**
1. Read README.md to check existing files
2. Identify new file: `【任务】【Author】策略名称_收益_回撤.html`
3. Extract strategy and factor information from HTML
4. Create `strategy/docs/策略名称.md` (simplified version)
5. Rename HTML to `策略名称.html`
6. Update `strategy/htmls/README.md` (index only)
7. Update `notes/good_factor.md` (factor-focused summary)
8. Report completion to user

## Quality Checklist

Before completing, verify:

- [ ] **🚫 NO EXTERNAL URL ACCESS** - Confirm you did NOT access any external URLs (MOST IMPORTANT)
- [ ] **🚫 NO WebFetch/WebSearch used** - Confirm you did NOT use WebFetch or WebSearch tools
- [ ] Strategy document created (simplified version)
- [ ] HTML file renamed appropriately
- [ ] Index file updated with new entry
- [ ] Notes file updated with factor-focused summary
- [ ] Factor principles and logic captured
- [ ] Attachment information extracted and documented
- [ ] NO performance data in notes
- [ ] NO strategy parameters in notes
- [ ] File paths are correct
- [ ] Consistent formatting maintained
