---
name: subject-timeline-builder
description: Build Chinese interactive clinical subject timeline HTML from uploaded study protocols, EDC data listings, finding listings, and PK listings. Use when the user asks for “受试者时间线”, subject timeline, patient profile timeline, patient profile HTML, or wants to reconstruct per-subject visit dates, medical history, adverse events, prior/concomitant medications, findings, PK, and center/subject filters from clinical trial listings.
---

# Subject Timeline Builder

## Core Rule

Do not hardcode rules from a previous project. Derive study periods, visit names, medication/history definitions, sheet mappings, and column mappings from the current uploaded protocol and listings. If a rule or field cannot be confidently derived, run precheck and ask the user to confirm before building.

Fully deconstruct every uploaded file before building. Do not skip protocol, listing, grouping, finding, or PK review because the task is long. If field matching is unclear or disputed, stop and tell the user exactly which data were not identified or were ambiguous; do not make an unconfirmed decision.

## Workflow

1. Inventory inputs.
   - Protocol: PDF/DOCX/TXT/HTML that defines study period, visits, medication rules, and history rules.
   - Data listing: EDC export workbook with subject, visit, AE, MH, CM, lab/PK or other sheets.
   - Finding listing: optional self-inspection/finding workbook or CSV.
   - PK listing: optional workbook or sheet; may be inside data listing.
   - Subject grouping/randomization table: required when group labels, randomized-only output, or screen-failure labels are requested. If no uploaded file contains grouping/randomization/screening result information, ask the user to upload or identify it.

2. Ask required build-scope questions before mapping.
   - Ask whether to include all subjects or only randomized subjects.
   - Ask whether to include USV/计划外访视 data. If included, show USV only at subject level and place each USV by its actual USV date on the timeline.
   - If the user selects randomized-only and randomization/screen-failure fields cannot be found, stop and ask the user to identify the relevant file/sheet/columns.

3. Precheck workbook structure.
   - Run `scripts/inspect_workbook.py` on every workbook.
   - Identify candidate sheets and exact headers for visits, subject demographics, AE, MH, CM, findings, and PK.
   - Record missing or ambiguous fields in a short precheck list.

4. Read and deconstruct the protocol.
   - Extract visit schedule and normalize visit order.
   - Extract concomitant medication and prior medication definitions.
   - Extract current/past medical history definitions.
   - Extract PK sampling schedule if relevant.
   - Extract the study flow table and use it to define actual visit labels and order. If the table uses Vxx/访视xx labels, include them. If it does not, label visits directly by Dxx or Wxx as shown in the protocol.
   - If no concomitant medication definition is found, ask whether to use: “non-study medications whose use range overlaps first study drug administration through treatment-period final visit”.
   - If no current medical history definition is found, ask whether to use: “conditions whose date range overlaps first study drug administration through the subject’s last study visit”.

5. Resolve definitions into explicit windows.
   - For this skill, “合并用药” often means non-study medication whose use interval overlaps the protocol-defined study window. If the protocol says “研究期间”, use first actual visit through last actual visit. If it says “研究药物治疗期间”, use first study drug administration/D1 through treatment end.
   - Use partial dates conservatively: `YYYY-MM-uk` starts at day 1 for interval starts and ends at month end for interval ends; `YYYY-uk-uk` starts Jan 1 and ends Dec 31.
   - Do not use fields such as “该治疗是否在本研究治疗期间使用？” as the sole classifier when protocol definitions are date-window based. Treat them as source context only.
   - CM reason display: when the reason is “其他及既往病史/其他既往及现病史” or “不良事件”, extract the specific linked MH/AE from the corresponding listing columns. Also inspect “备注” columns as fallback. If the linked MH/AE cannot be extracted, stop for user confirmation.
   - AE display must include outcome/转归 when available.

6. Create a mapping JSON.
   - Use `references/config-schema.md`.
   - Include sheet names, column names, visit order, study-window rules, optional research drug name patterns, and optional display labels.
   - Include grouping/randomization mapping when available. Display group labels after subject IDs in cards and subject dropdowns, e.g. `01005 | 筛败` or `S03005 | 安慰剂组`.
   - If the user opts out of a module, omit its mapping and ensure the HTML does not show that module.

7. Build the HTML.
   - Run `scripts/build_timeline_html.py --listing <data.xlsx> --config <mapping.json> --output <timeline.html>`.
   - Add `--pk-listing`, `--finding-listing`, and `--group-listing` when those data are separate files and mappings are present.
   - The output must cover all centers and all subjects. Use the top-right filters to locate center and subject.

8. Validate before delivery.
   - Confirm subject count, center count, CM/MH/AE/PK row counts, and missing window count.
   - Spot-check at least one subject with pre-study medication, one with study-period medication, one with ongoing medication, one with AE, and one with PK if present.
   - Verify no encoded drug names are used when a raw medication name column is available.
   - Verify subject labels show group/screen-failure information when grouping is enabled.
   - Verify AE rows show outcome/转归.
   - Verify CM reasons show linked AE/MH details for AE/MH-related medication reasons.
   - Verify USV dates appear at their actual date positions when USV is enabled.
   - Open or parse the HTML and ensure the filters, cards, and Chinese labels render.

## Precheck Questions

Ask concise questions only for blockers:

- Required file missing: “未提供 data listing，无法构建受试者级访视、AE、MH、CM。是否先仅基于 finding listing 构建问题索引？”
- Ambiguous definition: “方案中未明确合并用药是否按研究期间或治疗期间定义。是否按首次试验药物用药至治疗期末次访视重叠定义合并用药？”
- Ambiguous raw/coded medication names: “CM sheet 中存在多个药物名称列。是否使用靠前的原始录入药名列？”
- Optional module absent: “未识别 PK listing。是否不展示 PK 模块？”
- Missing grouping: “未识别受试者分组/随机结果/筛选结果信息。请上传受试者分组表或指出对应 sheet 和列。”
- Population scope: “请确认 HTML 纳入所有受试者，还是仅纳入已随机受试者？”
- USV scope: “请确认是否纳入计划外访视（USV）数据；若纳入，将按 USV 实际日期显示在个例时间线中。”

If the user chooses not to show a module, remove that module from the config and from the HTML output.

## Resources

- `scripts/inspect_workbook.py`: print workbook sheets, headers, duplicate headers, and sample rows for mapping.
- `scripts/build_timeline_html.py`: generic configurable HTML builder.
- `references/config-schema.md`: required mapping JSON fields and examples.
- `references/protocol-extraction.md`: protocol facts to extract and fallback rules.
