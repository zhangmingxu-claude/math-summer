# 乐乐的数学冒险 — 项目上下文

## 项目概述
单文件 HTML5 PWA 数学学习平台，面向幼小衔接（5-7岁孩子）。
- **线上地址**: https://zhangmingxu-claude.github.io/math-summer/
- **GitHub**: https://github.com/zhangmingxu-claude/math-summer
- **核心文件**: `index.html`（~1700行，含全部 HTML/CSS/JS）
- **当前版本**: v3.7
- **技术栈**: 纯原生 JS + Web Audio API + Service Worker + 嵌入式拼音映射表

## 文件结构
```
math-summer/
  index.html       ← 唯一核心文件（修改内容只改这个）
  manifest.json     ← PWA 名称/图标（乐乐的数学冒险）
  sw.js             ← Service Worker（离线缓存）
  CLAUDE.md         ← 本文件（新对话自动恢复上下文）
  开发文档.txt       ← 详细开发文档
```

## 关键架构

### 页面系统
- `page-map` → 30天闯关地图（5列网格）
- `page-learn` → 学习日（讲堂→练习→擂台）
- `page-weekend` → 周末生活数学任务（子任务勾选+家长评星+自理加分）
- `page-badges` → 成就徽章墙
- `page-parent` → 家长面板（密码1234，含学习报告+护眼/拼音设置）

### 学习日三阶段
```
📖 小老师讲堂（翻页浏览）
  └─ 最后页点"开始闯关"
🎯 闯关练习（numPadPractice 内置键盘）
  └─ 三关完成
⚡ 趣味擂台（numPadArena 内置键盘，始终可进入）
  └─ 时间到 → 错题回顾 → 完成
```

### 数据结构
- `LECTURES` 对象：22天讲堂内容，每天5张幻灯片
  - 格式：`{t:标题, c:HTML正文, h:提示(可选), q:互动问题(可选), a:答案, choices:选项(可选)}`
- `STUDY_DAYS` 数组：22天练习配置（每天3关，每关有type/count/hint）
- `WEEKEND_TASKS` 数组：8个周末任务（含bonusTask自理任务+bonusStars奖励）
- `STATE` 对象：运行时状态（部分持久化到 localStorage key `math_summer_v3`）
- `SCHOOL_TIPS` 数组：30条小学小知识彩蛋
- `TUTOR_TIPS` 对象：28个知识点的家长辅导建议

### 拼音注音系统
- **嵌入式映射表** `PINYIN_MAP`：792个汉字→拼音，在`<head>`内
- **查表法渲染**：`processPinyin(el)` 遍历DOM文本节点查询映射表
- **缓存标记**：`data-pinyin-done` 属性，翻页不重复处理
- **智能模式**：学习日自动开启，周末自动关闭（家长可设置）
- **开关**：右上角🔤按钮，状态存 `localStorage('pinyin_enabled')`

### 内置数字键盘（双键盘体系）
- `numPadPractice` — 练习区键盘（在 practiceArea 内）
- `numPadArena` — 擂台区键盘（在 arenaCard 内）
- 两个键盘各自独立，不存在 DOM 移动，互不干扰
- **状态即DOM**：`_padSuffix()` 直接检查 `numPadArena` 是否可见，不维护变量
- `showNumPad(target)` — 切换显示/隐藏对应键盘
- `numPadInput/Delete/Confirm` — 通过 `_padSuffix()` 自动识别当前活跃键盘
- 最大2位输入限制 + 单键删除 + 防误触设计

### 擂台激活
- **分离设计**：`activateArena()` 只激活 / `submitArena()` 只提交
- 按钮始终可用，点击激活擂台（disabled 仅在擂台进行中）
- 擂台结束自动恢复按钮状态

### 题目生成
- `genQ(dayId, levelIdx)` → 根据 STUDY_DAYS 配置生成题目
- `genArenaQ(dayId)` → 生成擂台题目
- 题型通过 type 字段决定（compare/pattern/wordProblem/clockReadHour 等）
- 答错算理引导：第一次错→显示 hint + 重试，第二次错→显示答案
- 错题间隔复习：2天/5天/10天三级节点

### 休息提醒
- 25分钟全局计时器（不随页面切换重置）
- 5分钟强制休息锁定期
- 随机休息建议（跳绳/看绿植/喝水/伸懒腰等）

### 家长面板
- 学习报告：知识点彩色掌握条 + 薄弱项辅导建议 + 最近记录
- 偏好设置：护眼模式、拼音智能模式
- 管理功能：解锁天数、恢复进度、重置、导出JSON
- 错题本：按天分组 + 复习次数标记

## 常见修改任务

### 修改讲堂内容
编辑 `LECTURES` 对象，修改 t/c/h/q/a/choices 字段。

### 添加新题目类型
1. 在 `_genQRaw` 的 switch 中添加新 case
2. 在 `genArenaQ` 的 switch 中添加对应擂台逻辑
3. 在 `STUDY_DAYS` 中引用新 type

### 更新拼音映射表
参考 `开发文档.txt` 第四章。

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
持久化字段：currentDay, completedDays, totalStars, totalScore, dayStars, arenaTotal, badges, streakBest, wrongBook, learningLog
会话字段：learnPhase, currentLevel, lectureStep, arenaActive 等
