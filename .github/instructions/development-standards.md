---
applyTo: "**/*.ts,**/*.tsx"
---

# Development Standards

These are enforced patterns. Violations are treated as critical findings in code review.

---

## Hook pattern

Every connector or service call goes in a custom hook under `src/hooks/`. Components never call services directly.

```typescript
// src/hooks/useRecords.ts
import { useState, useEffect } from 'react';
import type { Record } from '../types';
import { RecordService } from '../services/mock';

export function useRecords() {
  const [records, setRecords] = useState<Record[]>([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    RecordService.getAll().then(setRecords).finally(() => setLoading(false));
  }, []);

  return { records, loading };
}
```

```typescript
// src/pages/RecordsPage.tsx - only calls the hook, never the service
import { useRecords } from '../hooks/useRecords';

export default function RecordsPage() {
  const { records, loading } = useRecords();
}
```

---

## User identity

Never hardcode a user name. Always use `getContext()` from `@microsoft/power-apps/app`.

```typescript
// src/hooks/useCurrentUser.ts
import { useState, useEffect } from 'react';
import type { CurrentUser } from '../types';

export function useCurrentUser(): CurrentUser {
  const [user, setUser] = useState<CurrentUser>({ fullName: 'User', initials: 'U' });

  useEffect(() => {
    import('@microsoft/power-apps/app')
      .then(({ getContext }) => getContext())
      .then(ctx => {
        const name = ctx.user.fullName ?? ctx.user.userPrincipalName ?? 'User';
        const parts = name.trim().split(' ');
        const initials = parts.length >= 2
          ? (parts[0][0] + parts[parts.length - 1][0]).toUpperCase()
          : name.slice(0, 2).toUpperCase();
        setUser({ fullName: name, initials });
      })
      .catch(() => {});
  }, []);

  return user;
}
```

---

## SDK imports

```typescript
// Correct
import { getContext } from '@microsoft/power-apps/app';
import { trackEvent } from '@microsoft/power-apps/telemetry';

// Wrong - will fail at build time
import { getContext } from '@microsoft/power-apps';
```

---

## OData safety

Always include `$top` and `$select` in retrieve calls. Never issue unbounded queries.

```typescript
// Wrong
DataverseService.getAll()

// Correct
DataverseService.retrieveMultipleRecords('entity', '?$top=250&$select=col1,col2')
```

---

## Dataverse quirks

- Picklist fields: read as `number`, write by setting the numeric option value. Do not send the label string.
- Virtual fields: read-only. Exclude from write payloads.
- Lookups: set as `{ entityType: 'tablename', id: 'guid' }`. Not as a flat GUID string.
- OData filter on lookup: filter on `_lookupfield_value`, not the lookup field name.
- Always include `$select`. Omitting it returns all fields including virtual ones.
- Entity prefix (`cr{nnn}_`) varies per environment. Read it from the generated model file.

---

## Versioning

- Show version in the UI (topbar chip, footer, or about panel)
- Format: `v{major}.{minor}.{patch}`. Start at `v0.1.0` for pre-release.
- Increment patch for fixes, minor for new features, major for breaking data model changes.

---

## Theme

Default to dark theme. Use CSS variables. No hardcoded hex colours in component files.

```css
--bg-1: main background
--bg-2: card/panel background
--text-1: primary text
--text-2: secondary/muted text
--blue: primary action colour
--green: success
--amber: warning
--red: danger/error
```

---

## TypeScript strict mode

- Remove all unused imports before building (TS6133)
- Explicitly type all state and props. Avoid implicit `any`.
- Optional chaining before accessing connector response `.value` arrays
