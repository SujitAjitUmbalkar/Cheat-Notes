# **Future and CompletableFuture – Java Multithreading**

# **1. Future**

## **1.1 What is Future?**

**Definition:** `Future` is an interface in Java that represents the result of a task that is being executed asynchronously (in another thread).

It allows us to submit a task to an ExecutorService and continue doing other work without waiting for that task to finish immediately. Later, we can retrieve its result using `get()`.

**Why do we need Future?**

Suppose we are building a Spring Boot application and need to fetch user details from another service. This operation takes 5 seconds.

Instead of making the main thread wait for 5 seconds, we can submit this task to another thread and continue doing other work.

When we need the result, we can retrieve it using Future.

## **1.2 How to create a Future?**

We generally use the `submit()` method of ExecutorService.

**Important:** `submit()` returns a Future object.

```
try (ExecutorService executor = Executors.newFixedThreadPool(4)) {

    Callable<String> task = new Callable<String>() {
        @Override
        public String call() throws Exception {
            Thread.sleep(5000);
            return "Sujit";
        }
    };

    Future<String> future = executor.submit(task);

    log.info("Task submitted");

    log.info("Result: {}", future.get());
}
```

**Understanding the code:**

1. We create an ExecutorService with 4 worker threads.
2. We create a Callable task that sleeps for 5 seconds and returns `"Sujit"`.
3. We submit the task using `submit()`.
4. The ExecutorService executes the task on a worker thread.
5. Meanwhile, the main thread can continue its work.
6. When the main thread calls `future.get()`, it waits until the result is available.

**How Future works**

**Main Thread**

Submits task using `submit()`

**Main Thread**

Continues executing other statements.

Waits at get()

**Worker Thread**

Executes Callable task.

Returns result.

## **1.3 Important methods of Future**

There are four important methods you should know.

**1. get()**

- Used to retrieve the result of a task.
- If the task is still running, the calling thread waits until the result becomes available.
- It can throw `InterruptedException` and `ExecutionException`.

```
String result = future.get();
```

**2. isDone()**

- Checks whether the task has completed or not.
- Returns `true` if the task is completed, otherwise `false`.
- It does not block the calling thread.

```
while (!future.isDone()) {
    log.info("Task is still running...");
}

log.info("Result: {}", future.get());
```

Here, we repeatedly check whether the task is completed. This is called **Polling**.

Once `isDone()` returns `true`, we can retrieve the result using `get()` without waiting for the task to finish.

**3. isCancelled()**

Checks whether the task was cancelled.

```
log.info("Cancelled: {}", future.isCancelled());
```

**4. cancel()**

Attempts to cancel the task.

```
future.cancel(true);
```

- `true`: Attempts to interrupt the thread executing the task.
- `false`: Does not interrupt the task if it is already running.

Cancellation is only an attempt; it does not guarantee that the task will stop.

## **1.4 Advantages of Future**

1. Allows tasks to execute asynchronously.
2. Main thread does not need to wait immediately after submitting a task.
3. Allows us to retrieve results from background tasks.
4. Provides methods to check task completion and cancel tasks.

## **1.5 Disadvantages of Future**

1. `get()` blocks the calling thread until the result is available.
2. We may need to repeatedly check `isDone()` to know whether the task is completed. This is called polling.
3. We cannot directly tell Future what to do automatically after a task completes.
4. It does not provide built-in methods for chaining multiple tasks.
5. Combining results from multiple Future objects requires additional coding.

**This is where CompletableFuture becomes useful.**

# **2. CompletableFuture**

## **2.1 What is CompletableFuture?**

**Definition:** `CompletableFuture` is a class in Java that represents the result of an asynchronous task. It provides all the basic features of Future along with additional features like automatically executing another task after completion, combining tasks and handling exceptions.

**Why do we need CompletableFuture?**

Let's understand the problem with Future.

Suppose:

1. We fetch user details from one service.
2. After getting user details, we fetch that user's orders.
3. After getting orders, we prepare the final response.

With Future, we generally need to call `get()` to retrieve each result before moving to the next operation.

This makes the main thread wait repeatedly.

With CompletableFuture, we can tell Java:

- When the first task completes, execute the second task.
- When the second task completes, execute the third task.

We don't need to manually wait using `get()` between every operation.

**In simple words:** CompletableFuture allows us to define what should happen after a task finishes.

## **2.2 How to create CompletableFuture?**

There are two main methods.

### **A. supplyAsync()**

**Definition:** `supplyAsync()` is used when we want to execute a task asynchronously and the task returns a result.

It accepts a `Supplier` and returns a `CompletableFuture`.

