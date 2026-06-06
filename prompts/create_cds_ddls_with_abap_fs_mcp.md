# Create CDS DDLS With ABAP FS MCP

Copy this prompt into Codex when starting a CDS Data Definition / DDLS task.

```text
Please first read and strictly follow this workflow document:

docs/codex/ABAP_FS_MCP_Codex_CDS_Data_Definition.md

Task:
Create / maintain CDS Data Definition objects.

Variables:
<CONNECTION_ID> = <CONNECTION_ID>
<PACKAGE> = <PACKAGE>
<TABLE_FUNCTION_DDLS> = <TABLE_FUNCTION_DDLS>
<AMDP_CLASS> = <AMDP_CLASS>
<QUERY_DDLS> = <QUERY_DDLS>
<TRANSPORT_TASK> = <TRANSPORT_TASK>
<OUTPUT_DIR> = outputs/

Execution requirements:
1. First call get_connected_systems.
2. Then call search_abap_objects for all target objects.
3. Read existing source before changing anything.
4. DDLS objects must be created only as CDS Data Definitions.
5. Do not create ZTF_* as Table.
6. Do not use Table / TABL / DT for CDS Table Function.
7. Use Class only for ABAP Class or AMDP Class.
8. Use Service Definition or Service Binding only if explicitly requested.
9. Do not enable screenshots, browser automation, visual automation, or VS Code UI automation by default.
10. If a popup, QuickPick, package selector, transport selector, save prompt, activation prompt, SAP login confirmation, or CREATION_CANCELLED occurs, stop and give me exact VS Code manual steps.
11. After I reply "continue", "done", or "next", verify the SAP object state again through MCP tools.
12. Run ATC analysis when appropriate or when requested.
13. Report inspected objects, changed objects, object types, verification results, and any remaining manual steps.
14. Do not expose real company system information in outputs.
```

Optional DEMO values for public examples:

```text
<PACKAGE> = ZDEMO_PACKAGE
<TABLE_FUNCTION_DDLS> = ZTF_DEMO_FI_AR_OVD
<AMDP_CLASS> = ZCL_DEMO_FI_AR_OVD_AMDP
<QUERY_DDLS> = ZC_DEMO_FI_AR_OVD_QRY
```
