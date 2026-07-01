# 🚀 Spring Boot Annotations

---
🔷 Spring Beans – Complete Single Table

| Category  | Annotation / Method | Where Used | Purpose / What it Does                       |
| --------- | ------------------- | ---------- | -------------------------------------------- |
| Core Bean | `@Component`        | Class      | Marks class as Spring-managed bean           |
| Core Bean | `@Service`          | Class      | Service layer specialization of `@Component` |
| Core Bean | `@Repository`       | Class      | DAO layer + exception translation            |
| Core Bean | `@Controller`       | Class      | MVC controller                               |
| Core Bean | `@RestController`   | Class      | REST API controller                          |
| Core Bean | `@Configuration`    | Class      | Defines configuration class                  |
| Core Bean | `@Bean`             | Method     | Manually creates and registers bean          |
| Dependency Injection | `@Autowired`         | Field / Constructor / Setter | Auto inject dependency           |
| Dependency Injection | `@Qualifier("name")` | Field / Param                | Resolve multiple beans           |
| Dependency Injection | `@Primary`           | Class                        | Default bean selection           |
| Dependency Injection | `@Inject`            | Field / Constructor          | Java alternative to `@Autowired` |
| Dependency Injection | `@Resource`          | Field                        | Inject by name                   |
| Scope    | `@Scope("singleton")`   | Class      | Default single instance |
| Scope    | `@Scope("prototype")`   | Class      | New instance each time  |
| Scope    | `@Scope("request")`     | Class      | Per HTTP request        |
| Scope    | `@Scope("session")`     | Class      | Per session             |
| Scope    | `@Scope("application")` | Class      | Per servlet context     |
| Lifecycle | `@PostConstruct`                        | Method     | Called after bean init |
| Lifecycle | `@PreDestroy`                           | Method     | Called before destroy  |
| Lifecycle | `InitializingBean.afterPropertiesSet()` | Method     | Init callback          |
| Lifecycle | `DisposableBean.destroy()`              | Method     | Destroy callback       |
| Lifecycle | `initMethod` (in `@Bean`)               | Method ref | Custom init method     |
| Lifecycle | `destroyMethod` (in `@Bean`)            | Method ref | Custom destroy method  |
| Configuration | `@Value("${key}")`         | Field      | Inject property value   |
| Configuration | `@PropertySource`          | Class      | Load properties file    |
| Configuration | `@ConfigurationProperties` | Class      | Bind properties to POJO |
| Conditional / Advanced | `@Conditional`      | Class / Method | Load bean conditionally      |
| Conditional / Advanced | `@Profile("dev")`   | Class          | Environment-based beans      |
| Conditional / Advanced | `@Lazy`             | Class / Field  | Lazy initialization          |
| Conditional / Advanced | `@DependsOn`        | Class          | Control initialization order |
| Context Methods | `ApplicationContext.getBean()` | Code       | Fetch bean manually    |
| Context Methods | `BeanFactory.getBean()`        | Code       | Low-level bean access  |
| Context Methods | `containsBean()`               | Code       | Check bean existence   |
| Context Methods | `getBeanDefinitionNames()`     | Code       | List all beans         |
| Aware Interfaces | `BeanNameAware`           | Class      | Get bean name             |
| Aware Interfaces | `BeanFactoryAware`        | Class      | Access BeanFactory        |
| Aware Interfaces | `ApplicationContextAware` | Class      | Access ApplicationContext |
| Bean Lifecycle Flow | Bean Instantiation   | Internal   | Object creation        |
| Bean Lifecycle Flow | Dependency Injection | Internal   | Inject dependencies    |
| Bean Lifecycle Flow | `@PostConstruct`     | Method     | Init logic             |
| Bean Lifecycle Flow | Bean Ready           | Internal   | Ready to use           |
| Bean Lifecycle Flow | `@PreDestroy`        | Method     | Cleanup                |

---

# 🏗️ MVC Architecture & Core Concepts

