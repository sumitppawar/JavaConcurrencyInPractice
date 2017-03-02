Notes
When to use volatile
----------
1. when variable is only modified by single thread.
2. when variable is not taking part in define other variables state
3. when synchronization is not required.

Problems with Synchronized Collection (not concurrent collection java 5)
----------
**Iterator is not thread safe In synchronized collections**

1. When accessing synchronized collection by multiple thread, iterator is not thread safe.
2. It follow **Fail First** approach, that is if collection is modified by another thread we will get Concurrent Modification exception.
3. One way to avoid exception is synchronise iterator loop, using collection reference as guard object.

```Java
public void method(Vector<String> v) {
    Iterator<String> itr = v.iterator();
    while(itr.hasNext()) {
         // Throws ConcurrentModificationException when
         //Passed vector is modified by another thread
        System.out.println(itr.next());
    }
    
    //Thread Safe Iterator
    synchronized(v) {
        while(itr.hasNext()) {
            System.out.println(itr.next());
        }
    }
}
```

Putting synchronized on loop may cause performance issue.
Another way is to make iteration on Copy of Collection.
But Copying of collection may hit memory issue, its upto situation.
Even Iterator throws ConcurrentModifcationException if  another thread modifies collection, but that also not sure.
Internally iterator has a count for concurrent modification trace, and that is not thread safe.
There is may be a case that other thread modifies collections, but we will not get concurrent modification exception.

Thread Confinement 
-----------
>Object is called thread confined when it is only accessable by single thread.

1. That is when object handled to a thread, thread is exclusively owner of that object,other thread can't access or modify it unless thread release it to maintener.
2. JDBC ```Connection``` object is good example of thread confinement. It is confined to single request thread.

### Ad-hoc Thread Confinement
### Stack Confinement
1. Use of local variable, always confined to single thread.
### ThreadLocal
1. Use of ```ThreadLocal``` Lib class.
2. Provides get and set method that mentains seperate copy of value for each thread that uses it.
3. get returns most recent value passed to set from current executing thread.
4. Example

```java
private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {
    public Connection initialValue() {
        return DriverManager.getConnection(DB_URL);
    }
};
public Connection getConnection() {
    return connectionHolder.get()
}
```

5. When get is called for first time, method ```initialValue()``` is execute.

Deque and Work Stealing
-----------
1. Java 6 has added two new collection ```Deque``` (pronouced as 'deck') and ```BlockingDeque```, double ended queue that are allows efficient insertion and removal from both end.
2. ```Deque``` and ```BlockingDeque``` extends ```Queue``` and ```BlockingQueue``` respectivily.
3. Their implementation includes ```ArrayDeque``` and ```LinkedBlockingDeque```.
4. Unlike ```BlockingQueue``` every consumer has its own deue, once its own deque is exahusted, consumer will steal task from other threads deque.
5. This type of producer consumer are mainly suited for task where consumer can be aslo producer, when consumer find out new task it will add its own deque.
6. Typical example is Web crawler, when crawler find new link in page, it will add new task to its deque.
7. By ***Work Stealing*** pattern, sytem thread will be always busy.

Synchronizer
----------
1. Synchronizer is an Object that manages workflow among threads.
2. ```BlockingQueue``` is an examle of synchronizer.
3. Othere type of synchronizer are lateches, semaphore, barriers.
4. All synchronizer has some common properties, they all have some state. 
5. Based on state, it forces to incoming thread to wait.
6. They provide way to manupulate state and effient wait mechanism.

Interruptible Methods
----------
1. When methods throws InterruptedException, it means method is blocking and may be interrupted.
2. Most common use of interruption is to cancel thread work.
3. Interruption is a cooperative mechanism and we must have plan to respond it.
4. In java lib, either propogate it or if not possible to propogate interrupt current thread(restore the iterrupt).
```java

//Restore Interrupt in case it not possible to throws
public class RunnableTask extends Runnable {
    public void run() {
        try {
            Thread.sleep();
        } catch(InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```


