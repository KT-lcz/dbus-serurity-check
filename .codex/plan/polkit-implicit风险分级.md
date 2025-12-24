# polkit implicit 风险分级

## 上下文

- 目录：`/acer/docs/DBus security`
- 目标：为 `tools/check_polkit_action_implicit.py` 增加风险分级输出，规则为 yes=高风险，auth_self/auth_self_keep=待人工分析，并补充文本/JSON 输出字段。

## 计划

- [x] 更新工具风险分级逻辑与输出字段（文本/JSON）。
- [x] 更新架构与变更记录，补充计划记录。

## 记录

- 开始时间：2025-12-24T13:20:16+08:00
- 结束时间：2025-12-24T13:20:16+08:00

## 遗留项

- 无。

## 自检

- 未运行：未提供可执行的测试指令，且依赖运行环境具备 `pkaction`。