| Main Topic | Layer / Responsibility | Annotations Used (with 1-line purpose) | Functions / Methods Used (with purpose) |
| :--- | :--- | :--- | :--- |
| **MVC Architecture** | Overall application design | `@Controller` – Handles requests and returns views<br>`@RestController` – Handles REST requests and returns JSON | `model.addAttribute()` – Sends data to view |
| **Application Flow** | End-to-end execution | **MVC flow** | Request → Controller → Service → Repository → DB → Response |
| **Dependency Injection** | Inject required objects | `@Autowired` – Inject bean<br>`@Component` – Generic Spring bean<br>`@Configuration` – Configuration class<br>`@Bean` – Manual bean creation | **Constructor injection** – Preferred DI method |
| **Web Layer** | Presentation layer (Handles HTTP reqs & responses) | `@RequestMapping` – Base URL mapping<br>`@GetMapping` – Handle GET requests<br>`@PostMapping` – Handle POST requests<br>`@PutMapping` – Handle PUT requests<br>`@PatchMapping` – Handle PATCH requests<br>`@DeleteMapping` – Handle DELETE requests | **Controller methods** – Entry point for API calls |
| **Request Handling** | Read incoming request data | `@PathVariable` – Read value from URL path<br>`@RequestParam` – Read query parameters (supports `required=true/false`)<br>`@RequestBody` – Convert JSON to Java object | **Automatic data binding** by Spring |
| **Response Handling** | Send response to client | `@ResponseBody` – Convert Java object to JSON<br>`ResponseEntity<T>` – Control response body and status | `ok()` – 200 OK<br>`notFound()` – 404 NOT FOUND<br>`badRequest()` – 400 BAD REQUEST |
| **Service Layer** | Business logic processing | `@Service` – Marks business logic layer | **Service methods** – Implement core logic |
| **Persistence Layer** | Database interaction | `@Repository` – DAO layer<br>`JpaRepository` – Provides CRUD operations | N/A |
| **Entity Layer** | Map Java class to DB table | `@Entity` – Marks DB entity<br>`@Table` – Custom table name<br>`@Id` – Primary key<br>`@GeneratedValue` – PK generation strategy<br>`@Column` – Column specifics (name, nullable, length)<br>`@CreationStamp` / `@UpdationStamp` – Audit timestamps | **Entity getters/setters** – Access DB fields |
| **DTO Layer** | Data transfer between layers | **DTO (POJO)** – No Spring annotation<br>`@NotNull` – Must not be null<br>`@NotBlank` – Must not be empty<br>`@Email` – Email validation<br>`@Size` – Length constraint<br>`@JsonProperty` – JSON field mapping<br>`@JsonIgnore` – Exclude field from JSON | **Getter/setter** – Carry request/response data |
| **ModelMapper (Config)** | Conversion Between DTO and Entity | N/A | `.map()` – Basic mapping<br>`.getConfiguration().setSkipNullEnabled(true)` – Prevent null overwrites<br>`.typeMap(Source.class, Dest.class)` – Define custom config<br>`.addMapping(Source::getX, Dest::setY)` – Map fields with different names |
| **Object Mapping** | Convert Entity ↔ DTO | `ModelMapper` – Automatic object mapping | `map(source, target)` – Convert objects |
| **JSON Serialization** | Control JSON structure | `@JsonProperty` – Custom JSON field name<br>`@JsonIgnore` – Exclude field from JSON<br>`@JsonFormat` – Format dates | **Jackson** auto serialization/deserialization |
| **Validation** | Validate input data | `@Valid` – Enable validation<br>`@NotNull`, `@Email`, `@Size` – Validation rules | **Validation** before controller execution |
| **Optional Handling** | Handle null safely | `Optional<T>` – Avoid null values | `map()` – Transform value<br>`orElse()` – Default value<br>`orElseThrow()` – Throw exception |
| **Exception Handling** | Error handling mechanism | `@ExceptionHandler` – Local exception handler<br>`@ControllerAdvice` – Global exception handler | **Custom error responses** |
| **PATCH Handling** | Partial update logic | No annotation (logic based) | `ReflectionUtils.findField()` – Find field<br>`setAccessible(true)` – Access private field<br>`ReflectionUtils.setField()` – Update value |
---

# 🔄 Object Mapping

| Tool | Purpose |
|------|----------|
| ModelMapper | Automatic object mapping |
| map(source, target) | Convert objects |

