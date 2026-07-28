---
name: app-regulatory-remediation
description: Use when handling App regulatory reports, privacy compliance inspections, app-store review feedback, SDK or permission audit findings, or security remediation notices. Guides an App developer through report interpretation, engineering verification, missing-info questions, remediation planning, validation evidence, release gates, and a formal整改报告 for Android, iOS, HarmonyOS, Flutter, React Native, H5/WebView, or mini-program projects.
---

# App Regulatory Remediation Skill

## Purpose

Use this skill when a developer receives a regulator report, privacy compliance inspection, app-store review feedback, SDK/data-collection warning, permission audit, or security remediation notice and needs a practical remediation report.

The agent's job is to convert unclear compliance language into:

- a developer-readable issue list
- verifiable engineering facts
- targeted follow-up questions
- a minimal remediation plan
- validation and evidence requirements
- a formal Markdown remediation report

Do not assume the user understands regulatory details. Read the report first, explain it plainly, then inspect engineering artifacts.

## Core Rules

- Base conclusions on supplied reports, code, built artifacts, screenshots, or explicit user confirmation.
- Mark uncertain items as `待确认`; never turn them into confirmed facts.
- Do not invent completed tests, screenshots, build numbers, legal review, publication status, or regulator acceptance.
- Prefer final release artifacts over source files when available.
- Keep changes or recommendations minimal and traceable to the report.
- Separate platform, channel, region, version, and language scope.
- If code can answer a question, inspect code instead of asking the user.
- If business/legal judgment is required, ask the right owner instead of guessing.

## Inputs To Request

If not already provided, ask for the smallest missing set:

- App name, package name / bundle ID, current version, target version.
- Affected platforms: Android, iOS, HarmonyOS, Flutter, React Native, H5/WebView, mini program.
- Affected regions and release channels.
- Regulator report, review feedback, email, screenshot, PDF, Word template, or ticket.
- Project root or key files.
- Current privacy policy, app-store text, in-app permission disclosure, or consent copy.
- Expected deadline and evidence format.

Do not block on all inputs. Start with the report and project path if those are available.

## Workflow

### 1. Read And Translate The Report

Extract only from the report at this stage:

- inspection object: app, platform, version, package/bundle ID, region, agency, case ID
- all inspection items
- result for each item: compliant, non-compliant, warning, recommendation, unknown
- regulator wording and developer-readable meaning
- mentioned permissions, SDKs, APIs, pages, personal data fields, flows, or store listing items
- required deadline and evidence submission format

Output a table:

| 编号 | 监管问题 | 结论 | 开发视角解释 | 监管要求 | 证据来源 |
|---|---|---|---|---|---|

Stop and ask if the report cannot be read or if pages are missing.

### 2. Convert Report Items To Engineering Checks

Map regulatory issues to concrete checks:

| 监管表达 | 工程核对方向 |
|---|---|
| 权限未完整披露 | Compare manifest/plist/entitlements, runtime requests, store text, privacy policy, and system settings. |
| 未区分必需/可选权限 | For each permission, identify purpose, request timing, denial impact, and required/optional classification. |
| 拒绝权限后服务不可用 | Test startup, login, homepage, and feature-level denial paths. |
| 未取得用户同意 | Check privacy dialog, permission prompt, SDK initialization timing, and consent storage. |
| 过度收集个人信息 | Trace forms, APIs, analytics, SDKs, logs, and local storage. |
| 撤回同意不足 | Check system settings, in-app toggles, account deletion, opt-out, and data export/deletion paths. |
| 第三方 SDK 风险 | Check SDK list, initialization timing, permissions, network calls, privacy policy disclosure, and opt-out. |
| 商店公示不一致 | Compare app-store listing, privacy labels/data safety forms, screenshots, and final app behavior. |

### 3. Inspect Engineering Artifacts

Use fast local search first. Prefer `rg` and built outputs when available.

Android checks:

- `AndroidManifest.xml`, flavor/channel manifests, library manifests.
- merged manifest and manifest merger report.
- Gradle target SDK, product flavors, app IDs.
- runtime permission request call sites.
- permission denial and permanent denial handling.
- app-store text and privacy policy references.

Typical search patterns:

```text
uses-permission|requestPermissions|XXPermissions|Permission\.|POST_NOTIFICATIONS|READ_MEDIA|ACCESS_FINE_LOCATION|READ_PHONE_STATE|READ_CONTACTS|CAMERA|RECORD_AUDIO
```

iOS checks:

- `Info.plist`, `InfoPlist.strings`, entitlements.
- Archive/final plist when available.
- permission request APIs and permission_handler macros.
- ATT, notifications, location, camera, photo, microphone, contacts, Bluetooth, local network, Face ID.
- localized purpose strings.

Typical search patterns:

```text
NS.*UsageDescription|NSUserTrackingUsageDescription|InfoPlist.strings|requestAuthorization|Permission\.|CLLocation|PHPhotoLibrary|AVCapture|CNContact|CBManager|UNUserNotificationCenter
```

HarmonyOS checks:

- `module.json5`, `oh-package.json5`, permission declarations.
- ArkTS/ArkUI permission request calls.
- privacy statement and app-market declarations.
- build profile and release artifact configuration.

Hybrid checks:

