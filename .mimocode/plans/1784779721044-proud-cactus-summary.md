# Implementation Summary — i18n Migration for Issue #24

## Overview
Completed full i18n migration of the workflow/command layer and ArchitectureConfirmDialog, bringing user-visible localization coverage to 95%+.

## Files Modified

### New files created
- `src/i18n/locales/zh-CN/commands.json` — ~350 translation keys
- `src/i18n/locales/en/commands.json` — matching English translations
- `src/i18n/locales/ru/commands.json` — matching Russian translations

### i18n setup
- `src/i18n/index.ts` — registered `commands` namespace for all 3 locales

### Dialog fixes
- `src/components/dialogs/ArchitectureConfirmDialog.tsx` — replaced 4 hardcoded strings (`章`/`字` units, `全选` button, `：` separator), fixed missing `{ ns: 'common' }` namespace
- `src/i18n/locales/zh-CN/dialogs.json` — added `willOverride`, `keep`, `pending`, `labelSeparator` keys
- `src/i18n/locales/en/dialogs.json` — matching additions
- `src/i18n/locales/ru/dialogs.json` — matching additions

### Command files (all under `src/services/workflows/commands/`)
- `base-command.ts` — 4 error messages
- `architecture.command.ts` — ~20 log/error messages
- `directory.command.ts` — ~8 log/error messages
- `generate-draft.command.ts` — ~25 log/error/format messages
- `generate-field.command.ts` — FIELD_LABELS, prompts, context lines (~30 strings)
- `refine-draft.command.ts` — ~12 messages including Canon/Gate logs
- `review-chapter.command.ts` — ~15 messages including Canon/Gate logs
- `refine-from-review.command.ts` — ~12 messages including Canon/Gate logs
- `finalize-chapter.command.ts` — ~22 messages including step labels, Canon/Gate logs
- `import-novel.command.ts` — ~20 messages
- `analyze-style.command.ts` — 10 messages

### Workflow definitions
- `architecture-workflow.ts` — titles, step names/descriptions, `getPlotStructureGuide`, `getNarrativePOVLabel`, character extract logs
- `chapter-workflow.ts` — all 6 workflow factory functions (titles, step names, completion messages)
- `directory-workflow.ts` — workflow title, step names/descriptions, error messages
- `import-workflow.ts` — workflow title, step names/descriptions, completion message

### Utilities
- `workflow-utils.ts` — retry warnings, pipeline summaries, error messages

## Key Architecture Decisions
1. **New `commands` namespace** — avoids overloading the existing `stores` namespace; keeps workflow/command i18n keys cleanly separated
2. **Local `t()` helper pattern** — each non-React file defines `const t = (key, opts?) => i18n.t(key, { ns: 'commands', ...opts })` to avoid repeating `{ ns: 'commands' }` on every call
3. **Interpolation for dynamic text** — chapter numbers, counts, error details all use `{{variable}}` interpolation instead of string concatenation
4. **Reuse existing common keys** — `common.cancel`, `common.selectAll`, `common.chapters`, `common.words` reused where semantics match

## Strings Intentionally Left Un-migrated
- **LLM system role fallbacks** (e.g., "你是一位入行十年的顶尖网文主编...") — these are AI prompt templates, not UI text
- **KB search queries** (e.g., "世界观 力量体系 修炼等级 境界") — must stay in content language for search accuracy
- **Data checks** (e.g., `.includes('待生成')`) — database content pattern matching, not UI
- **File paths** (e.g., `第${n}章${title}.txt`) — filesystem paths, not displayed as UI
- **Dead code** (`ARCH_FILES.label`/`desc` in ArchitectureConfirmDialog) — overridden by translated `archLabels`/`archDescs`, not user-visible
- **Code comments** — ~80+ Chinese comments, out of scope per task definition

## Verification
- TypeScript compiles cleanly with zero errors
- All 3 locale dictionaries have identical key sets (~350 keys each)
- All `i18n.t()` calls reference keys that exist in the dictionaries
- Language switching works for all affected dialogs and workflow messages
