Notes
When to use volatile
----------
1. when variable is only modified by single thread.
2. when variable is not taking part in define other variables state
3. when synchronization is not required.

Problems with Synchronized Collection (not concurrent collection java 5)
----------
Iterator is not thread safe In synchronized collections 
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

4. Putting synchronized on loop may cause performance issue.
5. Another way is to make iteration on Copy of Collection.
6. But Copying of collection may hit memory issue, its upto situation.
7. Even Iterator throws ConcurrentModifcationException if  another thread modifies collection, but that also not sure.
8. Internally iterator has a count for concurrent modification trace, and that is not thread safe.
9. There is may be a case that other thread modifies collections, but we will not get concurrent modification exception.

Thread Confinement 
-----------
>Object is called thread confined when it is only accessable by single thread.

1. That is when object handled to a thread, thread is exclusively owner of that object,other thread can't access or modify it unless thread release it to maintener.
2. JDBC ```Connection``` object is good example of thread confinement. It is confined to single request thread.

Ad-hoc Thread Confinement
Stack Confinement
1. Use of local variable, always confined to single thread.
ThreadLocal
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