---

# 🧾 JSON Serialization

| Annotation | Purpose |
|------------|----------|
| `@JsonProperty` | Custom JSON field name |
| `@JsonIgnore` | Exclude field |
| `@JsonFormat` | Format date |

---


# 🧾 Bean Validation

| Annotation | Purpose | Parameters |
|------------|----------|------------|
| `@NotNull` | Not null | message |
| `@NotEmpty` | Not null & size > 0 | message |
| `@NotBlank` | Not blank string | message |
| `@Size` | Length range | min, max |
| `@Min` | ≥ value | value |
| `@Max` | ≤ value | value |
| `@Email` | Valid email | regexp |
| `@Pattern` | Regex match | regexp |
| `@Positive` | > 0 | — |
| `@PositiveOrZero` | ≥ 0 | — |
| `@Negative` | < 0 | — |
| `@NegativeOrZero` | ≤ 0 | — |
| `@Past` | Past date | — |
| `@PastOrPresent` | Past/present | — |
| `@Future` | Future date | — |
| `@FutureOrPresent` | Future/present | — |
| `@Digits` | Integer & fraction control | integer, fraction |
| `@DecimalMin` | ≥ decimal | value, inclusive |
| `@DecimalMax` | ≤ decimal | value, inclusive |
| `@AssertTrue` | Must be true | — |
| `@AssertFalse` | Must be false | — |
| `@Valid` | Enable nested validation | — |

---

# ⚠️ Exception Handling

| Annotation / Function | Use |
|-----------------------|-----|
| `@RestControllerAdvice` | Global REST exception handler (JSON response) |
| `@ControllerAdvice` | Global MVC exception handler |
| `@ExceptionHandler` | Handle specific exception |
| `@Data` | Lombok shortcut for getters/setters |
| `@Builder` | Builder pattern |

---

# 🏷️ @Table Deep Structure

| Property | Purpose |
|----------|----------|
| name() | Table name |
| schema() | Schema |
| catalog() | Database |
| uniqueConstraints() | Composite unique |
| indexes() | Add DB index |

Structure:

@Table  
├─ name()  
├─ schema()  
├─ catalog()  
├─ uniqueConstraints()  
│    └─ @UniqueConstraint  
│         ├─ name()  
│         └─ columnNames()  
└─ indexes()  
     └─ @Index  
          ├─ name()  
          ├─ columnList()  
          └─ unique()  

---

# 🚀 Spring Data JPA – Query Method Keywords

| Category | Keyword | Meaning | Example |
|----------|----------|----------|----------|
| Basic | findBy | Select query | findByTitle(String title) |
| AND/OR | And | AND condition | findByTitleAndPrice(...) |
| AND/OR | Or | OR condition | findByTitleOrSku(...) |
| Comparison | GreaterThan | > | findByPriceGreaterThan(...) |
| Comparison | LessThan | < | findByQuantityLessThan(...) |
| Null | IsNull | IS NULL | findBySkuIsNull() |
| String | Containing | LIKE %value% | findByTitleContaining("Parle") |
| String | StartingWith | value% | findByTitleStartingWith("Par") |
| String | EndingWith | %value | findByTitleEndingWith("G") |
| Collection | In | IN clause | findBySkuIn(List<String>) |
| Between | Between | BETWEEN | findByPriceBetween(...) |
| Order | OrderBy | ORDER BY | findByOrderByPriceDesc() |
| Count | countBy | Return count | countByActiveTrue() |
| Exists | existsBy | Return boolean | existsBySku(...) |
| Delete | deleteBy | Delete rows | deleteBySku(...) |
| Limit | Top / First | Limit results | findTop3ByOrderByPriceDesc() |
| JPQL | @Query | Custom JPQL | @Query("select p from Product p") |
| Native | nativeQuery=true | Raw SQL | @Query(value="select * from product", nativeQuery=true) |
| Pagination | Pageable | Pagination | findAll(Pageable p) |
| Sorting | Sort | Dynamic sorting | findAll(Sort.by("price")) |
| Return Type | Optional<T> | Single safe result | Optional<Product> |
| Return Type | List<T> | Multiple rows | List<Product> |
| Return Type | Page<T> | Paginated | Page<Product> |
| Projection | Interface / DTO | Partial columns | List<ProductView> |

