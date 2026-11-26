
# Java Executor Framework – Complete Notes

## 1. Multithreading Basics

### What is a Thread?

- A **Thread** is an independent path of execution inside a program.
- Think of a thread as a worker.
- A **Task** is the actual work performed by that worker.
- Multithreading allows multiple tasks to make progress concurrently.

**Example:** An application can download a file while updating a progress bar.

### Why do we need Multithreading?

- To perform multiple tasks concurrently.
- To improve application responsiveness.
- To utilize CPU resources efficiently.
- To avoid making one task wait unnecessarily for another.

### Creating a Thread Manually

```
public class Main {
    public static void main(String[] args) {

        Thread t1 = new Thread(() -> {
            System.out.println("Worker is doing a task");
        });

        t1.start();
    }
}
```

**Important:**

- `start()` starts a new thread.
- `run()` directly calls the method on the current thread; it does not create a new thread.

### Why not create a new Thread for every Task?

Creating too many threads causes:

- More memory consumption.
- Increased CPU overhead.
- Expensive thread creation and management.
- Performance problems when the application handles many tasks.

**Solution: Thread Pool**

A Thread Pool maintains reusable worker threads.

Instead of creating a new thread for every task, existing workers execute multiple tasks one after another.

---

## 2. Java Executor Framework

### What is Executor Framework?

- Introduced in **JDK 5**.
- Available in the `java.util.concurrent` package.
- It separates task submission from task execution.
- Instead of manually creating and managing threads, we submit tasks to an Executor.

The Executor decides how the submitted tasks should be executed.

### Basic Execution Flow

1. Create a task using `Runnable` or `Callable`.
2. Submit the task to an Executor.
3. The Executor manages its execution.
4. A worker thread executes the task.
5. After completion, the worker can be reused for another task.

**Remember:**

- Task = What work needs to be done?
- Thread = Who performs the work?
- Executor = Who manages task execution?
- Thread Pool = Collection of reusable worker threads.

---

## 3. Executor Interface

### What is Executor?

- `Executor` is the basic interface of the Executor Framework.
- It provides a method called `execute()`.
- It is used to submit a `Runnable` task for execution.

```
void execute(Runnable command);
```

### Example

```
public class Main {
    public static void main(String[] args) {

        // 1. Create a task
        Runnable task = () -> {
            System.out.println("Hello from the task");
        };

        // 2. Create Executor implementation
        Executor executor = new Executor() {
            @Override
            public void execute(Runnable command) {
                Thread thread = new Thread(command);
                thread.start();
            }
        };

        // 3. Submit task
        executor.execute(task);
    }
}
```

### Understanding the Code

1. We create a `Runnable` task containing the actual work.
2. We implement the `Executor` interface.
3. Inside `execute()`, we create and start a new thread.
4. Finally, we submit the task using `executor.execute(task)`.

**Important:** This is a custom Executor implementation. Other implementations can use thread pools or even execute tasks in the calling thread.

### What is Runnable?

- `Runnable` represents a task that needs to be executed.
- It contains the `run()` method.
- It does not return a result through its interface.

---

## 4. ExecutorService Interface

### What is ExecutorService?

- `ExecutorService` extends the `Executor` interface.
- It provides additional methods for task submission, result retrieval, cancellation, and lifecycle management.
- It is commonly used when working with thread pools.

### Important Methods

| Method               | Purpose                                                                   |
| -------------------- | ------------------------------------------------------------------------- |
| `execute()`          | Submit a Runnable task.                                                   |
| `submit()`           | Submit a task and receive a Future.                                       |
| `shutdown()`         | Stop accepting new tasks and finish submitted tasks.                      |
| `shutdownNow()`      | Attempt to interrupt running tasks and return tasks that haven't started. |
| `isShutdown()`       | Check whether shutdown has started.                                       |
| `isTerminated()`     | Check whether all tasks have completed after shutdown.                    |
| `awaitTermination()` | Wait for executor termination up to a specified time.                     |
| `invokeAll()`        | Submit multiple tasks and wait for all to complete.                       |
| `invokeAny()`        | Return one successful completed task's result.                            |

