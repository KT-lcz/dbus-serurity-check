# 变更记录

## 2025-12-22T16:04:07+08:00

### 修改目的

- 将 DBus 安全检查 CSV 转换为可执行的 Markdown 检查清单，便于评审/门禁/交付流程直接使用。

### 修改范围

- 新增 `DBus 安全检查-检查清单.md`
- 新增 `doc/architecture.md`
- 新增 `doc/changelog.md`
- 新增 `.codex/plan/DBus安全检查清单转换.md`

### 修改内容

- 按“检查项”聚合 CSV 记录（空“检查项”行归并到上一条检查项），生成分章节清单。
- 每条检查以 `- [ ]` 形式输出，并保留字段：检查阶段/检查方法/流程/输出/是否需要 AI/处理方法/备注。
- 对多行字段进行结构化展开（列表/分行），提升可读性与可执行性。

### 对整体项目的影响

- 文档从“表格”升级为“可执行清单”，更易在 CI 门禁、CD 交付检查与人工评审中落地。
- 不改变原始 CSV 数据；如 CSV 更新，需要同步更新 Markdown 清单以保持一致。

## 2025-12-22T16:20:26+08:00

### 修改目的

- 实现一个可复用的 Python CLI 工具，用于按传参检查单个 systemd service 的 Cap/User/Group 信息，支撑“DBus服务权限检查”落地自动化。

### 修改范围

- 新增 `tools/check_service_cap.py`
- 新增 `.codex/plan/systemd-service-cap检查工具.md`
- 更新 `doc/architecture.md`
- 更新 `doc/changelog.md`

### 修改内容

- 通过 `systemctl show <service> --property=...` 获取并解析 `User/Group/SupplementaryGroups/CapabilityBoundingSet/AmbientCapabilities` 等字段。
- 按检查表规则计算 `effective_capabilities`（root → `CapabilityBoundingSet`；非 root → `AmbientCapabilities`），并输出文本或 JSON。
- 提供明确退出码：成功 `0`、service 不存在 `2`、其他错误 `1`。

### 对整体项目的影响

- 将“service Cap”检查从纯人工步骤提炼为可执行工具，便于在 CD/交付环节复用与标准化。
- 工具依赖运行环境可访问 `systemctl`/system bus；在容器或无 systemd 环境下会返回错误并提示原因。

## 2025-12-22T16:35:44+08:00

### 修改目的

- 增强 `check_service_cap.py`：支持从文件批量读取 service；支持通过 `--expected-caps` 校验期望 Cap，不一致时返回退出码 `3`，便于门禁自动判定。

### 修改范围

- 更新 `tools/check_service_cap.py`
- 新增 `.codex/plan/systemd-service-cap工具增强.md`
- 更新 `doc/architecture.md`
- 更新 `doc/changelog.md`

### 修改内容

- 新增 `--services-file`：从文件读取待检查的 unit（每行一个）。
- 新增 `--expected-caps`：从文件读取期望 Cap（每行一个），与实际 `effective_capabilities` 对比；不一致时标记为 mismatch 并返回退出码 `3`。
- 增强 JSON 输出：输出 `results` 列表及 `summary` 汇总，便于 CI 消费。

### 对整体项目的影响

- 可一次性对一组 service 做一致性检查；与期望 Cap 文件配合可直接作为门禁脚本使用。
- JSON 输出结构发生变化（统一输出为包含 `results/summary` 的对象），如有既有解析脚本需同步调整。

## 2025-12-22T16:49:18+08:00

### 修改目的

- 修复批量模式下 summary 的 `not_found` 误判：兼容 service 列表文件/期望 cap 文件的 UTF-8 BOM 与零宽字符，避免 unit 名被污染导致 `LoadState=not-found`。

### 修改范围

- 更新 `tools/check_service_cap.py`
- 新增 `.codex/plan/service列表summary误判修复.md`
- 更新 `doc/changelog.md`

### 修改内容

- 读取文件统一使用 `utf-8-sig`，自动剥离 BOM。
- 额外清理零宽字符（如 `\ufeff`/`\u200b` 等），并对命令行传入的 service 名同样做规范化。

### 对整体项目的影响

- 解决常见“肉眼看起来正确但实际 unit 名包含不可见字符”的场景，summary 与实际查询一致性更高。

## 2025-12-22T17:05:28+08:00

### 修改目的

- 新增文件系统范围检查工具：读取 service 的 `ProtectSystem/ProtectHome/*Paths` 等配置，输出可读/可写范围摘要，并在显式使用 `/var/lib` `/var/run` `/run` 时提示优先使用 `StateDirectory=`/`RuntimeDirectory=`。

### 修改范围

- 新增 `tools/check_service_fs_scope.py`
- 新增 `.codex/plan/systemd-service-fs-scope检查工具.md`
- 更新 `doc/architecture.md`
- 更新 `doc/changelog.md`

### 修改内容

- 通过 `systemctl show <service> --property=...` 获取并解析 `ReadWritePaths/ReadOnlyPaths/InaccessiblePaths/ProtectSystem/ProtectHome/PrivateTmp/NoNewPrivileges` 等字段。
- 派生 `ReadableScope/WritableScope`，并输出文本或 JSON（含 `results/summary`）。
- 若 `*Paths` 中显式出现 `/var/lib` `/var/run` `/run` 路径前缀，输出提示字段 `prefer_state_runtime_directory_hint`。

### 对整体项目的影响

- “文件路径注入检查”可由工具提供结构化证据，减少人工翻看 unit 文件/配置的成本。
- 输出为派生结果，仍建议与 unit 文件评审、运行时验证结合使用。
