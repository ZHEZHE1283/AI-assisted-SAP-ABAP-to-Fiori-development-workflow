# ABAP FS MCP / Codex 通用蒸馏：创建 CDS Data Definition / CDS Table Function / CDS Query View

生成时间：2026-06-05  
协作模式优化：2026-06-06  
适用场景：通过 VS Code ABAP FS MCP 创建或维护 ABAP CDS Data Definition（DDLS）对象  
适用对象类型：CDS View、CDS Table Function、CDS Query View、Consumption View  
不绑定单一业务对象：本文件不是只服务于 `ZTF_DEMO_FI_AR_OVD`，以后 `ZTF_DEMO_FI_AR_OVE`、`ZTF_DEMO_FI_AR_OVF`、`ZTF_MM_xxx`、`ZC_DEMO_xxx_QRY` 等同类对象都可以套用。  

---

## 1. 结论先放前面

本蒸馏文件沉淀的是一套**通用操作方法**，不是某一次具体对象的救火记录。

以后凡是遇到下面这类对象：

```text
CDS View
CDS Table Function
CDS Query View
Consumption View
```

都不要按普通透明表思路创建。

默认入口是 ABAP FS MCP 后台工具：

```text
先用 search_abap_objects / get_abap_object_info 确认对象状态。
如需创建，可以先尝试 create_object_programmatically。
但只要创建步骤触发 VS Code / ADT 弹窗、返回 CREATION_CANCELLED、或需要包/传输人工确认，
默认立刻切换成人机协作暂停点。
不要默认启用视觉或 VS Code UI 自动化。
```

暂停点的默认协作方式是：

```text
Codex 给出精确的 VS Code 手动操作步骤。
用户在 VS Code 里完成创建/选择包/选择传输/保存/激活等弹窗确认。
用户完成后回复：继续 / 已完成 / 下一步。
Codex 再回到 MCP 后台核验对象状态，并继续写源码、ATC、激活检查。
```

如果后台创建失败，默认先让用户手动使用：

```text
VS Code ABAP FS Explorer
-> 找到目标包
-> 右键目标包
-> ABAP Create object
-> 类型选择 CDS Data Definitions
```

注意 ABAP FS UI 兜底时，可视化列表里显示的是复数：

```text
CDS Data Definitions
```

不要选错成：

```text
Table
```

不要走 SAP 官方 ADT 扩展命令：

```text
adt-vscode.createObject
```

如果用户明确授权 Codex 代操作 VS Code UI，并且确实需要命令兜底，使用 ABAP FS 命令：

```text
abapfs.create
```

但 `abapfs.create` 属于 VS Code UI/命令兜底，只在用户明确要求 Codex 代操作 VS Code UI 时才由 Codex 自动触发。

---

## 2. 为什么不能把 Table Function 当 Table 创建

CDS Table Function 的名字里虽然带 `Table Function`，但它不是普通透明表。

它的 ABAP Repository 对象类型仍然是：

```text
DDLS
```

也就是 CDS Data Definition。

所以：

```text
ZTF_*
ZC_*_QRY
ZI_*
```

这类 CDS 源对象，创建入口都应该优先判断为：

```text
CDS Data Definitions
```

而不是：

```text
Table
TABL
DT
Structure
```

---

## 3. UI 兜底时的 ABAP FS 创建选项

这一段必须保留。

原因是：如果用户明确授权 Codex 做 VS Code UI 自动化，最容易出错的不是源码，而是**创建对象时选错类型**。

下面是通过 VS Code 里 ABAP FS 的 `ABAP Create object` QuickPick 确认到的对象类型。
它只作为 UI 兜底参考，不代表默认要启用视觉。

> 注意：表格里的“本次建议”分成两层理解：  
> 1. **通用建议**：以后类似 DDLS 创建任务都适用。  
> 2. **案例示例**：以 `ZTF_DEMO_FI_AR_OVD` / `ZC_DEMO_FI_AR_OVD_QRY` 为例，但不代表以后只能创建 OVD。

