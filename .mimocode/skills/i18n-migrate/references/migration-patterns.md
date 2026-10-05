# Migration Patterns Reference

Detailed patterns for common i18n migration scenarios.

## React Component Patterns

### Simple Label

```tsx
// Before
<span>Settings</span>

// After
const { t } = useTranslation('settings');
<span>{t('title')}</span>
```

### Conditional Text

```tsx
// Before
{isLoading ? 'Loading...' : 'Done'}

// After
{isLoading ? t('loading') : t('done')}
```

### JSX Attributes

```tsx
// Before
<input placeholder="Enter name" />

// After
<input placeholder={t('placeholders.name')} />
```

### Pluralization

```tsx
// Before
`${count} items`

// After
t('itemCount', { count })
// Key: "itemCount": "{{count}} items" (English handles plural in value)
```

## Non-React Patterns

### Service/Store with Local Helper

```ts
import i18n from '../i18n';

const t = (key: string, opts?: object) => i18n.t(key, { ns: 'stores', ...opts });

// Usage
export function getStatus() {
  return t('status.ready');
}
```

### Error Messages

```ts
// Before
throw new Error('File not found');

// After
throw new Error(t('errors.fileNotFound'));
```

### Notification Messages

```ts
// Before
showNotification('Saved successfully');

// After
showNotification(t('notifications.saved'));
```

## Locale File Structure

### Flat Keys

```json
{
  "title": "Settings",
  "save": "Save",
  "cancel": "Cancel"
}
```

### Nested Keys (by feature)

```json
{
  "dialog": {
    "confirm": "Are you sure?",
    "yes": "Yes",
    "no": "No"
  },
  "errors": {
    "notFound": "Not found",
    "permissionDenied": "Permission denied"
  }
}
```

### Keys with Variables

```json
{
  "welcome": "Hello, {{name}}!",
  "chapterCount": "{{total}} chapters",
  "wordCount": "{{count}} words"
}
```

## i18n Config Registration

```ts
// src/i18n/index.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';

import zhCNCommon from './locales/zh-CN/common.json';
import enCommon from './locales/en/common.json';
import ruCommon from './locales/ru/common.json';

i18n.use(initReactI18next).init({
  resources: {
    'zh-CN': { common: zhCNCommon, /* ... */ },
    'en': { common: enCommon, /* ... */ },
    'ru': { common: ruCommon, /* ... */ },
  },
  ns: ['common'],
  defaultNS: 'common',
  lng: localStorage.getItem('locale') || 'zh-CN',
});

export default i18n;
```

## String Category Checklist

| Category | Migrate? | Example |
|----------|----------|---------|
| Button labels | Yes | "Save", "Cancel" |
| Tab names | Yes | "Settings", "About" |
| Section headers | Yes | "General", "Advanced" |
| Input placeholders | Yes | "Enter name..." |
| Tooltips | Yes | "Click to expand" |
| Error messages | Yes | "File not found" |
| Success messages | Yes | "Saved successfully" |
| Loading states | Yes | "Loading..." |
| Empty states | Yes | "No items found" |
| Confirmation dialogs | Yes | "Are you sure?" |
| LLM prompts | Optional* | AI instruction text |
| Code comments | No | // internal note |
| Variable names | No | myVariable |
| Import paths | No | './utils' |
| Debug logs | No | console.log |
| Data content | No | Markdown headings |

*LLM prompts: Keep in source language or create separate `contentLocalized` fields.
