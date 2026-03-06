---
name: sap-commerce-development
description: >-
  Guides SAP Commerce Cloud backend development: type system (items.xml),
  ImpEx data management via patches extension, Spring XML bean configuration, 
  OCC REST API, extension architecture, builds, and testing.
  Use when working on SAP Commerce/Hybris backend code, items.xml, ImpEx files,
  Spring XML, OCC controllers, facades, services, DAOs, populators, or any 
  SAP Commerce Cloud extension development.
license: Apache-2.0
metadata:
  version: "2.0"
  platform-version: "2211"
---

# SAP Commerce Cloud Backend Development

## Architecture

SAP Commerce Cloud uses a layered architecture: **Type System → Service Layer → Facade Layer → OCC REST API**.

All custom code lives in extensions under `core-customize/hybris/bin/custom/`.

| Layer | Extension Suffix | Purpose |
|-------|-----------------|---------|
| Models & Services | `core` | Type system, business logic, DAOs |
| Facades & DTOs | `facades` | Data transformation, populators |
| REST API | `occ` | OCC controllers, WsDTOs |
| Data & Patches | `patches` | Patched data management via ImpEx |

## Extension Structure

Each extension contains:

```
<extension>/
├── extensioninfo.xml              # Metadata and dependencies
├── resources/
│   ├── <ext>-items.xml            # Type system definitions
│   ├── <ext>-spring.xml           # Spring bean configuration
│   └── <ext>-beans.xml            # DTO/bean definitions
├── src/                           # Java source
├── testsrc/                       # Test source
└── web/                           # Web module (OCC extensions)
```

## Type System (items.xml)

See [reference/type-system.md](reference/type-system.md) for full XML reference.

**Critical rules:**
- After `*-items.xml` changes → `ant clean all` (regenerates model classes)
- Then `ant updatesystem` to apply schema changes to the DB
- Every new item type needs `<deployment table="..." typecode="..."/>`
- **Typecodes are permanent** — never reuse or change them

### New Type

```xml
<itemtype code="CustomType" autocreate="true" generate="true"
          jaloclass="com.example.core.jalo.CustomType"
          extends="GenericItem">
    <deployment table="CustomTypes" typecode="NNNNN"/>
    <attributes>
        <attribute qualifier="code" type="java.lang.String">
            <modifiers unique="true" optional="false"/>
            <persistence type="property"/>
        </attribute>
    </attributes>
</itemtype>
```

### Extend Existing Type

```xml
<itemtype code="Product" autocreate="false" generate="false">
    <attributes>
        <attribute qualifier="customField" type="java.lang.String">
            <persistence type="property"/>
        </attribute>
    </attributes>
</itemtype>
```

## Patches Extension (Data Management)

See [reference/patches.md](reference/patches.md) for full patches and ImpEx reference.

**Critical rules:**
- Use the OOTB patches framework instead of `@SystemSetup` for data management
- Each patch maps to a ticket/task ID for traceability
- Patches are tracked in DB — already-executed patches are skipped
- Use `INSERT_UPDATE` as default ImpEx mode (idempotent)

### ImpEx Template

```impex
$catalog=productCatalog
$version=catalogversion(catalog(id[default=$catalog]),version[default='Online'])[unique=true,default=$catalog:Online]

INSERT_UPDATE Product;code[unique=true];name[lang=en];$version
;PROD001;My Product;
```

## Spring Configuration

See [reference/spring-config.md](reference/spring-config.md) for full Spring reference.

**Critical rules:**
- Beans in `<ext>-spring.xml`, override SAP beans via `<alias>`
- Never redefine bean IDs starting with `default` — create your own and alias
- Use `parent` attribute for abstract bean inheritance

### Service Bean

```xml
<alias name="defaultCustomService" alias="customService"/>
<bean id="defaultCustomService"
      class="com.example.core.services.impl.DefaultCustomService">
    <property name="modelService" ref="modelService"/>
    <property name="flexibleSearchService" ref="flexibleSearchService"/>
</bean>
```

### Override SAP Bean

```xml
<alias name="customCartService" alias="cartService"/>
<bean id="customCartService"
      class="com.example.core.services.impl.CustomCartServiceImpl"
      parent="defaultCartService">
    <property name="customDep" ref="customDep"/>
</bean>
```

## OCC REST API

See [reference/occ-api.md](reference/occ-api.md) for full OCC reference.

**Critical rules:**
- Controllers in the OCC extension, annotated with `@RestController`
- Convert models to WsDTOs via `DataMapper` — never expose internal models
- Define WsDTOs in `<project>occ-beans.xml`

### Controller

```java
@RestController
@RequestMapping(value = "/{baseSiteId}/resources")
@Api(tags = "Resources")
public class ResourceController extends BaseController {

    @Resource(name = "resourceFacade")
    private ResourceFacade resourceFacade;

    @GetMapping("/{id}")
    @ResponseStatus(HttpStatus.OK)
    public ResourceWsDTO getResource(@PathVariable final String id) {
        final ResourceData data = resourceFacade.getById(id);
        return getDataMapper().map(data, ResourceWsDTO.class);
    }
}
```

## Database Queries

Always use FlexibleSearch, never raw SQL:

```java
final FlexibleSearchQuery query = new FlexibleSearchQuery(
    "SELECT {pk} FROM {Product} WHERE {code} = ?code");
query.addQueryParameter("code", productCode);
SearchResult<ProductModel> result = flexibleSearchService.search(query);
```

## Testing

See [reference/testing.md](reference/testing.md) for full testing reference.

**Critical rules:**
- `@UnitTest` + `@ExtendWith(MockitoExtension.class)`
- AAA pattern: Arrange → Act → Assert
- Use `assertThrows`, not try-catch
- Use specific Mockito matchers for overloaded methods: `any(ConfigModel.class)`, not `any()`

## Build & Test

```bash
cd /path/to/project && . ./setenv.sh      # Setup environment
# From hybris/bin/platform:
ant build                                  # Quick compile
ant clean all                              # After items.xml changes
ant updatesystem                           # Apply type + patch changes to DB

# Tests
ant unittests -Dtestclasses.extensions="<ext>" \
              -Dtestclasses.suppress.junit.tenant=true
ant integrationtests -Dtestclasses.extensions="<ext>"
```

**Important:** Run `ant build` after changing test files before running tests.

## Feature Development Workflow

1. **Models** — Define/extend types in `*-items.xml` → `ant clean all`
2. **Service** — Implement business logic in the `core` extension
3. **Facade** — Create facade + populators in the `facades` extension
4. **API** — Add OCC endpoint in the `occ` extension (if REST API needed)
5. **Data** — Create a patch class + ImpEx files in the `patches` extension
6. **Tests** — Write unit tests → `ant build` → run tests

## Safety Rules

- **NEVER modify** `hybris/bin/platform/` or `hybris/bin/modules/` (SAP core)
- **Config location**: `core-customize/hybris/config/` (NOT `hybris/config/`)
- Use a project-specific prefix for all custom properties

## References

- [reference/type-system.md](reference/type-system.md) — items.xml: types, enums, relations, deployment
- [reference/patches.md](reference/patches.md) — Patches extension, ImpEx syntax, macros, lifecycle
- [reference/spring-config.md](reference/spring-config.md) — Spring beans, aliases, overrides, list/map modification
- [reference/occ-api.md](reference/occ-api.md) — OCC controllers, WsDTOs, beans.xml, DataMapper
- [reference/testing.md](reference/testing.md) — Unit tests, Mockito, AAA pattern
