---
layout: post
title: "Hash Map 源码学习"
date: 2013-06-11 10:34
comments: true
categories: 
---
前段时间看了看Hash Map 源代码，趁着端午小长假记录一下自己的学习所得。
Hash Map实际存储数据的对象是一个数组，数组的每第一个元素都是一个单链的头部。数组的每一个元素是一个内部类Entry 的对象。
	// 默认的Entry数组的大小
	static final int DEFAULT_INITIAL_CAPACITY = 16;

    	/**
     	* The maximum capacity, used if a higher value is implicitly specified
     	* by either of the constructors with arguments.
        * MUST be a power of two <= 1<<30.
        * Entry数组的最大值
     	*/
    	static final int MAXIMUM_CAPACITY = 1 << 30;

    	/**
     	* The load factor used when none specified in constructor.
        *默认加载因子是构造容器阈值的一个参数。
     	*/
    	static final float DEFAULT_LOAD_FACTOR = 0.75f;

    	/**
     	* The table, resized as necessary. Length MUST Always be a power of two.
        * Enrty 数组 存储实际数据的地方，长度必须是2的次幂
        */
          transient Entry[] table;

        /**
    	* The number of key-value mappings contained in this map.
        * 容器的大小
     	*/
          transient int size;

        /**
        * The next size value at which to resize (capacity * load factor).
    	* @serial
        * 容器的最大容量，由Entry数组长度＊加载因子决定。当装载的对象大于这个最 大容量的时候，就会进行扩容操作。
    	*/
          int threshold;

        /**
         * The number of times this HashMap has been structurally modified
         * Structural modifications are those that change the number of mappings in
         * the HashMap or otherwise modify its internal structure (e.g.,
         * rehash).  This field is used to make iterators on Collection-views of
         * the HashMap fail-fast.  (See ConcurrentModificationException).
         * 容器被改变的次数。新加入一个不存在的key的时候也算是改变容器
         */
          transient int modCount;	
先看看最为重要的 内部Entry 类。这个类实现Map的Entry<K,V>接口。有这么几个方法:getKey,getValue,setValue,equals,hashCode:
        static class Entry<K,V> implements Map.Entry<K,V> {
        //前面说过，HashMap是由数组链表构成的。这里的next就是指向链表的下一个节点
        final K key;
        V value;
        Entry<K,V> next;
        final int hash;

        /**
         * Creates new entry.
         */
        Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }

        public final K getKey() {
            return key;
        }

        public final V getValue() {
            return value;
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry e = (Map.Entry)o;
            Object k1 = getKey();
            Object k2 = e.getKey();
            if (k1 == k2 || (k1 != null && k1.equals(k2))) {
                Object v1 = getValue();
                Object v2 = e.getValue();
                if (v1 == v2 || (v1 != null && v1.equals(v2)))
                    return true;
            }
            return false;
        }

        public final int hashCode() {
            return (key==null   ? 0 : key.hashCode()) ^
                   (value==null ? 0 : value.hashCode());
        }

        public final String toString() {
            return getKey() + "=" + getValue();
        }

        /**
         * This method is invoked whenever the value in an entry is
         * overwritten by an invocation of put(k,v) for a key k that's already
         * in the HashMap.
         * 这个方法 在LinkedHashMap里面被重写了，LinkedHashMap可以按照存入的顺序获取对象，也能按照访问顺序获取对象，当按照访问顺序获取时：1，当put方法被调用，并且key已经存在的时候，调用该方法。2,当get方法被调用也需要调整访问顺序。以后写LinkedHashMap在详细写。
         */
        void recordAccess(HashMap<K,V> m) {
        }

        /**
         * This method is invoked whenever the entry is
         * removed from the table.
         * 当对象被从容器中移除时调用
         */
        void recordRemoval(HashMap<K,V> m) {
        }
    }
接下来看看构造函数：
    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);

        // Find a power of 2 >= initialCapacity
        //table 数组的长度必须是2的次方
        int capacity = 1;
        while (capacity < initialCapacity)
            capacity <<= 1;

        this.loadFactor = loadFactor;
        threshold = (int)(capacity * loadFactor);
        table = new Entry[capacity];
        init();
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative. 
     * 使用默认加载因子构造一个HashMap对象
     */
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the default initial capacity
     * (16) and the default load factor (0.75).
     * 使用默认加载因子和默认table容量构造一个HashMap对象
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
        threshold = (int)(DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR);
        table = new Entry[DEFAULT_INITIAL_CAPACITY];
        init();
    }

    /**
     * Constructs a new <tt>HashMap</tt> with the same mappings as the
     * specified <tt>Map</tt>.  The <tt>HashMap</tt> is created with
     * default load factor (0.75) and an initial capacity sufficient to
     * hold the mappings in the specified <tt>Map</tt>.
     *
     * @param   m the map whose mappings are to be placed in this map
     * @throws  NullPointerException if the specified map is null
     * 根据一个Map来构造另一个HashMap
     */
    public HashMap(Map<? extends K, ? extends V> m) {
        //table的大小取（m的大小／默认加载因子＋1）和默认容量中较大的值，使用默认加载因子。因此下面putAllForCreate就不用考虑容器扩容的问题。
        this(Math.max((int) (m.size() / DEFAULT_LOAD_FACTOR) + 1,
                      DEFAULT_INITIAL_CAPACITY), DEFAULT_LOAD_FACTOR);
        putAllForCreate(m);
    }
