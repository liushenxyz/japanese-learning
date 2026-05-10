# 日語単語帳 — 项目交接文档

## 项目概述

这是一个 **JLPT 日语备考词汇学习工具**，由三个独立 HTML 文件构成，无需构建工具，纯原生 JS/HTML/CSS。

核心流程：
```
Claude 对话分析试卷图片
    ↓ 下载 JSON 词库
离线背单词 HTML（本地运行）
```

---

## 文件清单

| 文件 | 用途 | 运行环境 |
|------|------|----------|
| `japanese_flashcards.html` | **主文件**：在线版，含 AI 提取、卡片学习、词库浏览、语法重点四个模块 | Claude.ai 环境（API 已注入） |
| `jp_analyzer_claude.html` | **分析工具**：上传试卷图片 → AI 解析 → 选词 → 下载 JSON | Claude.ai 环境 |
| `jp_flashcard_offline.html` | **离线版**：导入 JSON 词库 → 本地背单词，完全离线 | 本地浏览器 / 手机 |

---

## 主文件（japanese_flashcards.html）架构

### Tab 结构
```
topbar（词库统计）
subtabs：
  ├── 🃏 卡片学习   (tab-study)
  ├── 📚 词库浏览   (tab-list)
  ├── 📖 语法重点   (tab-grammar)
  └── 📷 AI 提取   (tab-upload)
```

### 数据结构

#### VOCAB_RAW — 词库数组（当前 84 条）
```js
{
  id: 1,
  kanji: '情景',           // 汉字
  kana: 'じょうけい',      // 假名
  roman: 'jōkei',          // 罗马音
  trans: '情景、景象',      // 中文释义
  pos: '名词',              // 词性（详细）
  pg: '名词',               // 词性分组（用于筛选：名词|动词|形容词|副词|外来語）
  level: 'n3',              // JLPT 级别：n5/n4/n3/n2/n1
  src: '問題1',             // 来源（第几题）
  count: 1,                 // 出现次数（跨试卷累计）
  ex_jp: '美しい情景が目に浮かんだ。',   // 例句日语
  ex_kana: 'うつくしい じょうけい...',   // 例句假名
  ex_roman: 'utsukushii jōkei...',       // 例句罗马音
  ex_zh: '美丽的情景浮现在眼前。',        // 例句中文
  gm: '<span class="grammar-tag n3">N3</span> 〜が目に浮かぶ...'  // 语法说明（含HTML标签）
}
```

#### GRAMMAR_DATA — 语法数组（当前 20 条）
```js
{
  id: 'g1',
  sentence: '今ごろになって焦ってもしかたない。',  // 原句
  kana: 'いまごろ に なって...',                   // 原句假名
  trans: '都到这个时候了，着急也没用。',            // 中文翻译
  point: '〜てもしかたない：即使……也没用，表无奈',  // 语法要点
  level: 'n3',                                     // 级别
  freq: 1,                                          // 出现频率（>=2 显示高频标记）
  ex_jp: '今ごろになって焦ってもしかたない。'        // 用于发音的句子
}
```

### 关键 CSS 变量
```css
--bg: #f4f2ed;           /* 页面背景 */
--surface: #ffffff;       /* 卡片白色 */
--surface2: #f0ede7;      /* 次级背景 */
--accent: #2d5a3d;        /* 主题绿色 */
--accent-light: #e7f2eb;  /* 绿色浅色 */
--text: #1c1917;          /* 主文字 */
--text2: #6b6560;         /* 次级文字 */
--text3: #a09b94;         /* 提示文字 */
--red: #8b3a2a;           /* 错误/危险 */
--gold: #7a5e18;          /* 高频标记 */
--gold-light: #fdf7e4;
```

### 等级色系
```js
const lvStyle = {
  n5: 'background:#fdf7e4;color:#7a5e18',
  n4: 'background:#e6f0fa;color:#1c4478',
  n3: 'background:#e7f2eb;color:#2d5a3d',
  n2: 'background:#fdf0ec;color:#8b3a2a',
  n1: 'background:#f3edf8;color:#5a2d7a'
};
```

### 关键函数索引
```
buildDeck()         词性筛选后重建卡片队列
renderCard()        渲染当前卡片（正反面）
fixCardHeight()     动态调整卡片高度（内容自适应）
renderList()        渲染词库表格（含出现次数、发音、例句）
renderGrammar()     渲染语法表格（含高频标记）
speak(text, btn)    调用 Web Speech API 朗读日语
renderQuiz(w)       自测模式：生成4选1选项（显示译文）
checkQ(chosen,cid)  自测答案判断
rate(level)         掌握程度评分 + localStorage 持久化
runAI()             调用 Anthropic API 解析图片（仅 Claude 环境）
```