- Flutter `permission_handler`, platform channels, native bridge handlers.
- React Native permission modules and native modules.
- H5/WebView JS bridge permission calls.
- third-party SDK initialization before consent.

Output engineering findings as:

| 项目 | 当前状态 | 证据 | 风险 | 建议 | 状态 |
|---|---|---|---|---|---|

Evidence should include file paths, line numbers, built artifact paths, screenshots, or exact report pages when possible.

### 4. Ask Missing Questions By Owner

Ask only questions that cannot be answered from the report or code.

Group questions:

- 产品: basic service definition, feature importance, feature fallback, region/channel scope.
- Android: final manifest, flavor rules, runtime request timing, denial handling.
- iOS: plist/localization/entitlements, permission_handler macros, notification/ATT/location timing.
- HarmonyOS or other platforms: platform permission declaration and runtime behavior.
- 测试: device matrix, OS versions, denial/revoke/permanent denial cases, evidence capture.
- 运营/商店: app-store languages, privacy labels/data safety forms, publication screenshots.
- 法务/合规: required vs optional classification, disclosure wording, regulator template wording.

For each question, include why it matters.

### 5. Build The Remediation Plan

Use priority levels:

- `P0`: blocks regulator closure, store approval, release, or user basic service availability.
- `P1`: important compliance hardening but can follow P0 if explicitly accepted.
- `P2`: privacy governance, cleanup, monitoring, or follow-up improvements.

For each remediation item, include:

- problem source
- target platform/channel/version
- owner
- minimal change
- acceptance criteria
- evidence to collect
- status: 未开始 / 进行中 / 已完成 / 待确认 / 延期

Do not recommend broad refactors unless the report directly requires them.

### 6. Define Validation And Evidence

Validation must prove that the final app matches disclosures and remains usable where required.

Common validation cases:

- fresh install and first launch
- privacy consent accepted/refused where applicable
- deny each runtime permission
- permanently deny / do not ask again
- revoke permission from system settings
- enter each feature that uses the permission
- login and homepage availability
- SDK initialization before/after consent
- store text, privacy policy, in-app prompt consistency
- final package artifacts match source expectations

Evidence should record:

- app version and build number
- package/bundle ID and channel
- device model and OS version
- test account/region where safe to record
- date and tester
- screenshots or videos before and after remediation
- final manifest/plist/entitlements/exported permission list
- app-store/backend publication screenshots

### 7. Generate The Formal Report

Use this structure unless the regulator template requires another format:

```markdown
# App 整改实施方案

> 文档版本：
> App 版本/构建号：
> 涉及平台：
> 涉及地区/渠道：
> 计划完成日期：
> 监管/审核编号：

# 1. 整改背景与范围
## 1.1 背景
## 1.2 范围
## 1.3 原则

# 2. 监管检查结论
| 编号 | 检查项 | 监管结论 | 监管要求 | 本次处理方式 |
|---|---|---|---|---|

# 3. 工程核对结论
## 3.1 Android
## 3.2 iOS
## 3.3 HarmonyOS/其他平台（如适用）
## 3.4 待确认事项

# 4. 整改方案
| 问题 | 优先级 | 整改措施 | 责任方 | 交付物 | 验收标准 |
|---|---|---|---|---|---|

# 5. 执行计划与职责
| 编号 | 优先级 | 阶段 | 责任方 | 产出与完成条件 | 状态 |
|---|---|---|---|---|---|

# 6. 验证与整改证据
## 6.1 正式包核对
## 6.2 功能和拒绝/撤回测试
## 6.3 商店/隐私政策/弹窗取证

# 7. 发布门禁
- [ ] 所有 P0 项关闭或有书面延期结论
- [ ] 最终包声明、实际调用、用户告知一致
- [ ] 未确认事项未写入对外最终承诺
- [ ] 测试和证据与同一版本对应
- [ ] 产品、测试、运营、法务完成各自确认

# 8. 风险与限制

# 附录 A：权限 / SDK / 数据项基线
| 平台 | 项目 | Key/API/SDK | 用途 | 申请/触发时机 | 拒绝/关闭影响 | 分类 | 处理建议 |
|---|---|---|---|---|---|---|---|
```

### 8. Final Quality Gate

Before returning the final answer or report, verify:

- Every regulator issue maps to at least one remediation or explicit non-applicability reason.
- Every engineering conclusion has evidence or is marked `待确认`.
- No fake completion language appears.
- Platform/channel/region/version scopes are not mixed.
- The recommended changes are minimal and traceable.
- The evidence plan can prove the final claims.
- The report can be handed to product, engineering, QA, operations, and legal without hidden assumptions.

## Response Style

When working interactively with a developer:

- Start with report interpretation before remediation.
- Explain compliance terms in developer language.
- Show concrete tables and file references.
- Ask grouped, owner-specific questions.
- Keep the plan surgical and verifiable.
- If the user says they do not understand the report, do not ask them to classify issues; do that from the report and ask only business facts.

## Example User Requests That Should Trigger This Skill

- `帮我分析这个监管报告，并输出整改方案`
- `我不懂这个 App 审核反馈，帮我看要改什么`
- `根据这个隐私合规检查报告生成整改报告`
- `检查 Android/iOS 权限是否和商店说明一致`
- `帮我整理权限整改实施方案`
- `输出监管整改取证清单`
- `我们收到某地区 App 合规检查，帮我拆解开发任务`
