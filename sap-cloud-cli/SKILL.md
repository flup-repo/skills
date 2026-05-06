---
name: sap-cloud-cli
description: >-
  Manage SAP Commerce Cloud Portal (CCV2) data using the sapccm CLI.
  Handles builds, deployments, data backups, data restores, service properties,
  and CLI configuration for a CCV2 subscription. Use when the user wants to
  list/create/cancel builds or deployments, manage data backups and restores,
  inspect or update service properties, or configure the sapccm tool.
argument-hint: "build list | build create | deployment list | deployment create | databackup create | datarestore create | service-properties get | ..."
---

# SAP Commerce Cloud CLI (`sapccm`)

## Setup

Load environment variables from the `.env` file in this skill's directory before running any command:

```bash
# From the skill directory
source /path/to/sap-cloud-cli/.env
# or export values directly:
export SUBSCRIPTION_CODE=SUBSCRIPTION_CODE_REDACTED
export D1_ENV_CODE=d1
export S1_ENV_CODE=s1
export P1_ENV_CODE=p1
```

The CLI binary is `sapccm`. It must be available in your `PATH` before running any command — if `sapccm` is not found, add its installation directory to `PATH` and verify with `sapccm --version`.

## Authentication Failure

If any command returns an authentication error, ask the user to update credentials:

```bash
sapccm config set client-id {CLIENT_ID_VALUE}
sapccm config set client-secret {CLIENT_SECRET_VALUE}
sapccm config set token-url {TOKEN_URL_VALUE}
```

Then retry the original command.

## Global Configuration

These properties, once set globally with `sapccm config set`, do not need to be passed as flags in subsequent commands. Override per-command by passing the explicit flag.

```bash
sapccm config set subscription-code $SUBSCRIPTION_CODE
sapccm config set application-code commerce-cloud
```

Inspect current config:
```bash
sapccm config list
sapccm config show KEY
```

Delete a property:
```bash
sapccm config delete KEY
```

## Environment Codes

| Variable       | Value | Purpose           |
|----------------|-------|-------------------|
| D1_ENV_CODE    | d1    | Development env   |
| S1_ENV_CODE    | s1    | Staging env       |
| P1_ENV_CODE    | p1    | Production env    |

## Command Reference

### `sapccm build`

```bash
# List last 50 builds (newest first)
sapccm build list [--subscription-code=<subscriptionCode>]

# Show details of a specific build
sapccm build show [--subscription-code=<subscriptionCode>] [--build-code=<buildCode>]

# Download build logs
sapccm build logs [--subscription-code=<subscriptionCode>] [--build-code=<buildCode>] [--file-path=<filePath>]

# Create a build (waits for completion by default; use --no-wait to return immediately)
sapccm build create [--subscription-code=<subscriptionCode>] \
  [--application-code=<applicationCode>] \
  [--branch=<branch>] \
  [--name=<name>] \
  [--no-wait]
```

**Build field reference:**
- `build-code`: ISO 8601 date format `YYYYMMDD.N` (e.g. `20230712.1`)
- `application-code`: `commerce-cloud` or `commerce-cloud-datahub`
- `branch`: Git branch name
- `name`: Display name (quoted string)
- `STATUS`: `UNKNOWN` → `SCHEDULED` → `BUILDING` → `SUCCESS` or `FAIL` → eventually `DELETED`

**Log files:** Saved to `build-logs.zip` by default (in the sapccm installation directory); override with `--file-path` to control the output location.

---

### `sapccm deployment`

```bash
# List deployments for a build
sapccm deployment list [--subscription-code=<subscriptionCode>] [--build-code=<buildCode>]

# Show details of a deployment
sapccm deployment show [--subscription-code=<subscriptionCode>] [--deployment-code=<deploymentCode>]

# Create a deployment
sapccm deployment create [--subscription-code=<subscriptionCode>] \
  [--environment-code=<environmentCode>] \
  [--build-code=<buildCode>] \
  [--database-update-mode=<databaseUpdateMode>] \
  [--strategy=<strategy>] \
  [--no-wait]

# Get cancellation options for an ongoing deployment
sapccm deployment cancel-options [--subscription-code=<subscriptionCode>] \
  --deployment-code=<deploymentCode>

# Cancel an ongoing deployment
sapccm deployment cancel [--subscription-code=<subscriptionCode>] \
  --deployment-code=<deploymentCode> \
  --rollback-database=<true|false>
```

**Deployment field reference:**
- `deployment-code`: 6-digit numeric identifier (e.g. `374268`)
- `database-update-mode`: `NONE` (default), `UPDATE`, or `INITIALIZE` — case-sensitive
- `strategy`: `ROLLING_UPDATE` (default), `RECREATE`, or `GREEN` — case-sensitive
- `STATUS`: `SCHEDULED` → `DEPLOYING` → `DEPLOYED` or `FAIL` → eventually `UNDEPLOYED`

**Cancel notes:** Cancelling stops service instances and may leave the database in an inconsistent state. Use `--rollback-database=true` to mitigate. Always redeploy a previously successful build after cancellation.

---

### `sapccm databackup`

```bash
# List data backups for an environment (oldest first)
sapccm databackup list [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>]

# Show details of a specific backup
sapccm databackup show [--subscription-code=<subscriptionCode>] \
  [--environment-code=<environmentCode>] \
  --databackup-code=<databackupCode>

# Create a data backup (waits by default)
sapccm databackup create [--subscription-code=<subscriptionCode>] \
  [--environment-code=<environmentCode>] \
  --description=<description> \
  [--no-wait]

# Delete a data backup
sapccm databackup delete [--subscription-code=<subscriptionCode>] \
  [--environment-code=<environmentCode>] \
  --databackup-code=<databackupCode> \
  [--no-wait]
```