### execute() vs submit()

| Feature               | execute()                    | submit()                                 |
| --------------------- | ---------------------------- | ---------------------------------------- |
| Accepts               | Runnable                     | Runnable or Callable                     |
| Returns               | void                         | Future                                   |
| Result retrieval      | No                           | Possible through Future                  |
| Completion tracking   | No Future                    | Yes                                      |
| Exception observation | Depends on execution context | Through Future.get() for submitted tasks |

**When to use:**

- `execute()` → When I only need to execute a task.
- `submit()` → When I need to track completion, cancel a task, or retrieve its result.

### Important: Instantiating Executor and ExecutorService

Both `Executor` and `ExecutorService` are interfaces.

Therefore, we cannot directly instantiate them using `new`.

#### A. Executor

`Executor` is a Functional Interface because it contains only one abstract method.

It can be implemented using:

1. Anonymous Inner Class.
2. Lambda Expression.

Example:

```
Executor executor = command -> {
    new Thread(command).start();
};
```

#### B. ExecutorService

`ExecutorService` contains multiple abstract methods.

Therefore, we generally use concrete implementations instead of implementing it manually.

Common implementations:

| Implementation                | Purpose                                     |
| ----------------------------- | ------------------------------------------- |
| `ThreadPoolExecutor`          | Customizable thread pool.                   |
| `ScheduledThreadPoolExecutor` | Delayed and periodic task execution.        |
| `ForkJoinPool`                | Work-stealing and divide-and-conquer tasks. |

**Remember:** `Executors` is a utility class that provides factory methods to create ExecutorService implementations.

---

## 5. Runnable, Callable and Future

These three concepts are important when submitting tasks and handling their results.

### A. Runnable

- Represents a task that does not return a result through its interface.
- Contains the `run()` method.
- Cannot declare checked exceptions in `run()`.

```
Runnable task = () -> {
    System.out.println("Processing order");
};
```

### B. Callable

- Represents a task that returns a result.
- Contains the `call()` method.
- Can throw checked exceptions.
- Uses generics to define the return type.

```
Callable<Integer> task = () -> {
    return 10 + 20;
};
```

### Runnable vs Callable

| Runnable                          | Callable                     |
| --------------------------------- | ---------------------------- |
| `run()` method                    | `call()` method              |
| No result through its interface   | Returns a result             |
| Cannot declare checked exceptions | Can throw checked exceptions |
| Used with execute() or submit()   | Used with submit()           |

### C. Future

**What is Future?**

- `Future` represents the result of an asynchronous task.
- It acts as a handle through which we can check task completion, cancel the task, or retrieve its result.
- The result may not be available immediately.

### Important Future Methods

| Method               | Purpose                                      |
| -------------------- | -------------------------------------------- |
| `get()`              | Wait for completion and retrieve the result. |
| `get(timeout, unit)` | Wait for a limited time.                     |
| `isDone()`           | Check whether the task is completed.         |
| `isCancelled()`      | Check whether the task was cancelled.        |
| `cancel(boolean)`    | Attempt to cancel the task.                  |

### Example: Callable + Future

```
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {

        ExecutorService service =
                Executors.newSingleThreadExecutor();

        Callable<Integer> task = () -> 10 + 20;

        Future<Integer> future = service.submit(task);

        try {
            System.out.println(future.get()); // 30
        }
        catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        catch (ExecutionException e) {
            System.err.println("Task failed: " + e.getCause());
        }
        finally {
            service.shutdown();
        }
    }
}
```

### Execution Flow

1. Create an ExecutorService.
2. Create a Callable task.
3. Submit it using `submit()`.
4. Receive a Future object.
5. Call `future.get()` to retrieve the result.
6. Shut down the ExecutorService.

**Important:**

