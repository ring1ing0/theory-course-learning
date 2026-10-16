# 进度与证据记录

在首次建档、写入或恢复有歧义时读取；同一上下文内复用。记录放课程工作区，沿用现有布局，按需创建：

- `progress.json`：课程约束、来源范围、当前模块/目标摘要、待答题、复习队列、下一步。
- `knowledge/attempts.jsonl`：真实作答证据，按行追加；量少时可沿用既有记录文件。
- `knowledge/<章节>.md`：仅需长期保留的定义/条件、推导骨架、实质误区及来源。
- `lessons/<模块>.md`：需要保存时才建覆盖提纲或讲义，不与当前待答题一起展示答案。

## 进度结构

继续使用 `schemaVersion: 2`，兼容旧记录；新增字段按需添加，缺失不推测。

| 字段 | 内容 |
|---|---|
| `courseName`, `courseMode` | 课程名；`mechanism` / `formal_theory` / `exam_driven` / `mixed` |
| `examDate`, `constraints` | 已知日期与时间约束，未知用 `null` |
| `sources` | 来源 ID、路径、实际核验范围 |
| `weeklyPlan` | 有需要才保留计划，不每轮重写 |
| `modules` | ID、标题、基线、目标状态、教学状态及相关证据 ID |
| `cursor` | `moduleId`, `unitId`, `nextAction` |
| `pendingQuestions` | 待答题 ID、题文/稳定定位、目标、题源、阶段、提示及答案展示状态 |
| `weakPoints`, `misconceptions` | 仅需跟踪的实质问题及证据，缺省为空 |
| `reviewQueue` | 有复习理由的目标，不自动收录所有知识或错误 |
| `lastUpdated` | 实际带时区日期时间 |

模块 `baseline` 保留 `sourceRefs`, `prerequisites`, `objectives`, `evidenceRequired`。按需新增 `objectiveStates`：每项含 `id`, `objective`, `importance`（key/supporting）、`importanceReason`, `exposure`（unknown/taught/verified_prior）、`mastery`, `evidenceIds`。`taught` 只证明讲过；`verified_prior` 需来源或既往证据；未知不能当已教。题目的 `requiredKnowledge` 关联目标或已核验前置，发题前核对。

仅在需要区分范围时增加：`completionScope`（实际完成范围）、`remainingCoverage`（尚未讲/未核验范围）、目标的 `evidenceScope`（证据能支持到哪一步）及 `transferStatus`（条件变化/综合迁移状态）。可用 `targetAlignment` 关联代表题、核验状态、目标要求和验收证据。沿用现有结构，不为新增字段重建整份记录。

若教学衔接不足需要后续补讲，可在当前模块增加 `teachingFollowUp`：`reason`、`remainingCoverage`、`nextDemonstration`、`status`，分别说明依据、未完整呈现的教学内容、下一次示范及待办状态。它记录教师后续动作，与学生的 `mastery` 分开；只有实际补讲后才更新完成情况，维护文件本身不算完成教学或能力修复。跨会话已确认的讲授偏好可在课程级 `teachingApproach` 保存简短摘要并引用现有教学计划，避免复制整份 skill。

阶段 `phase` 取 `red/green/refactor/review`，表示当前教学动作，可跳过不需要的阶段。`teachingStatus` 取 `pending/in_progress/completed/paused`，与掌握度独立。

`mastery` 取 `unverified/needs_repair/provisional/delayed_verified`。优先维护目标状态，模块字段只总结关键目标；支持性细节或孤立笔误不自动令模块退级。关键目标有实质待修复项时标 `needs_repair`；无待修复项但关键覆盖不足则 `unverified`；全部关键目标初步通过才 `provisional`；全部关键目标另时段独立通过才 `delayed_verified`。概念选择题不支持独立证明/构造能力升级。复习失败仅更新受影响目标，保留其他证据。

`timedPerformance` 默认 `unverified`，有实际限时证据后保存结果与证据 ID；不从消息间隔推算用时。

课程估时与考试限时表现分开。需要时长规划时按 [pacing.md](pacing.md) 保存时间口径、节点估计/实际用时、余量和大节点 `redStatus`；不为普通单题强制建计时记录。恢复已完成或跳过 RED 的大节点时直接承接教学。

## 作答证据

