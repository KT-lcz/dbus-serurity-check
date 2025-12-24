# DBus system.d 检查工具增加 only-flagged

## 上下文

- 工具：`tools/check_dbus_system_conf.py`
- 痛点：`--json` 输出包含大量无命中项（扫描系统所有 `*.conf`），在 CI/脚本消费时噪音较大。
- 目标：增加 `--only-flagged`，在 `--json` 模式下仅输出命中项与错误项。

## 计划

- [x] 增加 `--only-flagged` 参数
- [x] 过滤 `--json` 的 `results`（保留 flagged 与 error）
- [x] 自检与记录变更（`doc/changelog.md`）

## 记录

- 开始时间：2025-12-23T10:23:09+08:00
- 结束时间：2025-12-23T10:23:44+08:00

## 自检

- `PYTHONDONTWRITEBYTECODE=1 python3 -m py_compile tools/check_dbus_system_conf.py`
- `python3 ./tools/check_dbus_system_conf.py --json --only-flagged`（results 数量应显著减少）