### localStorage 键名
```
jp_mastered    已掌握词汇 ID 数组（JSON）
```

---

## 数据来源（已处理的试卷）

| 试卷 | ID 范围 | 词汇数 | 语法数 |
|------|---------|--------|--------|
| 第1回（言語知識）| 1–34 | 34 | g1–g10（10条） |
| 混淆辨析词 | 19–34 | 16 | — |
| 第2回（言語知識）| 35–84 | 30 | g11–g20（10条） |
| **合计** | — | **84** | **20** |

### 高频语法（freq >= 2，跨试卷重复出现）
- `〜わけではない`（N3）— 出现2次，g4 + g14

---

## 设计规范

### 字体
- 汉字/界面：`Noto Sans JP 400`（无衬线，不加粗）
- 假名：`Noto Sans JP 500`，绿色 `#3d8b60`
- 罗马音：`DM Mono italic 300`
- 界面文字：`Noto Sans JP 400/500`

### 卡片正面布局
```
[来源标注]    [词性徽章]
     [汉字 72px 400]
   [假名 22px 绿色]
  [罗马音 斜体灰色]
      [🔊 发音]
```

### 卡片背面布局（白色，与正面一致）
```
[释义 + 词性]    [🔊]
─────────────────────
例句块（浅灰底）：
  原句（绿色）
  假名
  罗马音
  [🔊 朗读例句]  中文译文
─────────────────────
语法块（淡绿底）：
  出题语法说明
```

---

## 待开发功能（Claude Code 接手）

### 高优先级
- [ ] **数据持久化**：将 VOCAB_RAW 和 GRAMMAR_DATA 抽离为独立 `data.js`，支持动态加载
- [ ] **多套试卷管理**：按试卷分组，支持单独练习某套试卷
- [ ] **学习进度持久化**：localStorage 保存每张卡的评分记录（hard/ok/easy）
- [ ] **离线版同步**：`jp_flashcard_offline.html` 支持直接导入 `japanese_flashcards.html` 的完整 JSON 导出
- [ ] **语法表「跳转」**：点击语法行中的词汇，跳转到对应卡片

### 中优先级
- [ ] **PWA 化**：添加 `manifest.json` + Service Worker，支持手机端添加到主屏幕
- [ ] **间隔重复算法（SRS）**：基于艾宾浩斯，替换当前简单的 hard/ok/easy 逻辑
- [ ] **词库导出为 JSON**：从主文件导出当前全部词库，供离线版导入
- [ ] **count 字段跨次自动累计**：新词加入时若已存在则 +1，而非重复添加

### 低优先级
- [ ] **暗色模式**：CSS 变量已预留结构，需补充 `@media (prefers-color-scheme: dark)`
- [ ] **语法卡片化**：语法重点也做成可翻转卡片形式学习
- [ ] **统计页**：学习热力图、各 N 级掌握率、高频错题

---

## API 调用说明

主文件 `runAI()` 调用方式（Claude 环境无需 key）：
```js
fetch('https://api.anthropic.com/v1/messages', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({
    model: 'claude-sonnet-4-20250514',
    max_tokens: 4096,
    system: systemPrompt,
    messages: [{ role: 'user', content: [...] }]
  })
})
```

独立部署时需额外加：
```js
headers: {
  'Content-Type': 'application/json',
  'x-api-key': 'YOUR_API_KEY',
  'anthropic-version': '2023-06-01',
  'anthropic-dangerous-direct-browser-access': 'true'
}
```

---

## JSON 词库格式（导入/导出）

`jp_analyzer_claude.html` 导出格式，`jp_flashcard_offline.html` 可直接导入：
```json
{
  "vocab": [ ...VOCAB_RAW 数组元素... ],
  "exportedAt": "2025-05-10T03:00:00.000Z",
  "source": "Claude分析",
  "total": 30
}
```

离线版同时支持直接传入裸数组 `[...]`。

---

## 快速启动（Claude Code）

```bash
# 本地预览（无需安装依赖）
npx serve .
# 或
python3 -m http.server 8080
# 访问 http://localhost:8080/japanese_flashcards.html
```

若需独立部署并使用 AI 功能：
1. 在 HTML 的 `runAI()` 函数 headers 中加入 `x-api-key`
2. 或搭建简单 Node.js 代理转发请求（避免 key 暴露在前端）
