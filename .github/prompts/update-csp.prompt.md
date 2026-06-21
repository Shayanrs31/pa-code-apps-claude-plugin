---
mode: agent
tools: [codebase, runCommands, readFile]
description: Configure Content Security Policy for the current Power Apps code app environment
---

# Update Content Security Policy

You are configuring the Content Security Policy (CSP) for Power Apps code apps in the current environment.

Reference: https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/content-security-policy

> CSP is configured at the environment level and applies to all code apps in that environment. Requires environment administrator permissions.

---

## Step 1: Identify what to unblock

Ask: "What is being blocked? For example: an external font, an image from a CDN, an API call, or an embedded iframe?"

Map to directive:

| Need | Directive |
|---|---|
| External scripts | `script-src` |
| External images or CDN | `img-src` |
| External fonts | `font-src` |
| External API call from browser | `connect-src` |
| Embedded iframe | `frame-src` |
| External stylesheets | `style-src` |
| App embedded in external site | `frame-ancestors` |

---

## Step 2: Choose method

Ask: "Do you want to configure this in the Power Platform admin center (UI), or via PowerShell?"

---

## Option A: Admin center

1. Sign in to https://admin.powerplatform.microsoft.com
2. Manage → Environments → select environment
3. Settings → Product → Privacy + Security
4. Under Content security policy, select the App tab
5. Find the directive, toggle off default, add the custom source value
6. Save

---

## Option B: PowerShell

### Authenticate

```powershell
$tenantId = "<your-tenant-id>"
$clientId = "9cee029c-6210-4654-90bb-17e6e9d36617"
$token = azureauth aad --resource "https://api.powerplatform.com/" --tenant $tenantId --client $clientId --output token | ConvertTo-SecureString -AsPlainText -Force
```

### Read current settings first (always read before updating)

```powershell
$envId = "<your-env-id>"
$directives = (Get-CodeAppContentSecurityPolicy -Token $token -Env $envId).Directives
```

### Add a source

```powershell
$directives['font-src'] = @("'self'", 'https://fonts.gstatic.com')
Set-CodeAppContentSecurityPolicy -Token $token -Env $envId -Directives $directives
```

---

## Common scenarios

| Scenario | Directive | Source |
|---|---|---|
| Google Fonts | `font-src` | `https://fonts.gstatic.com` |
| Google Fonts stylesheet | `style-src` | `https://fonts.googleapis.com` |
| Azure Maps | `img-src` | `https://*.azuremaps.com` |
| Application Insights | `connect-src` | `https://*.applicationinsights.azure.com` |
| External REST API | `connect-src` | `https://api.yourservice.com` |
| Embed in Teams tab | `frame-ancestors` | `https://teams.microsoft.com` |

---

## Important

- Always read current directives before updating. Setting directives replaces the entire collection.
- Custom sources are merged with defaults (except when default is `'none'`, which is replaced).
