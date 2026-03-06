# OCC REST API Reference

## Contents
- Architecture overview
- Extension structure
- Controller pattern
- WsDTO definitions (beans.xml)
- Data mapping and field levels
- Error handling

## Architecture

OCC (Omni Commerce Connect) is the REST API layer built on Spring MVC.

```
HTTP Request → OCC Controller → Facade → Service → DAO → Database
                    ↓
              Model → WsDTO (via DataMapper)
                    ↓
              HTTP Response (JSON/XML)
```

## OCC Extension Structure

```
<project>occ/
├── src/com/example/occ/
│   └── controllers/
│       └── ResourceController.java
├── resources/
│   ├── <project>occ-beans.xml        # WsDTO definitions
│   └── <project>occ-spring.xml       # Spring config
├── web/
│   └── WEB-INF/
│       ├── web.xml
│       └── config/
│           └── v2-web-spring.xml     # Web context config
└── extensioninfo.xml
```

## Controller Pattern

```java
@RestController
@RequestMapping(value = "/{baseSiteId}/resources")
@Api(tags = "Resources")
public class ResourceController extends BaseController {

    @Resource(name = "resourceFacade")
    private ResourceFacade resourceFacade;

    @GetMapping
    @ResponseStatus(HttpStatus.OK)
    @ApiOperation(value = "Get all resources")
    public ResourceListWsDTO getResources(
            @ApiParam(value = "Page size") @RequestParam(defaultValue = "20") final int pageSize) {
        final List<ResourceData> results = resourceFacade.getAll(pageSize);
        final ResourceListWsDTO dto = new ResourceListWsDTO();
        dto.setResources(getDataMapper().mapAsList(results, ResourceWsDTO.class, null));
        return dto;
    }

    @GetMapping("/{resourceId}")
    @ResponseStatus(HttpStatus.OK)
    @ApiOperation(value = "Get resource by ID")
    public ResourceWsDTO getResource(@PathVariable final String resourceId) {
        final ResourceData data = resourceFacade.getById(resourceId);
        return getDataMapper().map(data, ResourceWsDTO.class);
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    @ApiOperation(value = "Create resource")
    public ResourceWsDTO createResource(@RequestBody final ResourceWsDTO request) {
        final ResourceData inputData = getDataMapper().map(request, ResourceData.class);
        final ResourceData result = resourceFacade.create(inputData);
        return getDataMapper().map(result, ResourceWsDTO.class);
    }
}
```

## WsDTO Definitions (beans.xml)

Define DTOs in `<project>occ-beans.xml`:

```xml
<bean class="com.example.occ.dto.ResourceWsDTO">
    <property name="code" type="String"/>
    <property name="name" type="String"/>
    <property name="active" type="Boolean"/>
    <property name="price" type="de.hybris.platform.commercewebservicescommons.dto.product.PriceWsDTO"/>
</bean>

<bean class="com.example.occ.dto.ResourceListWsDTO">
    <property name="resources" type="java.util.List&lt;com.example.occ.dto.ResourceWsDTO>"/>
</bean>
```

**Facade-level DTOs** go in `<project>facades-beans.xml`:

```xml
<bean class="com.example.facades.data.ResourceData">
    <property name="code" type="String"/>
    <property name="name" type="String"/>
    <property name="active" type="Boolean"/>
</bean>
```

## Data Mapping

The `DataMapper` (via `BaseController.getDataMapper()`) converts between Data objects and WsDTOs:

```java
// Single object
ResourceWsDTO dto = getDataMapper().map(data, ResourceWsDTO.class);

// List
List<ResourceWsDTO> dtos = getDataMapper().mapAsList(dataList, ResourceWsDTO.class, null);

// With field selection
ResourceWsDTO dto = getDataMapper().map(data, ResourceWsDTO.class, fields);
```

Configure field levels in `*-spring.xml`:
```xml
<bean parent="fieldSetLevelMapping">
    <property name="dtoClass" value="com.example.occ.dto.ResourceWsDTO"/>
    <property name="levelMapping">
        <map>
            <entry key="BASIC" value="code,name"/>
            <entry key="DEFAULT" value="code,name,active"/>
            <entry key="FULL" value="code,name,active,price"/>
        </map>
    </property>
</bean>
```

## Error Handling

Use `WebserviceValidationException` or return appropriate HTTP status:

```java
if (data == null) {
    throw new RequestParameterException("Resource not found",
        RequestParameterException.MISSING, "resourceId");
}
```

## Common Pitfalls

- Exposing internal models directly → always use WsDTOs
- Missing `@Api` annotations → incomplete Swagger documentation
- Not using `DataMapper` → manual mapping is error-prone
- Forgetting to register beans.xml DTOs → serialization fails
- Wrong request mapping → conflicts with existing OCC endpoints
