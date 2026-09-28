### Step 1 — Run Kafka, Kafbat & Configure JPA

1. **Run Kafka + Kafbat**

   * Start both using `docker-compose.yml`.
   * **Kafka** → message broker.
   * **Kafbat** → UI to monitor/manage Kafka.

2. **Configure Kafka connection**

   ```properties
   spring.kafka.bootstrap-servers=localhost:9092
   ```

   At this stage, **only configure the Kafka connection** — no producers, consumers, or topics yet.

3. **Configure JPA / Database**

   ```properties
   spring.datasource.url=jdbc:mysql://localhost:3306/kafka_db
   spring.datasource.username=root
   spring.datasource.password=****

   spring.jpa.hibernate.ddl-auto=update
   spring.jpa.show-sql=true
   ```

   * maintain different configuration files for Kafka configuration and JPA Configurations. 

**Logic:**

> Start Kafka & Kafbat → connect Spring Boot to Kafka → configure JPA/database.
> Producer, Consumer and Topic configuration will come in the later steps.

---

### Step 2 — Define Kafka Topics

1. **Define topic names in `application.yml`**

   ```yaml
   kafka:
     topics:
       user-created: user-created-topic
       user-random: user-random-topic
   ```

2. **Use these names in a Kafka configuration class**

   * Create `NewTopic` beans.
   * Define:

     * **Topic name**
     * **Number of partitions**
     * **Number of replicas**

   ```java
   @Bean
    public NewTopic userRandomTopic()
    {
        return new NewTopic(KAFKA_RANDOM_USER_TOPIC, 3, (short) 1);
    }
   ```

3. **Why `NewTopic`?**

   * Spring Boot uses these beans to **create the topics in Kafka** if they don't already exist.

**Logic:**

> `application.yml` → stores topic names
> → Config class reads those names
> → `NewTopic` beans define **name + partitions + replicas**
> → Kafka creates the topics.

---

### Step 3 — Create Event Class

1. **Create an `event` package**

   * Events represent the data/message that will be sent through Kafka.

2. **Create an Event class**

   * Use JSON as the event format.
   * Add `@Data`.
   * Add the required fields and an `id`.

   ```java
   @Data
   public class UserCreatedEvent {

       private Long id;
       private String name;
       private String email;
   }
   ```

3. **Use `id` as the Kafka key**

   * The `id` will be sent as the **Kafka message key**.
   * Kafka uses the key to determine the partition.
   * The same key is consistently mapped to the same partition (assuming the partition count remains unchanged).

**Logic:**

> Event class → Java object → serialized to JSON → sent to Kafka with `id` as key → Kafka determines the partition using the key.

---

### Step 4 — Configure Kafka Producer

1. **Configure Producer Serialization**

   * Configure serializers so Kafka knows how to convert Java objects into bytes before sending.
   * **Key serializer:** converts `Long` key → bytes.
   * **Value serializer:** converts `UserCreatedEvent` → JSON → bytes.

   ```yaml
   spring:
     kafka:
       producer:
         key-serializer: org.apache.kafka.common.serialization.LongSerializer
         value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
   ```

2. **Get Topic Name in Service**

   ```java
   @Value("${kafka.topics.user-created}")
   private String KAFKA_USER_CREATED_TOPIC;
   ```

3. **Inject `KafkaTemplate`**

   * Define the **key datatype** and **value datatype**:

   ```java
   private final KafkaTemplate<Long, UserCreatedEvent> kafkaTemplate;
   ```

   Here:

   * `Long` → message key (`id`)
   * `UserCreatedEvent` → message value

4. **Send the Event**

   ```java
   kafkaTemplate.send(
       KAFKA_USER_CREATED_TOPIC,
       userCreatedEvent.getId(),
       userCreatedEvent
   );
   
   ```
   ```
        public CreateUserRequestDto createUser(CreateUserRequestDto createUserRequestDto)
    {
        log.info("UserService:createUser");

        User user = modelMapper.map(createUserRequestDto, User.class);
        User savedUser = userRepository.save(user);

        UserCreatedEvent userCreatedEvent = modelMapper.map(savedUser, UserCreatedEvent.class);
        kafkaTemplate.send(KAFKA_USER_CREATED_TOPIC, userCreatedEvent.getId(), userCreatedEvent);

        return modelMapper.map(savedUser, CreateUserRequestDto.class);
    }

   ```

**Logic:**

