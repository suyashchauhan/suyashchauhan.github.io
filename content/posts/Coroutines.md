+++
date = '2025-05-11T15:02:02+05:30'
draft = true
title = 'Coroutines'
+++

### What they are for ?
write asynchronous or blocking code in such a fashion that it looks similar to unblocking code


### Some other options
 - Threads  
    Thread creation is expensive


Threads
```kt{linenos=inline}
fun main() = runBlocking {
    repeat(50_000) { // launch a lot of threads
        thread {
            Thread.sleep(5000L)
            println(".")
        }
    }
}
```
Coroutines
```kt{linenos=inline}
fun main() = runBlocking {
    repeat(50_000) { // launch a lot of coroutines
        launch {
            delay(5000L)
            println(".")
        }
    }
}
```
- Callback style code
- RxJava



### Kotlin DSL
Domain specific language


- Kotlin is a statically typed language (type of a variable is known at compile time)

- Kotlin functions are first-class, which means they can be stored in variables and data structures,
and can be passed as arguments to and returned from other higher-order functions

- Function Types -> Kotlin uses this to represent functions

- Function literals are functions that are not declared but are passed immediately as an expression

- Lambda expressions and anonymous functions are function literals. 

receiver type, which refers to the type being extended

Function types can optionally have an additional receiver type, 
which is specified before the dot in the notation: the type A.(B) -> C represents functions 
that can be called on a receiver object A with a parameter B and return a value C (also called as "function types with reciever")




Inside the body of the function literal, the receiver object passed to a call becomes an implicit **this**, so that you can access the members of that receiver object without any additional qualifiers, or access the receiver object using a this expression.

## Suspend functions

- When they are suspended, they return a Continuation
- suspend functions can only be called from inside coroutines or another suspending function
- suspend functions have the power to suspend the coroutines
- suspend functions are actually state machines with suspension points as states

### Who will store these states ?
Continuation
```kt{linenos=inline}
/**
 * Interface representing a continuation after a suspension point that returns a value of type `T`.
 */
@SinceKotlin("1.3")
public interface Continuation<in T> {
    /**
     * The context of the coroutine that corresponds to this continuation.
     */
    public val context: CoroutineContext

    /**
     * Resumes the execution of the corresponding coroutine passing a successful or failed [result] as the
     * return value of the last suspension point.
     */
    public fun resumeWith(result: Result<T>)
}



```


When function a calls function b, the virtual machine needs to store the state of a somewhere, as well as the address where execution should return once b is finished. All this is stored in a structure called call stack  


- Coroutine context , CoroutineScope , Shared Mutable state

## Coroutine Builder
coroutine builder, a bridge from the normal to the suspending world
launch and async are extension functions on CoroutineScope.  

launch
- start a coroutine, and it will run independently  

runBlocking  
- runBlocking is a very atypical builder. It blocks the thread 
it has been started on whenever **its** coroutine is suspended  

async
- similar to launch, but it is designed to produce a value

## Coroutine scope
- To create a scope out of a suspending function, we use the coroutineScope function.
```kt{linenos=inline}
public interface CoroutineScope {
    /**
     * The context of this scope.
     * Context is encapsulated by the scope and used for implementation of coroutine builders that are extensions on the scope.
     * Accessing this property in general code is not recommended for any purposes except accessing the [Job] instance for advanced usages.
     *
     * By convention, should contain an instance of a [job][Job] to enforce structured concurrency.
     */
    public val coroutineContext: CoroutineContext
}
```



## Structured concurrency
A parent provides a scope for its children, and they are called in this scope. This builds a relationship that is called a structured concurrency. Here are the most important effects of the parent-child relationship:

- children inherit context from their parent (but they can also overwrite it, as will be explained in the Coroutine context chapter);
- a parent suspends until all the children are finished (this will be explained in the Job and children awaiting chapter);
- when the parent is cancelled, its child coroutines are cancelled too (this will be explained in the Cancellation chapter);
- when a child raises an error, it destroys the parent as well (this will be explained in the Exception handling chapter).