# 🚀 Spring Boot JPA Auditing

| Name                                             | Type       | Package                                          | Used On                        | Purpose                            | Why Important                             |
| ------------------------------------------------ | ---------- | ------------------------------------------------ | ------------------------------ | ---------------------------------- | ----------------------------------------- |
| `@EnableJpaAuditing`                             | Annotation | `org.springframework.data.jpa.repository.config` | Main / Config class            | Enables JPA Auditing               | Without this auditing will not activate   |
| `@EntityListeners(AuditingEntityListener.class)` | Annotation | `jakarta.persistence`                            | Entity class                   | Registers auditing listener        | Connects entity with auditing system      |
| `AuditingEntityListener`                         | Class      | `org.springframework.data.jpa.domain.support`    | Used inside `@EntityListeners` | Listens to entity lifecycle events | Automatically sets audit fields           |
| `@CreatedDate`                                   | Annotation | `org.springframework.data.annotation`            | Entity field                   | Stores creation timestamp          | Tracks when entity was created            |
| `@LastModifiedDate`                              | Annotation | `org.springframework.data.annotation`            | Entity field                   | Stores last update timestamp       | Tracks when entity was updated            |
| `@CreatedBy`                                     | Annotation | `org.springframework.data.annotation`            | Entity field                   | Stores creator user                | Tracks who created entity                 |
| `@LastModifiedBy`                                | Annotation | `org.springframework.data.annotation`            | Entity field                   | Stores last modifier user          | Tracks who updated entity                 |
| `AuditorAware<T>`                                | Interface  | `org.springframework.data.domain`                | Custom class                   | Provides current logged-in user    | Required for @CreatedBy & @LastModifiedBy |
| `Optional<T> getCurrentAuditor()`                | Method     | `AuditorAware`                                   | Implemented method             | Returns current user               | Supplies user info to auditing system     |
| `SecurityContextHolder`                          | Class      | `org.springframework.security.core.context`      | Security integration           | Gets authentication object         | Used in real projects for logged-in user  |
| `Authentication`                                 | Interface  | `org.springframework.security.core`              | Security integration           | Represents logged-in user          | Used to extract username                  |
| `getName()`                                      | Method     | `Authentication`                                 | Security integration           | Returns username                   | Used inside AuditorAware                  |
| `@PrePersist`                                    | Annotation | `jakarta.persistence`                            | Lifecycle callback             | Triggered before insert            | Internally used by auditing               |
| `@PreUpdate`                                     | Annotation | `jakarta.persistence`                            | Lifecycle callback             | Triggered before update            | Internally used by auditing               |
| `LocalDateTime`                                  | Class      | `java.time`                                      | Entity field                   | Date-time storage                  | Recommended type for timestamps           |
| `Instant`                                        | Class      | `java.time`                                      | Entity field                   | UTC timestamp storage              | Good for distributed systems              |
| `@MappedSuperclass`                              | Annotation | `jakarta.persistence`                            | Base entity class              | Shares auditing fields             | Used for reusable audit base class        |
| `@Embeddable`                                    | Annotation | `jakarta.persistence`                            | Audit metadata class           | Embeds audit object                | Clean design for large systems            |
| `@Embedded`                                      | Annotation | `jakarta.persistence`                            | Entity field                   | Embeds audit class                 | Used with @Embeddable                     |
| `auditorAwareRef`                                | Attribute  | `@EnableJpaAuditing`                             | Config                         | Links custom AuditorAware bean     | Needed when multiple beans exist          |



# 🚀 SPRING TESTING 

