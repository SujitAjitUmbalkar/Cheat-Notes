# 🚀 Spring Boot Complete Notes (MVC + JPA + Validation + Exception Handling)

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



# 🧪 Spring Boot Testing – Annotations & Methods Cheat Sheet

| Category       | Annotation / Method      | Meaning                       | Example                                |
| -------------- | ------------------------ | ----------------------------- | -------------------------------------- |
| Core Test      | `@Test`                  | Marks test method             | `@Test void saveUser(){}`              |
| Display        | `@DisplayName`           | Custom test name              | `@DisplayName("Login Test")`           |
| Disable        | `@Disabled`              | Skip test                     | `@Disabled("Pending")`                 |
| Repeated       | `@RepeatedTest`          | Run multiple times            | `@RepeatedTest(5)`                     |
| Parameterized  | `@ParameterizedTest`     | Run with multiple inputs      | `@ParameterizedTest`                   |
| Parameterized  | `@ValueSource`           | Primitive input values        | `@ValueSource(strings={"A","B"})`      |
| Parameterized  | `@CsvSource`             | CSV inputs                    | `@CsvSource({"1,2","3,4"})`            |
| Parameterized  | `@MethodSource`          | Method-based data             | `@MethodSource("data")`                |
| Lifecycle      | `@BeforeEach`            | Runs before every test        | `@BeforeEach void init(){}`            |
| Lifecycle      | `@AfterEach`             | Runs after every test         | `@AfterEach void clean(){}`            |
| Lifecycle      | `@BeforeAll`             | Runs once before all tests    | `@BeforeAll static void setup(){}`     |
| Lifecycle      | `@AfterAll`              | Runs once after all tests     | `@AfterAll static void end(){}`        |
| Assertions     | `assertEquals()`         | Compare values                | `assertEquals(2,a+b)`                  |
| Assertions     | `assertNotEquals()`      | Values should differ          | `assertNotEquals(1,id)`                |
| Assertions     | `assertTrue()`           | Condition true                | `assertTrue(flag)`                     |
| Assertions     | `assertFalse()`          | Condition false               | `assertFalse(flag)`                    |
| Assertions     | `assertNull()`           | Object should be null         | `assertNull(user)`                     |
| Assertions     | `assertNotNull()`        | Object should exist           | `assertNotNull(user)`                  |
| Assertions     | `assertThrows()`         | Expect exception              | `assertThrows(Exception.class,()->{})` |
| Assertions     | `assertDoesNotThrow()`   | No exception expected         | `assertDoesNotThrow(()->{})`           |
| Assertions     | `assertAll()`            | Multiple assertions           | `assertAll(...);`                      |
| Assertions     | `fail()`                 | Force failure                 | `fail("Error")`                        |
| Spring Boot    | `@SpringBootTest`        | Load full application context | `@SpringBootTest`                      |
| MVC Test       | `@WebMvcTest`            | Test controller layer only    | `@WebMvcTest(UserController.class)`    |
| JPA Test       | `@DataJpaTest`           | Test repository layer         | `@DataJpaTest`                         |
| JDBC Test      | `@JdbcTest`              | Test JDBC components          | `@JdbcTest`                            |
| JSON Test      | `@JsonTest`              | Test JSON serialization       | `@JsonTest`                            |
| REST Client    | `@RestClientTest`        | Test REST clients             | `@RestClientTest`                      |
| Mocking        | `@Mock`                  | Create Mockito mock           | `@Mock UserRepo repo;`                 |
| Mocking        | `@InjectMocks`           | Inject mocks                  | `@InjectMocks UserService service;`    |
| Mocking        | `@MockBean`              | Mock Spring bean              | `@MockBean UserRepo repo;`             |
| Mocking        | `@Spy`                   | Partial mock                  | `@Spy List<String> list;`              |
| Mockito        | `when()`                 | Define behavior               | `when(repo.findById(1L))`              |
| Mockito        | `thenReturn()`           | Mock return value             | `thenReturn(user)`                     |
| Mockito        | `verify()`               | Verify method called          | `verify(repo).save(user)`              |
| Mockito        | `times()`                | Verify call count             | `verify(repo,times(2))`                |
| Mockito        | `never()`                | Verify never called           | `verify(repo,never())`                 |
| Mockito        | `any()`                  | Match any argument            | `any(User.class)`                      |
| Mockito        | `eq()`                   | Exact match                   | `eq(1L)`                               |
| Transaction    | `@Transactional`         | Rollback after test           | `@Transactional`                       |
| SQL            | `@Sql`                   | Run SQL scripts               | `@Sql("/data.sql")`                    |
| Profiles       | `@ActiveProfiles`        | Use test profile              | `@ActiveProfiles("test")`              |
| Config         | `@TestConfiguration`     | Test-only config              | `@TestConfiguration`                   |
| Properties     | `@TestPropertySource`    | Custom properties             | `@TestPropertySource(...)`             |
| Context        | `@DirtiesContext`        | Reload Spring context         | `@DirtiesContext`                      |
| Auto Configure | `@AutoConfigureMockMvc`  | Enable MockMvc                | `@AutoConfigureMockMvc`                |
| Web Testing    | `MockMvc`                | Test REST endpoints           | `mockMvc.perform(...)`                 |
| Web Testing    | `perform()`              | Send request                  | `perform(get("/users"))`               |
| Web Testing    | `andExpect()`            | Validate response             | `andExpect(status().isOk())`           |
| Web Testing    | `status()`               | HTTP status checks            | `status().isCreated()`                 |
| Web Testing    | `jsonPath()`             | Validate JSON                 | `jsonPath("$.name")`                   |
| HTTP           | `get()`                  | GET request                   | `get("/users")`                        |
| HTTP           | `post()`                 | POST request                  | `post("/users")`                       |
| HTTP           | `put()`                  | PUT request                   | `put("/users/1")`                      |
| HTTP           | `delete()`               | DELETE request                | `delete("/users/1")`                   |
| Security Test  | `@WithMockUser`          | Mock logged-in user           | `@WithMockUser`                        |
| Security Test  | `@WithAnonymousUser`     | Anonymous user                | `@WithAnonymousUser`                   |
| Security Test  | `csrf()`                 | Add CSRF token                | `.with(csrf())`                        |
| Exception Test | `assertThrowsExactly()`  | Exact exception type          | `assertThrowsExactly(...)`             |
| Async Test     | `@Async` + test          | Test async methods            | `future.get()`                         |
| Timeout        | `assertTimeout()`        | Max execution time            | `assertTimeout(...)`                   |
| Nested Tests   | `@Nested`                | Group related tests           | `class LoginTests{}`                   |
| Order          | `@TestMethodOrder`       | Order tests                   | `@TestMethodOrder(...)`                |
| Order          | `@Order`                 | Specify order                 | `@Order(1)`                            |
| Conditional    | `@EnabledOnOs`           | Run on specific OS            | `@EnabledOnOs(OS.WINDOWS)`             |
| Conditional    | `@EnabledOnJre`          | Run on JDK version            | `@EnabledOnJre(JRE.JAVA_21)`           |
| Repository     | `save()`                 | Save entity                   | `repo.save(user)`                      |
| Repository     | `findById()`             | Find by ID                    | `repo.findById(1L)`                    |
| Repository     | `findAll()`              | Get all rows                  | `repo.findAll()`                       |
| Repository     | `deleteById()`           | Delete row                    | `repo.deleteById(1L)`                  |
| Repository     | `existsById()`           | Check existence               | `repo.existsById(1L)`                  |
| Repository     | `count()`                | Count rows                    | `repo.count()`                         |
| REST Template  | `TestRestTemplate`       | Integration API testing       | `restTemplate.getForObject()`          |
| Environment    | `@DynamicPropertySource` | Dynamic properties            | `registry.add(...)`                    |
| Containers     | `@Testcontainers`        | Enable Testcontainers         | `@Testcontainers`                      |
| Containers     | `@Container`             | Define container              | `@Container PostgreSQLContainer<?>`    |
| Bean Loading   | `@ContextConfiguration`  | Custom context config         | `@ContextConfiguration(classes=...)`   |
| Bean Loading   | `@Import`                | Import config class           | `@Import(SecurityConfig.class)`        |
