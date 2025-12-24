# service 列表 summary 误判修复

## 上下文

- 目录：`/acer/docs/DBus security`
- 现象：执行 `python3 ./tools/check_service_cap.py --services-file ./service --json` 时，`results` 中每个 service 看起来正常，但 `summary` 显示 `not_found=1 ok=2`，与预期不符（应为 `ok=3`）。
- 目标：修复导致误判的根因，保证 `summary` 与 `results[].status` 一致且符合真实查询结果。

## 计划

- [x] 定位 summary 误判原因（优先排查 BOM/零宽字符导致的 unit 名异常）
- [x] 修复 service/caps 文件解析（兼容 UTF-8 BOM；清理零宽字符；空行安全处理）
- [x] 更新文档与变更记录（`doc/architecture.md`、`doc/changelog.md`）
- [x] 自检与回归验证（`--help`、`py_compile`；给出复现/确认指令）

## 记录

- 开始时间：2025-12-22T16:48:04+08:00
- 结束时间：2025-12-22T16:49:18+08:00

## 自检

- `python3 tools/check_service_cap.py --help` 正常
- `python3 -m py_compile tools/check_service_cap.py` 通过

## 反馈点

- 请在你的环境复跑：`python3 ./tools/check_service_cap.py --services-file ./service --json`，确认 `summary.ok` 与实际 service 数一致。