| Category       | Annotation / Method      | Meaning                                                                       | Example                                                                       |
| -------------- | ------------------------ | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| Core Test      | `@Test`                  | Marks a method as a JUnit test that should be executed by the test runner.    | `@Test void saveUser(){}` → Executes this method as a test.                   |
| Display        | `@DisplayName`           | Gives a custom, human-readable name to a test or test class in reports.       | `@DisplayName("Login Test")` → Shows "Login Test" instead of method name.     |
| Disable        | `@Disabled`              | Skips execution of the test until it is enabled again.                        | `@Disabled("Pending")` → Test is ignored with the given reason.               |
| Repeated       | `@RepeatedTest`          | Executes the same test multiple times automatically.                          | `@RepeatedTest(5)` → Runs the test 5 consecutive times.                       |
| Parameterized  | `@ParameterizedTest`     | Runs the same test with multiple input values.                                | `@ParameterizedTest` → Used with `@ValueSource`, `@CsvSource`, etc.           |
| Parameterized  | `@ValueSource`           | Supplies primitive or simple values to a parameterized test.                  | `@ValueSource(strings={"A","B"})` → Runs once with `"A"` and once with `"B"`. |
| Parameterized  | `@CsvSource`             | Supplies multiple comma-separated values for each test execution.             | `@CsvSource({"1,2","3,4"})` → Executes test with `(1,2)` and `(3,4)`.         |
| Parameterized  | `@MethodSource`          | Uses data returned from a method as test inputs.                              | `@MethodSource("data")` → Reads arguments from the `data()` method.           |
| Lifecycle      | `@BeforeEach`            | Executes before every individual test method.                                 | `@BeforeEach void init(){}` → Initializes objects before each test.           |
| Lifecycle      | `@AfterEach`             | Executes after every individual test method finishes.                         | `@AfterEach void clean(){}` → Cleans resources after each test.               |
| Lifecycle      | `@BeforeAll`             | Executes only once before all tests in the class.                             | `@BeforeAll static void setup(){}` → Creates shared test resources.           |
| Lifecycle      | `@AfterAll`              | Executes only once after all tests have completed.                            | `@AfterAll static void end(){}` → Releases shared resources.                  |
| Assertions     | `assertEquals()`         | Verifies that expected and actual values are equal.                           | `assertEquals(2,a+b)` → Passes only if `a+b` equals `2`.                      |
| Assertions     | `assertNotEquals()`      | Verifies that two values are different.                                       | `assertNotEquals(1,id)` → Passes if `id` is not `1`.                          |
| Assertions     | `assertTrue()`           | Verifies that a condition evaluates to true.                                  | `assertTrue(flag)` → Passes when `flag` is `true`.                            |
| Assertions     | `assertFalse()`          | Verifies that a condition evaluates to false.                                 | `assertFalse(flag)` → Passes when `flag` is `false`.                          |
| Assertions     | `assertNull()`           | Verifies that an object reference is null.                                    | `assertNull(user)` → Passes if `user` is `null`.                              |
| Assertions     | `assertNotNull()`        | Verifies that an object reference is not null.                                | `assertNotNull(user)` → Passes if `user` exists.                              |
| Assertions     | `assertThrows()`         | Verifies that the specified exception (or subclass) is thrown.                | `assertThrows(Exception.class,()->{})` → Passes if exception occurs.          |
| Assertions     | `assertDoesNotThrow()`   | Verifies that code executes without throwing any exception.                   | `assertDoesNotThrow(()->{})` → Passes if no exception is thrown.              |
| Assertions     | `assertAll()`            | Groups multiple assertions and executes all of them.                          | `assertAll(...);` → Reports all failed assertions together.                   |
| Assertions     | `fail()`                 | Immediately marks the test as failed.                                         | `fail("Error")` → Stops test execution with the message.                      |
| Spring Boot    | `@SpringBootTest`        | Loads the complete Spring Boot application context for integration testing.   | `@SpringBootTest` → Tests services, repositories, controllers together.       |
| MVC Test       | `@WebMvcTest`            | Loads only MVC components like controllers for web layer testing.             | `@WebMvcTest(UserController.class)` → Tests only `UserController`.            |
| JPA Test       | `@DataJpaTest`           | Loads only JPA components such as repositories and entities.                  | `@DataJpaTest` → Tests database queries without full application.             |
| JDBC Test      | `@JdbcTest`              | Loads JDBC-related beans for testing raw SQL operations.                      | `@JdbcTest` → Tests DAO using `JdbcTemplate`.                                 |
| JSON Test      | `@JsonTest`              | Tests JSON serialization and deserialization.                                 | `@JsonTest` → Verifies Jackson object mapping.                                |
| REST Client    | `@RestClientTest`        | Tests REST client components like `RestTemplate` or `WebClient`.              | `@RestClientTest` → Mocks external REST services.                             |
| Mocking        | `@Mock`                  | Creates a Mockito mock object without Spring context.                         | `@Mock UserRepo repo;` → Fake repository for unit testing.                    |
| Mocking        | `@InjectMocks`           | Injects created mocks into the object under test.                             | `@InjectMocks UserService service;` → Injects mocked repository into service. |
| Mocking        | `@MockBean`              | Replaces a Spring bean with a Mockito mock in the application context.        | `@MockBean UserRepo repo;` → Spring injects mock repository.                  |
| Mocking        | `@Spy`                   | Creates a partial mock that calls real methods unless stubbed.                | `@Spy List<String> list;` → Real list with selective mocking.                 |
| Mockito        | `when()`                 | Defines behavior for a mocked method call.                                    | `when(repo.findById(1L))` → Specifies expected behavior.                      |
| Mockito        | `thenReturn()`           | Specifies the value that the mocked method should return.                     | `thenReturn(user)` → Returns `user` instead of real DB result.                |
| Mockito        | `verify()`               | Verifies that a mocked method was invoked.                                    | `verify(repo).save(user)` → Ensures `save()` was called.                      |
| Mockito        | `times()`                | Specifies how many times a method should have been called.                    | `verify(repo,times(2))` → Expects exactly two calls.                          |
| Mockito        | `never()`                | Verifies that a method was never invoked.                                     | `verify(repo,never())` → Ensures method wasn't called.                        |
| Mockito        | `any()`                  | Matches any argument of the specified type.                                   | `any(User.class)` → Accepts any `User` object.                                |
| Mockito        | `eq()`                   | Matches an exact argument value.                                              | `eq(1L)` → Matches only value `1L`.                                           |
| Transaction    | `@Transactional`         | Runs each test inside a transaction that rolls back after completion.         | `@Transactional` → Database changes are automatically reverted.               |
| SQL            | `@Sql`                   | Executes SQL scripts before or after a test.                                  | `@Sql("/data.sql")` → Loads test data into database.                          |
| Profiles       | `@ActiveProfiles`        | Activates a specific Spring profile during testing.                           | `@ActiveProfiles("test")` → Uses `application-test.properties`.               |
| Config         | `@TestConfiguration`     | Defines configuration beans used only during testing.                         | `@TestConfiguration` → Creates test-specific beans.                           |
| Properties     | `@TestPropertySource`    | Overrides application properties for a test class.                            | `@TestPropertySource(...)` → Uses custom property values.                     |
| Context        | `@DirtiesContext`        | Marks the Spring context as dirty and reloads it after the test.              | `@DirtiesContext` → Creates a fresh application context.                      |
| Auto Configure | `@AutoConfigureMockMvc`  | Automatically configures `MockMvc` for controller testing.                    | `@AutoConfigureMockMvc` → Enables HTTP request simulation.                    |
| Web Testing    | `MockMvc`                | Simulates HTTP requests without starting a real server.                       | `mockMvc.perform(...)` → Tests REST endpoints.                                |
| Web Testing    | `perform()`              | Sends a simulated HTTP request using `MockMvc`.                               | `perform(get("/users"))` → Executes a GET request.                            |
| Web Testing    | `andExpect()`            | Verifies the response returned by the request.                                | `andExpect(status().isOk())` → Expects HTTP 200.                              |
| Web Testing    | `status()`               | Checks the HTTP response status code.                                         | `status().isCreated()` → Verifies HTTP 201 response.                          |
| Web Testing    | `jsonPath()`             | Verifies JSON response fields using JSONPath expressions.                     | `jsonPath("$.name")` → Checks the `name` property.                            |
| HTTP           | `get()`                  | Creates an HTTP GET request.                                                  | `get("/users")` → Requests user list.                                         |
| HTTP           | `post()`                 | Creates an HTTP POST request.                                                 | `post("/users")` → Creates a new user.                                        |
| HTTP           | `put()`                  | Creates an HTTP PUT request.                                                  | `put("/users/1")` → Updates user with ID 1.                                   |
| HTTP           | `delete()`               | Creates an HTTP DELETE request.                                               | `delete("/users/1")` → Deletes user with ID 1.                                |
| Security Test  | `@WithMockUser`          | Simulates an authenticated user during testing.                               | `@WithMockUser` → Bypasses actual login.                                      |
| Security Test  | `@WithAnonymousUser`     | Simulates an anonymous (not logged-in) user.                                  | `@WithAnonymousUser` → Tests public endpoint access.                          |
| Security Test  | `csrf()`                 | Adds a valid CSRF token to HTTP requests.                                     | `.with(csrf())` → Prevents CSRF validation failure.                           |
| Exception Test | `assertThrowsExactly()`  | Verifies that the exact exception type is thrown.                             | `assertThrowsExactly(...)` → Subclasses are not accepted.                     |
| Async Test     | `@Async` + test          | Tests asynchronous methods by waiting for completion.                         | `future.get()` → Waits for async task result.                                 |
| Timeout        | `assertTimeout()`        | Verifies that code completes within a specified time.                         | `assertTimeout(...)` → Fails if execution is too slow.                        |
| Nested Tests   | `@Nested`                | Groups related test classes inside one outer test class.                      | `class LoginTests{}` → Organizes login-related tests.                         |
| Order          | `@TestMethodOrder`       | Specifies the order in which test methods run.                                | `@TestMethodOrder(...)` → Executes tests in defined sequence.                 |
| Order          | `@Order`                 | Assigns execution order to an individual test.                                | `@Order(1)` → Runs before higher-order tests.                                 |
| Conditional    | `@EnabledOnOs`           | Runs the test only on specified operating systems.                            | `@EnabledOnOs(OS.WINDOWS)` → Executes only on Windows.                        |
| Conditional    | `@EnabledOnJre`          | Runs the test only on specified Java versions.                                | `@EnabledOnJre(JRE.JAVA_21)` → Executes only on Java 21.                      |
| Repository     | `save()`                 | Saves a new entity or updates an existing one.                                | `repo.save(user)` → Inserts or updates a database row.                        |
| Repository     | `findById()`             | Retrieves an entity by its primary key.                                       | `repo.findById(1L)` → Returns user with ID `1`.                               |
| Repository     | `findAll()`              | Retrieves all entities from the database table.                               | `repo.findAll()` → Returns every record.                                      |
| Repository     | `deleteById()`           | Deletes an entity using its primary key.                                      | `repo.deleteById(1L)` → Removes row with ID `1`.                              |
| Repository     | `existsById()`           | Checks whether an entity exists for the given ID.                             | `repo.existsById(1L)` → Returns `true` or `false`.                            |
| Repository     | `count()`                | Returns the total number of records in the table.                             | `repo.count()` → Counts all database rows.                                    |
| REST Template  | `TestRestTemplate`       | Sends real HTTP requests for integration testing.                             | `restTemplate.getForObject()` → Calls running application endpoints.          |
| Environment    | `@DynamicPropertySource` | Registers dynamic property values during test execution.                      | `registry.add(...)` → Sets container database URL.                            |
| Containers     | `@Testcontainers`        | Enables Testcontainers support for integration tests.                         | `@Testcontainers` → Starts Docker containers automatically.                   |
| Containers     | `@Container`             | Declares a Docker container managed by Testcontainers.                        | `@Container PostgreSQLContainer<?>` → Starts PostgreSQL for tests.            |
| Bean Loading   | `@ContextConfiguration`  | Specifies custom configuration classes or XML for loading the Spring context. | `@ContextConfiguration(classes=...)` → Loads selected configuration only.     |
| Bean Loading   | `@Import`                | Imports additional configuration or bean classes into the test context.       | `@Import(SecurityConfig.class)` → Includes `SecurityConfig` in the test.      |



