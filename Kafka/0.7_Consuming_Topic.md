# Concept 7 – Consuming Topics

According to the PDF, this concept contains four important points: 

1. Consumers read messages from partitions.
2. Consumers track the last consumed offset.
3. Kafka does not delete consumed messages.
4. Consumers can reprocess old messages.

Let's complete all four.

---

# 1. Consumer reads from Partitions

Consumer subscribes to a Topic.

Example:

```text
orders Topic

├── Partition 0
├── Partition 1
└── Partition 2
```

Kafka assigns one or more Partitions.

Example:

```text
Inventory Consumer

↓

Partition 0

Partition 2
```

Consumer reads events sequentially.

```text
Offset 0

↓

Offset 1

↓

Offset 2

↓

Offset 3
```

Consumer **does not search** for Rahul, Amit or Neha.

It simply processes every event it receives.

---

# 2. Consumer tracks the Last Offset

Suppose

```text
Partition 0

Offset 0

Offset 1

Offset 2

Offset 3

Offset 4
```

Consumer processes

```text
✓ Offset 0

✓ Offset 1

✓ Offset 2
```

Kafka remembers

```text
Last Consumed Offset = 2
```

If the application crashes,

after restarting,

Consumer continues from

```text
Offset 3
```

instead of Offset 0.

---

# 3. Kafka does NOT delete consumed messages

This is one of Kafka's biggest features.

Suppose Consumer reads

```text
Offset 0

Offset 1

Offset 2
```

Question:

Does Kafka delete these events?

**No.**

Partition still contains

```text
Offset 0

Offset 1

Offset 2

Offset 3

Offset 4
```

Nothing is removed simply because a consumer has read it. 

---

# Why?

Imagine another service starts tomorrow.

Example:

```text
Analytics Service
```

It also wants all previous order events.

If Kafka had deleted them,

Analytics Service could never process old orders.

So Kafka keeps the events.

---

# 4. Retention Policy

If Kafka never deletes messages,

won't the disk become full?

Yes.

So Kafka uses a **Retention Policy**.

Example:

```text
Keep messages for

7 Days

OR

100 GB
```

After the retention limit is reached,

Kafka removes the **oldest messages**. 

So deletion depends on **retention**, **not on consumption**.

---

# 5. Reprocessing Messages

Suppose

```text
Partition

Offset 0

Offset 1

Offset 2

Offset 3
```

Yesterday,

Inventory Service processed all of them.

Today,

a bug is fixed.

Now you want Inventory Service to process all events again.

Can Kafka do that?

**Yes.**

Consumer can start reading again from

```text
Offset 0
```

or any older offset.

This is called **Reprocessing**. 

---

# Complete Flow

```text
Producer
     │
Publishes Event
     │
     ▼
Topic
     │
     ▼
Partition
     │
Offset 0
Offset 1
Offset 2
Offset 3
     │
     ▼
Consumer
Reads Sequentially
Tracks Last Offset
Processes Event
```

Kafka **keeps** the events after they are consumed.

Only the retention policy eventually removes old data.

---

# Doubts Solved

### Q1. Does Consumer search for Rahul's event?

❌ No.

Consumer reads events sequentially from its assigned Partition.

---

### Q2. Does Kafka delete an event after it is consumed?

❌ No.

Events remain until the retention policy removes them.

---

### Q3. Can another Consumer read the same old events?

✅ Yes.

As long as the events are still retained.

---

### Q4. Can a Consumer read from the beginning again?

✅ Yes.

It can start from Offset 0 or any other offset.

---

### Q5. Why does Kafka keep consumed messages?

To allow:

* New consumers to process historical data.
* Reprocessing after fixing bugs.
* Recovery after failures.

---

# Remember

* Consumer subscribes to a **Topic**.
* Kafka assigns **Partitions**.
* Consumer reads events **sequentially**.
* Consumer tracks the **last consumed Offset**.
* Kafka **does not delete** consumed messages.
* Messages are deleted only by the **Retention Policy**.
* Consumers can **reprocess** old events by reading from an earlier Offset.

---

✅ **We have now completed the "Consuming Topics" section from your PDF.** 

The next section after the introductory concepts is **Consumer Groups** (which the PDF mentions in its architecture summary but does not explain in detail). That will be our next major concept.
