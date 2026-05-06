# SAP Commerce Cloud Service & Property Codes

## Service Codes

| Service Code | Description |
|---|---|
| `hcs_admin` | Admin service |
| `hcs_common` | Common/shared service |
| `hcs_platform_accstorefront` | Accelerator storefront |
| `hcs_platform_api` | OCC / REST API |
| `hcs_platform_backgroundProcessing` | Background processing / cron jobs |
| `hcs_platform_backoffice` | Backoffice admin UI |
| `hcs_datahub` | Data Hub |
| `hcs_js_apps` | JavaScript apps (e.g. Spartacus SSR) |

## Property Service Codes per Service

### `hcs_admin`

| Property Code | Description | Editable |
|---|---|---|
| `initialpassword` | Initial passwords for admin/anonymous | No — read-only array |
| `customer-properties` | Custom service properties (key=value) | Yes |

### `hcs_common`

| Property Code | Description | Editable |
|---|---|---|
| `customer-properties` | Custom service properties (key=value) | Yes |
| `security-properties` | Encryption keys (symmetric, token signing) | No — read-only array |

### `hcs_platform_accstorefront`, `hcs_platform_api`, `hcs_js_apps`

| Property Code | Description | Editable |
|---|---|---|
| `customer-properties` | Custom service properties (key=value) | Yes |
| `green-deployment-supported` | Indicates service supports Blue/Green deploy | Read indicator |

### `hcs_platform_backgroundProcessing`, `hcs_platform_backoffice`, `hcs_datahub`

| Property Code | Description | Editable |
|---|---|---|
| `customer-properties` | Custom service properties (key=value) | Yes |

## `customer-properties` Value Format

```
"key1=value1\nkey2=value2\nkey3=value3"
```

- Pairs are separated by `\n` (literal newline in the value string)
- Properties are sorted alphabetically by key after saving
- Empty lines are removed
- Minimum non-empty value: `"\n"`
- Setting an empty string `""` is invalid

## Applying Property Changes

After a `service-properties put`:

1. Go to Cloud Portal → Services page
2. Click **Apply Configurations**
3. Confirm and wait for all services to restart

A per-service **Restart** does NOT apply property changes (verify in HAC > Platform > Configuration).