```
try (ExecutorService executor = Executors.newFixedThreadPool(4)) {

    Supplier<String> task = new Supplier<String>() {
        @Override
        public String get() {
            return getName();
        }
    };

    CompletableFuture<String> future =
            CompletableFuture.supplyAsync(task, executor);

    log.info("Result: {}", future.get());
}
```

Here:

- `Supplier` is used because our task returns a String.
- `supplyAsync()` executes the task using the provided ExecutorService.
- `future.get()` retrieves the result and blocks until it is available.

**Note:** If we don't provide an ExecutorService, Java normally uses its common ForkJoinPool.

### **B. runAsync()**

**Definition:** `runAsync()` is used when we want to execute a task asynchronously but don't need any result in return.

It accepts a `Runnable` and returns `CompletableFuture<Void>`.

```
try (ExecutorService executor = Executors.newFixedThreadPool(4)) {

    Runnable task = new Runnable() {
        @Override
        public void run() {
            log.info("Sending email...");
        }
    };

    CompletableFuture<Void> future =
            CompletableFuture.runAsync(task, executor);

    future.get();
}
```

Here:

- `Runnable` is used because the task does not return anything.
- `runAsync()` executes the task asynchronously.
- `future.get()` waits until the task completes.

### **Difference between supplyAsync() and runAsync()**

| supplyAsync()                      | runAsync()                                 |
| ---------------------------------- | ------------------------------------------ |
| Used when a task returns a result. | Used when a task does not return a result. |
| Accepts `Supplier`.                | Accepts `Runnable`.                        |
| Returns `CompletableFuture<T>`.    | Returns `CompletableFuture<Void>`.         |

# **3. Most Important Feature: Callback Methods**

This is the main advantage of CompletableFuture over Future.

**What is a callback?**

A callback is simply an operation that we tell Java to execute after another task finishes.

For example, after fetching a username, we want to convert it into uppercase.

With CompletableFuture, we can attach another operation to the first task.

## **3.1 thenApply()**

**Definition:** `thenApply()` is used to perform an operation on the result of the previous task and return a modified result.

Example:

```
try (ExecutorService executor = Executors.newFixedThreadPool(4)) {

    CompletableFuture<String> future =
            CompletableFuture.supplyAsync(new Supplier<String>() {
                @Override
                public String get() {
                    return "Sujit";
                }
            }, executor);

    CompletableFuture<String> result = future.thenApply(
            new Function<String, String>() {
                @Override
                public String apply(String name) {
                    return name.toUpperCase();
                }
            }
    );

    log.info("Result: {}", result.get());
}
```

Output:

```
SUJIT
```

**How does it work?**

1. First task returns `"Sujit"`.
2. `thenApply()` automatically receives that result.
3. It converts the name into uppercase.
4. It returns the modified result.

We don't need to manually retrieve the first result using `get()` before writing the next operation.

## **3.2 thenAccept()**

**Definition:** `thenAccept()` is used when we want to perform an operation using the previous task's result but don't need to return anything.

Example:

```
future.thenAccept(new Consumer<String>() {
    @Override
    public void accept(String name) {
        log.info("User name: {}", name);
    }
});
```

Here, we receive the username and print it. We don't return anything.

## **3.3 thenRun()**

**Definition:** `thenRun()` is used to execute another operation after the previous task completes. It does not receive the previous task's result and does not return anything.

Example:

```
future.thenRun(new Runnable() {
    @Override
    public void run() {
        log.info("Task completed successfully");
    }
});
```

Here, we don't need the previous task's result. We just want to execute another operation.

### **Difference between thenApply(), thenAccept() and thenRun()**

| Method         | Receives previous result? | Returns a result? |
| -------------- | ------------------------- | ----------------- |
| `thenApply()`  | Yes                       | Yes               |
| `thenAccept()` | Yes                       | No                |
| `thenRun()`    | No                        | No                |

**Easy way to remember**

- `thenApply()` → Take result, modify it and return it.
- `thenAccept()` → Take result and use it.
- `thenRun()` → Just execute another operation.

# **4. Synchronous vs Asynchronous Callback Methods**

You will see two versions of some CompletableFuture methods.

For example:

- `thenApply()`
- `thenApplyAsync()`

**thenApply():** The next operation may execute on the thread that completes the previous task or another thread that calls the completion method.

**thenApplyAsync():** The next operation is scheduled for asynchronous execution using the default executor or the ExecutorService we provide.

Example:

```
future.thenApplyAsync(
        new Function<String, String>() {
            @Override
            public String apply(String name) {
                return name.toUpperCase();
            }
        },
        executor
);
```

**Important:** The `Async` version does not necessarily mean a new thread is created every time. Java uses the available executor to execute the task.

The same idea applies to `thenAcceptAsync()` and `thenRunAsync()`.

# **5. Working with Multiple Tasks**

