# **ScheduledExecutorService – Notes**

### **1. What is ScheduledExecutorService?**

`ScheduledExecutorService` is an interface that extends `ExecutorService`.

It is used to execute tasks:

- Once after a specified delay.
- Repeatedly at fixed intervals.
- Repeatedly after a fixed delay following completion.

### **2. Creating ScheduledExecutorService**

We can create it using two factory methods:

**A. Multiple worker threads**

```
ScheduledExecutorService executor =
        Executors.newScheduledThreadPool(2);
```

**B. Single worker thread**

```
ScheduledExecutorService executor =
        Executors.newSingleThreadScheduledExecutor();
```

Both support scheduling tasks.

### **3. Why not use SingleThreadExecutor or FixedThreadPool?**

Because they are designed for normal task execution and don't provide scheduling methods.

| Executor                             | Purpose                                           |
| ------------------------------------ | ------------------------------------------------- |
| `newSingleThreadExecutor()`          | Normal tasks using one thread                     |
| `newFixedThreadPool(3)`              | Normal tasks using 3 threads                      |
| `newScheduledThreadPool(2)`          | Delayed and periodic tasks using multiple threads |
| `newSingleThreadScheduledExecutor()` | Delayed and periodic tasks using one thread       |

**Important:** If you need scheduling functionality, use `ScheduledExecutorService`.

### **4. Important methods**

| Method                     | Purpose                                                  |
| -------------------------- | -------------------------------------------------------- |
| `schedule()`               | Executes a task once after a specified delay.            |
| `scheduleAtFixedRate()`    | Executes repeatedly at a fixed rate.                     |
| `scheduleWithFixedDelay()` | Executes repeatedly with a fixed delay after completion. |
| `shutdown()`               | Initiates an orderly shutdown.                           |

### **5. Code Example (Without Lambda)**

```
import java.util.concurrent.*;

public class Main {

    public static void main(String[] args) {

        ScheduledExecutorService executor =
                Executors.newScheduledThreadPool(2);

        Runnable task = new Runnable() {
            @Override
            public void run() {
                System.out.println("Task executed: " +
                        Thread.currentThread().getName());
            }
        };

        // Execute once after 3 seconds
        executor.schedule(task, 3, TimeUnit.SECONDS);

        // Execute repeatedly every 5 seconds
        executor.scheduleAtFixedRate(
                task, 0, 5, TimeUnit.SECONDS
        );

        // Execute with 5 seconds delay after completion
        executor.scheduleWithFixedDelay(
                task, 0, 5, TimeUnit.SECONDS
        );

        // Shutdown when appropriate
        // executor.shutdown();
    }
}
```

*Note: This example demonstrates all three methods. Normally, choose the scheduling method you actually need.*

### **6. Difference between scheduleAtFixedRate() and scheduleWithFixedDelay()**

**A. scheduleAtFixedRate()**

- Next execution is based on the previous scheduled start time.
- Maintains a fixed execution rate.
- If the task takes longer than the period, the next execution starts as soon as possible after completion.

Example: Period = 5 seconds, task execution time = 2 seconds.

**Fixed Rate**

**Task 1**

0–2 sec

**Task 2**

5–7 sec

**Task 3**

10–12 sec

Scheduled start times: 0, 5, 10, 15...

**B. scheduleWithFixedDelay()**

- Next execution starts after the previous execution completes and the specified delay passes.
- Maintains a fixed waiting period between executions.

Example: Delay = 5 seconds, task execution time = 2 seconds.

**Fixed Delay**

**Task 1**

0–2 sec

**Task 2**

7–9 sec

**Task 3**

14–16 sec

Scheduled start times: 0, 7, 14, 21...

### **7. Quick Comparison**

| Fixed Rate                                   | Fixed Delay                          |
| -------------------------------------------- | ------------------------------------ |
| Fixed interval between scheduled start times | Fixed delay after task completion    |
| Based on previous scheduled time             | Based on previous completion time    |
| Useful for periodic scheduling               | Useful when tasks need a rest period |

### **8. Important points to remember**

1. `ScheduledExecutorService` extends `ExecutorService`.
2. `newScheduledThreadPool(2)` creates a scheduler with multiple worker threads.
3. `newSingleThreadScheduledExecutor()` creates a scheduler with one worker thread.
4. Successive executions of the same periodic task never overlap.
5. If a periodic task throws an exception, subsequent executions are suppressed.
6. Always shut down the executor when it is no longer needed.

**Easy trick:**

- Fixed Rate = Maintain a schedule.
- Fixed Delay = Wait after completing the task.
