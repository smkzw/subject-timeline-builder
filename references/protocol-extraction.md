# Protocol Extraction Checklist

Extract these facts from the current protocol before building.

## Visit Model

- Screening, baseline/D1, treatment visits, open-label visits, EOT, safety follow-up, unscheduled visits.
- Visit labels and order for display.
- Which visits define “study period”, “treatment period”, and “follow-up period”.
- Whether unscheduled visits should count in first/last actual visit. Default: include if the protocol says “研究期间” and the visit occurred.

## Medication Rules

Find the exact definition for:

- Prior medication / 既往用药.
- Concomitant medication / 合并用药.
- Whether the definition uses “研究期间”, “治疗期间”, “研究药物治疗期间”, “签署知情同意后”, or “首次给药后”.
- Whether study drug should be excluded from CM. Default: exclude research/study drug if it appears in CM, using configured name patterns.

If no definition is found, ask the user whether to use the default fallback:

- 合并用药: non-study medication whose date range overlaps first study drug administration through treatment-period final visit.

## Medical History Rules

Find the exact definition for:

- Past medical history / 既往史.
- Current medical history / 现病史.
- Whether ongoing conditions at screening, D1, first dose, or study end are current history.

If no definition is found, ask the user whether to use the default fallback:

- 现病史: condition whose date range overlaps first study drug administration through the subject’s last study visit.

## Date Handling

- Preserve original partial dates in displayed text.
- For interval classification only:
  - `YYYY-MM-uk`: start bound = first day of month, end bound = last day of month.
  - `YYYY-uk-uk`: start bound = Jan 1, end bound = Dec 31.
- If date imputation changes classification, flag it in precheck.

## Precheck Output

Before building, report:

- Files recognized and modules enabled.
- Sheet/column mapping summary.
- Definitions found in protocol, with exact short paraphrase and source section/page when available.
- Missing definitions or ambiguous fields requiring user confirmation.
