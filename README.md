### Notes
##### When to use volatile
1. when variable is only modified by single thread.
2. when variable is not taking part in define other variables state
3. when synchronization is not required.

##### Problems with Synchronized Collection (not concurrent collection java 5)
###### Iterator is not thread safe In synchronized collections 
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
