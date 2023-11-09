# Threading Cheat Sheet for C/C++

## Table of Contents
- [POSIX Threads (pthreads) in C](#posix-threads-pthreads-in-c)
  - [Header File](#header-file)
  - [Creating and Exiting Threads](#creating-and-exiting-threads)
  - [Waiting for Threads to Finish](#waiting-for-threads-to-finish)
  - [Thread Attributes](#thread-attributes)
  - [Mutex Initialization](#mutex-initialization)
  - [Mutex Lock and Unlock](#mutex-lock-and-unlock)
  - [Condition Variables](#condition-variables)
  - [Thread Cleanup](#thread-cleanup)
- [C++11 Thread Library](#c11-thread-library)
  - [Header Files](#header-files)
  - [Creating and Joining Threads](#creating-and-joining-threads)
  - [Mutexes](#mutexes)
  - [Condition Variables](#condition-variables)
  - [Async and Future](#async-and-future)
  - [Thread Local Storage](#thread-local-storage)
- [Compilation](#compilation)
  - [For C](#for-c)
  - [For C++](#for-c-1)
- [Examples](#examples)
  - [Basic Thread Creation and Joining](1.md)
  - [Using a Lambda as a Thread Function](2.md)
  - [Mutex for Thread Synchronization](3.md)
  - [Thread with Member Function](4.md)
  - [Detaching Threads](5.md)
  - [Thread with Function Parameters](6.md)
  - [Using `std::async` and `std::future`](7.md)
  - [Parallel Array Operations with Threads`](8.md)



# POSIX Threads (pthreads) in C

POSIX threads, or pthreads, is a POSIX standard for threads, providing a set of APIs to create and manage threads in a Unix/Linux environment.

## Creating and Exiting Threads

To create a thread, you use `pthread_create`, and to exit a thread, you use `pthread_exit`.

```c
#include <pthread.h>
#include <stdio.h>

// Thread function to be executed by all threads
void *myThreadFun(void *vargp) {
    printf("Hello from new thread - got %d\n", *((int*)vargp));
    pthread_exit(NULL);
}

int main() {
    pthread_t thread_id;
    int value = 42;
    printf("Before Thread\n");
    pthread_create(&thread_id, NULL, myThreadFun, (void *)&value);
    // Other code
    pthread_join(thread_id, NULL);
    printf("After Thread\n");
    exit(0);
}
```

## Thread Synchronization

Synchronization is crucial in concurrent programming to prevent race conditions. Mutexes are used to protect data integrity when threads access shared data.

### Mutex Initialization

Declare and initialize a mutex before using it.

```c
pthread_mutex_t lock = PTHREAD_MUTEX_INITIALIZER;
```

### Mutex Lock and Unlock

A mutex lock is used to block other threads from entering the critical section until the mutex is unlocked.

```c
pthread_mutex_lock(&lock);
// Critical section - only one thread can access this at a time
pthread_mutex_unlock(&lock);
```

## Condition Variables

Condition variables are used to block a thread until a particular condition is met.

```c
pthread_cond_t cond = PTHREAD_COND_INITIALIZER;

// Thread function
void *threadFunction(void *arg) {
    pthread_mutex_lock(&lock);
    pthread_cond_wait(&cond, &lock); // Wait for the condition
    // Proceed after the condition is signaled
    pthread_mutex_unlock(&lock);
    pthread_exit(NULL);
}
```

## Joining and Detaching Threads

Joining a thread makes the calling thread wait until the specified threadid thread terminates.

```c
pthread_join(thread_id, NULL);
```

Detaching a thread releases any resources once it has completed.

```c
pthread_detach(thread_id);
```

## Thread Attributes

Thread attributes allow you to set certain properties like stack size or scheduling policy for the threads.

```c
pthread_attr_t attr;
pthread_attr_init(&attr);
pthread_attr_setdetachstate(&attr, PTHREAD_CREATE_DETACHED);
```

## Thread-Cleanup

Once you are done with mutexes and condition variables, you should destroy them to free up resources.

```c
pthread_mutex_destroy(&lock);
pthread_cond_destroy(&cond);
```

Remember to link the pthread library when compiling your C program:

```bash
gcc -o my_program my_program.c -pthread
```

This chapter covers the basic use of POSIX threads in C. Always refer to the detailed documentation for a thorough understanding and best practices.

Ensure that you replace `my_program` with your actual program's name when compiling, and you replace `// Other code` with the actual code you wish to run in the `main` function.

 
# C++11 Thread Library

C++11 introduced a thread library which provides classes and functions to manage threads.

## Header Files

To use the C++11 threading facilities, include the required headers.

```cpp
#include <thread>
#include <mutex>
#include <condition_variable>
```

## Creating and Joining Threads

A thread is created by instantiating an `std::thread` object and passing a callable entity to its constructor.

```cpp
void threadFunction() {
    // Code to execute in the thread.
}

int main() {
    std::thread t(threadFunction); // Create a thread running threadFunction
    t.join(); // Wait for the thread to finish
    return 0;
}
```

## Mutexes

Mutexes in C++11 are used to protect shared data from being simultaneously accessed by multiple threads.

```cpp
std::mutex mtx;

void safeFunction() {
    std::lock_guard<std::mutex> lock(mtx); // RAII style mutex locking
    // Code that modifies shared data
}
```

## Condition Variables

Condition variables are used to synchronize threads via notifications.

```cpp
std::condition_variable cv;
bool ready = false;
std::mutex mtx;

void workerThread() {
    std::unique_lock<std::mutex> lock(mtx);
    cv.wait(lock, []{ return ready; }); // Wait until 'ready' becomes true
    // Do work after the wait
}

int main() {
    std::thread worker(workerThread);
    {
        std::lock_guard<std::mutex> lk(mtx);
        ready = true; // Set 'ready' to true
    }
    cv.notify_one(); // Notify the waiting thread
    worker.join(); // Wait for the thread to finish
    return 0;
}
```

## Async and Future

C++11 provides `std::async`, which can run a task asynchronously (potentially in a new thread) and return a `std::future` that will hold the result.

```cpp
#include <future>

int performComputation() {
    // Compute and return result
    return 42;
}

int main() {
    std::future<int> result = std::async(performComputation);
    std::cout << "The result is " << result.get() << std::endl; // get() waits for the computation to finish and retrieves the result
    return 0;
}
```

## Thread Local Storage

C++11 introduced thread local storage, allowing variables to have a distinct value for each thread.

```cpp
thread_local int threadSpecificData = 0;

void threadFunction() {
    threadSpecificData = /* some value unique to the thread */;
}
```

# Compilation

Make sure to enable C++11 (or later) support in your compiler and link any necessary libraries.

```bash
g++ -std=c++11 -pthread -o my_program my_program.cpp
```