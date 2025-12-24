# DBus system.d root service 方法暴露检查工具

## 上下文

- 目录：`/acer/docs/DBus security`
- 输入 1：系统目录 `/etc/dbus-1/system.d/`、`/usr/share/dbus-1/system.d/` 下所有 `*.conf`（XML）
- 输入 2：`--services-file` 指定的 D-Bus bus name 列表（按行分隔）
- 目标：结合 conf 中 `allow own` / `deny` 策略，识别允许 root `own` 的 bus name，并输出该 service **未被 default policy deny 覆盖**的 methods（`service -> object path -> interface -> method`），同时输出该 service 的 conf 文件与所属 deb 包（`dpkg-query -S`）。

## 计划

- [x] 扩展 conf 扫描：索引 `allow own` 与 default policy 下 `deny send_*`
- [x] 增加 `--services-file`：按行读取 bus name 列表
- [x] 判定 root service：存在 `<policy user="root"><allow own="...">`
- [x] 使用 `busctl --system introspect --xml-interface` 递归枚举 object tree 的 methods
- [x] 以 default policy deny 规则匹配并剔除受管控 methods，输出剩余 methods
- [x] 支持 `--json` 与 `--only-flagged`（仅输出 flagged/error/not-found 的 results）
- [x] 更新 `doc/architecture.md`、`doc/changelog.md`

## 记录

- 开始时间：2025-12-23T11:05:45+08:00
- 结束时间：2025-12-23T11:09:05+08:00

## 自检

- `PYTHONDONTWRITEBYTECODE=1 python3 -m py_compile tools/check_dbus_system_conf.py`
- `python3 ./tools/check_dbus_system_conf.py --services-file <(printf "org.debian.apt\n")`（本容器 system bus 不可用，会返回 error；在真实系统应可枚举方法）
