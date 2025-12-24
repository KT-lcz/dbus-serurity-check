# systemd service 文件系统范围检查工具（Python）

## 上下文

- 目录：`/acer/docs/DBus security`
- 目标：实现一个 Python CLI 工具，通过 `systemctl show --property=...` 检查 service 的文件系统沙箱参数，并输出该 service 的可读/可写范围摘要。
- 重点参数：`ReadWritePaths` `ProtectSystem` `ProtectHome` `PrivateTmp` `InaccessiblePaths` `ReadOnlyPaths` `NoNewPrivileges`。
- 额外规则：若检测到显式使用 `/var/lib` `/var/run` `/run`（路径前缀），输出提醒字段：优先使用 `StateDirectory=`/`RuntimeDirectory=`。
- 兼容性：与 `tools/check_service_cap.py` 行为保持一致（单个/批量、`--json`、summary、退出码、输入文件 BOM/零宽字符处理）。

## 计划

- [x] 明确输出模型与规则（可读/可写范围表达、提示触发条件、退出码）
- [x] 实现文件系统范围检查工具（`systemctl show` → 解析 → 派生范围 → 输出）
- [x] 更新文档与变更记录（`doc/architecture.md`、`doc/changelog.md`）
- [ ] 自检与回归验证（`--help`、`py_compile`；请用户在真实 systemd 环境验证）

## 记录

- 开始时间：2025-12-22T17:03:01+08:00
- 结束时间：2025-12-22T17:06:41+08:00

## 自检

- `python3 tools/check_service_fs_scope.py --help` 正常
- `python3 -m py_compile tools/check_service_fs_scope.py` 通过
- `--services-file` 支持 UTF-8 BOM/零宽字符（通过合成样例文件验证）
