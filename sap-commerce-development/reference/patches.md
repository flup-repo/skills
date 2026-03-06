# Patches Extension & ImpEx Reference

## Contents
- Patches framework overview
- Setting up the patches extension
- Creating patches
- Patch execution lifecycle
- Auto-execution on CCv2
- ImpEx syntax reference
- ImpEx conventions in patches
- Best practices

## Patches Framework Overview

The **patches extension** (`hybris/bin/modules/platform/patches`) is an OOTB framework for managing data changes across environments. Instead of manually importing ImpEx, patches automate data management during `ant updatesystem`.

Key concepts:
- A **Patch** is a Java class that runs during system update
- Each patch imports ImpEx files and/or executes custom Java code
- Patches are **tracked in the DB** — executed patches are skipped on subsequent updates
- Patches can be **rerunnable** or one-time
- The `patchesdemo` extension provides implementation examples

## Setting Up the Patches Extension

### 1. Add Required Extensions

In `localextensions.xml`:
```xml
<extension name="patches"/>
<extension name="patchesbackoffice"/>      <!-- Optional: Backoffice UI -->
<extension name="projectpatches"/>         <!-- Your custom patches extension -->
```

### 2. Extension Directory Structure

```
projectpatches/
├── extensioninfo.xml
├── resources/
│   ├── projectpatches-spring.xml
│   └── projectpatches/
│       └── patches/
│           ├── TICKET-001/
│           │   ├── 01-types.impex
│           │   └── 02-data.impex
│           └── TICKET-002/
│               ├── 01-update.impex
│               └── post/
│                   └── 01-cleanup.impex
├── src/com/example/patches/
│   ├── setup/
│   │   └── PatchesSystemSetup.java
│   ├── patch/
│   │   ├── AbstractProjectPatch.java
│   │   ├── TICKET001.java
│   │   └── TICKET002.java
│   └── structure/
│       └── FileStructureVersion.java
└── testsrc/
```

### 3. Extension Dependencies

In `extensioninfo.xml`:
```xml
<extensioninfo>
    <extension name="projectpatches" ...>
        <requires-extension name="patches"/>
        <requires-extension name="projectcore"/>
        <!-- Add all extensions whose types you reference in ImpEx -->
    </extension>
</extensioninfo>
```

## Creating Patches

### Abstract Patch Base Class

```java
package com.example.patches.patch;

import de.hybris.platform.patches.Patch;
import de.hybris.platform.patches.Release;
import de.hybris.platform.patches.organisation.StructureState;

public abstract class AbstractProjectPatch implements Patch {

    private static final String IMPEX_BASE = "/projectpatches/patches/";

    abstract String getTicketId();

    protected void executeCustomJavaCode() {
        // Override in subclasses when needed
    }

    @Override
    public void createProjectData(StructureState structureState) {
        importImpexesFromFolder(IMPEX_BASE + getTicketId() + "/");
        executeCustomJavaCode();
        importImpexesFromFolder(IMPEX_BASE + getTicketId() + "/post/");
    }

    @Override
    public String getPatchId() { return getTicketId(); }

    @Override
    public String getPatchName() { return getTicketId(); }

    @Override
    public String getPatchDescription() { return getTicketId(); }

    @Override
    public Release getRelease() { return () -> getTicketId(); }

    @Override
    public StructureState getStructureState() { return FileStructureVersion.V1; }
}
```

### Concrete Patch (ImpEx Only)

```java
@Component
public class TICKET001 extends AbstractProjectPatch {
    @Override
    String getTicketId() { return "TICKET-001"; }
}
```

ImpEx files go in: `resources/projectpatches/patches/TICKET-001/`

### Patch with Custom Java Code

```java
@Component
public class TICKET002 extends AbstractProjectPatch {
    @Override
    String getTicketId() { return "TICKET-002"; }

    @Override
    protected void executeCustomJavaCode() {
        // Custom data migration logic
    }
}
```

### SystemSetup for Patch Execution

```java
@SystemSetup(extension = "projectpatches")
public class PatchesSystemSetup extends AbstractPatchesSystemSetup {

    @Autowired(required = false)
    private List<AbstractProjectPatch> patchList;

    @PostConstruct
    private void updatePatchesList() {
        if (patchList != null) {
            super.setPatches(new ArrayList<>(patchList));
        }
    }

    @Override
    @SystemSetup(type = Type.PROJECT, process = Process.UPDATE)
    public void createProjectData(final SystemSetupContext ctx) {
        super.createProjectData(ctx);
    }

    @Override
    @SystemSetupParameterMethod
    public List<SystemSetupParameter> getInitializationOptions() {
        return super.getInitializationOptions();
    }
}
```

