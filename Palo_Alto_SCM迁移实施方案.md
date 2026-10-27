# PAN-OS 防火墙配置迁移到 SCM 实施指南

> **示例**：上海站点（`shanghai-base-new1.xml` → `Shanghai`）配置导入 SCM，适用于后续各站点迁移（命令中 `<站点>.xml` / `<站点folder>` 按实际替换，如 `wuxi.xml` / `Wuxi`）。

**流程两阶段**
- **阶段 A · Folder 级（第 1-4 步）**：解析 → 干跑 → 导入 folder 容器 → 剩余 UI 补配；仅暂存 SCM 云端，**不影响现网防火墙**
- **阶段 B · 设备级（第 5-9 步）**：Onboard → 解析 → 干跑 → 导入 device 容器 → Push 验证；需变更窗口执行

> **本指南验证基线**：`shanghai-base-new1.xml`（PAN-OS 11.1.0, 设备 NB_PA_A, 详见 11.1.13），`--folder Shanghai` 下 Folder 924 项 + Device 45 项已完整干跑验证（见 `/tmp/simulate.log`），可作为模板复用。

---

## 一、脚本与文件

### 1.1 Folder 级脚本（与设备级同构的两步流程）

**`parse_panos_folder_xml.py`** — folder 级 XML → 导入文件（只读输入，不连接 SCM）

```
输入:  shanghai-base-new1.xml --folder Shanghai
产出:  Shanghai_folder_config.json              可导入的配置对象 (Folder 容器, 与 device 对称，阶段 A)
        Shanghai_folder_migration_summary.md     落地清单（含 UI 手工项，与 device 对称）
        命名: 统一 {folder}_folder_config.json / {folder}_device_config.json，ls {folder}_* 即归组
```

**`import_folder_to_scm.py`** — JSON → SCM folder

```
输入:  Shanghai_folder_config.json
行为:  全量导入 / --only 按类型导入 / 不加 --apply 为干跑预览
        仅 --apply 时写入 SCM，不写任何文件
```

### 1.2 设备级脚本（与 folder 级同构的两步流程）

**`parse_panos_device_xml.py`** — 设备级 XML → 导入文件（只读输入，不连接 SCM）

```
输入:  shanghai-base-new1.xml --folder Shanghai
产出:  Shanghai_device_config.json              设备级可导入配置（接口/IKE/IPsec/路由，阶段 B）
        Shanghai_device_migration_summary.md     设备级落地清单 (与 folder 对称)
选项:  -o / --summary 改名；--vrf-name 指定 VRF 名；--folder 统一 folder 前缀 (不传回退 xml 名, 兼容)
        命名: 统一 {folder}_device_config.json / {folder}_folder_config.json，ls {folder}_* 即归组
```

**`import_device_to_scm.py`** — JSON → SCM（接口写 folder 作变量，LR/VPN 写 device 容器）

```
输入:  Shanghai_device_config.json
前提:  防火墙已 Onboard；目标 folder 已存在
参数:  --device <序列号>（--apply 必填）
       --folder <站点folder>  接口类对象(ethernet/tunnel/loopback)创建于 folder 作为"接口变量"
                              （官方模型：设备级逻辑路由器/IPsec VPN 只能引用 folder 级接口变量）
行为:  全量导入 / --only 按类型导入 / 不加 --apply 为干跑预览（不发任何网络请求）
其他:  VRF 大写名自动转小写；逻辑路由器自动声明 ARE 路由栈
```

```bash
# 标准三步: 解析 → 干跑 → 正式导入 (接口以 folder 变量模式创建, 统一 folder 前缀)
python3 parse_panos_device_xml.py shanghai-base-new1.xml --folder Shanghai
python3 import_device_to_scm.py Shanghai_device_config.json --device <序列号> --folder Shanghai
python3 import_device_to_scm.py Shanghai_device_config.json --device <序列号> --folder Shanghai --apply
```

### 1.3 文件清单

**输入源（每站点一个，防火墙导出）**
- `<站点>.xml` — 如 `shanghai-base-new1.xml`（folder 级 + 设备级解析共用同一导出文件）；其他站点如 `wuxi.xml` 同理

