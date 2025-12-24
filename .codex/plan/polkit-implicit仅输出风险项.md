# polkit implicit 仅输出风险项

## 上下文

- 目录：`/acer/docs/DBus security`
- 目标：为 `tools/check_polkit_action_implicit.py` 增加 `--only-flagged`，只输出风险 action 信息，并更新相关文档。

## 计划

- [x] 增加 `--only-flagged` 参数并调整 JSON/文本输出逻辑。
- [x] 更新架构与变更记录，补充计划记录。

## 记录

- 开始时间：2025-12-24T13:27:48+08:00
- 结束时间：2025-12-24T13:27:48+08:00

## 遗留项

- 无。

## 自检

- 未运行：未提供可执行的测试指令，且依赖运行环境具备 `pkaction`。
