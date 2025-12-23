# DBus Security 检查清单与工具集

本目录用于落地 DBus 安全检查表，并提供若干最小可用的自动化工具，用于在交付/门禁阶段收集证据、发现常见高风险配置面。

- 检查清单：`DBus 安全检查-检查清单.md`
- 架构设计：`doc/architecture.md`
- 变更记录：`doc/changelog.md`
- 工具脚本：`tools/*.py`

## 环境要求

- Python：`3.10+`（脚本使用了 `X | None` 等语法）
- 系统：建议 Debian/Ubuntu + systemd（脚本依赖 `systemctl/busctl` 与 `dpkg-query`）
- 命令依赖（按需）：
  - `systemctl`：systemd service 检查
  - `busctl`：D-Bus system bus introspection（会连接 system bus，且默认允许 auto-start）
  - `dpkg-query`：通过文件路径反查 deb 包归属
  - `getcap`：读取 file capabilities（通常来自 `libcap2-bin`）
  - `pkaction`：读取 polkit action 配置（通常来自 `policykit-1`）

> 注意：在容器/受限环境中，`systemctl`/`busctl`/`pkaction` 可能因无法连接 system bus / polkit authority 而失败；工具会输出英文错误信息并返回非 0 退出码。

## 通用约定（输入文件）

所有“列表文件”参数均遵循相同格式：

- 每行一个条目（service / actionid / package / bus name）
- 忽略空行与以 `#` 开头的注释行
- 支持 UTF-8 BOM，并会清理零宽字符（避免不可见字符污染参数）

示例：

```text
# one item per line
dbus.service
ssh.service
```

## 工具说明

### 1) `tools/check_service_cap.py`

通过 `systemctl show` 检查 systemd service 的用户/组与 capabilities，并可与期望能力集做对比。

**用法**

```bash
python3 "./tools/check_service_cap.py" "dbus.service"
python3 "./tools/check_service_cap.py" --services-file "./services.txt" --json
python3 "./tools/check_service_cap.py" --services-file "./services.txt" --expected-caps "./expected_caps.txt"
```

**能力判定规则（EffectiveCapabilities）**

- `User` 为空或为 `root`：使用 `CapabilityBoundingSet`
- `User` 非 root：使用 `AmbientCapabilities`

**输出（文本）**

- `User/Group/SupplementaryGroups/Groups`
- `CapabilityBoundingSet/AmbientCapabilities/EffectiveCapabilities`
- `Rule`：表明使用了哪条判定规则
- 若指定 `--expected-caps`：输出 `ExpectedCapabilities/MatchExpected/MissingCapabilities/UnexpectedCapabilities`

**输出（JSON）**

- 顶层：`results`（逐 service 结果）、`summary`（计数汇总）
- `results[].status`：`ok` / `not-found` / `mismatch` / `error`

**退出码**

- `0`：全部检查正常
- `2`：存在 `not-found`
- `3`：存在 `mismatch`（仅在指定 `--expected-caps` 时可能出现）
- `1`：其他错误
- `127`：缺少外部命令（如 `systemctl`）

### 2) `tools/check_service_fs_scope.py`

通过 `systemctl show` 读取 service 的沙箱/路径相关参数，派生该 service 对文件系统的可读/可写范围摘要。

**用法**

```bash
python3 "./tools/check_service_fs_scope.py" "dbus.service"
python3 "./tools/check_service_fs_scope.py" --services-file "./services.txt" --json
```

**输出（文本）关键字段**

- 原始字段：`ProtectSystem/ProtectHome/PrivateTmp/NoNewPrivileges/*Paths/StateDirectory/RuntimeDirectory`
- 派生字段：
  - `ReadableScope`：
    - `All`：默认可读全局
    - `All except ...`：存在不可访问路径（例如 `ProtectHome=yes` 或 `InaccessiblePaths=`）
  - `WritableScope`：
    - `Only ...`：`ProtectSystem=strict` 时，近似“默认只读根”，仅允许显式可写路径
    - `All except read-only roots ... and inaccessible paths ...; writable exceptions ...`：其余情况下的摘要
- 提示字段：
  - `HintPreferStateRuntimeDirectory`：当 `*Paths` 中显式使用 `/var/lib`、`/var/run`、`/run` 时提示优先使用 `StateDirectory=`/`RuntimeDirectory=`

> 说明：该工具基于有限字段做“范围摘要”，不覆盖所有可能影响文件系统视图的 systemd 指令（例如 bind mounts 等），建议与 unit 文件评审结合使用。

**输出（JSON）**

- 顶层：`results`、`summary`
- `results[].read_scope` / `results[].write_scope`：派生结构化范围
- `results[].prefer_state_runtime_directory_hint`：提示信息

**退出码**

- `0`：全部检查正常
- `2`：存在 `not-found`
- `1`：其他错误
- `127`：缺少外部命令（如 `systemctl`）

### 3) `tools/check_deb_binaries_privilege.py`

输入已安装的 deb 包列表，枚举包内可执行文件并检测：

- file capabilities（`getcap`）
- setuid / setgid（S 位）

仅输出存在 capabilities 或 S 位的二进制，以及其所属包。

**用法**

```bash
python3 "./tools/check_deb_binaries_privilege.py" "openssh-server"
python3 "./tools/check_deb_binaries_privilege.py" --packages-file "./packages.txt" --json
```

**输出（文本）**