| UI 选项 | 主要用途 | 通用建议 | 案例示例 |
| --- | --- | --- | --- |
| `Authorization field` | 权限字段 | DDLS 创建任务不用，除非用户明确要求做权限对象 | 本次 OVD 不用 |
| `Authorization object` | 权限对象 | DDLS 创建任务不用，除非用户明确要求做权限控制 | 本次 OVD 不用 |
| `Behavior Definition` | RAP Behavior Definition | RAP 才用；当前 Fiori Elements V2 + OData V2 / DDLS 路线不用 | 本次 OVD 不用 |
| `CDS Access Control` | CDS DCL 权限控制 | 只有要创建 DCL 权限控制时才用；不要拿它创建 CDS View / Table Function | 本次 OVD 不用 |
| `CDS Annotation definitions` | CDS 注解定义 | 这是注解定义对象，不是普通 CDS View 源对象 | 本次 OVD 不用 |
| `CDS Data Definitions` | CDS View / CDS Table Function / Query View 源对象 | **创建 `ZTF_*`、`ZI_*`、`ZC_*` 这类 CDS 源对象必须优先选它** | 创建 `ZTF_DEMO_FI_AR_OVD`、`ZC_DEMO_FI_AR_OVD_QRY` 必须选它 |
| `CDS metadata extensions` | CDS Metadata Extension | 只有单独做 Metadata Extension 时才用；不要拿它创建主 CDS | 本次 OVD 不用 |
| `Class` | ABAP Class | 只有要创建 AMDP Class / 普通 Class 时才用；已存在就不要重建 | `ZCL_DEMO_FI_AR_OVD_AMDP` 已存在，不要重建 |
| `Data Element` | 数据元素 | DDIC 基础对象；除非用户明确要求补数据元素，否则不用 | 本次 OVD 不用 |
| `Domain` | 域 | DDIC 基础对象；除非用户明确要求补域，否则不用 | 本次 OVD 不用 |
| `Function Group` | 函数组 | 传统函数模块开发才用 | 本次 OVD 不用 |
| `Function group include` | 函数组 include | 传统函数组内部 include 才用 | 本次 OVD 不用 |
| `Function module` | 函数模块 | 传统 FM 开发才用 | 本次 OVD 不用 |
| `Include` | Include 程序 | 传统 Report 拆分才用；DDLS 创建不用 | 本次 OVD 不用 |
| `Interface` | ABAP Interface | ABAP OO 接口才用 | 本次 OVD 不用 |
| `Message class` | 消息类 | 消息号 / MESSAGE 文本维护才用 | 本次 OVD 不用 |
| `Package` | 包 | 只有要创建新包才用；已有业务包不要重建 | `<PACKAGE>` 已存在，不要重建 |
| `Program` | 报表 / 程序 | 传统 Report / Executable Program 才用；不要拿它创建 CDS | 本次 OVD 不用 |
| `Service binding` | OData 服务绑定 | 后续发布服务时可能要创建；不要提前创建，除非用户明确要求 | 后续如果要发布服务再创建 |
| `Service definition` | OData 服务定义 | 后续暴露 CDS 服务时可能要创建；不要误拿它创建 Query View | 后续如果要发布服务再创建 |
| `Structure` | DDIC 结构 | DDIC 结构对象；不是 CDS Table Function | 本次 OVD 不用 |
| `Table` | 数据库表 / 结构类 DDIC 对象 | **不要用于 CDS Table Function；看到 `ZTF_*` 也不能想当然选 Table** | 不要用于 `ZTF_DEMO_FI_AR_OVD` |

原蒸馏记录里的关键结论也要保留：

```text
ABAP FS 创建对象支持程序、类、接口、函数组、数据元素、域、数据库表、CDS、消息类、包等类型。

但 programmatic creation 仍可能弹出包 / 传输请求对话框，不是完全无交互。

因此 Codex 不能假装已经完全自动创建成功。
如果 UI 弹窗需要用户确认，必须明确告诉用户当前卡在哪一步。
```

### 3.1 这张表以后怎么用

