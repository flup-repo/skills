# Spring Configuration Reference

## Contents
- Bean definitions and aliases
- Overriding SAP beans
- Dependency injection
- Abstract beans and parent inheritance
- Modifying existing lists/maps
- Application context hierarchy
- Common SAP beans

## Bean Definition Files

Each extension has `<extensionname>-spring.xml` in its `resources/` directory.
Loaded automatically in extension build order (declared in `extensioninfo.xml`).

## Defining a Bean with Alias

For beans that other extensions might override:

```xml
<alias name="defaultCustomService" alias="customService"/>
<bean id="defaultCustomService"
      class="com.example.core.services.impl.DefaultCustomService">
    <property name="modelService" ref="modelService"/>
    <property name="flexibleSearchService" ref="flexibleSearchService"/>
</bean>
```

Reference via the alias: `ref="customService"`.

## Overriding SAP Beans

SAP Commerce uses the **alias pattern** for extensibility:

```xml
<!-- SAP defines: -->
<alias alias="cartService" name="defaultCartService"/>
<bean id="defaultCartService" class="de.hybris.platform.order.impl.DefaultCartService"/>

<!-- Override by redefining the alias: -->
<alias alias="cartService" name="customCartService"/>
<bean id="customCartService"
      class="com.example.core.services.impl.CustomCartServiceImpl"
      parent="defaultCartService">
    <property name="customDep" ref="customDep"/>
</bean>
```

**Key rule:** Always override via `<alias>`, never redefine the `defaultXxx` bean ID directly.

## Parent Bean Inheritance

Use `parent` to inherit properties from an abstract or concrete bean:

```xml
<bean id="customAction"
      class="com.example.core.actions.CustomAction"
      parent="abstractAction">
    <property name="myService" ref="customService"/>
</bean>
```

## Dependency Injection

### Setter Injection (standard in SAP Commerce)
```xml
<bean id="myBean" class="com.example.core.MyClass">
    <property name="service" ref="customService"/>
    <property name="timeout" value="5000"/>
</bean>
```

### Constructor Injection
```xml
<bean id="myBean" class="com.example.core.MyClass">
    <constructor-arg ref="customService"/>
    <constructor-arg value="5000"/>
</bean>
```

### List/Set Properties
```xml
<property name="strategies">
    <list>
        <ref bean="strategy1"/>
        <ref bean="strategy2"/>
    </list>
</property>
```

## Modifying Existing Lists/Maps

Standard Spring merge doesn't work across extensions. Use SAP's directives:

### Adding to a List
```xml
<bean id="customListMerge"
      depends-on="existingBeanId"
      parent="listMergeDirective">
    <property name="add" ref="newEntry"/>
    <property name="listPropertyDescriptor" value="propertyName"/>
</bean>
```

### Adding to a Map
```xml
<bean id="customMapMerge"
      depends-on="existingBeanId"
      parent="mapMergeDirective">
    <property name="key" value="myKey"/>
    <property name="value" ref="newValueBean"/>
</bean>
```

## Application Context Hierarchy

```
GlobalApplicationContext (shared singleton)
  └── Core ApplicationContext (per tenant)
       └── Web ApplicationContext (per web module: OCC, backoffice)
```

- Core context beans are visible to web context (parent relationship)
- Web context beans cannot override core context beans
- Extension load order determines which definition wins (last loaded wins)

## Extension Load Order

Controlled by `<requires-extension>` in `extensioninfo.xml`:
```xml
<requires-extension name="projectcore"/>
```
Extensions loaded later can override beans from earlier extensions.

## Common SAP Beans

| Bean ID | Interface | Purpose |
|---------|-----------|---------|
| `modelService` | `ModelService` | CRUD operations on models |
| `flexibleSearchService` | `FlexibleSearchService` | Database queries |
| `userService` | `UserService` | Current user/session |
| `sessionService` | `SessionService` | Session management |
| `configurationService` | `ConfigurationService` | Read properties |
| `eventService` | `EventService` | Publish/subscribe events |
| `catalogVersionService` | `CatalogVersionService` | Catalog version management |
| `commonI18NService` | `CommonI18NService` | Localization |
| `businessProcessService` | `BusinessProcessService` | Process engine |

## Common Pitfalls

- Defining a bean with the same ID in two extensions → last loaded wins silently
- Forgetting `<alias>` → cannot be overridden by downstream extensions
- Wrong `parent` reference → missing property initialization
- Circular dependencies → application context fails to start
