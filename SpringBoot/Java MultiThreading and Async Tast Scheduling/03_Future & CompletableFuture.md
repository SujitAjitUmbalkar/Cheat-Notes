# **Future and CompletableFuture (Java Concurrency)**

## **1. Future**

**Definition:** `Future` is an interface in Java that represents the result of an asynchronous computation. It allows us to execute a task in another thread and retrieve its result later without blocking the main thread immediately.

**Important methods:**

| Method          | Purpose                                       |
| --------------- | --------------------------------------------- |
| `get()`         | Retrieves the result; blocks until available. |
| `isDone()`      | Checks whether the task is completed.         |
| `isCancelled()` | Checks whether the task was cancelled.        |
| `cancel()`      | Attempts to cancel the task.                  |

**Example:**

```
try (ExecutorService executor = Executors.newFixedThreadPool(4)) {

    Callable<String> task = new Callable<String>() {
        @Override
        public String call() throws Exception {
            return getName();
        }
    };

    Future<String> future = executor.submit(task);

    log.info("Result: {}", future.get());
}
```

**Limitations of Future:**

- `get()` blocks the calling thread until the result is available.
- Cannot directly attach callbacks to execute when the task completes.
- Does not provide built-in methods for chaining multiple asynchronous tasks.

## **2. CompletableFuture**

**Definition:** `CompletableFuture` is a class that implements the `Future` and `CompletionStage` interfaces. It supports asynchronous programming by allowing us to attach callbacks, chain multiple tasks, handle exceptions and combine multiple asynchronous operations without manually blocking after every task.

**Example:**

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

**Important methods:**

| Method            | Purpose                                                                            |
| ----------------- | ---------------------------------------------------------------------------------- |
| `supplyAsync()`   | Executes an asynchronous task that returns a result.                               |
| `runAsync()`      | Executes an asynchronous task without returning a result.                          |
| `thenApply()`     | Transforms the previous task's result.                                             |
| `thenAccept()`    | Consumes the result without returning anything.                                    |
| `thenRun()`       | Executes the next task without using the previous result.                          |
| `thenCompose()`   | Chains dependent asynchronous tasks.                                               |
| `thenCombine()`   | Combines results of two independent tasks.                                         |
| `exceptionally()` | Handles exceptions and provides a fallback result.                                 |
| `whenComplete()`  | Performs an action when the task completes, whether successfully or exceptionally. |
| `join()`          | Retrieves the result and throws an unchecked exception if the task fails.          |

## **3. Future vs CompletableFuture**

| Feature                | Future              | CompletableFuture       |
| ---------------------- | ------------------- | ----------------------- |
| Asynchronous execution | Yes                 | Yes                     |
| Retrieve result        | `get()`             | `get()` / `join()`      |
| Callback support       | No                  | Yes                     |
| Task chaining          | No                  | Yes                     |
| Combine multiple tasks | No built-in support | Yes                     |
| Exception handling     | Limited             | Built-in methods        |
| Manual completion      | No                  | Yes, using `complete()` |

## **4. When to use?**

- **Future:** When you need to execute a background task and retrieve its result later.
- **CompletableFuture:** When you need multiple asynchronous operations, task chaining, callbacks, combining results or handling exceptions.

**Remember**

`Future` = Submit a task and retrieve its result.

`CompletableFuture` = Submit a task, process its result, chain tasks and handle exceptions asynchronously.

**Important:** `CompletableFuture` does not automatically make every operation asynchronous. For example, `thenApply()` may execute on the thread that completes the previous stage. Use `thenApplyAsync()` when you specifically want asynchronous execution of that stage. Also, calling `get()` or `join()` still blocks the calling thread until the result is available.