以后用户说：

```text
创建 ZTF_DEMO_FI_AR_OVE
创建 ZTF_DEMO_FI_AR_OVF
创建 ZC_DEMO_FI_AR_OVE_QRY
创建 ZC_DEMO_FI_AR_OVF_QRY
```

Codex 应该自动判断：

```text
这些都是 DDLS / CDS Data Definition 路线。
应该选 CDS Data Definitions。
不能选 Table。
不能选 Service definition。
不能选 Program。
```

如果用户说：

```text
创建 ZCL_DEMO_FI_AR_OVE_AMDP
```

Codex 才能判断：

```text
这是 ABAP Class。
应该选 Class。
但创建前必须 search_abap_objects 检查是否已存在。
```

如果用户说：

```text
创建服务定义
创建服务绑定
发布 OData
```

Codex 才能考虑：

```text
Service definition
Service binding
```

但不能在用户只要求创建 DDLS 时，擅自创建服务对象。

---

## 4. 通用变量命名

以后不要把本次对象名直接写死在通用提示词里。

统一使用下面这些变量：

```text
<CONNECTION_ID>             ABAP FS 连接，例如 <CONNECTION_ID>

<PACKAGE>                   目标包，例如 <PACKAGE>

<BASE_REPORT>               参考的原始报表，例如 ZDEMO_REPORT
                            如果本次不是报表迁移，可以为空

<TABLE_FUNCTION_DDLS>       CDS Table Function 的 DDLS 名称
                            例如 ZTF_DEMO_FI_AR_OVD / ZTF_DEMO_FI_AR_OVE / ZTF_DEMO_FI_AR_OVF

<AMDP_CLASS>                Table Function 对应的 AMDP Class
                            例如 ZCL_DEMO_FI_AR_OVD_AMDP / ZCL_DEMO_FI_AR_OVE_AMDP

<QUERY_DDLS>                Query View / Consumption View 的 DDLS 名称
                            例如 ZC_DEMO_FI_AR_OVD_QRY / ZC_DEMO_FI_AR_OVE_QRY

<TARGET_DDLS>               当前要创建或维护的任意 DDLS 对象

<TRANSPORT_TASK>            当前传输请求或任务，例如 <TRANSPORT_TASK>

<OUTPUT_DIR>                Codex 输出源码目录，例如 outputs/

<SOURCE_FILE>               准备写入对象的本地源码文件
                            例如 outputs/<TARGET_DDLS>.ddls.asddls
```

---

## 5. 通用执行纪律

以后交给 Codex / MCP 执行时，必须先声明下面这些纪律。

```text
你现在通过 VS Code ABAP FS MCP 访问 SAP。

【连接约束】
1. 优先使用用户本次指定的 ABAP FS 连接，例如 <CONNECTION_ID>。
2. 必须先调用 get_connected_systems 确认连接可用。
3. 不要假设对象已经存在，必须先 search_abap_objects 检查。

【范围约束】
1. 只允许处理用户本次明确指定的包、类、DDLS、服务对象。
2. 不允许擅自修改用户未指定的对象。
3. 如果需要新增 Service Definition、Service Binding、DDIC Table、Class、Program、Include，必须先向用户确认。
4. 修改原则：最小改动、先跑通、不重构、不顺手优化无关逻辑。
5. 不默认启用视觉、截图、Computer Use 或 VS Code UI 自动化。

【CDS Data Definition 创建规则】
1. 凡是 CDS View、CDS Table Function、CDS Query View，本质都应创建为 DDLS / CDS Data Definition。
2. 不要把 CDS Table Function 当成普通透明表创建。
3. 不要选择 Table / TABL / DT。
4. 后台优先路径是：
   create_object_programmatically，objectType 使用 DDLS/DF。
5. 如果后台创建失败、返回 CREATION_CANCELLED、或需要 VS Code 弹窗确认：
   默认不要继续自动化。
   先暂停，把创建动作交给用户手动完成。
   用户回复“继续 / 已完成 / 下一步”后，再用 MCP 搜索核验。
6. 手动创建时，VS Code ABAP FS Explorer 中路径是：
   包右键 -> ABAP Create object -> CDS Data Definitions。
7. QuickPick 中正确选项是复数形式：
   CDS Data Definitions。
8. 不要调用 SAP 官方 ADT 命令：
   adt-vscode.createObject。
9. 只有用户明确要求 Codex 代操作 VS Code UI 时，才考虑调用 ABAP FS 命令：
   abapfs.create。
```