**脚本产出（可随时重新生成，建议保留作验收记录，统一 folder 前缀）**
- `<folder>_folder_config.json` / `<folder>_folder_migration_summary.md` — folder 级导入文件与落地清单 (`ls <folder>_*` 即归组, 与 device 对称，阶段 A)
- `<folder>_device_config.json` / `<folder>_device_migration_summary.md` — 设备级导入文件与落地清单 (`--folder` 统一前缀; 不传 --folder 时回退 `<xml名>_device_config.json` 兼容，阶段 B)
- `<folder>_folder_config_import_errors.json` / `<folder>_device_config_import_errors.json` — 导入失败项完整报错（仅失败时产生, 随对应 JSON 同前缀；`import_folder_to_scm.py` 与 `import_device_to_scm.py` 同款落盘逻辑，含未截断原始报文，可直接提交 PANW 支持）

**实测产物示例（上海站点）**
- `Shanghai_folder_config.json` — 924 项（对象类 625 + 策略类 299）
- `Shanghai_device_config.json` — 45 项（eth 7 + tunnel 6 + IKE/IPsec 各6 + VPN 3+3 + LR 2）

**文档**
- `SCM_MIGRATION_GUIDE.md` — 本指南（手工维护，以 `shanghai-base-new1.xml` 为验证模板）
- 实施方案 PPT、步骤截图 DOCX、项目调研表 XLSX — 汇报/分享材料

### 1.4 配置分类与落地方向（以 Shanghai 924+45 为例，`Shanghai_folder_migration_summary.md` 四类为准）

- **第一类 · API 自动导入** — 标签(1)、地址(501)、地址组(34)、服务(64)、服务组(11)、URL 分类(4)、Zone(10)、安全策略(186)、NAT(49)、PBF(64) —— `import_folder_to_scm.py` 完成，计 924 项
- **第二类 · 由 Device 脚本接管完成** — 接口(7)、隧道(6)、虚拟路由器(2)、IKE/IPsec crypto(各6)、IKE 网关(3)/IPsec 隧道(3) 等设备级配置 —— 阶段 B `parse_panos_device_xml.py` + `import_device_to_scm.py` 自动化（接口作 folder 变量，计 45 项），Folder 清单中标记 `[x] 已接管`，无需 UI 重复配置
- **第三类 · SCM UI 手工配置（SCM 可管，非保留本地）** — GlobalProtect 网关/门户、日志设置/日志转发、Zone Protection 等 —— 需在 SCM UI 手工完成（`SCM_MIGRATION_GUIDE.md` 第三类，`Shanghai_folder_migration_summary.md` 第三类 `[ ]` 勾选）
- **第四类 · 保留设备本地（SCM 不管理）** — 本地证书（如 GlobalProtect/ex-gp-gw-*）、本地管理员账号（admin/chenxu/T02421）、管理接口/初始化配置 —— 保留在防火墙本地，不迁移

---

## 二、实施前准备工作

