# Unit Testing Reference

## Contents
- Test class structure
- Naming conventions
- Mockito patterns
- Common test patterns
- SAP Commerce test specifics

## Test Class Structure

```java
@UnitTest
@ExtendWith(MockitoExtension.class)
class DefaultCustomServiceTest {

    private static final String TEST_CODE = "TEST123";

    @InjectMocks
    private DefaultCustomService service;

    @Mock
    private ModelService modelService;

    @Mock
    private FlexibleSearchService flexibleSearchService;

    @Test
    void testGetProduct_WithValidCode_ReturnsProduct() {
        // Arrange
        final ProductModel product = new ProductModel();
        product.setCode(TEST_CODE);
        when(flexibleSearchService.search(any(FlexibleSearchQuery.class)))
            .thenReturn(new SearchResult<>(List.of(product), 1, 1, 0));

        // Act
        final ProductModel result = service.getProduct(TEST_CODE);

        // Assert
        assertEquals(TEST_CODE, result.getCode());
        verify(flexibleSearchService).search(any(FlexibleSearchQuery.class));
    }
}
```

## Key Rules

- `@UnitTest` annotation (from `de.hybris.bootstrap.annotations.UnitTest`)
- `@ExtendWith(MockitoExtension.class)` — no JUnit 4 runners
- **Package-private** classes and methods (no `public`)
- **AAA pattern**: Arrange → Act → Assert

## Mockito Patterns

### Stubbing
```java
when(service.method(any())).thenReturn(value);
when(service.method(eq("specific"))).thenReturn(value);
doNothing().when(service).voidMethod(any());
doThrow(new RuntimeException()).when(service).failingMethod();
```

### Verification
```java
verify(service).method(any());
verify(service, never()).method(any());
verify(service, times(2)).method(any());
// Don't use times(1) — it's the default
```

### Overloaded Methods (Critical!)
```java
// WRONG: ambiguous method reference
verify(service).getConfig(any(), anyString());

// CORRECT: use specific type matchers
verify(service).getConfig(any(ConfigModel.class), anyString());
```

## Common Test Patterns

### Testing Exceptions
```java
@Test
void testValidate_WithNull_ThrowsException() {
    assertThrows(IllegalArgumentException.class,
        () -> service.validate(null));
}
```

### Populator Testing
```java
@Test
void shouldPopulateCodeFromSource() {
    final ProductModel source = new ProductModel();
    source.setCode("ABC");

    final ProductData target = new ProductData();
    populator.populate(source, target);

    assertEquals("ABC", target.getCode());
}
```

## SAP Commerce Specifics

### Running Tests
```bash
# All tests in an extension
ant unittests -Dtestclasses.extensions="<extensionname>" \
              -Dtestclasses.suppress.junit.tenant=true

# IMPORTANT: Package-level filtering doesn't work reliably
# DON'T use: -Dtestclasses.packages=com.example.core.services
```

Always `ant build` after changing test source files before running tests.

## Pitfalls

- Using `any()` with overloaded methods → use `any(SpecificType.class)`
- Forgetting `ant build` before running tests → stale compiled test classes
- Using package-level test filtering → tests not found
- Mocking incomplete chains → NPE in production code paths
- Not stubbing all chained calls (e.g., `getParent().getCode()`)
- Using try-catch instead of `assertThrows`