---

## 6. 通用操作路线：创建任意 DDLS 对象

下面这套步骤适用于：

```text
<TABLE_FUNCTION_DDLS>
<QUERY_DDLS>
ZI_* View
ZC_* Consumption View
其他 CDS Data Definition
```

### 6.1 确认连接

```text
调用 get_connected_systems

期望看到：
<CONNECTION_ID>
```

### 6.2 检查目标包

```text
检查 <PACKAGE> 是否存在。
不要在未确认包存在的情况下创建对象。
```

### 6.3 检查目标 DDLS 是否已经存在

```text
调用 search_abap_objects

connectionId = <CONNECTION_ID>
pattern      = <TARGET_DDLS>
types        = DDLS
```

判断规则：

```text
如果找到 <TARGET_DDLS>：
    读取对象信息，确认是否是用户本次目标对象。
    不要重复创建。

如果没有找到 <TARGET_DDLS>：
    才进入创建流程。
```

### 6.4 使用 ABAP FS 创建对象

```text
默认先用 MCP 后台创建：
create_object_programmatically
objectType = DDLS/DF
name = <TARGET_DDLS>
description = 按用户本次业务填写
packageName = <PACKAGE>
transportRequest = 用户指定的 <TRANSPORT_TASK>

如果后台创建成功：
立刻 search_abap_objects 核验 <TARGET_DDLS> 是否存在。
存在后再进入写源码步骤。

如果后台创建失败 / CREATION_CANCELLED / 需要弹窗：
1. 先报告失败原因和当前状态。
2. 不默认启用视觉。
3. 不默认进入 VS Code UI。
4. 给用户一个明确的手动创建任务，只创建当前这一个 <TARGET_DDLS>。
5. 要求用户完成后回复：继续 / 已完成 / 下一步。
6. 用户回复后，Codex 必须重新 search_abap_objects 核验对象存在。

手动创建路径：
VS Code ABAP FS Explorer -> <PACKAGE> 右键 -> ABAP Create object -> CDS Data Definitions。
```

创建时不要选：

```text
Table
Structure
Service definition
Service binding
Program
adt-vscode.createObject
```

给用户的暂停提示建议固定成：

```text
当前卡点：MCP 后台创建 <TARGET_DDLS> 返回 <ERROR>，需要 VS Code 弹窗确认。

请在 VS Code 操作：
1. 打开 ABAP FS Explorer，确认连接是 <CONNECTION_ID>。
2. 找到包 <PACKAGE>。
3. 右键 <PACKAGE>，选择 ABAP Create object。
4. 类型选择 CDS Data Definitions。
5. 对象名填 <TARGET_DDLS>，描述填 <DESCRIPTION>。
6. 包/传输弹窗选择 <PACKAGE> 和 <TRANSPORT_TASK>。
7. 创建完成后先不要改无关对象。

预期结果：<TARGET_DDLS> 出现在包 <PACKAGE> 的 Core Data Services / Data Definitions 下。
完成后回复：继续 / 已完成 / 下一步。
```

### 6.5 写入源码

创建完成后，将本地准备好的源码写入新对象。

源码文件建议命名为：

```text
<OUTPUT_DIR>/<TARGET_DDLS>.ddls.asddls
```

ABAP FS workspace URI 通常类似：

```text
adt://<CONNECTION_ID>/System Library/<PACKAGE>/Core Data Services/Data Definitions/<TARGET_DDLS>.ddls.asddls
```

如果读取 inactive 源码，优先使用：

```text
/source/main?version=inactive
```

