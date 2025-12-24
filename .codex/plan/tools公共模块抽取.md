# tools 公共模块抽取（DRY）

## 上下文

- 目录：`/acer/docs/DBus security`
- 现状：多个工具重复实现了“按行读取列表文件（含 BOM/零宽字符处理）”“执行外部命令”“systemctl show 解析”“缺少命令 vs 缺少输入文件的报错区分”等逻辑。
- 目标：抽取 `tools/_common.py` 作为内部复用模块，保持行为一致的同时降低维护成本与重复 bug 风险。

## 计划

- [x] 设计公共模块 API（最小可用、避免过度抽象）
- [x] 实现 `tools/_common.py`
- [x] 重构现有工具引用（`check_service_cap.py` / `check_service_fs_scope.py` / `check_deb_binaries_privilege.py` / `check_polkit_action_implicit.py`）
- [x] 更新文档与自检（`doc/architecture.md`、`doc/changelog.md`、py_compile）

## 记录

- 开始时间：2025-12-23T09:40:09+08:00
- 结束时间：2025-12-23T09:46:23+08:00

## 自检

- `PYTHONDONTWRITEBYTECODE=1 python3 -m py_compile tools/_common.py tools/check_service_cap.py tools/check_service_fs_scope.py tools/check_deb_binaries_privilege.py tools/check_polkit_action_implicit.py` 通过
