### Notes
##### When to use volatile
1. when variable is only modified by single thread.
2. when variable is not taking part in define other variables state
3. when synchronization is not required.

##### Problems with java concurrent API
Iterator is not thread safe
1. When accessing concurrent collection by multiple, iterator is not thread safe.
2. It follow **Fail First** approach, that it collection is modified by another thread we will get Concurrent Modification exception.
3. One way is synchronise terator loop, using collection refernce as guard object.
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
4. Putting synchronized may cause performance issue.
5. Another way is to make iteration of Copy of Collection.
6. But Copy of collection may hit memory issue.Its upto situation what to do to Concurrent iteration concurrent modification.
7. Even Iterator throws ConcurrentModifcationException if  another thread modifies collection, but that also not sure.
8. Internally  Iterator has a count for concurrent modification trac, and that is not thread safe.
9. There is may case that even thread modifies, but we not have concurrent modification exception.
