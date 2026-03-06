# Type System Reference (items.xml)

## Contents
- File structure and ordering
- Item types (creating and extending)
- Attributes and modifiers
- Enum types
- Collection and map types
- Relations (1:N, M:N)
- Deployment tables and typecodes
- Build lifecycle

## File Structure

The `*-items.xml` file must follow this element ordering:

1. `<atomictypes>` — core Java types (rarely needed)
2. `<collectiontypes>` — typed collections (list, set)
3. `<enumtypes>` — enumeration types
4. `<maptypes>` — key-value map types
5. `<relations>` — relationships between types
6. `<itemtypes>` — item type definitions

Location: `<extension>/resources/<extensionname>-items.xml`

## Creating a New Type

```xml
<itemtype code="CustomType" autocreate="true" generate="true"
          jaloclass="com.example.core.jalo.CustomType"
          extends="GenericItem">
    <deployment table="CustomTypes" typecode="29001"/>
    <attributes>
        <attribute qualifier="code" type="java.lang.String">
            <modifiers unique="true" optional="false"/>
            <persistence type="property"/>
            <description>Unique identifier</description>
        </attribute>
        <attribute qualifier="name" type="localized:java.lang.String">
            <persistence type="property"/>
        </attribute>
        <attribute qualifier="active" type="java.lang.Boolean">
            <defaultvalue>Boolean.TRUE</defaultvalue>
            <persistence type="property"/>
        </attribute>
    </attributes>
</itemtype>
```

Key attributes:
- `autocreate="true"` — create the type (first time)
- `generate="true"` — generate Java model class
- `extends` — parent type (default: `GenericItem`)
- `<deployment>` — **required** for new types; maps to DB table

## Extending an Existing Type

```xml
<itemtype code="Product" autocreate="false" generate="false">
    <attributes>
        <attribute qualifier="customField" type="java.lang.String">
            <persistence type="property"/>
        </attribute>
    </attributes>
</itemtype>
```

Rules:
- `autocreate="false"` — type already exists
- `generate="false"` — don't regenerate the base model class
- No `<deployment>` needed (uses existing table)

## Attribute Types

| Java Type | Usage |
|-----------|-------|
| `java.lang.String` | Text fields |
| `java.lang.Integer` | Integers |
| `java.lang.Long` | Long integers |
| `java.lang.Double` | Decimal numbers |
| `java.lang.Boolean` | Boolean flags |
| `java.util.Date` | Date/time values |
| `localized:java.lang.String` | Localized text (per language) |
| `<ItemTypeCode>` | Reference to another item type |

## Attribute Modifiers

```xml
<modifiers read="true" write="true" search="true"
           optional="true" unique="false" initial="false"/>
```

| Modifier | Default | Description |
|----------|---------|-------------|
| `read` | true | Readable via getter |
| `write` | true | Writable via setter |
| `search` | true | Searchable via FlexibleSearch |
| `optional` | true | Can be null |
| `unique` | false | Unique constraint |
| `initial` | false | Set once at creation, then immutable |

## Enum Types

```xml
<enumtype code="CustomStatus" dynamic="true">
    <value code="NEW"/>
    <value code="ACTIVE"/>
    <value code="CLOSED"/>
</enumtype>
```

- `dynamic="true"` — values can be added at runtime via ImpEx (recommended)
- `dynamic="false"` — values fixed at compile time (generates Java enum)

## Collection Types

```xml
<collectiontype code="StringList" elementtype="java.lang.String" type="list"/>
<collectiontype code="ProductSet" elementtype="Product" type="set"/>
```

`type` can be `list`, `set`, or `collection`.

## Relations

### One-to-Many

```xml
<relation code="Product2Condition" generate="true" localized="false" autocreate="true">
    <sourceElement type="Product" qualifier="product" cardinality="one">
        <modifiers optional="false"/>
    </sourceElement>
    <targetElement type="Condition" qualifier="conditions" cardinality="many">
        <modifiers read="true" write="true"/>
    </targetElement>
</relation>
```

No `<deployment>` needed — foreign key stored on the "many" side.

### Many-to-Many

```xml
<relation code="Product2Warehouse" generate="true" localized="false" autocreate="true">
    <deployment table="Prod2Warehouse" typecode="29002"/>
    <sourceElement type="Product" qualifier="products" cardinality="many"/>
    <targetElement type="Warehouse" qualifier="warehouses" cardinality="many"
                   collectiontype="set"/>
</relation>
```

**Requires `<deployment>`** with unique typecode (creates a link table).

## Deployment Rules

- Every new `<itemtype>` needs a `<deployment>` with unique `typecode`
- Every M:N `<relation>` needs a `<deployment>` with unique `typecode`
- **Typecodes are permanent** — never reuse or change after initial deployment
- Use a project-specific typecode range (e.g., 29000–29999)

## Build Lifecycle After Changes

1. `ant clean all` — regenerates model classes from items.xml
2. Start the platform or `ant updatesystem` — applies schema changes to DB

## Common Pitfalls

- Forgetting `ant clean all` after items.xml changes → stale model classes
- Reusing typecodes → corrupt type system
- Missing `<deployment>` on new types → build failure
- Using `autocreate="true"` on existing types → duplicate type error
- Changing `<deployment>` table/typecode after initial deployment → data loss