1. **SCM 许可证**：Essentials 或 Pro 已激活
2. **创建 folder**：SCM 中建好站点文件夹 `Shanghai`（其他站点如 `Wuxi`/`Zhangjiagang` 按需创建，名称与 `--folder` 保持一致）
3. **服务账号**：hub.paloaltonetworks.com > Common Services > IAM，角色含**配置写权限**，记录 `client_id` / `client_secret` / `TSG ID`
4. **导出配置**：防火墙 `Device > Setup > Operations > Export Named Configuration Snapshot`，得到 `shanghai-base-new1.xml`（仅导出，不影响运行）
5. **环境变量**（一次性写入 `~/.zshrc`，永久生效）：


   ```

   > 验证：`env | grep SCM_` 应显示三项；未生效则 `source ~/.zshrc` 后重跑

6. **PAN-OS 版本** ≥ 10.2.3（本模板 11.1.0，满足；`detail-version 11.1.13`）

---

## 三、实施步骤（最终方案要点）

> 全程分两阶段：**阶段 A = Folder 级**（配置上云，不影响现网）；**阶段 B = 设备级**（设备接入，需变更窗口）。每个导入步骤都先干跑预览，确认无误后再 `--apply`。
>
> **模板复用**：下述命令以 `shanghai-base-new1.xml` / `Shanghai` 为示例验证通过，其他站点仅替换文件名与 folder 名（如 `wuxi.xml --folder Wuxi`）。

### 阶段 A · Folder 级步骤（配置上云，不影响现网）

#### 第 1 步 · 解析 XML

```bash
cd /Users/xiahongbo/Documents/配置迁移
python3 parse_panos_folder_xml.py shanghai-base-new1.xml --folder Shanghai
# 其他站点例: python3 parse_panos_folder_xml.py wuxi.xml --folder Wuxi
```

> 默认产出 `Shanghai_folder_config.json` 与 `Shanghai_folder_migration_summary.md`（统一 folder 前缀，与 device 对称），可加 `-o` / `--summary` 改名（需带文件名值）。实测产出 924 项（见本段文件清单）。

#### 第 2 步 · 干跑预览（不写入 SCM）

```bash
python3 import_folder_to_scm.py Shanghai_folder_config.json --folder Shanghai
# 其他站点: python3 import_folder_to_scm.py Wuxi_folder_config.json --folder Wuxi
```

> 输出导入类型、数量、目标文件夹（示例：`tags 1 / addresses 501 / security_rules 186 ... 目标 Shanghai`）。**确认无误后再进入第 3 步。**

#### 第 3 步 · 正式导入（--apply）

```bash
python3 import_folder_to_scm.py Shanghai_folder_config.json --folder Shanghai --apply
```

- 按依赖顺序导入：tags → addresses → address_groups → services → service_groups → url_categories → zones → pbf_rules → security_rules → nat_rules
- 结果标记：`+` 新建 ｜ `=` 已存在跳过(409) ｜ `X` 失败（按错误修 JSON 后重跑；失败时落盘 `Shanghai_folder_config_import_errors.json`，与 Device 同款）
- **幂等**：重复执行安全，同名对象自动跳过
- **不影响现网**：配置仅暂存 SCM 云上，无任何设备操作

#### 第 4 步 · 按清单完成剩余 UI 配置

打开 `Shanghai_folder_migration_summary.md` 逐项勾选（与 `Shanghai_device_migration_summary.md` 对称，`ls Shanghai_*` 即归组）。其中接口/隧道/IKE/IPsec/路由等设备级项已改由阶段 B 脚本自动导入，本步只需处理脚本覆盖不了的部分：

- [ ] HA 主备（API 只读无法写入）
- [ ] GlobalProtect 网关/门户
- [ ] 日志设置/日志转发
- [ ] 管理设置

> ✅ **阶段 A 完成**：folder 配置已在 SCM 中，可在 UI 校验，不影响现网。验收通过后进入阶段 B。

### 阶段 B · 设备级步骤（设备接入，需变更窗口）

#### 第 5 步 · Onboard 防火墙

1. SCM `Configuration > Onboarding`，将防火墙 onboard 到 `Shanghai`（其他站点 onboard 到对应 folder）
2. 确认防火墙已注册、连通正常（第 8 步 device 容器导入依赖设备注册）

> ⚠️ Onboard 时防火墙切换到 ARE 路由模式并重启，请在变更窗口执行。

#### 第 6 步 · 解析设备级 XML

```bash
python3 parse_panos_device_xml.py shanghai-base-new1.xml --folder Shanghai
# 其他站点例: python3 parse_panos_device_xml.py wuxi.xml --folder Wuxi
```

> 默认产出 `Shanghai_device_config.json` 与 `Shanghai_device_migration_summary.md`（统一 folder 前缀，与 folder 对称；不传 --folder 时回退 `<xml名>_device_config.json` 兼容），实测含 7×eth + 6×tunnel + 2×mgt profile + 10 zone + 3 VPN + 2 逻辑路由器（45 项）。

#### 第 7 步 · 干跑预览（不发任何网络请求）

```bash
python3 import_device_to_scm.py Shanghai_device_config.json --device <设备序列号> --folder Shanghai
# 例: python3 import_device_to_scm.py Shanghai_device_config.json --device 0123A456789 --folder Shanghai
```

> 显示各类型数量、目标容器（接口 → folder 变量，其余 → device）与设备序列号。**确认无误后再进入第 8 步。**

#### 第 8 步 · 正式导入（--apply）

```bash
python3 import_device_to_scm.py Shanghai_device_config.json --device <设备序列号> --folder Shanghai --apply
```

- **接口变量模型（官方要求）**：接口类对象（ethernet/tunnel/loopback）与管理 profile 创建于 folder 层作为"Layer 3 接口变量"并带默认接口赋值（默认配置自动复用+更新）；逻辑路由器/IKE 网关/IPsec 隧道引用这些 `$变量名`——官方《Configure Layer 3 Interfaces (SCM)》明确"选择 folder/snippet 即创建接口变量"；设备容器内直建接口按裸名引用会报 400 "is not a valid reference"
- 按依赖顺序导入：interface_management_profiles → ethernet_interfaces → loopback_interfaces → tunnel_interfaces → zones（成员自动改写为 `$接口变量`）→ logical_routers（自动声明 ARE 路由栈）→ ike_crypto_profiles → ipsec_crypto_profiles → ike_gateways → ipsec_tunnels
- **Zone 成员修复**：zone 创建于 folder 层，`network.layer3` 为 SCM 字符串数组格式（非 PAN-OS 的 member 对象）；同名 zone 已存在时 PUT 精确替换成员；导入前自动清理他 scope（如 ngfw-shared onboarding 脚手架）对本站点接口变量的错误占用，避免 409 already in use
- **VRF 命名（实测结论）**：租户仅放行 VRF 名 `default`，自定义名均报 INVALID_KEYWORD（多 VRF 命名需联系 PANW 开通 ARE 能力）。脚本自动按方案链改名重试（`vrf_前缀 → LR加-scm后缀 → 统一default`）。**转换细节**：PAN-OS 中 `virtual-router ISP1` 的 `vrf.name` 同名 `ISP1` 会先静默小写 `ISP1→isp1`（`import_device_to_scm.py:480`，CLI不再单独提示，仅内部转换）再归一 `isp1→default`；`逻辑路由器 Name` 列仍为 `ISP1` 大写不变（VRF名在详情页），`test` 同理 `test→default`
- **folder 层限制（实测结论）**：device 容器局部配置**不解析 `$接口变量`**（push 报 "interface '$xxx' is not a valid reference"），逻辑路由器对 device 容器写入返回 403——故接口变量、zone、逻辑路由器、IKE 网关、IPsec 隧道全部创建于 folder 层；仅 crypto profiles 等无变量引用的对象留在 device 容器。VPN 对象迁移后脚本自动清理 device 容器旧同名对象（残留会导致 push 失败且与 folder 副本名字冲突）
- **`?device=` 查询的"视图对象"（勿删）**：带 `?device=` 参数查询 ethernet/tunnel 接口与 interface-management-profiles 时，返回结果包含 folder 层对象的设备级解析视图——接口显示为 default_value 裸名（id 与 folder 变量相同）、管理 profile 带 `override:true + override_id:<folder id>`（值与 folder 层一致）。这些是 SCM 自动生成的下发关联，**不是残留，删除会破坏推送关联**；仅当 device 层对象 id 与 folder 不同且无 override 字段时才是真残留（如早期误建的 VPN 对象）。另见 `redist-collector-profile`（All/default）与 `auto-vpn-ping-profile`（ngfw-shared/Auto-VPN）等继承默认项，同 onboarding 脚手架 zone 一样无害、无需处理
- **管理 profile 保真度**：PAN-OS interface-management-profile 支持 ping/http/https/ssh/snmp/telnet/response-pages 七种协议；解析器已含全部七项（snmp 曾遗漏导致 folder 层 mgt profile 丢 snmp=yes，2026-08-26 已修复并回写 SCM）
- **同名已存在对象**：逻辑路由器已存在时自动 PUT 合并更新（接口取并集、路由/ECMP/管理距离以清单为准）；静态路由名含空格自动转下划线（SCM 名称规则）
- 失败项完整报错写入 `Shanghai_device_config_import_errors.json` / `Shanghai_folder_config_import_errors.json`（含未截断的原始报文，可直接用于排查或提交 PANW 支持；不传 --folder 时为 `<xml名>_device_config_import_errors.json` / `<xml名>_folder_config_import_errors.json`；Folder 与 Device 同款落盘）
- **幂等**：同名对象 409 自动跳过，可安全重跑
- ⚠️ **pre-shared-key** 为 PAN-OS 加密密文，导入后在 UI 确认/更新 IKE 网关密钥

#### 第 9 步 · 推送与验证

1. SCM 中 **Push Config** 推送到该 folder（如 `Shanghai`）
2. Overview 查 Sync Status，用 **Config Diff** 对比设备本地与云端
3. Config Diff 有差异 → 按差异补配后重新 Push

> ✅ **阶段 B 完成**：该站点迁移收尾。

#### 第 10 步 · 下一站点

重复阶段 A → 阶段 B，各站点独立执行；阶段 A 验收通过后再安排阶段 B。其他站点示例：

```bash
python3 parse_panos_folder_xml.py wuxi.xml --folder Wuxi
python3 import_folder_to_scm.py Wuxi_folder_config.json --folder Wuxi --apply
python3 parse_panos_device_xml.py wuxi.xml --folder Wuxi
python3 import_device_to_scm.py Wuxi_device_config.json --device <Wuxi序列号> --folder Wuxi --apply
```

---

## 四、按类型选择性导入（--only）

不加 `--only` = 全量导入；加 `--only` = 只导入指定类型（可重复使用、自由组合，同样以 `--apply` 执行正式导入）。

### 支持的类型

**对象类**
- `tags` 标签
- `addresses` 地址对象
- `address_groups` 地址组
- `services` 服务对象
- `service_groups` 服务组
- `url_categories` URL 分类
- `zones` 安全域

**策略类**
- `pbf_rules` 策略转发
- `security_rules` 安全策略
- `nat_rules` NAT 策略

### 组合示例（以 Shanghai 为例）

```bash
# 只导入网络对象（地址+组+服务+域）
python3 import_folder_to_scm.py Shanghai_folder_config.json --folder Shanghai \
  --only addresses --only address_groups --only services --only service_groups --only zones --apply

