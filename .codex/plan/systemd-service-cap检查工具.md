# systemd service Cap 检查工具（Python）

## 上下文

- 目录：`/acer/docs/DBus security`
- 目标：实现一个 Python CLI 工具，通过传参指定一个 systemd service，检查并输出该 service 的 User/Group/Capabilities 信息。
- 依据：`DBus 安全检查-工作表1.csv` 中“DBus服务权限检查”的检查逻辑。

## 计划

- [x] 梳理需求与输出格式（字段、规则、错误处理、退出码）
- [x] 实现 Python CLI 工具（`systemctl show` → 解析 → 输出）
- [x] 补充架构与变更记录（`doc/architecture.md`、`doc/changelog.md`）
- [x] 本地自检与优化建议（运行 `--help`、静态检查、可读性优化建议）

## 记录

- 开始时间：2025-12-22T16:18:17+08:00
- 结束时间：2025-12-22T16:21:47+08:00

## 自检

- `python3 tools/check_service_cap.py --help` 正常
- `python3 -m py_compile tools/check_service_cap.py` 通过