- `get()` blocks the calling thread until the result is available.
- `InterruptedException` occurs if the waiting thread is interrupted.
- `ExecutionException` indicates that the submitted task failed.
- `TimeoutException` can occur when using `get(timeout, unit)`.

---

## 6. ThreadPoolExecutor

### What is ThreadPoolExecutor?

- `ThreadPoolExecutor` is a concrete implementation of `ExecutorService`.
- It manages worker threads and a task queue.
- Workers execute submitted tasks and can be reused after completing them.

### Important Configurations

| Configuration              | Meaning                                                               |
| -------------------------- | --------------------------------------------------------------------- |
| `corePoolSize`             | Number of core worker threads maintained by the pool.                 |
| `maximumPoolSize`          | Maximum number of worker threads allowed.                             |
| `keepAliveTime`            | How long eligible idle threads beyond the core size can remain alive. |
| `workQueue`                | Stores tasks waiting for execution.                                   |
| `ThreadFactory`            | Creates worker threads.                                               |
| `RejectedExecutionHandler` | Defines what happens when a task cannot be accepted.                  |

### How ThreadPoolExecutor Works

1. If the number of workers is below `corePoolSize`, create a new worker.
2. Once core workers are busy, place incoming tasks in the queue.
3. If the queue becomes full, create additional workers up to `maximumPoolSize`.
4. If neither a worker nor queue capacity is available, apply the rejection policy.

**Note:** This is the standard behavior. Actual execution depends on queue type and configuration.

An unbounded queue generally prevents the pool from growing beyond its core size during normal operation.

### Example: Custom ThreadPoolExecutor

```
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {

        ThreadPoolExecutor executor =
            new ThreadPoolExecutor(
                2,                      // Core pool size
                4,                      // Maximum pool size
                30,                     // Keep alive time
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(10)
            );

        for (int i = 1; i <= 6; i++) {

            final int taskId = i;

            executor.execute(() -> {
                System.out.println(
                    "Task " + taskId + " on " +
                    Thread.currentThread().getName()
                );
            });
        }

        executor.shutdown();
    }
}
```

### Understanding the Configuration

- Core workers = 2.
- Maximum workers = 4.
- Queue capacity = 10.
- Idle extra workers can remain alive for 30 seconds.
- Default rejection policy = `AbortPolicy`.

The output order is not guaranteed because multiple threads execute tasks concurrently.

### Rejection Policies

When the executor cannot accept another task, it uses a rejection policy.

| Policy                | Behaviour                                                                               |
| --------------------- | --------------------------------------------------------------------------------------- |
| `AbortPolicy`         | Throws `RejectedExecutionException`.                                                    |
| `CallerRunsPolicy`    | Executes the task in the submitting thread if the executor is not shut down.            |
| `DiscardPolicy`       | Silently discards the rejected task.                                                    |
| `DiscardOldestPolicy` | Removes the oldest queued task and retries submission if the executor is not shut down. |

### Example: CallerRunsPolicy

```
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {

        ThreadPoolExecutor executor =
            new ThreadPoolExecutor(
                1,
                1,
                0L,
                TimeUnit.SECONDS,
                new ArrayBlockingQueue<>(1),
                new ThreadPoolExecutor.CallerRunsPolicy()
            );

        for (int i = 1; i <= 3; i++) {

            final int taskId = i;

            executor.execute(() -> {
                try {
                    Thread.sleep(1000);
                }
                catch (InterruptedException e) {
                    Thread.currentThread().interrupt();
                }

                System.out.println(
                    "Task " + taskId + " running on " +
                    Thread.currentThread().getName()
                );
            });
        }

        executor.shutdown();
    }
}
```

**Understanding:**

- Only one worker is allowed.
- Queue capacity is one.
- When both the worker and queue are occupied, the third task runs in the submitting thread through `CallerRunsPolicy`.

---

## 7. Creating Thread Pools Using Executors

### What is Executors?