> Java Event → `JsonSerializer` converts it to JSON → Kafka receives **key + value** → key determines the partition → event is stored in that partition.
---

### Step 5 — Verify the Complete Flow

Now verify the application **from startup → Kafka → Kafbat**.

1. **Start Kafka & Kafbat**

   ```bash
   docker compose up -d
   ```

   * Kafka broker must be running.
   * Kafbat UI should be accessible.

2. **Start the Spring Boot application**

   * Spring Boot connects to Kafka using:

   ```yaml
   spring:
     kafka:
       bootstrap-servers: localhost:9092
   ```

   * JPA connects to the database.
   * `NewTopic` beans create the configured topics if they don't already exist.

3. **Verify the topic in Kafbat**

   * Open Kafbat.
   * Go to **Topics**.
   * Check that your topic exists:

   ```text
   user-created-topic
   ```

   * Verify its **partitions** and **replicas**.

4. **Trigger the Producer**

   * Call the API/service that creates the `UserCreatedEvent`.
   * Producer executes:

   ```java
   kafkaTemplate.send(
       KAFKA_USER_CREATED_TOPIC,
       userCreatedEvent.getId(),
       userCreatedEvent
   );
   ```

5. **Verify the message in Kafbat**

   * Open `user-created-topic`.
   * View the produced message.
   * Verify:

     ```text
     Key   → userCreatedEvent.getId()
     Value → UserCreatedEvent as JSON
     ```
   * Verify that the message is present in the expected partition.

**Complete flow:**

```text
Docker
  ↓
Kafka + Kafbat
  ↓
Spring Boot starts
  ↓
Kafka + JPA configuration loaded
  ↓
NewTopic creates/validates topic
  ↓
API triggers Producer
  ↓
KafkaTemplate
  ↓
Key (id) + Event (JSON)
  ↓
Kafka chooses partition
  ↓
Message stored in topic
  ↓
Kafbat → verify topic + partition + message
```

**At this point:** Kafka infrastructure, topic creation, event creation, and producer flow are verified.

---

### Step 6 — Configure Kafka Consumer

1. **Create a `consumer` package** in the consumer service.

   * This service will **receive and process events** produced by the producer service.

2. **Configure the Consumer**

   * Consumer needs to know:

     * Kafka broker address
     * Key deserializer
     * Value deserializer
     * Consumer group ID

   ```yaml
   spring:
     kafka:
       bootstrap-servers: localhost:9092

       consumer:
         group-id: notification-service
         key-deserializer: org.apache.kafka.common.serialization.LongDeserializer
         value-deserializer: org.springframework.kafka.support.serializer.JsonDeserializer
   ```

3. **Why deserializers?**

   * Kafka stores messages as **bytes**.
   * The consumer must convert those bytes back into Java objects.
   * `LongDeserializer` → bytes → `Long`
   * `JsonDeserializer` → JSON → `UserCreatedEvent`

4. **Consumer needs to know the Event structure**

   * The producer sends `UserCreatedEvent`.
   * Therefore, the consumer also needs the **same event class structure** to deserialize the JSON correctly.

5. **Copy the `event` package**

   * Copy the producer's `event` package into the consumer service.
   * Keep the Event class structure the same.

   ```text
   Producer Service
   └── event
       └── UserCreatedEvent.java

   Consumer Service
   └── event
       └── UserCreatedEvent.java
   ```

6. **Configure trusted package**

   * `JsonDeserializer` needs to know which Java packages are allowed to be deserialized.
   * Configure the event package as a trusted package:

   ```yaml
   spring:
     kafka:
       consumer:
         properties:
           spring.json.trusted.packages: "com.codingshuttle.learnKafka.user_service.event"
   ```

**Logic:**

> Producer sends `UserCreatedEvent` as JSON → Kafka stores bytes → Consumer receives bytes → `JsonDeserializer` converts JSON back to `UserCreatedEvent`.

---

### Step 7 — Create Kafka Consumers

Now the consumer service actually **listens to topics and processes the events**.

### 1. Create a Kafka Listener

Use `@KafkaListener` and specify the **exact topic name** and **consumer group**.

```java
@KafkaListener(
        topics = "user-created-topic",
        groupId = "notification-service"
)
public void consumeUserCreatedEvent(UserCreatedEvent event) {

    System.out.println("Received event: " + event);

    // Operate on the event
    // send notification, save data, trigger another service, etc.
}
```

**Logic:**