Spring configuration:
```xml
<context:component-scan base-package="com.example.patches.patch"/>

<bean class="com.example.patches.setup.PatchesSystemSetup"
      parent="patchesSystemSetup">
    <property name="patches"><list/></property>
</bean>
```

## Patch Execution Lifecycle

1. `ant updatesystem` triggers `@SystemSetup` methods
2. `PatchesSystemSetup.createProjectData()` iterates all patches
3. For each patch, the framework checks if `PatchExecution` exists in DB
4. If not yet executed → runs `createProjectData()` on the patch
5. On success → creates `PatchExecutionModel` in DB (marks as done)
6. On subsequent updates → patch is skipped

## Auto-Execute Patches on CCv2

Reference in `manifest.json`:

```json
{
  "properties": [
    { "key": "configFile", "value": "/opt/hybris/bin/custom/projectcore/resources/sysupdate.json" }
  ]
}
```

The `sysupdate.json`:
```json
{
  "init": "Go",
  "initmethod": "update",
  "essential": "true",
  "localizetypes": "true",
  "projectpatches_sample": "true",
  "filteredPatches": "false"
}
```

---

## ImpEx Syntax Reference

### Modes

| Mode | Description |
|------|-------------|
| `INSERT` | Creates new records. Error if record exists. |
| `UPDATE` | Updates existing records. Error if not found. |
| `INSERT_UPDATE` | Creates or updates. **Preferred** — idempotent. |
| `REMOVE` | Deletes matching records. |

### Header Structure

```impex
INSERT_UPDATE Product;code[unique=true];name[lang=en];catalogVersion(catalog(id),version)[unique=true]
```

### Macros

```impex
$catalog=productCatalog
$version=catalogversion(catalog(id[default=$catalog]),version[default='Online'])[unique=true,default=$catalog:Online]
$lang=en

INSERT_UPDATE Product;code[unique=true];name[lang=$lang];$version
;PROD001;Product Name;
```

### Attribute Modifiers

| Modifier | Description | Example |
|----------|-------------|---------|
| `unique=true` | Marks unique key for matching | `code[unique=true]` |
| `default=value` | Default if not provided | `active[default=true]` |
| `lang=xx` | Localized attribute language | `name[lang=en]` |
| `mode=append` | Append to collection | `categories[mode=append]` |
| `mode=replace` | Replace collection values | `categories[mode=replace]` |
| `translator=class` | Custom value translator | `date[translator=...]` |
| `dateformat=pattern` | Date format | `createdAt[dateformat=yyyy-MM-dd]` |

### Referencing Other Types

```impex
# Simple reference
;product(code)
# Nested reference
;catalogVersion(catalog(id),version)
# Reference with default
;catalog(id)[default=productCatalog]
```

### Collection Values

```impex
INSERT_UPDATE Product;code[unique=true];supercategories(code)
;PROD001;CAT1,CAT2,CAT3
```

## ImpEx File Conventions in Patches

```
resources/projectpatches/patches/
├── TICKET-001/              # Folder = ticket ID
│   ├── 01-types.impex       # Numeric prefix for ordering
│   ├── 02-data.impex
│   └── post/                # Post-execution (after Java code)
│       └── 01-cleanup.impex
└── TICKET-002/
    └── 01-update.impex
```

Naming rules:
- Folder name = ticket/task ID
- Prefix files with numbers for ordering: `01-`, `02-`
- Use `post/` subfolder for ImpEx that must run after custom Java code

## Best Practices

1. **Use `INSERT_UPDATE`** unless you specifically need insert-only or update-only
2. **Mark all unique keys** with `[unique=true]`
3. **Use macros** for catalog versions, languages, and repeated references
4. **One patch per ticket** — keeps changes traceable
5. **Test ImpEx in HAC first** (Administration Console → ImpEx Import)
6. **Order matters** — import dependent types before referencing types
7. **Keep patches small** — easier to debug and rerun

## Common Pitfalls

- Missing `[unique=true]` → creates duplicates instead of updating
- Wrong catalog version → data goes to wrong catalog
- Trailing spaces in values → unexpected matching behavior
- Missing semicolons → columns shift and data corrupts
- Circular references → import order matters
- Missing extension dependency → types not available during patch execution
