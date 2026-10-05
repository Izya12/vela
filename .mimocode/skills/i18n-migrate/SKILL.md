---
name: i18n-migrate
description: "Migrate hardcoded user-visible strings to i18n (react-i18next or similar) in TypeScript/JavaScript projects. Use when auditing codebase for untranslated strings, adding translation keys to locale JSON files, replacing hardcoded text with t() or useTranslation() calls, verifying locale coverage, or completing i18n migration tasks. Trigger on mentions of i18n, localization, translate, migration, t(), useTranslation, locale."
---

# I18n Migration

Systematic workflow for migrating hardcoded UI strings to internationalized translation keys.

## Workflow

### 1. Audit

Find all hardcoded user-visible strings:

```
# For Chinese characters specifically:
grep -rn "[\x{4e00}-\x{9fff}]" <target-dir> --include="*.{ts,tsx,js,jsx}"

# For generic audit (any non-ASCII):
grep -rn "[^\x00-\x7F]" <target-dir> --include="*.{ts,tsx,js,jsx}"
```

**Skip these categories** (not UI text):
- LLM prompt content (AI instruction strings)
- Code comments
- Variable names / identifiers
- Data content (markdown headings written to DB)
- Debug/log strings only visible in dev console
- Import paths

### 2. Classify Strings

For each file, categorize strings:
- **UI labels**: Button text, tab names, section headers → migrate
- **Messages**: Success/error/info messages → migrate
- **Placeholders**: Input placeholders → migrate
- **Descriptions**: Helper text, tooltips → migrate
- **Prompt content**: LLM instructions → keep as-is (or create separate localized fields)
- **Data**: Content stored in DB → keep as-is

### 3. Migration Patterns

#### React Components

```tsx
// Before
<button>Submit</button>

// After
import { useTranslation } from 'react-i18next';

const { t } = useTranslation('namespace');
<button>{t('key')}</button>
```

#### Non-React Code (services, stores, commands)

```ts
// Before
throw new Error('Operation failed');

// After
import i18n from '../../i18n';
const t = (key: string, opts?: object) => i18n.t(key, { ns: 'namespace', ...opts });

throw new Error(t('errors.operationFailed'));
```

#### Template Literals with Variables

```tsx
// Before
`${chapter} / ${total} words`

// After
t('chapterWordCount', { chapter, total })
// Key: "chapterWordCount": "{{chapter}} / {{total}} words"
```

### 4. Namespace Strategy

Create namespaces by architectural layer:
- `common` — shared labels (cancel, ok, delete, etc.)
- `dialogs` — dialog/modal components
- `editors` — editor components
- `panels` — sidebar/panel components
- `settings` — settings modal
- `stores` — state management layer
- `commands` — workflow/command layer
- `pages` — page-level components

Register all namespaces in i18n config file.

### 5. Locale Files

**Always update ALL locales simultaneously.** For each new key:

```json
// zh-CN/namespace.json
{ "key": "中文值" }

// en/namespace.json
{ "key": "English value" }

// ru/namespace.json
{ "key": "Русское значение" }
```

### 6. Reuse Existing Keys

Before creating new keys, check if semantically equivalent keys exist:
- `common.cancel`, `common.ok`, `common.delete`
- `common.chapters`, `common.words`
- `common.selectAll`, `common.loading`

### 7. Verify

After migration batch:
```bash
npx tsc --noEmit          # Type check
npx vite build            # Build check
```

Fix any missing imports or type errors before proceeding.

## Common Pitfalls

1. **Missing import**: `useTranslation` or `i18n` not imported
2. **Wrong namespace**: Key exists in wrong namespace file
3. **Missing locale**: Key added to one locale but not others
4. **Variable mismatch**: Template `{{var}}` in JSON doesn't match code usage
5. **Type errors**: `t()` return type issues with TypeScript strict mode

## References

See `references/migration-patterns.md` for detailed patterns per file type.