如果读取 active 源码，使用：

```text
/source/main?version=active
```

---

## 7. Table Function + AMDP + Query View 的通用激活顺序

如果本次结构是：

```text
<TABLE_FUNCTION_DDLS>
<AMDP_CLASS>
<QUERY_DDLS>
```

建议按依赖关系激活：

```text
1. <TABLE_FUNCTION_DDLS>
2. <AMDP_CLASS>
3. <QUERY_DDLS>
```

原因：

```text
Query View 依赖 Table Function。
Table Function 的实现依赖 AMDP Class。
但 AMDP Class 里的 FOR TABLE FUNCTION 又需要 Table Function 的定义先存在。
```

如果 ABAP FS 弹出 Mass Activation 对话框，可以把本次相关 inactive 对象一起勾选。

但不要勾选无关对象。

---

## 8. MCP 工具使用边界

适合直接用 MCP 工具做的事情：

```text
get_connected_systems
search_abap_objects
get_abap_object_info
get_abap_object_lines
get_object_by_uri
get_abap_object_workspace_uri
run_atc_analysis
manage_transport_requests
execute_data_query
```

查询 SAP 表前必须先调用：

```text
get_abap_sql_syntax
```

创建对象时要谨慎：

```text
create_object_programmatically 名字看起来是全自动，
但 ABAP FS 文档说明它仍可能弹传输/包选择对话框。
因此默认路线应改成：

1. 先走 ABAP FS MCP 后台工具确认对象、读取源码、运行 ATC、准备源码。
2. 能用 MCP programmatic creation 创建的，优先后台创建。
3. 如果创建/保存/激活必须进入 VS Code UI 或弹窗，先停下来告诉用户卡在哪一步。
4. 不默认启用视觉/截图/Computer Use；只有用户明确授权时，才做 UI 自动化。
```

---

## 9. 后台优先 / UI 兜底注意事项

默认原则：

```text
能用 ABAP FS MCP 后台工具完成的，全部后台完成。
不要默认启用视觉。
不要默认截图。
不要默认使用 Computer Use / VS Code UI 自动化。
```

优先使用的 MCP 工具包括：

```text
get_connected_systems
search_abap_objects
get_abap_object_info
get_abap_object_lines
search_abap_object_lines
find_where_used
run_atc_analysis
get_abap_object_workspace_uri
create_object_programmatically
manage_transport_requests
```

注意 MCP 边界：

```text
外部 MCP 工具可以很好地搜索对象、读源码、跑 ATC、创建部分对象、分析依赖。
但对 VS Code 虚拟 adt:// 文件的直接编辑、实时 Problems 诊断、QuickPick 弹窗确认，可能仍属于 VS Code UI 能力。
```

遇到 MCP 做不到的动作时：

```text
先报告限制和当前阻塞点。
默认给用户一个可执行的 VS Code 手动任务。
不要把“需要弹窗确认”直接升级成“Codex 自动操作 UI”。

只有用户明确说：
你来操作 VS Code UI / 可以开视觉 / 可以用 Computer Use
才允许 Codex 启用视觉或 Computer Use 做 UI 自动化。
```

推荐默认选择是：用户手动处理 VS Code 弹窗，Codex 等待用户回复“继续 / 已完成 / 下一步”，然后继续用 MCP 后台核验。

### 9.1 人机协作开发模式

这套模板的最终目标不是纯后台演示，而是在 VS Code 里高效开发：

```text
ABAP 常规程序
ABAP Class / AMDP Class
CDS View
CDS Table Function
CDS Query / Consumption View
OData Service Definition / Service Binding
```

默认协作边界：

```text
Codex 负责：
1. 用 MCP 搜索对象、读取源码、分析依赖。
2. 生成或修改源码草案。
3. 运行 ATC / 查询对象状态 / 读取错误文档。
4. 判断需要创建、保存、激活、传输还是发布。
5. 给出精确的 VS Code 手动操作提示。
6. 用户完成后，用 MCP 重新读取后台状态核验。

用户负责：
1. SAP 登录、权限确认、传输请求弹窗、包选择弹窗。
2. VS Code QuickPick / 弹窗中需要人工确认的步骤。
3. 手动粘贴或保存 Codex 给出的源码，当 MCP 无法直接写入 adt:// 文件时。
4. 完成后回复“继续 / 已完成 / 下一步”。
```

