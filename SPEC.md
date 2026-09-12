# 高中生索引化错题/笔记/复习系统 · 规格

## 1. 项目与范围

- 项目：高中生索引化错题/笔记/复习系统。
- 当前只开发 **Android 单机 MVP**；电脑端录入程序**冻结**，仅保留规格。
- 用户：住宿高中生。平时手机轻录入，回家未来用电脑拍照完善 SJ 原题、第一次做题痕迹、BJ 页码，生成 PDF。**当前不开发电脑端**。

## 2. 术语

- SJ = 试卷
- BJ = 笔记
- BJ 单独列出，可检索，**页码必填**。

## 3. 科目

- 默认：语文、数学、英语、物理、化学、生物。
- 支持自定义，但不作为首批重点。

## 4. 技术约束

- Kotlin + Jetpack Compose + Room；单模块 MVP。
- 最低 Android 8，2-3GB 低端机流畅。
- 断网可录入/检索/复习/日程。
- **禁止**：登录云同步、强制 OCR、桌面端、重型编辑器、社交功能。
- **API Key 永不放 App**。

### 预留接口

`AiProvider`、`AsrProvider`、`SyncProvider`、`PdfExporter`、`DesktopImportExporter`。

Hilt/WorkManager/DataStore 可作为后续增强，非首版硬要求。

## 5. 数据模型

### SJ
`SJ(id, subject, title, source, examDate, totalScore, paperCode)`

`paperCode = subject-yyyyMMdd-source-seq`

### BJ
`BJ(id, subject, title, notebookCode, pageNo, tags, createdAt, photoPath?)`

### Mistake
`Mistake(id, subject, sjId?, questionNo?, bjId?, bjPageNo?, knowledgeTagIds, categoryTagIds, errorTypeRaw, errorTypeNorm, errorNoteRaw, errorNoteNorm, correctionNote?, photoPath?, difficulty, mastery0-5, status, nextReviewAt, createdAt, updatedAt, source:MANUAL/VOICE/IMPORT)`

### KnowledgeNode
`KnowledgeNode(id, subject, parentId?, level<=3, name, aliases, status:ACTIVE/PENDING/MERGED, mergedIntoId?, usageCount)`

### CategoryTag
`CategoryTag(id, name, status, mergedIntoId?, usageCount)`

### ScheduleTask
`ScheduleTask(id, title, date, startMinute?, durationMin, relatedMistakeIds, status)`

### AiPlan
`AiPlan(id, createdAt, inputSummary, resultJson, source:CLOUD/RULE, confidence)`

### VoiceCapture
`VoiceCapture(id, targetType, targetId, rawText, normText, asrSource, confidence, confirmed)`

### ExportBundle
zip 含 `bundle.json`、`images/`、`manifest.json`、`version`；当前实现导出与导入接口/校验规范；**桌面导入器不实现**。

## 6. 功能

1. **SJ/BJ 管理**：SJ 可挂整卷/页照片；BJ 页码必填；照片仅存，不 OCR。
2. **快速录入 <=20 秒**：选 SJ 或 BJ → 题号或 BJ 页码 → 知识点/分类/错因 → 一句"为什么错/下次信号"。
3. **语音**：错因/知识点/BJ 定位可语音；优先系统识别；保存 `rawText`/`normText`；清洗可撤销不覆盖原文；低置信待确认。
4. **词条**：最多 3 层；录入支持 A-Z、拼音首字母、模糊、最近、高频；新词默认 PENDING。
5. **词条治理**：专门页面；相近词融合/别名/层级建议；显示 源→目标、证据、影响条数、置信度；确认/忽略；合并重定向不删历史，可撤销。
6. **检索**：科目、SJ/BJ、知识点层级、分类、错因、掌握度、到期、是否有照片、来源；结果定位 SJ 题号/BJ 页码。
7. **复习**：今日到期；先回忆再看答案；会/模糊/不会；更新 `mastery`/`status`/`nextReviewAt`；统计重复错误率、同类题正确率、到期完成率。
8. **日程**：日/周；检索结果生成任务；完成/跳过/改期；每日上限可配。
9. **AI 漏洞**：仅经代理；输入脱敏统计/ID；输出严格 JSON：`hypothesis, evidence_mistake_ids, knowledge_path, error_pattern, next_actions, retest_after_days, estimated_minutes, confidence, insufficient_data`；无证据不下结论；每周最多 3 个漏洞，必须可回定位并给复测清单。
10. **降级**：断网/超时/欠费/非法 JSON → 本地规则生成，标 `source=RULE`，可恢复后刷新。
11. **PDF**：首版做复习报告/错题索引 PDF 接口与简单实现；完善原题 PDF 属电脑端规格。
12. **性能**：首屏不阻塞；分页；图片懒加载压缩；索引：`Mistake(subject, nextReviewAt)`、`BJ(subject, pageNo)`、`KnowledgeNode(subject, parentId, name)`。

## 7. 代理

Cloudflare Workers 或 Node/Express 二选一：

- `POST /ai/plan`
- `POST /ai/normalize`
- `POST /ai/tag-merge-suggest`
- 可选 `/asr`

OpenAI/国内模型 Provider 抽象；Key 仅服务端；token 鉴权、限流、超时、重试、日志、脱敏、schema 校验。

## 8. 桌面端冻结

只输出 `docs/desktop-spec.md`：导入 ExportBundle；回家拍照录入 SJ 原题/做题痕迹/红笔改错/BJ 页码；生成完善 PDF；OCR 可选插件；冲突按 `updatedAt+deviceId`；AI 清洗二次确认。**不实现**。

## 9. 交付顺序

1. 先文档/schema；
2. 再安卓骨架：SJ/BJ/快速录入/检索/复习/设置/导出接口；
3. 再补词条治理、代理、PDF、统计。

每次只交付当前任务 diff。

## 10. 验收

- 断网全流程可用；
- AI 失败降级；
- 录入 <=20 秒；
- 语音可撤销；
- 新词未确认不入正式树；
- 漏洞带证据定位；
- bundle 可校验；
- 低端机策略明确；
- 含迁移策略与测试清单。
