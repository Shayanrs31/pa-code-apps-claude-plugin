---
name: update-csp
description: Configure Content Security Policy for the current Power Apps code app environment. Use when the app needs to load external scripts, images, fonts, or connect to external APIs that are blocked by the default CSP.
user-invocable: true
allowed-tools: Read, Write, Bash
model: sonnet
---

# Skill: update-csp

Configure Content Security Policy (CSP) for Power Apps code apps in your environment.

Reference: https://learn.microsoft.com/en-us/power-apps/developer/code-apps/how-to/content-security-policy

---

## When to use this skill

Use this skill when your app needs to load resources from external sources that the default CSP blocks, for example:

- External fonts (Google Fonts, Adobe Fonts)
- External images or CDN assets
- External scripts (analytics, charting libraries)
- API connections from the browser (connect-src)
- Embedded iframes from external domains (frame-src)

---

## Prerequisites

- You must be an **environment administrator** to change CSP settings
- CSP is configured at the **environment level** and applies to all code apps in that environment
- To use the PowerShell API approach, install the Microsoft Authentication CLI: https://github.com/AzureAD/microsoft-authentication-cli

---

## Default CSP directives

These are the defaults applied to all code apps. Custom values you add are **appended** to the default. If the default is `'none'`, your value replaces it.

| Directive | Default |
|---|---|
| `frame-ancestors` | `'self' https://*.powerapps.com` |
| `script-src` | `'self' <platform>` |
| `img-src` | `'self' data: <platform>` |
| `style-src` | `'self' 'unsafe-inline'` |
| `font-src` | `'self'` |
| `connect-src` | `'none'` |
| `frame-src` | `'self'` |
| `form-action` | `'none'` |
| `base-uri` | `'self'` |
| `child-src` | `'none'` |
| `default-src` | `'self'` |
| `manifest-src` | `'none'` |
| `media-src` | `'self' data:` |
| `object-src` | `'self' data:` |
| `worker-src` | `'none'` |

---

## Workflow

### Step 1: Identify what to unblock

Ask the user: "What is being blocked? For example: an external font, an image from a CDN, an API call, or an embedded iframe?"

Map their answer to the correct directive:

| Need | Directive to update |
|---|---|
| Load external scripts | `script-src` |
| Load external images / CDN | `img-src` |
| Load external fonts | `font-src` |
| Call an external API from the browser | `connect-src` |
| Embed an external iframe | `frame-src` |
| Apply external stylesheets | `style-src` |
| Allow the app to be embedded in an external site | `frame-ancestors` |

### Step 2: Choose the configuration method

Ask: "Do you want to configure this in the Power Platform admin center (UI), or via PowerShell (API)?"

---

## Option A: Admin center (UI)

Guide the user through:

1. Sign in to https://admin.powerplatform.microsoft.com
2. Manage → Environments → select the environment
3. Settings → Product → Privacy + Security
4. Under Content security policy, select the **App** tab
5. Find the directive to update
6. Toggle off the default and add the custom source value
7. Save

Example for unblocking Google Fonts (`font-src`):

> Turn off the `font-src` toggle, then add `https://fonts.gstatic.com` to the source list. The resulting directive will be: `font-src 'self' https://fonts.gstatic.com`

---

## Option B: PowerShell API

### Authentication

```powershell
$tenantId = "<your-tenant-id>"
$clientId = "9cee029c-6210-4654-90bb-17e6e9d36617"
$token = azureauth aad --resource "https://api.powerplatform.com/" --tenant $tenantId --client $clientId --output token | ConvertTo-SecureString -AsPlainText -Force
```

### Helper functions

Paste both functions into your PowerShell session before running the examples below.

**Get-CodeAppContentSecurityPolicy**