- `Executors` is a utility class.
- It provides factory methods for creating common ExecutorService implementations.
- These methods internally use predefined configurations for thread pools and task queues.

### A. Fixed Thread Pool

```
ExecutorService executor =
        Executors.newFixedThreadPool(5);
```

**Features:**

- Creates a pool with five core workers.
- Uses a shared unbounded queue.
- Maximum concurrent task execution is five.
- Additional tasks wait in the queue.

Example:

```
ExecutorService executor =
        Executors.newFixedThreadPool(5);

for (int i = 1; i <= 10; i++) {

    final int id = i;

    executor.execute(() ->
        System.out.println("Task " + id)
    );
}

executor.shutdown();
```

### B. Single Thread Executor

```
ExecutorService executor =
        Executors.newSingleThreadExecutor();
```

**Features:**

- Uses one worker thread.
- Executes tasks sequentially.
- Tasks are normally executed in submission order.
- Can replace its worker if it terminates unexpectedly.

**Use case:** When tasks need to execute one after another.

### C. Scheduled Thread Pool

```
ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(2);
```

**Features:**

- Supports delayed task execution.
- Supports periodic task execution.
- Useful for scheduling repeated operations.

---

## 8. ScheduledExecutorService

### What is ScheduledExecutorService?

- It extends `ExecutorService`.
- It allows tasks to execute after a specific delay.
- It also supports repeated execution at fixed intervals.

### A. schedule() – Execute Once After a Delay

```
ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(1);

scheduler.schedule(
    () -> System.out.println("Runs after 5 seconds"),
    5,
    TimeUnit.SECONDS
);

scheduler.shutdown();
```

**Understanding:**

- Executes the task once.
- The task becomes eligible after five seconds.
- Actual execution can happen slightly later depending on scheduling and system load.
- Already submitted delayed tasks are allowed to execute after `shutdown()`.

### B. scheduleAtFixedRate() – Fixed Rate Execution

```
ScheduledExecutorService scheduler =
        Executors.newScheduledThreadPool(1);

scheduler.scheduleAtFixedRate(
    () -> System.out.println("Checking system"),
    0,              // Initial delay
    3,              // Period
    TimeUnit.SECONDS
);
```

**Understanding:**

- First execution starts without an initial delay.
- Subsequent executions follow a fixed schedule based on start times.
- If one execution takes longer than the period, the next execution starts late.
- The same periodic task does not overlap with itself.

### C. scheduleWithFixedDelay() – Fixed Delay Execution

```
scheduler.scheduleWithFixedDelay(
    () -> System.out.println("Cleaning files"),
    0,
    3,
    TimeUnit.SECONDS
);
```

**Understanding:**

- The next execution starts only after the previous execution finishes and the specified delay passes.
- If a task takes five seconds and the delay is three seconds, the next execution starts approximately eight seconds after the previous one started.

### Fixed Rate vs Fixed Delay

| Feature               | Fixed Rate                       | Fixed Delay                      |
| --------------------- | -------------------------------- | -------------------------------- |
| Method                | `scheduleAtFixedRate()`          | `scheduleWithFixedDelay()`       |
| Timing                | Based on scheduled start times   | Delay after completion           |
| Slow task             | Next execution may start late    | Delay is added after completion  |
| Same task overlapping | No                               | No                               |
| Uncaught exception    | Future executions are suppressed | Future executions are suppressed |

**Important:** If a periodic task throws an exception that escapes its execution, subsequent executions are suppressed.

To prevent this, handle expected exceptions inside the task.

---

## 9. Shutting Down ExecutorService

An ExecutorService should be shut down properly to release resources.

### Important Methods

| Method               | Purpose                                                                      |
| -------------------- | ---------------------------------------------------------------------------- |
| `shutdown()`         | Stops accepting new tasks and allows submitted tasks to finish.              |
| `shutdownNow()`      | Attempts to interrupt running tasks and returns tasks that have not started. |
| `awaitTermination()` | Waits for termination up to a specified time.                                |

