# DBus root service 未管控 method 状态标记

## 上下文

- 工具：`tools/check_dbus_system_conf.py`
- 痛点：当 root service 存在未被 default policy deny 覆盖的 methods 时，`status` 仍为 `ok`，不利于下游脚本基于状态分类处理。
- 目标：新增风险状态 `uncontrolled`，用于标识“root service 存在未管控 methods”。

## 计划

- [x] 计算残留 methods 后设置 `status=uncontrolled`
- [x] 在 `summary` 中增加 `uncontrolled` 统计
- [x] 自检与记录变更（`doc/changelog.md`）

## 记录

- 开始时间：2025-12-23T11:33:16+08:00
- 结束时间：2025-12-23T11:33:16+08:00

## 自检

- `PYTHONDONTWRITEBYTECODE=1 python3 -m py_compile tools/check_dbus_system_conf.py`