看看put方法：
    /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     * 计算Key对象的hash值，根据hash值和table的长度，计算出这个key应该处在的table数组的位置。之后遍历这个位置的链表如果发现key已经存在了，那么就替换这个key对应的value值，并返回旧的value值，否则就增加一个节点到这个链表结构中，并且容器修改修改次数＋＋。
     */
     public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        //调用key的hashCode方法，计算hash值。所以使用hashMap的对象要根据实际情况重写hashCode方法，默认的HashCode方法返回的是对象的内存地址。
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        //如果key不存在的话，增加一个节点
        addEntry(hash, key, value, i);
        return null;
     }
现在看看 hash方法,indexFor方法
     /**
     * Applies a supplemental hash function to a given hashCode, which
     * defends against poor quality hash functions.  This is critical
     * because HashMap uses power-of-two length hash tables, that
     * otherwise encounter collisions for hashCodes that do not differ
     * in lower bits. Note: Null keys always map to hash 0, thus index 0.
     */
     static int hash(int h) {
        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        //使用无符号右移操作符和异或操作符得到一个hash值来降低hash值冲突的概率
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
     }
      /**
      * Returns index for hash code h.返回数组下标，length-1是为了防止下标越界
      */
      static int indexFor(int h, int length) {
         return h & (length-1);
      }
现在看看 key=null的情况 
     private V putForNullKey(V value) {
         //null不具备hashCode方法。所以null 的hash值被默认设置为0.因此只有可能被放在table的第一个元素上。
         for (Entry<K,V> e = table[0]; e != null; e = e.next) {
             if (e.key == null) {
                 V oldValue = e.value;
                 e.value = value;
                 e.recordAccess(this);
                 return oldValue;
             }
         }
         modCount++;
         //如果不存在，增加一个节点，并且设置hash值为0,数组下表为0
         addEntry(0, null, value, 0);
         return null;
      }
一个小问题，为什么不在put方法里进行判断key是否为null。如果是就将i和hash赋值为0，为什么单独写一个putForNullKey，其中有大部分的代码都是可以重用的。

下面看看addEntry方法
       void addEntry(int hash, K key, V value, int bucketIndex) {
        // 获取 table bucketIndex处的元素，该元素是此处链表的表头
        Entry<K,V> e = table[bucketIndex];
        //将这个元素作为 新生成的Entry对象的next 指向的下一个元素。新生成的对象替代该元素成为此处链表的表头
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        //判断是否已经超过容器的极限值
         if (size++ >= threshold)
            //是的话，进行容器扩容操作
            resize(2 * table.length);
    }

接下来看看resize方法
     void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            //如果table的长度已经是最大值了，那么就将容器的容量极限值设置为integer的最大值，并且不在进行扩容操作。
            threshold = Integer.MAX_VALUE;
            return;
        }
        //否则进行扩容操作。
        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable);
        table = newTable;
        //重新设置容器的极限值
        threshold = (int)(newCapacity * loadFactor);
    }
transfer方法，该方法在linkedHashMap 出于效率的考虑重写过，这个以后在说。
      void transfer(Entry[] newTable) {
        Entry[] src = table;
        int newCapacity = newTable.length;
         //遍历老的容器，将容器的元素放到新的newTable数组链表中
        for (int j = 0; j < src.length; j++) {
            Entry<K,V> e = src[j];
            if (e != null) {
                src[j] = null;
                //遍历链表
                do {
                    Entry<K,V> next = e.next;
                    //获取该节点在新容器里的数组下标
                    int i = indexFor(e.hash, newCapacity);
                    //将e插入到新链表的表头
                    e.next = newTable[i];
                    newTable[i] = e;
                    //e 只想老练表的下一个元素
                    e = next;
                } while (e != null);
            }
        }
    }
注意到经过扩容之后，原先到HashMap的元素顺序已经完全打乱了。

还记得根据一个map创造一个HashMap对象的构造方法吗？构造方法调用了如下方法：
     private void putAllForCreate(Map<? extends K, ? extends V> m) {
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet())
            putForCreate(e.getKey(), e.getValue());
     }
       private void putForCreate(K key, V value) {
        int hash = (key == null) ? 0 : hash(key.hashCode());
        int i = indexFor(hash, table.length);

        /**
         * Look for preexisting entry for key.  This will never happen for
         * clone or deserialize.  It will only happen for construction if the
         * input Map is a sorted map whose ordering is inconsistent w/ equals.
         */
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k)))) {
                e.value = value;
                return;
            }
        }

        createEntry(hash, key, value, i);
    }

大部分代码比较直白，看看createEntry方法。相比与addEntry方法只是不需要扩容了，因为在构造函数里已经对容量做了计算
      void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }
在看和get相关的方法：
     public V get(Object key) {
        if (key == null)
            return getForNullKey();
        int hash = hash(key.hashCode());
        //是一个o（n）的遍历
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                return e.value;
        }
        return null;
    }
    private V getForNullKey() {
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {
            if (e.key == null)
                return e.value;
        }
        return null;
    }
    //根据key获取一个Entry对象
    final Entry<K,V> getEntry(Object key) {
        int hash = (key == null) ? 0 : hash(key.hashCode());
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash &&
                ((k = e.key) == key || (key != null && key.equals(k))))
                return e;
        }
        return null;
    }
都很简单，也不多说了。
再来看看HashMap的迭代器设计,map类本身并没有提供迭代器方法。你只能获取一个map的key的Set或者value的Collection或者entry的Set，然后调用这些set对象的迭代器方法获取迭代器对象。
    //这是Values 私有类。可以通过values()方法获取到一个map的Values对象，这个对象的iterator方法可以获取到该对象的迭代器对象。
    public Collection<V> values() {
        Collection<V> vs = values;
        return (vs != null ? vs : (values = new Values()));
    }
    private final class Values extends AbstractCollection<V> {
        public Iterator<V> iterator() {
            return newValueIterator();
        }
        public int size() {
            return size;
        }
        public boolean contains(Object o) {
            return containsValue(o);
        }
        public void clear() {
            HashMap.this.clear();
        }
    }
     
      Iterator<V> newValueIterator()   {
        return new ValueIterator();
      }
       //value迭代器对象，继承自HashIterator 重写了next方法，如果是EntrySet的话就是return nextEntry（），如果是KeySet就是return nextEntry().key。nextEntry()方法是HashIterator对象的方法。 
        private final class ValueIterator extends HashIterator<V> {
            public V next() {
                return nextEntry().value;
            }
        }

HashMap的私有迭代器抽象类，EntrySet的迭代器类，values的迭代器，keySet的迭代器类都是继承自这个类。注意到没有，这个类实现类Iterator接口，但是却没有实现next方法。还记得前一篇文章说的吗?抽象类实现接口的意义。因为这个next方法会根据几个子迭代器不同而改变，但是迭代器的其它方法却都是一样的。所以采用这种设计模式。
      private abstract class HashIterator<E> implements Iterator<E> {
        Entry<K,V> next;        // next entry to return
        int expectedModCount;   // For fast-fail
        int index;              // current slot
        Entry<K,V> current;     // current entry

        HashIterator() {
            expectedModCount = modCount;
            if (size > 0) { // advance to first entry
                Entry[] t = table;
                while (index < t.length && (next = t[index++]) == null)
                    ;
            }
        }

        public final boolean hasNext() {
            return next != null;
        }
        //final 方法不能被重写
        final Entry<K,V> nextEntry() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            Entry<K,V> e = next;
            if (e == null)
                throw new NoSuchElementException();

            if ((next = e.next) == null) {
                Entry[] t = table;
                while (index < t.length && (next = t[index++]) == null)
                    ;
            }
            current = e;
            return e;
        }

        public void remove() {
            if (current == null)
                throw new IllegalStateException();
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
            Object k = current.key;
            current = null;
            HashMap.this.removeEntryForKey(k);
            expectedModCount = modCount;
        }

    }
接下来看看EntrySet和KeySet的迭代器设计，和Values的大同小异,就是迭代器的next方法的返回值不一样。
     public Set<K> keySet() {
        Set<K> ks = keySet;
        return (ks != null ? ks : (keySet = new KeySet()));
    }

    private final class KeySet extends AbstractSet<K> {
        public Iterator<K> iterator() {
            return newKeyIterator();
        }
        public int size() {
            return size;
        }
        public boolean contains(Object o) {
            return containsKey(o);
        }
        public boolean remove(Object o) {
            return HashMap.this.removeEntryForKey(o) != null;
        }
        public void clear() {
            HashMap.this.clear();
        }
     }
     Iterator<K> newKeyIterator()   {
        return new KeyIterator();
     }
     private final class KeyIterator extends HashIterator<K> {
        public K next() {
            return nextEntry().getKey();
        }
     }
在贴一下EntySet的代码：
      private final class EntrySet extends AbstractSet<Map.Entry<K,V>> {
        public Iterator<Map.Entry<K,V>> iterator() {
            return newEntryIterator();
        }
        public boolean contains(Object o) {
            if (!(o instanceof Map.Entry))
                return false;
            Map.Entry<K,V> e = (Map.Entry<K,V>) o;
            Entry<K,V> candidate = getEntry(e.getKey());
            return candidate != null && candidate.equals(e);
        }
        public boolean remove(Object o) {
            return removeMapping(o) != null;
        }
        public int size() {
            return size;
        }
        public void clear() {
            HashMap.this.clear();
        }
     }
     Iterator<Map.Entry<K,V>> newEntryIterator()   {
        return new EntryIterator();
     }
     private final class EntryIterator extends HashIterator<Map.Entry<K,V>> {
       public Map.Entry<K,V> next() {
           return nextEntry();
       }
     }


