# 🚀 Spring Boot Complete Notes (MVC + JPA + Validation + Exception Handling)

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

