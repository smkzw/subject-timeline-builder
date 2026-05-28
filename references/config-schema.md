# Mapping Config Schema

Create one JSON file per build. Keep it explicit; do not rely on fuzzy matching during final build.

## Minimal Shape

```json
{
  "title": "研究名称 受试者时间线",
  "language": "zh-CN",
  "study_window": {
    "type": "first_visit_to_last_visit"
  },
  "history_window": {
    "type": "first_dose_to_last_visit",
    "start_visit_match": "基线"
  },
  "modules": {
    "ae": true,
    "mh": true,
    "cm": true,
    "findings": true,
    "pk": true
  },
  "population": "all",
  "include_usv": false,
  "sheets": {
    "visits": {
      "name": "SV--访视日期",
      "center": "研究中心",
      "subject": "受试者",
      "visit": "访视名称",
      "date": "访视日期",
      "occurred": "访视是否发生？"
    },
    "usv": {
      "name": "USV",
      "center": "试验中心名称",
      "subject": "受试者编号",
      "date": "检查日期(USVDAT)",
      "label": "USV"
    },
    "group": {
      "name": "Sheet1",
      "subject": "受试者编号",
      "group": "治疗组",
      "status": "受试者状态",
      "random_no": "随机号"
    },
    "cm": {
      "name": "CM--既往及合并用药治疗",
      "center": "研究中心",
      "subject": "受试者",
      "drug_name": "药物名称",
      "reason": "治疗原因",
      "reason_mh": "治疗原因为病史，请选择（可多选）",
      "reason_ae": "治疗原因为不良事件，请选择（可多选）",
      "reason_detail_columns": ["其他既往及现病史(CMMHNO)", "不良事件(CMAENO)", "备注(CMCO)"],
      "note": "备注(CMCO)",
      "dose": "单次剂量",
      "unit": "单位",
      "frequency": "给药频率",
      "route": "给药途径",
      "start": "开始日期",
      "ongoing": "是否持续",
      "end": "结束日期"
    }
  },
  "visit_order": [
    {"match": "筛选", "label": "筛选", "order": 0},
    {"match": "基线", "label": "基线", "order": 1},
    {"match": "2W", "label": "2W", "order": 2}
  ],
  "research_drug_name_patterns": ["研究药物", "试验药物"]
}
```

## Study Window Types

- `first_visit_to_last_visit`: first actual visit date through last actual visit date. Use when the protocol defines concomitant medication as “研究期间”.
- `first_dose_to_treatment_end`: first study drug/D1 through treatment end. Use when the protocol defines concomitant medication as “研究药物治疗期间”.
- `first_dose_to_last_visit`: first study drug/D1 through the subject’s final actual study visit. Often used for current medical history fallback.
- `custom_columns`: use explicit columns configured in `study_window.start_sheet/start_column/end_sheet/end_column`.

For `first_dose_*`, set `start_visit_match` when D1/baseline labels vary. For `first_dose_to_treatment_end`, set `end_visit_match` when the treatment-end label is known; otherwise the builder falls back to 24W-like labels and then the last visit.

## Optional Sheets

Add only if the module is enabled.

## Population and USV

- `population: "all"`: include every subject found in the mapped listings.
- `population: "randomized"`: include only subjects identified as randomized from `sheets.group` via treatment group or random number. If grouping/randomization cannot be mapped, stop and ask the user.
- `include_usv: true`: include unscheduled visits at subject level and place them by actual USV date. Configure `sheets.usv`.
- `include_usv: false`: exclude USV from the timeline.

Subject dropdown labels use `subject | group/status` when `sheets.group` is configured. If group status indicates screening failure, display `筛败`.

### AE

```json
"ae": {
  "name": "AE--不良事件",
  "center": "研究中心",
  "subject": "受试者",
  "term": "不良事件名称",
  "start": "最早开始日期",
  "grade": "最严重程度（CTCAE V5.0）",
  "relationship": "与研究用药的关系",
  "action": "对研究用药采取的措施",
  "outcome": "转归",
  "ongoing": "是否持续？",
  "end": "结束日期",
  "sae": "是否为严重不良事件（SAE）？"
}
```

### MH

```json
"mh": {
  "name": "MH--既往及现病史",
  "center": "研究中心",
  "subject": "受试者",
  "term": "疾病名称",
  "start": "开始日期",
  "ongoing": "是否持续",
  "end": "结束日期"
}
```

### PK

```json
"pk": {
  "name": "Sheet1",
  "subject": "辅助列：受试者筛选号",
  "visit": "访视名称",
  "result": "检测结果（ng/mL）",
  "lloq": "LLOQ  ",
  "note": "样本说明"
}
```

### Findings

```json
"findings": {
  "name": "Sheet1",
  "center": "中心",
  "subject": "受试者编号",
  "category": "问题分类",
  "visit": "涉及访视",
  "severity": "问题严重程度",
  "description": "原始问题描述",
  "response": "建议现场核查回复口径"
}
```

## Header Rule

If a sheet has duplicate headers, the scripts suffix later duplicates as `__2`, `__3`. For CM medication names, choose the raw/original medication name column, usually the earlier duplicate, unless the user explicitly requests coded names.