## Assertion Methods : JUnit vs AssertJ

| Use                  | JUnit                          | AssertJ                | JUnit Example                                    | AssertJ Example                                                     |
| -------------------- | ------------------------------ | ---------------------- | ------------------------------------------------ | ------------------------------------------------------------------- |
| Equality Check       | `assertEquals()`               | `isEqualTo()`          | `assertEquals(10, marks);`                       | `assertThat(marks).isEqualTo(10);`                                  |
| Not Equal            | `assertNotEquals()`            | `isNotEqualTo()`       | `assertNotEquals(5, num);`                       | `assertThat(num).isNotEqualTo(5);`                                  |
| True Check           | `assertTrue()`                 | `isTrue()`             | `assertTrue(flag);`                              | `assertThat(flag).isTrue();`                                        |
| False Check          | `assertFalse()`                | `isFalse()`            | `assertFalse(flag);`                             | `assertThat(flag).isFalse();`                                       |
| Null Check           | `assertNull()`                 | `isNull()`             | `assertNull(obj);`                               | `assertThat(obj).isNull();`                                         |
| Not Null Check       | `assertNotNull()`              | `isNotNull()`          | `assertNotNull(obj);`                            | `assertThat(obj).isNotNull();`                                      |
| Same Object          | `assertSame()`                 | `isSameAs()`           | `assertSame(a, b);`                              | `assertThat(a).isSameAs(b);`                                        |
| Different Object     | `assertNotSame()`              | `isNotSameAs()`        | `assertNotSame(a, b);`                           | `assertThat(a).isNotSameAs(b);`                                     |
| Array Equality       | `assertArrayEquals()`          | `containsExactly()`    | `assertArrayEquals(arr1, arr2);`                 | `assertThat(arr1).containsExactly(arr2);`                           |
| List Size            | `assertEquals(list.size())`    | `hasSize()`            | `assertEquals(3, list.size());`                  | `assertThat(list).hasSize(3);`                                      |
| Empty Check          | `assertTrue(list.isEmpty())`   | `isEmpty()`            | `assertTrue(list.isEmpty());`                    | `assertThat(list).isEmpty();`                                       |
| Not Empty            | `assertFalse(list.isEmpty())`  | `isNotEmpty()`         | `assertFalse(list.isEmpty());`                   | `assertThat(list).isNotEmpty();`                                    |
| Contains Item        | `assertTrue(list.contains())`  | `contains()`           | `assertTrue(list.contains("Java"));`             | `assertThat(list).contains("Java");`                                |
| Does Not Contain     | `assertFalse(list.contains())` | `doesNotContain()`     | `assertFalse(list.contains("Python"));`          | `assertThat(list).doesNotContain("Python");`                        |
| Starts With          | `assertTrue(str.startsWith())` | `startsWith()`         | `assertTrue(name.startsWith("Su"));`             | `assertThat(name).startsWith("Su");`                                |
| Ends With            | `assertTrue(str.endsWith())`   | `endsWith()`           | `assertTrue(name.endsWith("ar"));`               | `assertThat(name).endsWith("ar");`                                  |
| String Contains      | `assertTrue(str.contains())`   | `contains()`           | `assertTrue(name.contains("jee"));`              | `assertThat(name).contains("jee");`                                 |
| Greater Than         | `assertTrue(a>b)`              | `isGreaterThan()`      | `assertTrue(age > 18);`                          | `assertThat(age).isGreaterThan(18);`                                |
| Less Than            | `assertTrue(a<b)`              | `isLessThan()`         | `assertTrue(price < 100);`                       | `assertThat(price).isLessThan(100);`                                |
| Exception Testing    | `assertThrows()`               | `assertThatThrownBy()` | `assertThrows(Exception.class, () -> divide());` | `assertThatThrownBy(() -> divide()).isInstanceOf(Exception.class);` |
| Floating Point Check | `assertEquals(a,b,delta)`      | `isCloseTo()`          | `assertEquals(10.2, val, 0.5);`                  | `assertThat(val).isCloseTo(10.2, within(0.5));`                     |

## Simple Understanding

* JUnit provides traditional assertion methods.
* AssertJ provides fluent readable assertions using `assertThat()` chaining.

Example:

### JUnit

```java id="g1ks82"
assertTrue(name.startsWith("Su"));
```

### AssertJ

```java id="y6nqa1"
assertThat(name).startsWith("Su");
```