# 只导入策略（安全+NAT+PBF）
python3 import_folder_to_scm.py Shanghai_folder_config.json --folder Shanghai \
  --only security_rules --only nat_rules --only pbf_rules --apply

# 只导入地址
python3 import_folder_to_scm.py Shanghai_folder_config.json --folder Shanghai --only addresses --apply

# 只导入服务
python3 import_folder_to_scm.py Shanghai_folder_config.json --folder Shanghai --only services --apply
```

> 提示：`--only` 同样支持 dry-run 预览；类型有依赖（策略引用 Zone/地址）时建议先对象后策略。

---

## 五、其他常用变体

```bash
# 自定义输出文件名 (-o / --summary 需带文件名值, 省略则默认 Shanghai_folder_config.json / Shanghai_folder_migration_summary.md)
python3 parse_panos_folder_xml.py shanghai-base-new1.xml --folder Shanghai -o my_import.json --summary my_summary.md

# 其他站点各自解析导入 (统一 folder 前缀)
python3 parse_panos_folder_xml.py wuxi.xml --folder Wuxi
# 产出 Wuxi_folder_config.json + Wuxi_folder_migration_summary.md
python3 parse_panos_device_xml.py wuxi.xml --folder Wuxi
# 产出 Wuxi_device_config.json + Wuxi_device_migration_summary.md