Now let's understand how CompletableFuture helps us when we have multiple tasks.

There are two important methods.

## **5.1 thenCompose()**

**Definition:** `thenCompose()` is used when we have two tasks and the second task depends on the result of the first task.

Example:

Consider an e-commerce application:

1. First, fetch the user's ID.
2. Using that ID, fetch the user's orders.

```
CompletableFuture<String> userFuture =
        CompletableFuture.supplyAsync(new Supplier<String>() {
            @Override
            public String get() {
                return "USER101";
            }
        });

CompletableFuture<String> orderFuture = userFuture.thenCompose(
        new Function<String, CompletableFuture<String>>() {
            @Override
            public CompletableFuture<String> apply(String userId) {

                return CompletableFuture.supplyAsync(
                        new Supplier<String>() {
                            @Override
                            public String get() {
                                return "Orders for " + userId;
                            }
                        }
                );
            }
        }
);
```

**How does it work?**

- First task returns the user ID.
- `thenCompose()` receives that ID.
- It starts another asynchronous task to fetch orders.
- The final result is returned as a single `CompletableFuture`.

**Remember:** Use `thenCompose()` when the second asynchronous task depends on the result of the first task.

## **5.2 thenCombine()**

**Definition:** `thenCombine()` is used when we have two independent tasks running and want to combine their results after both are completed.

Example:

Suppose we want to fetch user details and product details independently.

```
CompletableFuture<String> userFuture =
        CompletableFuture.supplyAsync(new Supplier<String>() {
            @Override
            public String get() {
                return "Sujit";
            }
        });

CompletableFuture<String> productFuture =
        CompletableFuture.supplyAsync(new Supplier<String>() {
            @Override
            public String get() {
                return "Laptop";
            }
        });

CompletableFuture<String> finalResult = userFuture.thenCombine(
        productFuture,
        new BiFunction<String, String, String>() {
            @Override
            public String apply(String user, String product) {
                return user + " purchased " + product;
            }
        }
);
```

Output:

```
Sujit purchased Laptop
```

**How does it work?**

1. Both tasks can execute independently.
2. `thenCombine()` waits for both results.
3. It combines those results.
4. It returns a new CompletableFuture.

**Remember:** Use `thenCombine()` when two tasks are independent but their results are needed together.

### **Difference between thenCompose() and thenCombine()**

| thenCompose()                              | thenCombine()                     |
| ------------------------------------------ | --------------------------------- |
| Used for dependent tasks.                  | Used for independent tasks.       |
| Second task needs the first task's result. | Both tasks can run independently. |
| Chains two asynchronous tasks.             | Combines two task results.        |

# **6. Exception Handling in CompletableFuture**

What happens if an asynchronous task fails?

For example, an API call fails or a database operation throws an exception.

CompletableFuture provides methods to handle these situations.

## **6.1 exceptionally()**

**Definition:** `exceptionally()` is used to handle an exception and provide a default result if the previous task fails.

```
CompletableFuture<String> future =
        CompletableFuture.supplyAsync(new Supplier<String>() {
            @Override
            public String get() {
                throw new RuntimeException("Something went wrong");
            }
        });

CompletableFuture<String> result = future.exceptionally(
        new Function<Throwable, String>() {
            @Override
            public String apply(Throwable exception) {
                log.error("Error: {}", exception.getMessage());
                return "Default Name";
            }
        }
);
```

Here:

- The original task throws an exception.
- `exceptionally()` catches the failure.
- It returns `"Default Name"` instead.

## **6.2 whenComplete()**

**Definition:** `whenComplete()` is used to perform an operation when the previous task finishes, whether it succeeds or fails.

```
future.whenComplete(new BiConsumer<String, Throwable>() {
    @Override
    public void accept(String result, Throwable exception) {

        if (exception != null) {
            log.error("Task failed: {}", exception.getMessage());
        } else {
            log.info("Task completed: {}", result);
        }
    }
});
```

**Important difference:**

- `exceptionally()` can recover from an error by returning a fallback result.
- `whenComplete()` is mainly used to observe the result or error, such as logging. It does not recover from the error.

# **7. Other Useful CompletableFuture Methods**

These are useful to know, but you can practise them after understanding the main methods.

| Method            | Purpose                                                                                |
| ----------------- | -------------------------------------------------------------------------------------- |
| `get()`           | Retrieves the result and blocks until available.                                       |
| `join()`          | Retrieves the result, but throws an unchecked exception instead of checked exceptions. |
| `isDone()`        | Checks whether the task is completed.                                                  |
| `complete(value)` | Manually completes the CompletableFuture with a value.                                 |
| `cancel()`        | Attempts to cancel the task.                                                           |
| `allOf()`         | Waits for all given CompletableFutures to complete.                                    |
| `anyOf()`         | Completes when any one of the given CompletableFutures completes.                      |
| `handle()`        | Handles both success and failure and can return a new result.                          |