### Example: Proper Shutdown

```
executor.shutdown();

try {
    if (!executor.awaitTermination(10, TimeUnit.SECONDS)) {
        executor.shutdownNow();
    }
}
catch (InterruptedException e) {
    executor.shutdownNow();
    Thread.currentThread().interrupt();
}
```

### Important Points

- `shutdown()` does not wait for task completion.
- `awaitTermination()` waits for the executor to terminate.
- `shutdownNow()` attempts to interrupt running tasks.
- A task that ignores interruption may continue running.

---

## 10. Complete Example: Processing Orders

```
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {

        ExecutorService executor =
                Executors.newFixedThreadPool(3);

        for (int i = 1; i <= 6; i++) {

            final int orderId = i;

            executor.submit(() -> {

                System.out.println(
                    "Processing order " + orderId +
                    " on " +
                    Thread.currentThread().getName()
                );

            });
        }

        executor.shutdown();
    }
}
```

### Understanding the Flow

1. Create a thread pool with three workers.
2. Submit six order-processing tasks.
3. Three tasks can execute concurrently.
4. Remaining tasks wait in the queue.
5. When a worker finishes, it picks up another task.
6. `shutdown()` stops new submissions and allows existing tasks to finish.

**Remember:** The execution order is not guaranteed.

---

## 11. Quick Comparison – Important Concepts

| Concept                    | Remember It As                                     |
| -------------------------- | -------------------------------------------------- |
| `Executor`                 | Basic task execution interface.                    |
| `ExecutorService`          | Executor + task submission + lifecycle management. |
| `ThreadPoolExecutor`       | Configurable thread pool with workers and queue.   |
| `Executors`                | Utility class for creating common thread pools.    |
| `ScheduledExecutorService` | Executes delayed and periodic tasks.               |
| `Runnable`                 | Task without a result through its interface.       |
| `Callable`                 | Task that returns a result.                        |
| `Future`                   | Handle to retrieve or manage a task's outcome.     |

---

## 12. Common Mistakes

1. Task and Thread are not the same.
2. `execute()` does not guarantee creation of a new thread.
3. `submit()` returns a Future but does not wait for the result.
4. `Future.get()` can block the calling thread.
5. `Future.get()` can throw exceptions.
6. `shutdown()` does not wait for termination.
7. `shutdownNow()` cannot guarantee that every running task stops.
8. Fixed thread pools commonly use unbounded queues.
9. Fixed Rate is based on scheduled start times.
10. Fixed Delay is measured after the previous execution completes.
11. An uncaught exception in a periodic task suppresses future executions.
12. Task execution order is not guaranteed when multiple workers are used.

---

## 13. Final Summary

The Java Executor Framework helps manage task execution without manually creating and controlling every thread.

- **Executor:** Provides `execute()` to submit tasks.
- **ExecutorService:** Adds `submit()`, Future support, and shutdown methods.
- **Runnable:** Represents a task without a return value through its interface.
- **Callable:** Represents a task that returns a result.
- **Future:** Helps retrieve results and manage asynchronous task execution.
- **ThreadPoolExecutor:** Manages worker threads, queues, pool sizes, and rejection policies.
- **Executors:** Provides factory methods for creating common thread pools.
- **ScheduledExecutorService:** Handles delayed and repeated tasks.

**Main idea:** Submit tasks to an ExecutorService and let Java manage their execution using worker threads.

---

## 14. Practice Questions

