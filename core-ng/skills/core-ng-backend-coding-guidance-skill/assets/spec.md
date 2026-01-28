# Core-NG Framework

Comprehensive specification for Wonder's Core-NG framework - a forked version of [Neo's core-ng-project](https://github.com/neowu/core-ng-project) customized for Wonder's needs.

## Purpose

Core-NG is an opinionated Java web framework that provides:
- Dependency injection system
- MongoDB integration with automatic codec generation
- Kafka messaging integration
- HTTP/WebService framework with automatic API generation
- Configuration management
- Structured logging and monitoring

**Key Philosophy**: Convention over configuration. Following framework conventions leads to clean, maintainable code. Violating conventions leads to cryptic runtime errors.

---

## Requirements

### Requirement: Dependency Injection with bind()

The system SHALL use field-based dependency injection where ALL injectable services MUST be registered via `bind()` in a Module.

#### Scenario: Auto-create service with dependencies
- **WHEN** a service class with `@Inject` fields is bound using `bind(MyService.class)`
- **THEN** framework creates instance via reflection
- **AND** automatically injects all `@Inject` fields
- **AND** registers bean for future injection

```java
public class BOAllergenService {
    @Inject
    MongoCollection<Allergen> allergenCollection;

    @Inject
    BOItemService itemService;
}

// In Module
bind(BOAllergenService.class);  // Auto-creates and injects
```

#### Scenario: Manual instance creation with parameters
- **WHEN** a service requires constructor parameters
- **THEN** developer creates instance manually
- **AND** calls `bind(instance)` to inject fields and register

```java
public class ReportService {
    private final String reportPath;

    @Inject
    MongoCollection<Report> collection;

    public ReportService(String path) {
        this.reportPath = path;
    }
}

// In Module
bind(new ReportService("/tmp/reports"));  // Injects fields
```

#### Scenario: Binding fails without registration
- **WHEN** a class with `@Inject` field references unbound service
- **THEN** framework throws runtime error "No binding found for class X"

#### Scenario: Field injection only
- **WHEN** developer attempts constructor injection
- **THEN** framework does not support it (no error, just not injected)
- **NOTE**: Only field injection with `@Inject` is supported

---

### Requirement: Module System and Initialization Order

The system SHALL organize application components into Modules with proper initialization order.

#### Scenario: Module structure
- **WHEN** developer creates a new module
- **THEN** it MUST extend `core.framework.module.Module`
- **AND** override `initialize()` method
- **AND** perform all configuration and binding within `initialize()`

```java
public class ItemModule extends Module {
    @Override
    protected void initialize() {
        bind(BOItemService.class);
        bind(BOAllergenService.class);
    }
}
```

#### Scenario: Loading modules in correct order
- **WHEN** application initializes
- **THEN** infrastructure modules MUST be loaded before business modules
- **ORDER**: Properties → Database → Messaging → HTTP → Business logic

```java
public class AppModule extends Module {
    @Override
    protected void initialize() {
        loadProperties("sys.properties");
        load(new MongoModule());      // Infrastructure
        load(new KafkaModule());      // Infrastructure
        http().listenHTTP("8080");    // Infrastructure
        load(new ItemModule());       // Business
    }
}
```

#### Scenario: Module loading triggers config initialization
- **WHEN** `load(new MongoModule())` is called
- **THEN** framework calls MongoModule's `initialize()` method
- **AND** configs created via `config(MongoConfig.class)` are lazy-initialized
- **AND** all configs are validated after all modules loaded

---

### Requirement: MongoDB Collection and View Registration

The system SHALL require registration of ALL MongoDB entity classes and their static inner classes.

#### Scenario: Register main entity with collection()
- **WHEN** a class is annotated with `@Collection`
- **THEN** developer MUST call `config.collection(EntityClass.class)` in MongoModule
- **AND** framework generates encoders/decoders for the entity
- **AND** creates `MongoCollection<Entity>` bean for injection

```java
@Collection(name = "item_versions")
public class ItemVersion {
    @Id
    public ObjectId id;

    @Field(name = "item_number")
    public String itemNumber;
}

// In MongoModule
MongoConfig config = config(MongoConfig.class);
config.uri(requiredProperty("sys.mongo.uri"));
config.collection(ItemVersion.class);
```

#### Scenario: Register inner classes with view()
- **WHEN** entity contains static inner class fields
- **THEN** developer MUST call `config.view(InnerClass.class)` for EACH inner class
- **AND** framework generates codec for the inner class
- **REASON**: Inner classes are used in field declarations and need codecs for serialization

```java
@Collection(name = "item_versions")
public class ItemVersion {
    @Id
    public ObjectId id;

    @Field(name = "recipe")
    public Recipe recipe;  // Static inner class

    public static class Recipe {
        @Field(name = "components")
        public List<RecipeComponent> components;
    }

    public static class RecipeComponent {
        @Field(name = "item_version_id")
        public String itemVersionId;
    }
}

// In MongoModule - MUST register ALL inner classes
config.collection(ItemVersion.class);
config.view(ItemVersion.Recipe.class);           // Required!
config.view(ItemVersion.RecipeComponent.class);  // Required!
```

#### Scenario: Missing view() registration causes runtime error
- **WHEN** developer forgets to register an inner class with `view()`
- **AND** code attempts to update document containing that inner class
- **THEN** framework throws "No codec found for class X" at runtime
- **IMPACT**: Application crashes during `collection.update()` operations

#### Scenario: MongoDB entity field annotations
- **WHEN** defining MongoDB entity fields
- **THEN** top-level entity MUST have `@Collection(name = "collection_name")`
- **AND** MUST have exactly ONE field with `@Id` annotation
- **AND** ALL other fields MUST have `@Field(name = "field_name")` annotation
- **AND** MUST NOT have `@Property` annotation (that's for API only)

```java
// CORRECT
@Collection(name = "items")
public class Item {
    @Id
    public ObjectId id;

    @Field(name = "item_number")
    public String itemNumber;
}

// WRONG - Missing @Field
@Collection(name = "items")
public class Item {
    @Id
    public ObjectId id;

    public String itemNumber;  // ERROR: No @Field
}

// WRONG - Mixed annotations
@Collection(name = "items")
public class Item {
    @Id
    public ObjectId id;

    @Field(name = "name")
    @Property(name = "name")  // ERROR: Don't mix!
    public String name;
}
```

---

### Requirement: Kafka Message Publishing and Subscribing

The system SHALL support type-safe Kafka messaging with automatic bean registration.

#### Scenario: Publish message to Kafka topic
- **WHEN** developer calls `config.publish("topic", MessageClass.class)`
- **THEN** framework creates `MessagePublisher<MessageClass>` instance
- **AND** automatically binds it as injectable bean
- **AND** developer can inject and use publisher

```java
// Message class
public class ItemChanged {
    @Property(name = "item_number")
    public String itemNumber;

    @Property(name = "version_id")
    public Integer versionId;
}

// In KafkaModule
KafkaConfig config = kafka();
config.uri(requiredProperty("sys.kafka.uri"));
config.publish("item-changed", ItemChanged.class);

// Usage in service
public class ItemService {
    @Inject
    MessagePublisher<ItemChanged> publisher;

    public void notifyChange(String itemNumber) {
        ItemChanged msg = new ItemChanged();
        msg.itemNumber = itemNumber;
        publisher.publish(itemNumber, msg);  // key, value
    }
}
```

#### Scenario: Subscribe to Kafka topic
- **WHEN** developer calls `config.subscribe("topic", MessageClass.class, handler)`
- **THEN** framework automatically injects dependencies into handler
- **AND** routes messages from topic to handler
- **AND** handler processes messages asynchronously

```java
// Handler implementation
public class ItemChangedHandler implements MessageHandler<ItemChanged> {
    @Inject
    ItemVersionService itemVersionService;

    @Override
    public void handle(String key, ItemChanged message) {
        itemVersionService.processChange(message);
    }
}

// In KafkaModule
config.subscribe("item-changed", ItemChanged.class, new ItemChangedHandler());
```

#### Scenario: Message class requirements
- **WHEN** defining Kafka message class
- **THEN** ALL fields MUST have `@Property(name = "field_name")` annotation
- **AND** follows JSON serialization rules
- **AND** supports nested objects, lists, maps

---

### Requirement: WebService API Definition and Registration

The system SHALL support type-safe WebService APIs with automatic HTTP route generation.

#### Scenario: Define WebService interface
- **WHEN** creating a new API
- **THEN** developer defines interface with HTTP method annotations
- **AND** uses `@Path` for route patterns
- **AND** uses `@PathParam`, `@QueryParam` for parameters

```java
public interface AllergenWebService {
    @GET
    @Path("/api/allergen/:id")
    AllergenResponse get(@PathParam("id") String id);

    @POST
    @Path("/api/allergen")
    CreateResponse create(CreateAllergenRequest request);

    @PUT
    @Path("/api/allergen/:id")
    void update(@PathParam("id") String id, UpdateAllergenRequest request);
}
```

#### Scenario: Register WebService implementation
- **WHEN** implementing WebService interface
- **THEN** developer binds implementation class
- **AND** registers with `api().service()` to expose HTTP endpoints
- **AND** framework automatically validates and generates routes

```java
// Implementation
public class AllergenWebServiceImpl implements AllergenWebService {
    @Inject
    BOAllergenService allergenService;

    @Override
    public AllergenResponse get(String id) {
        return allergenService.get(id);
    }

    @Override
    public CreateResponse create(CreateAllergenRequest request) {
        return allergenService.create(request);
    }

    @Override
    public void update(String id, UpdateAllergenRequest request) {
        allergenService.update(id, request);
    }
}

// In Module
AllergenWebService impl = bind(AllergenWebServiceImpl.class);
api().service(AllergenWebService.class, impl);
```

#### Scenario: Request and Response beans
- **WHEN** defining API request/response classes
- **THEN** ALL fields MUST have `@Property(name = "field_name")` annotation
- **AND** can use validation annotations (`@NotNull`, `@NotBlank`, `@Size`, etc.)
- **AND** framework automatically validates before method execution

```java
public class CreateAllergenRequest {
    @NotNull
    @NotBlank
    @Property(name = "name")
    public String name;

    @Property(name = "display_name")
    public String displayName;

    @Size(min = 1, max = 100)
    @Property(name = "tags")
    public List<String> tags;
}
```

#### Scenario: Automatic API documentation
- **WHEN** WebService is registered with `api().service()`
- **THEN** framework exposes API documentation at `/_sys/api` endpoint
- **AND** includes all routes, request/response schemas, validation rules

---

### Requirement: HTTP Exception Handling

The system SHALL provide standard HTTP exceptions that map to appropriate status codes.

#### Scenario: Bad request for invalid user input
- **WHEN** user provides invalid input
- **THEN** service throws `BadRequestException` with error message
- **AND** framework returns HTTP 400 with error details

```java
if (Strings.isBlank(itemNumber)) {
    throw new BadRequestException("Item number is required");
}

if (isDuplicate(itemNumber)) {
    throw new BadRequestException("Duplicate item", "DUPLICATE_ITEM");
}
```

#### Scenario: Not found for missing resources
- **WHEN** requested resource doesn't exist
- **THEN** service throws `NotFoundException` with identifier
- **AND** framework returns HTTP 404

```java
ItemVersion item = itemVersionCollection.get(id)
    .orElseThrow(() -> new NotFoundException(
        Strings.format("item version not found, id={}", id)
    ));
```

#### Scenario: Conflict for business rule violations
- **WHEN** operation violates business rules
- **THEN** service throws `ConflictException` with reason
- **AND** framework returns HTTP 409

```java
if (itemVersion.versionStatus == VersionStatus.FINAL) {
    throw new ConflictException("Cannot modify published version");
}
```

#### Scenario: Standard exception types
- **WHEN** handling errors
- **THEN** use appropriate exception type:
  - `BadRequestException` (400) - Invalid user input
  - `UnauthorizedException` (401) - Authentication required
  - `ForbiddenException` (403) - Permission denied
  - `NotFoundException` (404) - Resource not found
  - `ConflictException` (409) - Business rule violation
  - `TooManyRequestsException` (429) - Rate limit exceeded

---

### Requirement: Configuration Property Management

The system SHALL provide type-safe configuration property access with validation.

#### Scenario: Required property access
- **WHEN** application requires a configuration property
- **THEN** developer uses `requiredProperty("key")`
- **AND** framework throws error at startup if property missing

```java
String mongoUri = requiredProperty("sys.mongo.uri");
String kafkaUri = requiredProperty("sys.kafka.uri");
```

#### Scenario: Optional property access
- **WHEN** configuration property is optional
- **THEN** developer uses `property("key")` which returns `Optional<String>`
- **AND** provides default value via `orElse()` or `orElseGet()`

```java
Optional<String> optional = property("optional.key");
String value = property("slack.channel").orElse("#default");
int timeout = property("http.timeout")
    .map(Integer::parseInt)
    .orElse(30);
```

#### Scenario: Property file loading
- **WHEN** application starts
- **THEN** developer loads properties via `loadProperties("filename")`
- **AND** supports variable substitution in property values
- **AND** later loaded properties override earlier ones

```java
loadProperties("sys.properties");
loadProperties("app-${app.env}.properties");  // Variable substitution
```

#### Scenario: Property validation
- **WHEN** all modules initialized
- **THEN** framework validates all declared properties are used
- **AND** validates all used properties are declared
- **AND** reports unused or undeclared properties

---

### Requirement: Controller Pattern for Custom Endpoints

The system SHALL support custom Controller implementation for endpoints not following WebService pattern.

#### Scenario: Implement custom controller
- **WHEN** endpoint doesn't fit WebService pattern (e.g., migration, admin tools)
- **THEN** developer implements `Controller` interface
- **AND** manually accesses Request object
- **AND** returns response object

```java
public class MigrationController implements Controller {
    @Inject
    MongoCollection<Item> itemCollection;

    @Override
    public Object execute(Request request) {
        String id = request.pathParam("id");
        Boolean dryRun = request.queryParam("dry_run")
            .map(Boolean::parseBoolean)
            .orElse(true);

        // ... migration logic

        return new MigrationResponse();
    }
}
```

#### Scenario: Register controller route
- **WHEN** controller is implemented
- **THEN** developer registers route via `http().route()`
- **AND** optionally registers request/response beans

```java
http().route(HTTPMethod.POST, "/api/migration/:id", new MigrationController());
http().bean(MigrationResponse.class);  // For API documentation
```

---

## Common Mistakes and Solutions

### Mistake: Forgetting to bind new service

**Problem**: Created new service but forgot to add `bind()` in Module

```java
// Wrong - Service created but not bound
public class NewService {
    @Inject
    ItemService itemService;  // Will fail at runtime!
}

// Usage elsewhere
public class OtherService {
    @Inject
    NewService newService;  // Error: No binding found!
}
```

**Solution**: Always bind new services in appropriate Module

```java
// In ItemModule.java
bind(NewService.class);  // Add this!
```

### Mistake: Missing view() for inner classes

**Problem**: Registered entity but forgot inner classes

```java
// Wrong - Runtime codec error
config.collection(ItemVersion.class);
// Missing inner class registrations!
```

**Solution**: Register ALL static inner classes

```java
config.collection(ItemVersion.class);
config.view(ItemVersion.Recipe.class);
config.view(ItemVersion.RecipeComponent.class);
config.view(ItemVersion.Ingredient.class);
// ... all inner classes
```

### Mistake: Mixing @Field and @Property

**Problem**: Using @Property on MongoDB entity fields

```java
// Wrong - Validation error
@Collection(name = "items")
public class Item {
    @Field(name = "name")
    @Property(name = "name")  // Don't mix!
    public String name;
}
```

**Solution**: Separate concerns - use @Field for MongoDB, @Property for API

```java
// MongoDB entity
@Collection(name = "items")
public class Item {
    @Field(name = "name")
    public String name;
}

// API view (separate class)
public class ItemView {
    @Property(name = "name")
    public String name;
}
```

### Mistake: Not using bind() before api().service()

**Problem**: Manually creating service without bind()

```java
// Wrong - Dependencies not injected
MyServiceImpl service = new MyServiceImpl();
api().service(MyService.class, service);
```

**Solution**: Always use bind() to inject dependencies

```java
// Correct
MyServiceImpl service = bind(MyServiceImpl.class);
api().service(MyService.class, service);
```

### Mistake: Multiple public constructors

**Problem**: Class has multiple constructors

```java
// Wrong - Framework error
public class MyService {
    public MyService() {}
    public MyService(String param) {}  // Only one allowed!
}
```

**Solution**: Single constructor with parameters, or default constructor with @Inject

```java
// Option 1: Constructor with parameters (use bind(instance))
public class MyService {
    private final String param;

    @Inject
    ItemService itemService;

    public MyService(String param) {
        this.param = param;
    }
}
bind(new MyService("value"));

// Option 2: Default constructor with field injection
public class MyService {
    @Inject
    ItemService itemService;
}
bind(MyService.class);
```

---

## Best Practices

### Prefer auto-creation with bind(Class)
- Use `bind(MyService.class)` when no constructor parameters needed
- Framework handles dependency injection automatically
- Cleaner and more maintainable

### Organize modules by domain
- One module per functional area (Item, Allergen, Vendor, etc.)
- Keep `initialize()` clean by extracting helper methods
- Load infrastructure before business modules

### Separate MongoDB entities from API views
- MongoDB entities use `@Collection`, `@Field`, `@Id`
- API views use `@Property`, validation annotations
- Prevents annotation conflicts
- Allows different field names for DB vs API

### Use standard exceptions
- Leverage built-in exception types
- Add custom error codes for client disambiguation
- Let framework handle error response formatting

### Validate early in Module initialization
- Load properties first
- Validate required configs during Module init
- Fail fast at startup rather than runtime

---

## Framework Constraints

### Hard Limits
- Only ONE public constructor per class (for auto-binding)
- Only FIELD injection (no constructor injection)
- MongoDB @Id field must be ObjectId or String
- WebService interface methods must have HTTP annotations
- Message handlers must be non-lambda classes (for injection)

### Validation Rules
- All MongoDB entity fields must have @Field annotation
- All API bean fields must have @Property annotation
- All inner classes must be registered with view()
- WebService implementation must match interface exactly

### Performance Characteristics
- Codec generation is one-time at startup (not runtime)
- Bean lookup is O(1) HashMap access
- HTTP route matching uses prefix tree
- MongoDB uses connection pooling (default pool size configurable)

---

## Related Documentation

- Upstream project: https://github.com/neowu/core-ng-project
- Upstream Wiki: https://github.com/neowu/core-ng-project/wiki
- Wonder fork: https://github.com/food-truck/wonder-core-ng-project
- Change logs: WONDER-CHANGELOG.md, CHANGELOG.md in core-ng repo
