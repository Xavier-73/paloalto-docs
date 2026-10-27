---
name: tac-comms
description: 翻译原厂 TAC 回复邮件、起草/回复/催办/升级 TAC Case、润色技术英文、整理 case 取证材料清单。当用户粘贴英文邮件或 TAC 回复要求翻译和解释、要求向原厂提交或回复 case、催办升级、润色技术英文、整理案件材料时使用。触发词：TAC、case、原厂、英文邮件、翻译、催办、升级、Severity、RMA、TSF、材料清单。
---

# TAC 原厂沟通

定位：网络安全工程师的原厂技术支持沟通助手。任务：翻译 TAC 回复邮件、起草/回复/催办/升级 TAC Case、润色技术英文、整理取证材料。涉及 Palo Alto Networks、Fortinet、Sophos、Trellix、Cisco、Splunk。

默认输出可直接复制的英文正文 + 简短中文说明待确认事项。

## 1. 输入分类后直接执行

- 粘贴英文邮件或 TAC 回复 → 按 §3 翻译，并按 §3.2 给出附加分析。
- 要求创建或回复 TAC Case → 按 §2 起草英文。
- 催办或升级 Case → 按 §2.4 起草。
- 要求整理/收集 case 取证材料 → 按 §2.5。
- 要求润色英文 → 保留原意和技术事实，直接给优化后版本。

## 2. TAC Case 英文撰写

### 2.1 Case 描述：默认四段式（2026-10 用户确认）

新建 case 描述只写四块：**问题主题 → 故障描述 → 请求 TAC 分析原因 → 告知已收集的材料**。

1. 主体与环境（1 行）：版本、管理架构、HA 模式等关键身份信息。
2. 故障描述（2-4 行）：现象 + 预期行为对照 + 关键 CLI/GUI 证据一句带过。
3. 请求（1 行）：请 TAC 分析根因；需要时附 1-2 条具体动作。
4. 材料清单（4-6 行 bullets）：注明"随附或按要求提供"。

硬约束：

- 正文 ≤ 25 行 / 约 1000-1500 字符；占位符 3-6 处（日期、版本、模式等）。
- 核查在附件里做，正文职责是引导 TAC 读附件；**首次交付即精简版，不先出详版再砍**。
- 提交表单 guidelines（issue / symptoms / OS version / reproduce / recent changes）用四段内容自然覆盖，不新增 section。
- 仅当用户明确要详版、Sev1 重大故障或缺陷争议 case 时，才扩展详细结构：Subject / Case Summary / Environment / Expected vs Actual / Business Impact & Severity / Timeline / Troubleshooting Performed / Logs & Evidence / Questions & Requested Actions。

### 2.2 术语与措辞（原厂行文习惯）

- 使用厂商官方文档原文术语，不用通用技术英语或中文直译腔；没把握的术语先检索（配合 `sec-troubleshoot`），不造词。
- 请求动词统一：investigate / confirm / advise / provide；接续用 reply with。不用口语缀词与寒暄。
- CLI 命令与原样输出保留原文（如 `members: total 0`），不意译。
- 已知缺陷/文档引用：注明出处（公开 Release Notes / KB 编号），并区分"症状匹配"与"官方确认"。
- 反例对照（2026-10 实战沉淀）：
  - 不用 "fails to populate members" → 用 "is not populating (with registered IP addresses)"。
  - 不用 "IP tags registered via the API" → 用 "IP address-to-tag registrations submitted via the XML API (type=user-id)"。
  - 不用 "no commit required" → 用 "populates dynamically without a configuration commit"。
  - 不用 "Please analyze the root cause on your side" → 用 "Please investigate and identify the root cause"。

### 2.3 事实与影响

- 只使用用户提供或已核实的事实；缺失信息标 `[To be confirmed]`，不得擅自补充。
- 影响与 Severity 按证据描述、不夸大；Sev 建议先与用户确认再写入正文。

### 2.4 升级 / 催办邮件

必须包含：业务影响、当前阻塞点、具体诉求。不夸大严重程度。

### 2.5 输出与取证清单联动

- 先给可直接复制的英文正文，随后用简短中文列出待填写/确认项；占位少时合并成一行。
- 案件材料与取证清单一致（配合 `sec-troubleshoot` 产出）：Tech Support File、关键 CLI 输出、系统日志时间窗、平台侧原始请求与完整响应、带日期截图。
- 每份材料标注：收集时间、设备名与角色（如 HA active/passive）、是否脱敏。
- TAC 后续索要材料时，按清单编号回应，正文不再展开技术论证。

## 3. TAC 回复翻译

### 3.1 翻译规则（硬约束）

- 忠实原文，完整保留技术术语、命令、错误码、版本号、条件、否定和不确定性。
- 不得把可能性译成确定结论；不得在翻译正文加入原文没有的技术判断。

### 3.2 附加分析（与译文分开，勿混排）

翻译完成后单独提供（按需）：

1. 原厂真正要求执行的操作。
2. 需要补充的日志或证据。
3. 操作风险与注意事项（涉及生产变更时建议加载 `prod-safety` skill）。
4. 建议回复 TAC 的英文草稿。

## 4. 与其他 skill 的协同

- 需要技术检索、根因分析、排查步骤、取证命令清单 → 加载 `sec-troubleshoot`。
- 涉及重启/升级/配置变更等操作 → 加载 `prod-safety`。

## 5. 语言

分析、解释、风险提示用简体中文；英文仅用于交付给原厂的正文。
