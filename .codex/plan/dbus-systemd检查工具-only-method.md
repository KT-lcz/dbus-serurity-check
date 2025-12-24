# DBus systemd 检查工具：--only-method（JSON 裁剪）

## 上下文

- 目录：`/acer/docs/DBus security`
- 目标：为 `tools/check_dbus_system_conf.py` 增加 `--only-method` 参数，在 `--services-file + --json` 模式下仅输出 method 三元组（`dbus_path/interface/method`），便于下游直接消费。

## 约定/决策

- `--only-method` 仅在 `--services-file` 且 `--json` 时生效；其他组合直接报错（避免语义歧义）。
- 输出格式为 JSON 数组（stdout）：`[{dbus_path, interface, method}, ...]`；不输出 `results/summary/missing_dirs` 等 envelope 字段。
- 若输入包含多个 service，则输出为所有 service 的 method 三元组并集（不携带 service 归属信息）。

## 计划

- [x] 明确 `--only-method` 输出语义与约束
- [x] 实现参数解析与 JSON 输出裁剪
- [x] 更新 `README.md` 与 `doc/changelog.md`
- [x] 自检（`py_compile`）并收尾

## 记录

- 开始时间：2025-12-23T17:20:49+08:00
- 结束时间：2025-12-23T17:23:40+08:00

## 自检

- `PYTHONDONTWRITEBYTECODE=1 python3 -m py_compile tools/check_dbus_system_conf.py`
- `PYTHONDONTWRITEBYTECODE=1 python3 ./tools/check_dbus_system_conf.py --services-file ./service --json --only-method`