1. What is the difference between a Task and a Thread?
2. Why do we need a Thread Pool?
3. What is the Java Executor Framework?
4. What is the difference between `execute()` and `submit()`?
5. What is the difference between `Executor` and `ExecutorService`?
6. Why can't we directly instantiate an interface?
7. What is the difference between Runnable and Callable?
8. What is Future and why do we use it?
9. What does `Future.get()` do?
10. What is the difference between `corePoolSize` and `maximumPoolSize`?
11. How does ThreadPoolExecutor manage incoming tasks?
12. What happens when the task queue becomes full?
13. Explain the four rejection policies.
14. What is the difference between Fixed Thread Pool and Single Thread Executor?
15. What is ScheduledExecutorService?
16. Differentiate between `scheduleAtFixedRate()` and `scheduleWithFixedDelay()`.
17. What is the difference between `shutdown()` and `shutdownNow()`?
18. Why should we use `awaitTermination()`?

---

## 15. Practical Revision Code – All Major Concepts

This example demonstrates manual thread creation, Executor, thread pools, ThreadPoolExecutor, Callable, Future, scheduling, and shutdown.

```
import java.util.concurrent.*;

public class Main {
    public static void main(String[] args) {

        // 1. Manual Thread Creation
        Thread manualThread = new Thread(() -> {
            System.out.println("1. Manual thread running on "
                    + Thread.currentThread().getName());
        });

        manualThread.start();


        // 2. Custom Executor Implementation
        Executor customExecutor = command -> {
            new Thread(command).start();
        };

        customExecutor.execute(() -> {
            System.out.println("2. Custom Executor running on "
                    + Thread.currentThread().getName());
        });


        // 3. Executors Factory Methods
        ExecutorService fixedPool =
                Executors.newFixedThreadPool(2);

        ExecutorService singlePool =
                Executors.newSingleThreadExecutor();

        fixedPool.execute(() -> {
            System.out.println("3. Fixed pool task running");
        });

        singlePool.execute(() -> {
            System.out.println("3. Single pool task running");
        });


        // 4. ThreadPoolExecutor
        ThreadPoolExecutor detailedThreadPool =
                new ThreadPoolExecutor(
                    1,
                    1,
                    10L,
                    TimeUnit.SECONDS,
                    new ArrayBlockingQueue<>(1),
                    new ThreadPoolExecutor.CallerRunsPolicy()
                );

        detailedThreadPool.execute(() -> {
            System.out.println("4. Custom ThreadPoolExecutor task");
        });


        // 5. Callable and Future
        ExecutorService callableService =
                Executors.newSingleThreadExecutor();

        Callable<Integer> callableTask = () -> {
            Thread.sleep(100);
            return 100 + 200;
        };

        Future<Integer> future =
                callableService.submit(callableTask);

        try {
            System.out.println("5. Future isDone: "
                    + future.isDone());

            System.out.println("5. Callable result: "
                    + future.get());

            System.out.println("5. Future isDone: "
                    + future.isDone());
        }
        catch (Exception e) {
            e.printStackTrace();
        }


        // 6. ScheduledExecutorService
        ScheduledExecutorService scheduler =
                Executors.newScheduledThreadPool(2);

        // Execute once after a delay
        scheduler.schedule(() -> {
            System.out.println("6. Delayed task executed");
        }, 1, TimeUnit.SECONDS);

        // Execute at fixed rate
        scheduler.scheduleAtFixedRate(() -> {
            System.out.println("6. Fixed rate task executed");
        }, 0, 2, TimeUnit.SECONDS);

        // Execute with fixed delay
        scheduler.scheduleWithFixedDelay(() -> {
            System.out.println("6. Fixed delay task executed");
        }, 0, 2, TimeUnit.SECONDS);


        // Allow scheduled tasks to run briefly
        try {
            Thread.sleep(3000);
        }
        catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }


        // 7. Shutdown
        fixedPool.shutdown();
        singlePool.shutdown();
        detailedThreadPool.shutdown();
        callableService.shutdown();

        // Stop recurring scheduled tasks
        scheduler.shutdownNow();

        try {
            if (!fixedPool.awaitTermination(3, TimeUnit.SECONDS)) {
                fixedPool.shutdownNow();
            }
        }
        catch (InterruptedException e) {
            fixedPool.shutdownNow();
            Thread.currentThread().interrupt();
        }
    }
}
```