**Example of allOf():**

Suppose we have three independent tasks and want to continue only after all three are completed.

```
CompletableFuture<String> task1 = CompletableFuture.supplyAsync(
        new Supplier<String>() {
            @Override
            public String get() {
                return "User";
            }
        }
);

CompletableFuture<String> task2 = CompletableFuture.supplyAsync(
        new Supplier<String>() {
            @Override
            public String get() {
                return "Order";
            }
        }
);

CompletableFuture<String> task3 = CompletableFuture.supplyAsync(
        new Supplier<String>() {
            @Override
            public String get() {
                return "Payment";
            }
        }
);

CompletableFuture<Void> allTasks =
        CompletableFuture.allOf(task1, task2, task3);

allTasks.get();

log.info("All tasks completed");
```

`allOf()` returns `CompletableFuture<Void>`. It does not automatically give us a list of all the results. We retrieve individual results from their respective futures if needed.

# **8. Future vs CompletableFuture**

| Feature                      | Future              | CompletableFuture  |
| ---------------------------- | ------------------- | ------------------ |
| Asynchronous execution       | Yes                 | Yes                |
| Retrieve result              | `get()`             | `get()` / `join()` |
| Check completion             | `isDone()`          | `isDone()`         |
| Callback methods             | No                  | Yes                |
| Transform result             | No built-in support | `thenApply()`      |
| Use result without returning | No built-in support | `thenAccept()`     |
| Execute next task            | No built-in support | `thenRun()`        |
| Chain dependent tasks        | No built-in support | `thenCompose()`    |
| Combine independent tasks    | No built-in support | `thenCombine()`    |
| Exception handling           | Limited             | Built-in methods   |
| Manually complete task       | No                  | Yes                |

## **9. Advantages and Disadvantages**

**Future – Advantages**

1. Simple way to execute tasks asynchronously.
2. Allows us to retrieve results from background tasks.
3. Provides methods to check completion and cancel tasks.

**Future – Disadvantages**

1. `get()` blocks the calling thread.
2. Polling using `isDone()` may waste CPU time.
3. Does not provide built-in callback and chaining methods.
4. Combining multiple task results requires additional coding.

**CompletableFuture – Advantages**

1. Supports asynchronous task execution.
2. Allows us to execute another operation automatically after a task completes.
3. Supports chaining and combining multiple tasks.
4. Provides built-in exception-handling methods.
5. Can use a custom ExecutorService to manage task execution.

**CompletableFuture – Disadvantages**

1. Has more methods, so it can initially be difficult to understand.
2. Debugging multiple chained tasks can be difficult.
3. Calling `get()` or `join()` still blocks the calling thread.
4. Incorrectly using asynchronous tasks can make code unnecessarily complicated.

# **10. Final Revision**

| Concept           | Remember                                                   |
| ----------------- | ---------------------------------------------------------- |
| Future            | Represents the result of an asynchronous task.             |
| `submit()`        | Submits a task and returns Future.                         |
| `get()`           | Retrieves result and waits if necessary.                   |
| `isDone()`        | Checks whether the task has completed.                     |
| CompletableFuture | Future with additional features for handling task results. |
| `supplyAsync()`   | Runs a task that returns a result.                         |
| `runAsync()`      | Runs a task that doesn't return a result.                  |
| `thenApply()`     | Modifies the previous result.                              |
| `thenAccept()`    | Uses the previous result without returning anything.       |
| `thenRun()`       | Executes another task without using the previous result.   |
| `thenCompose()`   | Runs a dependent asynchronous task.                        |
| `thenCombine()`   | Combines results of independent tasks.                     |
| `exceptionally()` | Handles errors and provides a fallback result.             |
| `whenComplete()`  | Runs an action when a task finishes.                       |
| `allOf()`         | Waits for all tasks.                                       |
| `anyOf()`         | Completes when any one task completes.                     |

**The most important thing to remember**

**Future:** Submit a task, continue working, and retrieve its result when needed.

**CompletableFuture:** Submit a task and tell Java what to do when it completes, including processing results, starting dependent tasks, combining results and handling errors.

## **11. What should you learn first?**

Don't try to memorize all the methods at once. Follow this order:

1. `supplyAsync()` and `runAsync()`
2. `thenApply()`, `thenAccept()` and `thenRun()`
3. `thenApplyAsync()`
4. `thenCompose()` and `thenCombine()`
5. `exceptionally()` and `whenComplete()`
6. `allOf()` and `anyOf()`

Once you understand these, you'll have a good foundation for using CompletableFuture in real Spring Boot backend applications.
