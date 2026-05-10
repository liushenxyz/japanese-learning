# CLAUDE.md — Claude Code 项目指令

## 项目简介

JLPT 日语备考词汇学习工具。用户在 Claude.ai 对话中分析试卷图片 → 提取词汇 → 下载 JSON → 导入离线 HTML 背单词。

## 文件说明

- `japanese_flashcards.html` — 主文件（在线版，含 AI 提取）
- `jp_flashcard_offline.html` — 离线背单词版（导入 JSON 使用）
- `jp_analyzer_claude.html` — Claude 端分析工具
- `README.md` — 完整项目文档

## 当前状态

- 词库：84 条（两次试卷 + 混淆辨析）
- 语法：20 条（含高频标记）
- 功能完整：卡片学习、自测模式、词库表格、语法表格、AI 提取
- 数据全部硬编码在 HTML 内（待抽离）

## 开发约定

### 不要改动的部分
- CSS 变量命名（--accent, --surface 等）
- VOCAB_RAW 和 GRAMMAR_DATA 的字段结构
- `speak()` 函数（Web Speech API 调用）
- `fixCardHeight()` 函数（卡片高度自适应逻辑）

### 代码风格
- 纯原生 JS，无框架，无构建工具
- CSS 全部写在 `<style>` 内，JS 全部写在 `<script>` 内
- 单文件原则（除非明确拆分）

### 数据字段必须完整
新增词条时以下字段必须有值：
`id, kanji, kana, roman, trans, pos, pg, level, src, count, ex_jp, ex_kana, ex_roman, ex_zh, gm`

pg 只能是：`名词 | 动词 | 形容词 | 副词 | 外来語`
level 只能是：`n5 | n4 | n3 | n2 | n1`

## 优先开发任务

1. **数据抽离**：将 VOCAB_RAW 和 GRAMMAR_DATA 移到 `data.js`，HTML 动态 fetch
2. **PWA**：`manifest.json` + `sw.js`，手机可添加主屏幕
3. **JSON 导出**：主文件加「导出词库」按钮，供离线版导入
4. **SRS 算法**：替换简单 hard/ok/easy 为艾宾浩斯间隔重复

## 本地运行

```bash
python3 -m http.server 8080
# http://localhost:8080/japanese_flashcards.html
```

AI 功能（独立部署）需在 `runAI()` 的 fetch headers 加：
```js
'x-api-key': process.env.ANTHROPIC_API_KEY,
'anthropic-version': '2023-06-01',
'anthropic-dangerous-direct-browser-access': 'true'
```