当需要用户手动操作时，Codex 的提示格式应固定：

```text
当前卡点：说明为什么 MCP 做不到。
请在 VS Code 操作：列出 1-5 个具体步骤。
预期结果：用户完成后应该看到什么或后台对象应达到什么状态。
完成后回复：继续 / 已完成 / 下一步。
```

重要约束：

```text
不要为了等待弹窗完成而启用视觉。
不要用截图轮询 VS Code 状态。
用户说“继续 / 已完成 / 下一步”后，再用 MCP 后台工具确认对象是否创建、保存、激活或 ATC 是否通过。
只有用户明确要求 Codex 代操作 VS Code UI 时，才启用视觉 / Computer Use。
```

只有在用户明确允许 UI 自动化后，才参考下面规则。

要调用 ABAP FS 创建命令，用：

```text
abapfs.create
```

不要用：

```text
adt-vscode.createObject
```

`adt-vscode.createObject` 是 SAP 官方 ADT 扩展命令，会先弹 Destination / Logon 选择，不是本次 ABAP FS Explorer 里的创建路线。

如果通过键盘或 SendKeys 输入 QuickPick 过滤词，要注意中文输入法会干扰英文输入。

更稳的做法是：

```text
先把英文过滤词放到剪贴板
再 Ctrl+V 粘贴
```

推荐粘贴的过滤词是：

```text
CDS Data Definitions
```

不要盲目用方向键猜位置，因为焦点一错就可能触发 VS Code 的 Go to Line 或别的输入框。

---

## 10. 给下一轮 Codex 的通用提示词

以后可以直接复制下面这一段给 Codex。

