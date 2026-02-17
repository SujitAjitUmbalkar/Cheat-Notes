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