```powershell
function Get-CodeAppContentSecurityPolicy {
  [CmdletBinding()]
  param(
    [Parameter(Mandatory = $true)] [string]$Env,
    [Parameter(Mandatory = $true)] [securestring]$Token,
    [uri]$ApiEndpoint = 'https://api.powerplatform.com/'
  )
  $ErrorActionPreference = 'Stop'
  $escapedEnv = [System.Uri]::EscapeDataString($Env)
  $uri = [Uri]::new($ApiEndpoint, "/environmentmanagement/environments/$escapedEnv/settings?api-version=2022-03-01-preview&`$select=PowerApps_CSPReportingEndpoint,PowerApps_CSPEnabledCodeApps,PowerApps_CSPConfigCodeApps")
  $resp = Invoke-RestMethod -Uri $uri -Method Get -Authentication Bearer -Token $Token
  $data = $resp.objectResult[0]
  if ($null -ne $data.PowerApps_CSPConfigCodeApps) {
    $parsed = $data.PowerApps_CSPConfigCodeApps | ConvertFrom-Json -AsHashtable -Depth 10
    $config = @{}
    foreach ($directive in $parsed.Keys) {
      $config[$directive] = $parsed[$directive].sources | Select-Object -ExpandProperty source
    }
  }
  @{
    ReportingEndpoint = $data.PowerApps_CSPReportingEndpoint
    Enabled           = $data.PowerApps_CSPEnabledCodeApps ?? $true
    Directives        = $config
  }
}
```

**Set-CodeAppContentSecurityPolicy**

```powershell
function Set-CodeAppContentSecurityPolicy {
  [CmdletBinding(SupportsShouldProcess = $true)]
  param(
    [Parameter(Mandatory = $true)] [string]$Env,
    [Parameter(Mandatory = $true)] [securestring]$Token,
    [uri]$ApiEndpoint = 'https://api.powerplatform.com/',
    [AllowNull()] [string]$ReportingEndpoint,
    [bool]$Enabled,
    [hashtable]$Directives
  )
  $payload = @{}
  if ($PSBoundParameters.ContainsKey('ReportingEndpoint')) { $payload['PowerApps_CSPReportingEndpoint'] = $ReportingEndpoint }
  if ($PSBoundParameters.ContainsKey('Enabled')) { $payload['PowerApps_CSPEnabledCodeApps'] = $Enabled }
  if ($PSBoundParameters.ContainsKey('Directives')) {
    $allowed = @('Frame-Ancestors','Script-Src','Img-Src','Style-Src','Font-Src','Connect-Src','Frame-Src','Form-Action','Base-Uri','Child-Src','Default-Src','Manifest-Src','Media-Src','Object-Src','Worker-Src')
    $textInfo = [System.Globalization.CultureInfo]::InvariantCulture.TextInfo
    $config = @{}
    foreach ($key in $Directives.Keys) {
      $directive = $textInfo.ToTitleCase($key)
      if ($allowed -notcontains $directive) { throw "Invalid CSP directive: $directive" }
      $config[$directive] = @{ sources = @($Directives[$key] | ForEach-Object { @{ source = $_ } }) }
    }
    $payload['PowerApps_CSPConfigCodeApps'] = ($config | ConvertTo-Json -Depth 10 -Compress)
  }
  $escapedEnv = [System.Uri]::EscapeDataString($Env)
  $uri = [Uri]::new($ApiEndpoint, "/environmentmanagement/environments/$escapedEnv/settings?api-version=2022-03-01-preview")
  Invoke-RestMethod -Uri $uri -Method Patch -Authentication Bearer -Token $Token -Body ($payload | ConvertTo-Json -Depth 10) -ContentType 'application/json' | Out-Null
}
```

### Read current settings

```powershell
Get-CodeAppContentSecurityPolicy -Token $token -Env "<your-env-id>"
```

### Add a source to a directive

**Important:** Always read first, then update. Setting directives replaces the entire collection.

```powershell
$envId = "<your-env-id>"
$directives = (Get-CodeAppContentSecurityPolicy -Token $token -Env $envId).Directives

# Add Google Fonts to font-src
$directives['font-src'] = @("'self'", 'https://fonts.gstatic.com')

Set-CodeAppContentSecurityPolicy -Token $token -Env $envId -Directives $directives
```

### Enable reporting

```powershell
Set-CodeAppContentSecurityPolicy -Token $token -Env "<your-env-id>" -ReportingEndpoint "https://your-report-endpoint.com/csp"
```

### Disable CSP enforcement (report-only mode)

```powershell
Set-CodeAppContentSecurityPolicy -Token $token -Env "<your-env-id>" -Enabled $false
```

---

## Common scenarios

| Scenario | Directive | Source to add |
|---|---|---|
| Google Fonts | `font-src` | `https://fonts.gstatic.com` |
| Google Fonts stylesheet | `style-src` | `https://fonts.googleapis.com` |
| Azure Maps tiles | `img-src` | `https://*.azuremaps.com` |
| Application Insights | `connect-src` | `https://*.applicationinsights.azure.com` |
| External REST API | `connect-src` | `https://api.yourservice.com` |
| Embed app in Teams tab | `frame-ancestors` | `https://teams.microsoft.com` |

---

## Important notes

- CSP settings are **environment-wide** - changes affect all code apps in that environment
- Custom sources are **merged** with defaults, not replaced (except when default is `'none'`)
- To reset a directive to its default, omit it from the directives collection when calling Set
- To disable a directive entirely, pass an empty array for that directive
- Requires environment administrator permissions
