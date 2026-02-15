# 🚀 Spring Boot Complete Notes (MVC + JPA + Validation + Exception Handling)

---


# 🏗️ MVC Architecture & Core Concepts

| Main Topic | Layer / Responsibility | Annotations Used (1-line purpose) | Functions / Methods Used |
|------------|------------------------|-----------------------------------|--------------------------|
| MVC Architecture | Overall application design | `@Controller` – Handles requests & returns views <br> `@RestController` – Handles REST & returns JSON | `model.addAttribute()` – Send data to view |
| Application Flow | End-to-end execution | MVC Flow | Request → Controller → Service → Repository → DB → Response |
| Dependency Injection | Inject required objects | `@Autowired` – Inject bean <br> `@Component` – Generic bean <br> `@Configuration` – Config class <br> `@Bean` – Manual bean creation | Constructor Injection – Preferred DI |

---

# 🌐 Web / Presentation Layer

| Annotation | Purpose |
|------------|----------|
| `@RequestMapping` | Base URL mapping |
| `@GetMapping` | Handle GET requests |
| `@PostMapping` | Handle POST requests |
| `@PutMapping` | Handle PUT requests |
| `@PatchMapping` | Handle PATCH requests |
| `@DeleteMapping` | Handle DELETE requests |

---

# 📥 Request Handling

| Annotation | Purpose |
|------------|----------|
| `@PathVariable` | Read value from URL path |
| `@RequestParam` | Read query parameter |
| `@RequestBody` | Convert JSON → Java Object |
| `required=true/false` | Optional query param |

---

# 📤 Response Handling

| Annotation / Class | Purpose |
|--------------------|----------|
| `@ResponseBody` | Convert Java → JSON |
| `ResponseEntity<T>` | Control body + status |
| `ok()` | 200 OK |
| `badRequest()` | 400 BAD REQUEST |
| `notFound()` | 404 NOT FOUND |

---

# 🧠 Service Layer

| Annotation | Purpose |
|------------|----------|
| `@Service` | Business logic layer |

---

# 💾 Persistence Layer

| Annotation / Interface | Purpose |
|------------------------|----------|
| `@Repository` | DAO layer |
| `JpaRepository` | Provides CRUD operations |

---

# 🗃️ Entity Layer

| Annotation | Purpose |
|------------|----------|
| `@Entity` | Marks DB entity |
| `@Table` | Custom table name |
| `@Id` | Primary key |
| `@GeneratedValue(strategy=IDENTITY/AUTO/SEQUENCE)` | Auto ID generation |
| `@Column(name="", nullable="", length=50)` | Customize column |
| `@CreationTimestamp` | Auto set creation time |
| `@UpdateTimestamp` | Auto set update time |

---

# 📦 DTO Layer

| Annotation | Purpose |
|------------|----------|
| `@NotNull` | Field must not be null |
| `@NotBlank` | Must not be empty |
| `@Email` | Email validation |
| `@Size` | Field length constraint |
| `@JsonProperty` | Rename JSON field |
| `@JsonIgnore` | Hide field from JSON |

---

# 🔁 ModelMapper

| Method | Purpose |
|--------|----------|
| `map()` | Convert DTO ↔ Entity |
| `setSkipNullEnabled(true)` | Prevent null overwrite |
| `typeMap()` | Custom mapping config |
| `addMapping()` | Map different field names |

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
