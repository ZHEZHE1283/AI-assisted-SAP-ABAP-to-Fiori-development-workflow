# AGENTS.md

## Project role

This repository is an AI-assisted SAP ABAP to Fiori workflow kit.

It documents how to use Codex with VS Code, ABAP FS MCP, CDS, OData V2, and Fiori Elements V2.

## Important safety rules

- Do not use or expose company real code.
- Do not expose SAP server IP, client, username, password, token, cookie, or transport-sensitive data.
- Use DEMO names in public examples.
- Prefer minimal changes.
- Do not refactor unrelated logic.
- Do not modify files or SAP objects outside the user-requested scope.

## ABAP FS MCP rules

When the task involves ABAP FS MCP, CDS Data Definition, CDS Table Function, CDS Query View, Consumption View, DDLS, AMDP, or OData V2, read this file first:

docs/codex/ABAP_FS_MCP_Codex_CDS_Data_Definition.md

Then follow its rules.

## Human-in-the-loop rule

If ABAP FS MCP cannot complete an operation because VS Code, ADT, package selection, transport request, QuickPick, save, activation, or SAP login confirmation is required:

1. Stop.
2. Explain the current blocking point.
3. Give the user exact VS Code manual steps.
4. Wait for the user to reply: continue / done / next.
5. After that, verify the SAP object state again through MCP tools.

Do not use screenshots, visual automation, or UI automation unless the user explicitly authorizes it.

## Object creation rules

- CDS View, CDS Table Function, CDS Query View, and Consumption View must be created as CDS Data Definitions / DDLS.
- Do not create ZTF_* as Table.
- Do not use Table / TABL / DT for CDS Table Function.
- Use Service Definition only when the user explicitly asks to create a service definition.
- Use Service Binding only when the user explicitly asks to create a service binding.
- Use Class only for ABAP Class or AMDP Class.

## Default execution style

- First inspect.
- Then plan.
- Then modify only the requested object.
- Then verify.
- Then report clearly.
