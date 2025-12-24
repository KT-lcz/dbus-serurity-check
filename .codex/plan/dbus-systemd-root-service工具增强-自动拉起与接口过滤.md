# DBus system.d root service 工具增强：自动拉起与接口过滤

## 上下文

- 工具：`tools/check_dbus_system_conf.py`
- 目标：按需求调整 root service methods 模式：
  - root 判定：只要 conf 中允许 root `own`，即视为 root service（即使同时允许其他非 default own）。
  - 方法枚举：默认允许自动拉起 service（busctl auto-start）。
  - 输出过滤：排除通用接口 `org.freedesktop.DBus.Introspectable` / `org.freedesktop.DBus.Properties` / `org.freedesktop.DBus.Peer` 下的方法。

## 计划

- [x] 调整 busctl introspect 默认 auto-start
- [x] 增加通用接口过滤列表并在枚举阶段排除
- [x] 自检与记录变更（`doc/changelog.md`）

## 记录

- 开始时间：2025-12-23T11:27:47+08:00
- 结束时间：2025-12-23T11:27:47+08:00

## 自检

- `PYTHONDONTWRITEBYTECODE=1 python3 -m py_compile tools/check_dbus_system_conf.py`