- `Package: ...`
- `Binary: <path> | Capabilities: ... | Setuid: ... | Setgid: ... | Mode: ...`

**输出（JSON）**

- 顶层：`results`、`summary`
- `results[].findings[]`：
  - `path`：二进制路径
  - `capabilities`：`getcap` 输出（为空则无）
  - `setuid` / `setgid`：布尔值
  - `mode_octal`：权限位（含 suid/sgid/sticky 的 4 位八进制）

**退出码**

- `0`：全部检查正常
- `2`：存在 `not-found`（包未安装）
- `1`：其他错误
- `127`：缺少外部命令（如 `dpkg-query`/`getcap`）

### 4) `tools/check_polkit_action_implicit.py`

读取 actionid 列表，执行 `pkaction -a <id> -v` 并检查以下字段是否命中高风险取值：

- `implicit any`
- `implicit inactive`
- `implicit active`

命中值：`yes` / `auth_self` / `auth_self_keep`（大小写不敏感）

若命中，则输出 actionid、对应 `.policy` 文件以及所属 deb 包（通过 `dpkg-query -S` 反查）。

**用法**

```bash
python3 "./tools/check_polkit_action_implicit.py" "org.freedesktop.packagekit.system-sources-refresh"
python3 "./tools/check_polkit_action_implicit.py" --actions-file "./actionids.txt" --json
```

**输出（文本）**

- `ActionId`
- `Packages`
- `ImplicitAny/ImplicitInactive/ImplicitActive`
- `PolicyFiles`（若可定位）

**退出码**

- `0`：全部检查完成（即使存在命中项也返回 0）
- `2`：存在 `not-found`（action id 不存在或不可查询）
- `1`：其他错误（例如 `pkaction` 无法连接 authority）
- `127`：缺少外部命令（如 `pkaction`/`dpkg-query`）

### 5) `tools/check_dbus_system_conf.py`

扫描 DBus system bus 配置目录下所有 `*.conf`（XML），提供两种模式：

#### 模式 A：default policy 下 `allow own`（默认）

定位 `<policy context="default">` 下的 `<allow own="...">`，输出对应 conf 文件与所属 deb 包。

```bash
python3 "./tools/check_dbus_system_conf.py"
python3 "./tools/check_dbus_system_conf.py" --json --only-flagged
```

文本输出字段：

- `ConfFile`
- `Packages`
- `AllowOwnInDefaultPolicy`

#### 模式 B：root service 未被 default deny 覆盖的方法面（`--services-file`）

读取 system bus name 列表（每行一个），并：

1. 识别 “root service”：conf 中存在 `<policy user="root"><allow own="SERVICE"/>`
2. 对 root service 使用 `busctl --system introspect --xml-interface --auto-start=yes` 递归枚举 methods
3. 从 methods 中排除 `<policy context="default">` 下的 `deny send_*` 覆盖项
4. 输出剩余 methods（按 `service -> object path -> interface -> method` 结构）

同时输出该 service 对应 conf 文件与所属 deb 包（由 conf 文件通过 `dpkg-query -S` 反查）。

```bash
python3 "./tools/check_dbus_system_conf.py" --services-file "./dbus_services.txt"
python3 "./tools/check_dbus_system_conf.py" --services-file "./dbus_services.txt" --json --only-flagged
```

**状态（`results[].status`）**

- `not-found`：未在 conf 中找到该 service 的 `allow own`
- `not-root`：找到 `allow own` 但未允许 `root` own
- `ok`：root service 且剔除 default deny 后无残留 methods
- `uncontrolled`：root service 且剔除 default deny 后仍存在残留 methods
- `error`：introspect/解析失败（例如无法连接 system bus）

**过滤**

- 会排除以下通用接口下的方法（减少噪音）：
  - `org.freedesktop.DBus.Introspectable`
  - `org.freedesktop.DBus.Properties`
  - `org.freedesktop.DBus.Peer`

**deny 匹配语义（用于排除 methods）**

从 `<policy context="default">` 下的 `deny` 读取以下字段进行匹配：

- `send_destination`（必需）
- 可选：`send_path` / `send_path_prefix` / `send_interface` / `send_member` / `send_type`

当 `deny` 未指定 `send_member` 时，表示“匹配更大范围”，因此会排除该范围内的所有 methods（例如仅 `send_destination` 或 `send_destination+send_interface`）。

> 重要：该模式仅基于 “default policy 的 deny” 做排除；工具输出的是“未被 default deny 覆盖的方法集”，不等价于“任何调用方都可调用”。建议结合 conf 中的 `allow send_*` 与业务身份模型进一步评估。

**退出码**

- `0`：全部检查完成（`uncontrolled` 不影响退出码）
- `2`：存在 `not-found`（仅在 `--services-file` 模式下）
- `1`：其他错误
- `127`：缺少外部命令（如 `dpkg-query`/`busctl`）

## CI/交付集成建议

- 输出结构化证据：优先使用 `--json` 并保存产物（便于归档与后续 diff）
- 降噪：使用 `--only-flagged` 仅保留风险项与错误项（适用于 `check_dbus_system_conf.py` 的 JSON 输出）

示例：

```bash
python3 "./tools/check_service_cap.py" --services-file "./services.txt" --json > "./cap_report.json"
python3 "./tools/check_dbus_system_conf.py" --services-file "./dbus_services.txt" --json --only-flagged > "./dbus_report.json"
```

