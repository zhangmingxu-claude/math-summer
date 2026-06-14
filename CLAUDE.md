# 乐乐的数学冒险 — 项目上下文

## 项目概述
单文件 HTML5 PWA 数学学习平台，面向幼小衔接（5-7岁孩子）。
- **线上地址**: https://zhangmingxu-claude.github.io/math-summer/
- **GitHub**: https://github.com/zhangmingxu-claude/math-summer
- **核心文件**: `index.html`（1483行，含全部 HTML/CSS/JS）
- **技术栈**: 纯原生 JS + Web Audio API + Service Worker + 嵌入式拼音映射表

## 文件结构
```
math-summer/
  index.html       ← 唯一核心文件（修改内容只改这个）
  manifest.json     ← PWA 名称/图标
  sw.js             ← Service Worker（离线缓存）
  开发文档.txt       ← 详细开发文档
```

## 关键架构

### 页面系统
- `page-map` → 30天闯关地图（5列网格）
- `page-learn` → 学习日（讲堂→练习→擂台三阶段）
- `page-weekend` → 周末生活数学任务
- `page-badges` → 成就徽章墙
- `page-parent` → 家长面板（密码1234）

### 数据结构
- `LECTURES` 对象：22天讲堂内容（数组索引0=第1天），每天5张幻灯片
  - 每张幻灯片：`{t:标题, c:HTML正文, h:提示(可选), q:互动问题(可选), a:答案(可选), choices:选项(可选)}`
- `STUDY_DAYS` 数组：22天练习配置（每天3关，每关有type/count）
- `WEEKEND_TASKS` 数组：8个周末任务
- `STATE` 对象：运行时状态（部分持久化到 localStorage key `math_summer_v3`）

### 拼音注音系统（v3.3）
- **嵌入式映射表** `PINYIN_MAP`：792个汉字→拼音，零外部依赖
- **查表法渲染**：`processPinyin(el)` 遍历DOM文本节点，命中则包裹`<ruby>`标签
- **智能跳过**：表情/数字/英文/标点在 `PINYIN_MAP` 中无映射，自动跳过
- **缓存标记**：处理完的元素打 `data-pinyin-done` 属性，翻页不重复处理
- **开关**：学习页右上角🔤按钮，状态存 `localStorage('pinyin_enabled')`
- **CSS**：`ruby-position: over` + `-webkit-ruby-position: before`（Safari兼容）

### 题目生成
- `genQ(dayId, levelIdx)` → 根据 STUDY_DAYS 配置生成题目
- `genArenaQ(dayId)` → 生成擂台题目（关联当天知识点）
- 题目类型 type 字段决定生成逻辑（compare/pattern/wordProblem/clockReadHour等）

## 常见修改任务

### 修改讲堂内容
编辑 `LECTURES` 对象（约520行），修改 t/c/h/q/a/choices 字段。
新增汉字会自动被拼音映射表覆盖（前提是映射表里有该字）。

### 添加新题目类型
1. 在 `_genQRaw` 的 switch 中添加新 case
2. 在 `genArenaQ` 的 switch 中添加对应擂台逻辑
3. 在 `STUDY_DAYS` 中引用新 type

### 更新拼音映射表
如果新增了映射表中没有的汉字，参考 `开发文档.txt` 第四章的步骤。

### 修改 CSS 样式
所有样式在 `<style>` 标签内（约12-176行），使用 CSS 变量（`:root` 中定义）。

## 部署
```bash
git add index.html [其他文件]
git commit -m "描述改动"
git push origin master
# GitHub Pages 自动部署，约30秒生效
```
若改了 sw.js，需更新 `CACHE_NAME` 版本号。

## 状态持久化
localStorage key: `math_summer_v3`
持久化字段：currentDay, completedDays, totalStars, totalScore, dayStars, arenaTotal, badges, streakBest, wrongBook
会话字段：learnPhase, currentLevel, lectureStep, arenaActive 等（不持久化）

## 音效系统
Web Audio API，函数：sfxCorrect(), sfxWrong(), sfxLevelUp(), sfxStar(), sfxClick(), sfxComplete()
开关：页面右上角 🔊 按钮，状态存 `soundEnabled` 变量
