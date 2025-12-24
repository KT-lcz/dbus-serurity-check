# polkit actionid 隐式授权检查工具（Python）

## 上下文

- 目录：`/acer/docs/DBus security`
- 目标：实现一个 Python CLI 工具，读取 actionid 列表文件（每行一个 actionid），逐个执行 `pkaction -a <actionid> -v`，检查：
  - `implicit any`
  - `implicit inactive`
  - `implicit active`
- 判定：上述字段若为 `yes` / `auth_self` / `auth_self_keep`，则输出该 actionid、所在包与对应字段值。
- 兼容性：与既有工具保持一致（支持单个/批量、`--json`、summary、输入 BOM/零宽字符处理、退出码风格）。

## 计划

- [x] 确认解析规则与输出字段（字段命名、not-found/error 判定、包归属获取方式）
- [x] 实现 actionid 检查工具（`pkaction` 解析 + policy 文件定位 + `dpkg-query -S`）
- [x] 更新架构与变更记录（`doc/architecture.md`、`doc/changelog.md`）
- [x] 自检与优化建议（`--help`、`py_compile`；请用户在真实 polkit 环境验证）

## 记录

- 开始时间：2025-12-22T17:50:21+08:00
- 结束时间：2025-12-22T17:52:52+08:00

## 自检

- `python3 tools/check_polkit_action_implicit.py --help` 正常
- `python3 -m py_compile tools/check_polkit_action_implicit.py` 通过