# 指定 SCM 环境（默认生产）
SCM_API_URL=https://api.strata.paloaltonetworks.com python3 import_folder_to_scm.py Shanghai_folder_config.json --folder Shanghai --apply
```

---

## 六、常见错误排查

- **缺少凭证** — 环境变量未设置 → `source ~/.zshrc` 后重跑；`env | grep SCM_` 验证
- **HTTP 401** — token 过期/无效 → 脚本自动重试认证；检查 `SCM_CLIENT_ID/SECRET/TSG_ID` 是否与 IAM 一致
- **HTTP 403** — 角色权限不足 → IAM 升级（需配置写权限）
- **HTTP 409** — 对象已存在 → 正常，幂等跳过
- **HTTP 400/422** — 字段校验失败（如 VRF 名、变量引用）→ 按 `*_import_errors.json` 原始报文修正 JSON 重跑
- **Zone 找不到** — Zone 未导入 → 先 `--only zones --apply`
- **Config Diff 有差异** — 第二类未完成 → 补 UI 配置后重新 Push

---

## 七、验证记录（模板：shanghai-base-new1.xml → Shanghai）

> 干跑验证于 2026-08-28 完成，无需凭证与网络，仅解析与本地校验。

```text
Folder 解析: tags 1, addresses 501, address_groups 34, services 64, service_groups 11,
            url_categories 4, zones 10, security_rules 186, nat_rules 49, pbf_rules 64
            → 合计 924 项，产出 Shanghai_folder_config.json (266KB)

Folder 干跑: DRY-RUN 全部类型预览通过，目标 folder:Shanghai

Device 解析: interface_management_profiles 2, ethernet_interfaces 7, tunnel_interfaces 6,
            zones 10, ike_crypto_profiles 6, ipsec_crypto_profiles 6,
            ike_gateways 3, ipsec_tunnels 3, logical_routers 2 → 共 45 项

Device 干跑: DRY-RUN 45 项预览通过，接口变量容器 folder/Shanghai，序列号占位符校验通过
```

> 后续站点替换 `<站点>.xml` / `<站点folder>` 后，同样先执行上述 4 个干跑步骤（Folder 解析→Folder干跑→Device解析→Device干跑）确认后再 `--apply`。
