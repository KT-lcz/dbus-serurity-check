# deb 二进制 Cap 与 S 位检查工具（Python）

## 上下文

- 目录：`/acer/docs/DBus security`
- 目标：实现一个 Python CLI 工具，读取 deb 包名列表文件（每行一个包名），列出包内所有可执行文件，并检查：
  - file capabilities（`getcap`）
  - `setuid`/`setgid`（S 位）
- 输出：仅输出“存在 Cap 或 S 位”的二进制，并标注二进制路径与所属包名；支持 JSON 与 summary。

## 计划

- [x] 梳理输入输出与规则（包列表格式、二进制判定、错误处理、退出码）
- [x] 实现工具（`dpkg-query -L` → 过滤可执行文件 → `getcap`/`stat` → 输出）
- [x] 更新文档与变更记录（`doc/architecture.md`、`doc/changelog.md`）
- [x] 自检与优化建议（`--help`、`py_compile`；请用户在真实环境验证）

## 记录

- 开始时间：2025-12-22T17:24:59+08:00
- 结束时间：2025-12-22T17:28:40+08:00

## 自检

- `python3 tools/check_deb_binaries_privilege.py --help` 正常
- `python3 -m py_compile tools/check_deb_binaries_privilege.py` 通过
- 以已安装包 `iputils-ping`/`passwd` 验证：可识别 capabilities（`/usr/bin/ping`）与 setuid/setgid（`/usr/bin/passwd` 等）