**Data backup field reference:**
- `databackup-code`: format `snap-NNNNNNNNNNNNN` (Unix timestamp in ms, e.g. `snap-1689563111707`)
- `STATUS`: `CREATING` → `CREATED` or `PREREQUISITES_NOT_MET` / `ERROR`; delete path: `DELETING` → `DELETED` or `DELETE_FAILED`
- `CAN BE RESTORED` / `CAN BE DELETED`: boolean flags shown in list/show output
- After deletion, the backup still appears in list/show but with `STATUS=DELETED` and both flags `false`

**Polling:** When `--no-wait` is not set, polls every 1 minute for up to 72 hours (creation) or 2 hours (deletion). Press `Ctrl+C` to release the terminal; the operation continues in the background.

---

### `sapccm datarestore`

```bash
# List data restores for an environment
sapccm datarestore list [--subscription-code=<subscriptionCode>] [--environment-code=<environmentCode>]

# Show details of a specific restore
sapccm datarestore show [--subscription-code=<subscriptionCode>] \
  [--environment-code=<environmentCode>] \
  --datarestore-code=<datarestoreCode>

# Create a data restore (restore a backup to an environment)
sapccm datarestore create [--subscription-code=<subscriptionCode>] \
  --source-environment-code=<sourceEnvironmentCode> \
  [--environment-code=<environmentCode>] \
  --databackup-code=<databackupCode> \
  [--no-wait]
```

**Data restore field reference:**
- `datarestore-code`: format `restore-NNNNNNNNNNNNN` (Unix timestamp in ms)
- `source-environment-code`: environment where the backup was taken
- `environment-code`: target environment (must be set explicitly, even if same as source)
- `STATUS`: `PREPARING` → `RESTORING` → `RESTORED` or `PREREQUISITES_NOT_MET` / `ERROR`
- A successful cross-environment restore creates a new data backup in the target environment

---

### `sapccm service-properties`

```bash
# Get a service property
sapccm service-properties get [--subscription-code=<subscriptionCode>] \
  [--environment-code=<environmentCode>] \
  --service-code=<serviceCode> \
  --property-service-code=<servicePropertyCode>

# Set/update a service property
sapccm service-properties put [--subscription-code=<subscriptionCode>] \
  [--environment-code=<environmentCode>] \
  --service-code=<serviceCode> \
  --property-service-code=<servicePropertyCode> \
  --property-service-value=<servicePropertyValue>
```

**Service codes (`service-code`):**
`hcs_admin`, `hcs_common`, `hcs_platform_accstorefront`, `hcs_platform_api`, `hcs_platform_backgroundProcessing`, `hcs_platform_backoffice`, `hcs_datahub`, `hcs_js_apps`

**Property service codes (`property-service-code`) by service:**

| Service | Property Code | Notes |
|---------|--------------|-------|
| `hcs_admin` | `initialpassword` | Read-only array; do not modify |
| `hcs_admin` | `customer-properties` | Editable key=value pairs |
| `hcs_common` | `customer-properties` | Editable key=value pairs |
| `hcs_common` | `security-properties` | Read-only encryption keys |
| `hcs_platform_accstorefront`, `hcs_platform_api`, `hcs_js_apps` | `customer-properties` | Editable |
| `hcs_platform_accstorefront`, `hcs_platform_api`, `hcs_js_apps` | `green-deployment-supported` | Blue/green indicator |
| `hcs_platform_backgroundProcessing`, `hcs_platform_backoffice`, `hcs_datahub` | `customer-properties` | Editable |

**`customer-properties` value format:** Key=value pairs separated by `\n`:
```
"key1=value1\nkey2=value2"
```
Minimum non-empty value is `"\n"`. Properties are sorted alphabetically; empty lines are removed.

**Apply changes:** After a `put`, changes are not immediately active. Go to the Cloud Portal → Services page → click **Apply Configurations** to restart all services. A per-service **Restart** alone does not apply property changes.

---

## `--no-wait` Flag Behaviour

| Usage | Equivalent |
|-------|-----------|
| `--no-wait` (no value) | `--no-wait=true` — returns terminal immediately |
| omitted | `--no-wait=false` — streams logs until done |
| `--no-wait=false` or `--no-wait=""` | Same as omitted |

---

## Typical Workflows

### Deploy a new build to d1

```bash
# 1. Create the build
sapccm build create --branch=main --name="My Feature" --application-code=commerce-cloud

# 2. Note the build-code from output (e.g. 20240506.1), then deploy
sapccm deployment create \
  --environment-code=$D1_ENV_CODE \
  --build-code=20240506.1 \
  --database-update-mode=UPDATE \
  --strategy=ROLLING_UPDATE
```

### Back up d1 and restore to s1

```bash
# 1. Create backup from d1
sapccm databackup create --environment-code=$D1_ENV_CODE --description="Pre-release backup"

# 2. Note snap-NNNNN code, then restore to s1
sapccm datarestore create \
  --source-environment-code=$D1_ENV_CODE \
  --environment-code=$S1_ENV_CODE \
  --databackup-code=snap-NNNNNNNNNNNNN
```

### Update a customer property in d1

```bash
sapccm service-properties put \
  --environment-code=$D1_ENV_CODE \
  --service-code=hcs_common \
  --property-service-code=customer-properties \
  --property-service-value="myProp=myValue\nanotherProp=anotherValue"
```

---

## Reference

- [commands.md](reference/commands.md) — Full command syntax cheat sheet
- [service-codes.md](reference/service-codes.md) — All service and property codes
