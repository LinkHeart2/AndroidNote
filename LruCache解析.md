# LruCache了解一下

## 一、LruCache是什么

该类最开始出现在Android 3.1中，是高速缓存，其中包含对有限数量的值的强引用。每次值被访问时，它都会移到队列的开头。将值添加到缓存中后如果缓存满了，该队列末尾的值将被逐出，并且可能会引发垃圾回收。
默认情况下，缓存大小以条目数衡量。重写 sizeOf以不同的单位调整缓存大小。如下例，此缓存限制存放4MiB的Bitmap：

```java
    int cacheSize = 4 * 1024 * 1024; // 4MiB
    LruCache<String, Bitmap> bitmapCache = new LruCache<String, Bitmap>(cacheSize) {
        protected int sizeOf(String key, Bitmap value) {
            return value.getByteCount();
       }
    }
```

## 二、解析源码

构造方法：

```java
    /**
     * @param maxSize 对于不覆盖sizeOf的缓存，这是缓存中的最大条目数。
     * 对于所有其他缓存，这是此缓存中条目大小的最大和。
     */
    public LruCache(int maxSize) {
        if (maxSize <= 0) {
            throw new IllegalArgumentException("maxSize <= 0");
        }
        this.maxSize = maxSize;
        this.map = new LinkedHashMap<K, V>(0, 0.75f, true);
    }

```

使用了LinkHashMap作为实际缓存的容器，其构造方法的第三个参数是accessOrder，设置为true表示按照访问顺序排序。

**public void resize(int maxSize)**
设置缓存的大小

**LruCache#get**
返回key的值（如果它存在于缓存中或可以由create创建）。如果返回值，则将其移到队列的开头。如果未缓存值并且无法创建，则返回null。

```java
    public final V get(K key) {
        if (key == null) {
            throw new NullPointerException("key == null");
        }

        V mapValue;
        synchronized (this) {
            mapValue = map.get(key);
            if (mapValue != null) {
                hitCount++;
                return mapValue;
            }
            missCount++;
        }

        /*
         * 尝试创建一个值。这可能会花费很长时间，并且当create（）返回时，map可能会有所不同。
         * 如果在create（）工作时将冲突值添加到map中，则将该值保留在map中并释放创建的值。
         */

        V createdValue = create(key);
        if (createdValue == null) {
            return null;
        }

        synchronized (this) {
            createCount++;
            mapValue = map.put(key, createdValue);

            if (mapValue != null) {
                // 发生了冲突，因此撤消前面插入的创建的值
                map.put(key, mapValue);
            } else {
                size += safeSizeOf(key, createdValue);
            }
        }

        if (mapValue != null) {
            entryRemoved(false, key, createdValue, mapValue);
            return mapValue;
        } else {
            trimToSize(maxSize);
            return createdValue;
        }
    }
```

**LruCache#put**
为key缓存value。该值将插入到队列的开头。

```java
    public final V put(K key, V value) {
        if (key == null || value == null) {
            throw new NullPointerException("key == null || value == null");
        }

        V previous;
        synchronized (this) {
            putCount++;
            size += safeSizeOf(key, value);
            previous = map.put(key, value);
            if (previous != null) {
                size -= safeSizeOf(key, previous);
            }
        }

        if (previous != null) {
            entryRemoved(false, key, previous, value);
        }

        trimToSize(maxSize);
        return previous;
    }
```

**LruCache#trimToSize**
删除最旧的条目，直到剩余条目总数等于或小于请求的大小。

```java
    public void trimToSize(int maxSize) {
        while (true) {
            K key;
            V value;
            synchronized (this) {
                if (size < 0 || (map.isEmpty() && size != 0)) {
                    throw new IllegalStateException(getClass().getName()
                            + ".sizeOf() is reporting inconsistent results!");
                }

                if (size <= maxSize) {
                    break;
                }

                //调用了LinkedHashMap#eldest来获取最近最少使用的条目
                Map.Entry<K, V> toEvict = map.eldest();
                if (toEvict == null) {
                    break;
                }

                key = toEvict.getKey();
                value = toEvict.getValue();
                map.remove(key);
                size -= safeSizeOf(key, value);
                evictionCount++;
            }

            entryRemoved(true, key, value, null);
        }
    }
```

