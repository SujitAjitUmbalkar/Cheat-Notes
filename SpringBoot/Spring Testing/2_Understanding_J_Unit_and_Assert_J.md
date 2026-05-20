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