## Coroutine Context 
```kt{linenos=inline}
public interface CoroutineContext {
    /**
     * Returns the element with the given [key] from this context or `null`.
     */
    public operator fun <E : Element> get(key: Key<E>): E?

    /**
     * Accumulates entries of this context starting with [initial] value and applying [operation]
     * from left to right to current accumulator value and each element of this context.
     */
    public fun <R> fold(initial: R, operation: (R, Element) -> R): R

    /**
     * Returns a context containing elements from this context and elements from  other [context].
     * The elements from this context with the same key as in the other one are dropped.
     */
    public operator fun plus(context: CoroutineContext): CoroutineContext =
        if (context === EmptyCoroutineContext) this else // fast path -- avoid lambda creation
            context.fold(this) { acc, element ->
                val removed = acc.minusKey(element.key)
                if (removed === EmptyCoroutineContext) element else {
                    // make sure interceptor is always last in the context (and thus is fast to get when present)
                    val interceptor = removed[ContinuationInterceptor]
                    if (interceptor == null) CombinedContext(removed, element) else {
                        val left = removed.minusKey(ContinuationInterceptor)
                        if (left === EmptyCoroutineContext) CombinedContext(element, interceptor) else
                            CombinedContext(CombinedContext(left, element), interceptor)
                    }
                }
            }

    /**
     * Returns a context containing elements from this context, but without an element with
     * the specified [key].
     */
    public fun minusKey(key: Key<*>): CoroutineContext

    /**
     * Key for the elements of [CoroutineContext]. [E] is a type of element with this key.
     */
    public interface Key<E : Element>

    /**
     * An element of the [CoroutineContext]. An element of the coroutine context is a singleton context by itself.
     */
    public interface Element : CoroutineContext {
        /**
         * A key of this coroutine context element.
         */
        public val key: Key<*>

        public override operator fun <E : Element> get(key: Key<E>): E? =
            @Suppress("UNCHECKED_CAST")
            if (this.key == key) this as E else null

        public override fun <R> fold(initial: R, operation: (R, Element) -> R): R =
            operation(initial, this)

        public override fun minusKey(key: Key<*>): CoroutineContext =
            if (this.key == key) EmptyCoroutineContext else this
    }
}

```
operations ->plus, minus


## Job context 

- a job represents a cancellable thing with a lifecycle
- each coroutine has its own job
- provider structured concurrency points 2,3,4
- To check the state in code, we use the properties isActive, isCompleted, and isCancelled.
- Job is a coroutine context, we can access it using coroutineContext[Job].
- When a coroutine has its own (independent) job, it has nearly no relation to its parent. It only inherits other contexts, but other results of the parent-child relationship will not apply. This causes us to lose structured concurrency, which is a problematic situation that should be avoided
- join method > This is a suspending function that suspends until a concrete job reaches a final state (either Completed or Cancelled).
```kt{linenos=inline}
public interface Job : CoroutineContext.Element { }
```

Jobs states 

| **State**       | **isActive** | **isCompleted** | **isCancelled** |
|----------------|--------------|------------------|------------------|
| New *(optional initial state)*       | false        | false          | false          |
| Active *(default initial state)*     | true         | false          | false          |
| Completing *(transient state)*       | true         | false          | false          |
| Cancelling *(transient state)*       | false        | false          | true           |
| Cancelled *(final state)*            | false        | true           | true           |
| Completed *(final state)*            | false        | true           | false          |


## Cancellation  
- cancellation is **co-operative**
- once a job is cancelled it cannot acts as a parent for any new coroutines
- ends the job at the first suspension point 
- if job has children they are also cancelled

Freeing up resources after cancellation  
1. 
```kt{linenos=inline}

job.invokeOnCompletion { exception: Throwable? ->
        println("Finished")
    }
```
2. 
```kt{linenos=inline}
finally {
            println("Finally")
            withContext(NonCancellable) {
                delay(1000L)
                println("Cleanup done")
            }
        }
```


**Q: How to do suspending tasks after cancellation ?**  
Ideally after cancellation we can't launch new coroutines 
```kt{linenos=inline}
finally {
        withContext(NonCancellable) {
            println("job: I'm running finally")
            delay(1000L)
            println("job: And I've just delayed for 1 sec because I'm non-cancellable")
        }
    }
```


## Exception Handling 

## Scope functions

- coroutineScope

```kt{linenos=inline}
suspend fun getUserName() {
    delay(1000)
    println("User: suyash")
}

suspend fun getUserData() {
    delay(1000)
    println("location, hobbies")
}


suspend fun main() = coroutineScope {
    printTime("begin")
    val userName = async { getUserName() }
    val userData = async { getUserData() }
    printTime("end ${userName.await()} ${userData.await()}")

}
```


## Dispatchers 

Main - the default one  
IO - when we block threads with I/O operations limit- 64 by default  
Default - limit of 20 for mine

- limitedParallelism used on Dispatchers.Default makes a dispatcher with an additional limit. Using limitedParallelism on Dispatcher.IO makes a dispatcher independent of Dispatcher.IO. 
However, they all share the same infinite pool of threads.

**Q: Dispatcher limited to a single thread ?**
Solves the shared state problems


## Shared state problems

## Constructing a coroutine scope




## Channel
why ? Share Information b/w coroutines

Channel supports any number of senders and receivers, and every value that is sent to a channel is received only once.


Hot data streams are eager, produce elements independently of their consumption,
and store the elements. Cold data streams are lazy, perform their operations on-demand, and store nothing.

## Flow



### References

[1]