**public final V remove(K key)**
删除key条目（如果存在）

**protected void entryRemoved(boolean evicted, K key, V oldValue, V newValue) {}**
当有条目被移除或调用put导致值被替换时会调用该方法，默认是空实现
evicted参数为true表示是缓存满了为了腾出空间而移除的，其它情况为false

**protected V create(K key)**
当key未命中缓存时调用该方法生成对应key的值，默认返回null

**protected int sizeOf(K key, V value)**
以用户定义的单位返回key和value的条目大小。默认实现返回1，因此size是条目数，max size是最大条目数。

**public final void evictAll()**
清除缓存，对每个已删除条目调用entryRemoved

## 三、LinkedHashMap

最近最少使用的顺序是由LinkedHashMap来确保的，使用了一个双向链表

```java
    /**
     * 双向链表的头（最老）
     */
    transient LinkedHashMapEntry<K,V> head;

    /**
     * 双链表的末尾（最小）
     */
    transient LinkedHashMapEntry<K,V> tail;

    /**
     * 此LinkedHashMap的迭代排序方法：true 表示访问顺序， false 表示插入顺序。
     * @serial
     */
    final boolean accessOrder;
```

前面提到LruCache初始化LinkedHashMap时accessOrder为true，按访问顺序排序。
accessOrder为true的情况下，双向链表的头是最近最少使用的条目，尾是最近最多使用的条目。

我们看看这个accessOrder在哪里起到了作用

```java
    public V get(Object key) {
        Node<K,V> e;
        if ((e = getNode(hash(key), key)) == null)
            return null;
        if (accessOrder)
            afterNodeAccess(e);
        return e.value;
    }
```

在LinkedHashMap#get方法中，accessOrder为true就调用LinkedHashMap#afterNodeAccess

```java
    void afterNodeAccess(Node<K,V> e) { // 将节点移到最后
        LinkedHashMapEntry<K,V> last;
        if (accessOrder && (last = tail) != e) {
            LinkedHashMapEntry<K,V> p =
                (LinkedHashMapEntry<K,V>)e, b = p.before, a = p.after;
            p.after = null;
            if (b == null)
                head = a;
            else
                b.after = a;
            if (a != null)
                a.before = b;
            else
                last = b;
            if (last == null)
                head = p;
            else {
                p.before = last;
                last.after = p;
            }
            tail = p;
            ++modCount;
        }
    }
```

在put的过程中，如果HashMap中不存在对应键的条目，就用了newNode来生成新的条目，LinkedHashMap重写了该方法，把新条目连接到列表末尾

```java
    Node<K,V> newNode(int hash, K key, V value, Node<K,V> e) {
        LinkedHashMapEntry<K,V> p =
            new LinkedHashMapEntry<K,V>(hash, key, value, e);
        linkNodeLast(p);
        return p;
    }

    // 连接到列表末尾
    private void linkNodeLast(LinkedHashMapEntry<K,V> p) {
        LinkedHashMapEntry<K,V> last = tail;
        tail = p;
        if (last == null)
            head = p;
        else {
            p.before = last;
            last.after = p;
        }
    }
```

前面提到移除条目时调用LinkedHashMap#eldest获取最近最少使用条目，该方法直接返回了head

```java
    public Map.Entry<K, V> eldest() {
        return head;
    }
```

## 四、实现一个简易LruCache

```java
import java.util.LinkedHashMap;
import java.util.Map;

public class LruCache<K,V> extends LinkedHashMap<K,V>{
    int cacheSize;
    public LruCache(int cacheSize){
        super((int) Math.ceil(cacheSize / 0.75),0.75f,true);
        this.cacheSize = cacheSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry<K, V> eldest) {
        return size()>cacheSize;
    }
}
```

1. 继承了LinkedHashMap，调用父类构造方法，设置双向链表按访问顺序排序。

2. 重写 LinkedHashMap#removeEldestEntry 方法，当 size 大于指定缓存容量时，就返回了true，LinkedHashMap就会删掉最旧的元素。
