# systemd service Cap 检查工具增强

## 上下文

- 目录：`/acer/docs/DBus security`
- 现有工具：`tools/check_service_cap.py`
- 目标增强：
  1. 支持从文件读取需要检查的 service 列表（每行一个 service）。
  2. 新增 `--expected-caps` 参数：从文件读取期望 Cap（每行一个 Cap），与实际结果不一致时返回退出码 `3`。

## 计划

- [x] 设计批量与期望 Cap 行为（输入优先级、输出、退出码聚合）
- [x] 实现 `--services-file` 与 `--expected-caps`（解析、对比、返回码）
- [x] 更新文档与变更记录（`doc/architecture.md`、`doc/changelog.md`）
- [x] 自检并给出优化建议（`--help`、`py_compile`、可维护性建议）

## 记录

- 开始时间：2025-12-22T16:33:32+08:00
- 结束时间：2025-12-22T16:36:38+08:00

## 自检

- `python3 tools/check_service_cap.py --help` 正常
- `python3 -m py_compile tools/check_service_cap.py` 通过
