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
1. When method throws InterruptedException, it means method is blocking and may be interrupted.
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
Latches (Synchronizer) 
----------
1. Latch is a synchronizer that delay threads progress till it reaches to terminal state.
2. Latch act as gate, all thread has to wait to open gate.
3. Latch is used when one task depends on certain ohter tasks to complete.
4. For Example Singer has wait untill band is ready.
```java
public void latchExample() {
    final CountDownLatch latch = new CountDownLatch(1);
    
    Thread band = new Thread(new Runnable() {
        public void run() {
            System.out.println("Band is ready");
            latch.countDown();
        }
    });
    
    Thread singer = new Thread(new Runnable() {
        public void run() {
            try {
                //Wait till Band is ready
                latch.await();
                System.out.println("Singing");
            } catch(InterruptedException e) {
                e.printStackTrace();
            }
        }
    });
    
    //Singer will wait to band to start
    singer.start();
    band.start();
}
```
FutureTask (Synchronizer) 
----------
1. ```FutureTask``` is wrapper for a task result, which may complete in future.
2. Can be used to safely publication of a result  from excuting thread to calling thread.
3. ```FutureTask<T>``` has some methods to get results state, value or cancle current task.
4. ```FutureTask``` can used to precompute the some value, which will needed for futhure processing.
5. Executor framework uses ```FutureTask``` to compute asynchronous task.

```java
public class PreLoader {

	private final FutureTask<String> futureTask = new FutureTask<>(new Callable<String>() {
		@Override
		public String call() throws Exception {
			return loadDateInfo();
		}
	});
	
	private final Thread t = new Thread(futureTask);
	
	public void start() {
		t.start();
	}
	
	public String getInfo() throws InterruptedException, ExecutionException {
		return futureTask.get();
	}
	
}

```

Semaphore (Synchronizer) 
----------
1. Counting Semaphore is used to controll the number of activity that can access a given resource or perform given action at the time.
2. Semaphore has given number of permits, activity can take permit if available else it will block untill permits is avialable.
3. release mthod return permots to semaphore when its done with resource.
4. Semaphore can be used to create bounded collection. Following example shows bounded hashSet
```java
public class BoundedeHashSet<T> {
	private final Semaphore sem;
	private final Set<T> set;
	
	public BoundedeHashSet(int bound) {
		this.set = Collections.synchronizedSet(new HashSet<T>());
		this.sem = new Semaphore(bound);
	}
	
	public boolean add(T o) throws InterruptedException {
		sem.acquire();
		boolean wasAdded = false;
		try {
			wasAdded = set.add(o);
			return wasAdded;
		} finally {
			if(!wasAdded) {
				sem.release();
			}
		}
	}
	
	public boolean remove(T e) {
		boolean wasRemoved = false;
		try {
			wasRemoved = set.remove(e);
			return wasRemoved;
			
		} finally {
			if(wasRemoved) {
				sem.release();
			}
		}
	}
}
```