```text
你现在通过 VS Code ABAP FS MCP 访问 SAP。

【本次变量】
<CONNECTION_ID> = 请使用用户本次指定的连接
<PACKAGE> = 请使用用户本次指定的包
<TABLE_FUNCTION_DDLS> = 请使用用户本次指定的 Table Function DDLS；如果本次没有则留空
<AMDP_CLASS> = 请使用用户本次指定的 AMDP Class；如果本次没有则留空
<QUERY_DDLS> = 请使用用户本次指定的 Query / Consumption DDLS；如果本次没有则留空
<TRANSPORT_TASK> = 请使用用户本次指定的传输请求或任务
<OUTPUT_DIR> = 请使用用户本次指定的源码输出目录

【连接约束】
1. 先调用 get_connected_systems，确认 <CONNECTION_ID> 可用。
2. 不要猜连接，不要切换到用户未指定的系统。

【范围约束】
1. 只允许处理用户本次明确指定的对象。
2. 不允许擅自修改其他包、其他类、其他 DDLS、其他服务对象。
3. 如果需要新增 Service Definition、Service Binding、DDIC Table、Class、Program、Include，必须先问用户。
4. 修改原则：最小改动、先跑通、不重构、不顺手优化无关逻辑。
5. 不默认启用视觉、截图、Computer Use 或 VS Code UI 自动化。
6. 只有用户明确授权 UI 自动化时，才允许进入 VS Code 界面操作。

【人机协作开发模式】
1. 默认由 Codex 使用 ABAP FS MCP 后台工具完成搜索、读取、分析、ATC、对象状态核验和源码准备。
2. 如果遇到 SAP 登录、权限确认、传输请求、包选择、QuickPick、保存/激活弹窗等必须在 VS Code 中确认的步骤：
   不启用视觉，不截图，不代点。
   先暂停并给用户手动操作指引。
3. 给用户的手动操作指引必须包含：
   当前卡点、请在 VS Code 操作、预期结果、完成后回复“继续 / 已完成 / 下一步”。
4. 用户回复“继续 / 已完成 / 下一步”后，Codex 必须回到 MCP 后台工具重新核验对象状态、源码、ATC 或激活结果。
5. 只有用户明确说“你来操作 VS Code UI / 可以开视觉 / 可以用 Computer Use”时，才进入 UI 自动化。

【创建对象类型选择】
1. 如果目标是 CDS View / CDS Table Function / CDS Query View / Consumption View：
   选择 CDS Data Definitions。
2. 如果目标是 AMDP Class / 普通 ABAP Class：
   选择 Class。
3. 如果目标是 Service Definition：
   选择 Service definition。
4. 如果目标是 Service Binding：
   选择 Service binding。
5. 如果目标是普通透明表：
   才能选择 Table。
6. 不要看到 ZTF_* 就选择 Table，ZTF_* 在这里是 CDS Table Function 的 DDLS，不是 DDIC Table。

【创建 DDLS 的规则】
1. 凡是 CDS View、CDS Table Function、CDS Query View、Consumption View，都按 DDLS / CDS Data Definition 处理。
2. 不要把 CDS Table Function 当成普通 Table 创建。
3. 不要选择 Table / TABL / DT。
4. 后台优先路径是：
   create_object_programmatically，objectType 使用 DDLS/DF。
5. 如果后台创建失败，且错误表明必须走 VS Code QuickPick / 弹窗：
   默认先暂停并交给用户手动创建。
   不要默认询问是否允许 UI 自动化。
   不要默认启用视觉 / Computer Use。
6. 手动创建路径是：
   VS Code ABAP FS Explorer -> <PACKAGE> 右键 -> ABAP Create object -> CDS Data Definitions。
7. QuickPick 里正确显示名是：
   CDS Data Definitions。
8. 不要调用 SAP 官方 ADT 命令：
   adt-vscode.createObject。
9. 只有用户明确要求 Codex 代操作 VS Code UI 时，才考虑 ABAP FS 命令：
   abapfs.create。

【通用执行步骤】
1. 调用 get_connected_systems。
2. 检查 <PACKAGE> 是否存在。
3. 对每个本次目标 DDLS，先 search_abap_objects：
   pattern = <TARGET_DDLS>
   types = DDLS。
4. 如果 <TARGET_DDLS> 不存在：
   先询问用户是否允许创建该 DDLS。
5. 用户允许后，优先调用 create_object_programmatically：
   objectType = DDLS/DF。
   name = <TARGET_DDLS>。
   packageName = <PACKAGE>。
   description 按本次业务填写。
   transportRequest 使用 <TRANSPORT_TASK>。
6. 如果 create_object_programmatically 无法完成，且必须走 VS Code UI：
   停下来说明失败原因。
   给用户一个只创建当前 <TARGET_DDLS> 的 VS Code 手动操作清单。
   要求用户创建完成后回复“继续 / 已完成 / 下一步”。
   未获用户明确 UI 自动化授权时，不启用视觉，不截图，不操作 VS Code。
7. 创建完成后，写入：
   <OUTPUT_DIR>/<TARGET_DDLS>.ddls.asddls。
8. 如果存在 Table Function + AMDP + Query View 依赖，激活顺序为：
   <TABLE_FUNCTION_DDLS> -> <AMDP_CLASS> -> <QUERY_DDLS>。
9. 激活后运行 run_atc_analysis，优先检查用户本次指定的主对象。

【UI 自动化注意】
0. 本段只在用户明确授权 UI 自动化后适用。
1. QuickPick 输入英文过滤词时，优先用剪贴板粘贴。
2. 避免中文输入法干扰。
3. 不要盲目用方向键猜位置。
4. 过滤词建议使用：
   CDS Data Definitions。
```

---

## 11. 公开 DEMO 案例附录

本附录只用于说明如何把一次具体 DDLS 创建任务变量化。不要在公开仓库中保留真实公司系统、真实业务包、真实传输、真实对象状态、真实报表名、真实业务规则或本地用户目录。

### 11.1 DEMO 变量表

