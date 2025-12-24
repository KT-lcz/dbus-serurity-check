# DBus system.d 默认策略 own 检查工具

## 上下文

- 目录：`/acer/docs/DBus security`
- 输入：系统目录 `/etc/dbus-1/system.d/`、`/usr/share/dbus-1/system.d/` 下所有 `*.conf`（XML）
- 目标：实现第一项检查能力——当 `<allow own="...">` 位于 `<policy context="default">` 下时，输出该配置、所在 conf 文件与所属 deb 包（`dpkg-query -S`）

## 计划

- [x] 扫描两个目录并枚举 `*.conf`
- [x] 解析 XML，定位 default policy 下的 `<allow own>`
- [x] 对命中项执行 `dpkg-query -S` 反查所属包
- [x] 输出文本报告与 `--json` 结构化结果
- [x] 更新 `doc/architecture.md` 与 `doc/changelog.md`

## 记录

- 开始时间：2025-12-23T10:13:57+08:00
- 结束时间：2025-12-23T10:17:01+08:00

## 自检

- `PYTHONDONTWRITEBYTECODE=1 python3 -m py_compile tools/_common.py tools/check_dbus_system_conf.py tools/check_polkit_action_implicit.py`
- `python3 ./tools/check_dbus_system_conf.py`（本环境扫描到 2 个命中项）