每条真实作答保存以下信息，未知用 `null`；长回答可引用本地记录：

- `id`, `questionId`, `moduleId`, `phase`, `at`：稳定 ID、归属、阶段和实际带时区时间。
- `question`, `sourceRef`, `objective`, `response`：可恢复题目、来源、目标和用户原答，选项字母也可作为原答。
- `verdict`：`correct/partial/incorrect/unclear`；未答保持待答，不记错误。
- `reason`, `errorType`, `impact`：判断依据、错因；影响取 `slip/local/core`，不确定可空，不因错一次就推定系统性问题。
- `hintLevel`, `solutionShown`, `independent`：提示级别 0 无、1 方向、2 关键步骤、3 完整解答；只有未获帮助且未见本题解答才可独立。判断独立不等于判断证据充分。
- `verification`：`observed` 为具体作答或可信结果；`self_reported` 为自述，不据此升级掌握度。
- `elapsedSeconds`, `repairOf`：实际测得/用户明确报告的时间；关联被修复错误 ID，否则空。

可选 `taskDemand`（recognition/direct_application/condition_change/integration/construction）、`scaffolding`（none/analogous_example/steps_provided）说明认知要求及支架。`independent` 表示是否获得本题提示/答案，不能单独证明迁移。若展示了针对本题的步骤或关键中间结果，应如实记提示；先前讲过通用原理不自动算本题提示。维护时修订旧结论需说明理由，保留原答与既有证据，不伪造新的尝试。

同一提交用稳定 ID 去重，重复提交不产生第二份独立证据。小错可留在作答记录中，但不自动建立误区、修复任务或复习项。

跟踪的弱项/误区含 `id`, `moduleId`, `objectiveId`, `issue`, `status`（suspected/confirmed/repaired）、`evidenceIds`。标记修复需新的独立证据；旧 `recheckPassed` 不能单独证明。看过答案后仍未验证的关键问题保留待修复，但允许继续新课。

`repaired` 必须限定范围，必要时加 `repairScope` 和 `transferStatus`。一个小例子纠正了局部关系，不代表所有容量题、证明或综合题都已修复；不能因新的目标更难而抹掉局部正确证据。

## 恢复与复习

读取顺序：`cursor` → `pendingQuestions` → 当前目标/教学基线摘要 → 少量到期项。仅对判断有疑问的目标查最近相关证据；不要整份读入作答日志、所有章节和讲义。

分批回答只移走已处理题目；跳过留简短备注，不补造答案。切换内容保留原断点。得到作答、改变阶段或暂停时更新最小必要记录，合并同轮写入，简短告知结果与下一步。

复习项含 `id`, `moduleId`, `objectives`, `reason`（关键节点/实质弱项/反复错误/用户指定及具体依据）、`dueDate`, `intervalDays`, `lastEvidenceId`, `status`（pending/done）。没有重要性或实际弱项理由的细节不单独排期。

间隔从实际练习日计算；复习时生成未见任务，有真实结果才关闭当前项并决定是否安排下一项。独立成功可拉长，核心理解失败需修复并缩短；孤立笔误不自动缩短。以课程本地日期显示；队列不提供离线定时通知。旧复习项缺少理由时结合已有证据判断，无依据的保留待核验，不一律当核心弱项。

## 写入与兼容

1. 更新前读取现有记录，保留未知字段和路径。已有文件没有 `schemaVersion` 时先备份，保留旧计划和完成历史；缺少证据的掌握状态注明“历史记录未包含证据”，不否定实际学习经历。旧版没有目标级记录时按需补充，不重做全课诊断。
2. 先保存证据，再写进度临时文件；解析 JSON 并检查题目/模块/证据引用后替换正式文件。使用稳定 ID 防止重试重复；并发时检查原文件是否变化，合并后再保存。
3. 损坏或失败时保留原文件与可恢复数据，明确保存/恢复范围，必要时给续学摘要；不虚报成功或悄悄重置。继续当前教学，不为补档强制用户重做已完成题目。

4. 周次安排指定 `planAuthority`；当前断点、待答题和作答证据以最新进度为准。并行窗口更新时只合并授权字段，保留新问题和新尝试；交接摘要的待答数量从最新进度生成，不沿用过期值。历史备份不是当前计划来源。