```text
Kafka Topic
     ↓
@KafkaListener
     ↓
Deserialize JSON
     ↓
UserCreatedEvent
     ↓
Business logic
```

---

### 2. Use `groupId` to define Consumer Groups

The **group ID determines which consumers work together**.

For example:

```java
@KafkaListener(
        topics = "user-created-topic",
        groupId = "notification-service"
)
public void notificationConsumer(UserCreatedEvent event) {
    // Send notification
}
```

Another service can consume the **same topic** with a different group:

```java
@KafkaListener(
        topics = "user-created-topic",
        groupId = "analytics-service"
)
public void analyticsConsumer(UserCreatedEvent event) {
    // Store/analyze user data
}
```

The same event can therefore be consumed by **both groups**:

```text
             user-created-topic
                     │
          ┌──────────┴──────────┐
          ↓                     ↓
notification-service      analytics-service
     group                    group
          ↓                     ↓
   Send notification      Analytics
```

---

### 3. Multiple Consumers in the Same Group

You can also have multiple consumers belonging to the **same group**:

```java
@KafkaListener(
        topics = "user-created-topic",
        groupId = "notification-service"
)
public void consumer1(UserCreatedEvent event) {
    // Process event
}
```

```java
@KafkaListener(
        topics = "user-created-topic",
        groupId = "notification-service"
)
public void consumer2(UserCreatedEvent event) {
    // Process event
}
```

Kafka automatically distributes the topic's **partitions** between consumers in the same group.

```text
user-created-topic
 ├── Partition 0 ──→ consumer1
 ├── Partition 1 ──→ consumer2
 ├── Partition 2 ──→ consumer1
```

> Within one consumer group, **a partition is consumed by only one consumer at a time**.

---

### 4. Operate on the Event

The listener receives the already-deserialized event:

```java
@KafkaListener(
        topics = "user-created-topic",
        groupId = "notification-service"
)
public void consume(UserCreatedEvent event) {

    Long userId = event.getId();
    String name = event.getName();
    String email = event.getEmail();

    System.out.println("User created: " + userId);

    // Business operation
    notificationService.sendWelcomeEmail(email);
}
```

### Final Logic

```text
Producer
   ↓
UserCreatedEvent
   ↓
Kafka Topic
   ↓
Partition
   ↓
Consumer Group
   ↓
@KafkaListener
   ↓
JSON → UserCreatedEvent
   ↓
Business Logic
```

**Remember the key rule:**

> **Same topic + same group → consumers share the work.**
> **Same topic + different groups → each group receives its own copy of the event.**
> **Yes. If groupId is defined directly in @KafkaListener, it overrides the default group-id from application.yml for that listener.**


---

### Step 8 — Verify the Consumer

Now verify the complete **Producer → Kafka → Consumer** flow.

1. **Start Kafka + Kafbat**

   ```bash
   docker compose up -d
   ```

2. **Start the Consumer Service**

   * Make sure it connects successfully to Kafka.
   * Check that there are no deserialization or trusted-package errors.

3. **Start the Producer Service**

   * Trigger the API that creates a `UserCreatedEvent`.

4. **Producer sends the event**

   ```java
   kafkaTemplate.send(
       "user-created-topic",
       userCreatedEvent.getId(),
       userCreatedEvent
   );
   ```

5. **Kafka stores the event**

   ```text
   Producer
      ↓
   user-created-topic
      ↓
   Partition
   ```

6. **Consumer receives the event**

   ```java
   @KafkaListener(
       topics = "user-created-topic",
       groupId = "notification-service"
   )
   public void consume(UserCreatedEvent event) {

       System.out.println("Received: " + event);
   }
   ```

7. **Verify in the consumer logs**

   You should see something similar to:

   ```text
   Received: UserCreatedEvent(
       id=101,
       name=Jeetu,
       email=jeetu@gmail.com
   )
   ```

8. **Verify in Kafbat**

   * Open `user-created-topic`.
   * Check that the message exists.
   * Verify its **key** and **JSON value**.
   * The consumer should be reading that same event.

### Complete verification flow

```text
Producer Service
      ↓
KafkaTemplate
      ↓
Kafka Topic
      ↓
Partition
      ↓
Consumer Group
      ↓
@KafkaListener
      ↓
JSON Deserialization
      ↓
UserCreatedEvent
      ↓
Business Logic
```

### What this confirms

If the event appears in the consumer logs, then these parts are working:

**Producer → Serialization → Kafka → Topic/Partition → Consumer Group → Deserialization → Consumer**.