```text
<CONNECTION_ID>       = <CONNECTION_ID>
<PACKAGE>             = <PACKAGE>
<AMDP_CLASS>          = ZCL_DEMO_FI_AR_OVD_AMDP
<TABLE_FUNCTION_DDLS> = ZTF_DEMO_FI_AR_OVD
<QUERY_DDLS>          = ZC_DEMO_FI_AR_OVD_QRY
<TRANSPORT_TASK>      = <TRANSPORT_TASK>
<OUTPUT_DIR>          = outputs/
```

### 11.2 DEMO 对象状态

```text
ZCL_DEMO_FI_AR_OVD_AMDP 已存在于包 <PACKAGE>，类型是 CLAS/OC。
ZTF_DEMO_FI_AR_OVD 当前未在系统中找到。
ZC_DEMO_FI_AR_OVD_QRY 当前未在系统中找到。

使用 create_object_programmatically 创建 DDLS 时，如果返回 CREATION_CANCELLED，说明创建流程需要 VS Code / ADT 人工确认。
Codex 应暂停，给用户 VS Code 手动创建清单，用户创建完成后回复“继续”，Codex 再用 MCP 核验并写源码。
```

### 11.3 DEMO 源码文件

```text
outputs/ZTF_DEMO_FI_AR_OVD.ddls.asddls
outputs/ZC_DEMO_FI_AR_OVD_QRY.ddls.asddls
```

### 11.4 DEMO 激活顺序

```text
1. ZTF_DEMO_FI_AR_OVD
2. ZCL_DEMO_FI_AR_OVD_AMDP
3. ZC_DEMO_FI_AR_OVD_QRY
```

### 11.5 DEMO 后台修复要点

真实业务规则不应放入公开模板。公开文档只保留方法论：

```text
1. 先确认原始报表或旧逻辑的输入字段。
2. 再确认 CDS Table Function 输出字段是否覆盖这些字段。
3. AMDP 中从底层明细一路带到计算层。
4. Query View 只消费已经确认的 Table Function 字段。
5. 修改后按 Table Function -> AMDP Class -> Query View 顺序激活。
6. 最后跑 ATC 或至少读取对象状态核验。
```

---

## 12. 以后新增 OVE / OVF 时怎么套

如果下一次目标变成：

```text
ZTF_DEMO_FI_AR_OVE
ZC_DEMO_FI_AR_OVE_QRY
ZCL_DEMO_FI_AR_OVE_AMDP
```

只需要替换变量：

```text
<TABLE_FUNCTION_DDLS> = ZTF_DEMO_FI_AR_OVE
<QUERY_DDLS>          = ZC_DEMO_FI_AR_OVE_QRY
<AMDP_CLASS>          = ZCL_DEMO_FI_AR_OVE_AMDP
```

如果再下一次目标变成：

```text
ZTF_DEMO_FI_AR_OVF
ZC_DEMO_FI_AR_OVF_QRY
ZCL_DEMO_FI_AR_OVF_AMDP
```

同样只替换变量：

```text
<TABLE_FUNCTION_DDLS> = ZTF_DEMO_FI_AR_OVF
<QUERY_DDLS>          = ZC_DEMO_FI_AR_OVF_QRY
<AMDP_CLASS>          = ZCL_DEMO_FI_AR_OVF_AMDP
```

通用规则不变：

```text
CDS Table Function 不是 Table。
CDS View / Table Function / Query View 都走 CDS Data Definitions。
创建前先 search_abap_objects。
写入源码后按依赖激活。
Table Function -> AMDP Class -> Query View。
范围必须锁在用户本次指定对象内。
```

---

## 13. 最终沉淀口径

这份文件的核心不是：

```text
创建某个固定的真实业务对象。
```

而是：

```text
以后所有同类 DDLS 创建任务，都按变量化对象处理。
具体对象名只是本次案例参数，不是通用模板本身。
```

所以 Codex 后续执行时，必须先读取用户本次指定的变量，再套用这套流程。

不要把附录里的 DEMO 案例误认为永久固定目标。
