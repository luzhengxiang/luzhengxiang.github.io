---
title: java集合
date: 2020-03-15 16:47:57
tags: java
---



# java中集合大纲

## 首先我们看结构类图
![](/img/java/集合/collection_main.jpg)


在java中，主要是collection和map两个集合接口，我们在java中使用到的所有集合都是继承或者实现了这两个接口。


## Collection
### Collection 接口中定义的方法
```   java
   int size();  //返回集合中元素个数
   boolean isEmpty();  //判断集合是否为空
   boolean contains(Object o); //判断集合中是否包含某元素
   Iterator<E> iterator(); //迭代器
   Object[] toArray(); //返回一个包含了本集合类中所有元素的数组。
   <T> T[] toArray(T[] a);
   boolean add(E e); //在集合末尾添加元素
   boolean remove(Object o); //若当前集合中有值与0的值相等的元素，则删除
   boolean containsAll(Collection<?> c);
   boolean addAll(Collection<? extends E> c);
   boolean removeAll(Collection<?> c);
   default boolean removeIf(Predicate<? super E> filter) {
        Objects.requireNonNull(filter);
        boolean removed = false;
        final Iterator<E> each = iterator();
        while (each.hasNext()) {
            if (filter.test(each.next())) {
                each.remove();
                removed = true;
            }
        }
        return removed;
    }
    boolean retainAll(Collection<?> c);
    void clear();
    boolean equals(Object o);
    int hashCode();
    @Override
    default Spliterator<E> spliterator() {
        return Spliterators.spliterator(this, 0);
    }
    default Stream<E> stream() {
        return StreamSupport.stream(spliterator(), false);
    }
    default Stream<E> parallelStream() {
        return StreamSupport.stream(spliterator(), true);
    }
```

### 常用Collection的实现类
Collection 接口的接口 对象的集合（单列集合）</br>
├——-—-List 接口：元素按进入先后有序保存，可重复</br>
│—————- ├ LinkedList 接口实现类， 链表结构， 插入删除， 没有同步， 线程不安全</br>
│—————- ├ ArrayList 接口实现类， 数组结构， 随机访问， 没有同步， 线程不安全</br>
│—————- └ Vector 接口实现类 数组， 同步， 线程安全</br>
│ ———————-└ Stack 是Vector类的实现类</br>
└——-Set 接口： 仅接收一次，不可重复，并做内部排序</br>
├————└HashSet 使用hash表（数组）存储元素</br>
│————————└ LinkedHashSet 链表维护元素的插入次序</br>
└ —————-TreeSet 底层实现为二叉树，元素排好序</br>

#### List

#### Set

##### HashSet
HashSet底层数据结构采用哈希表实现，元素无序且唯一，线程不安全，效率高，可以存储null元素，元素的唯一性是靠所存储元素类型是否重写hashCode()和equals()方法来保证的，如果没有重写这两个方法，则无法保证元素的唯一性。<br>
具体实现唯一性的比较过程：存储元素首先会使用hash()算法函数生成一个int类型hashCode散列值，然后已经的所存储的元素的hashCode值比较，如果hashCode不相等，则所存储的两个对象一定不相等，此时存储当前的新的hashCode值处的元素对象；如果hashCode相等，存储元素的对象还是不一定相等，此时会调用equals()方法判断两个对象的内容是否相等，如果内容相等，那么就是同一个对象，无需存储；如果比较的内容不相等，那么就是不同的对象，就该存储了，此时就要采用哈希的解决地址冲突算法，在当前hashCode值处类似一个新的链表， 在同一个hashCode值的后面存储存储不同的对象，这样就保证了元素的唯一性。

##### LinkedHashSet 
LinkedHashSet底层数据结构采用链表和哈希表共同实现，链表保证了元素的顺序与存储顺序一致，哈希表保证了元素的唯一性。线程不安全，效率高。
##### TreeSet 
TreeSet底层数据结构采用二叉树来实现，元素唯一且已经排好序；<br>
唯一性同样需要重写hashCode和equals()方法，二叉树结构保证了元素的有序性。<br>
根据构造方法不同，分为自然排序（无参构造）和比较器排序（有参构造），自然排序要求元素必须实现Compareable接口，并重写里面的compareTo()方法，元素通过比较返回的int值来判断排序序列，返回0说明两个对象相同，不需要存储；比较器排需要在TreeSet初始化是时候传入一个实现Comparator接口的比较器对象，或者采用匿名内部类的方式new一个Comparator对象，重写里面的compare()方法；

#### List和Set总结：
（1）、List,Set都是继承自Collection接口.<br>
（2）、List特点：元素有放入顺序，元素可重复 ，Set特点：元素无放入顺序，元素不可重复，重复元素会覆盖掉，（注意：元素虽然无放入顺序，但是元素在set中的位置是有该元素的HashCode决定的，其位置其实是固定的，加入Set 的Object必须定义equals()方法 ，另外list支持for循环，也就是通过下标来遍历，也可以用迭代器，但是set只能用迭代，因为他无序，无法用下标来取得想要的值。）<br>
（3）.Set和List对比：
Set：检索元素效率低下，删除和插入效率高，插入和删除不会引起元素位置改变。
List：和数组类似，List可以动态增长，查找元素效率高，插入删除元素效率低，因为会引起其他元素位置改变。
（4）、ArrayList与LinkedList的区别和适用场景
Arraylist：
优点：ArrayList是实现了基于动态数组的数据结构,因为地址连续，一旦数据存储好了，查询操作效率会比较高（在内存里是连着放的）。
缺点：因为地址连续， ArrayList要移动数据,所以插入和删除操作效率比较低。
LinkedList：
优点：LinkedList基于链表的数据结构,地址是任意的，所以在开辟内存空间的时候不需要等一个连续的地址，对于新增和删除操作add和remove，LinedList比较占优势。LinkedList 适用于要头尾操作或插入指定位置的场景
缺点：因为LinkedList要移动指针,所以查询操作性能比较低。
适用场景分析：
当需要对数据进行对此访问的情况下选用ArrayList，当需要对数据进行多次增加删除修改时采用LinkedList。

Vector <br>
Vector 线程安全 与arraylist的实现一样也是一个动态数组。但是因为他很多方法用了synchronized来修饰是线程同步的，效率很低，一般不建议使用

#### 常见的面试题
如何解决arraylist线程不安全的问题
1，使用Vector
2，使用Collections.synchronizedList。它会自动将我们的list方法进行改变，最后返回给我们一个加锁了List
``` java
protected static List<Object> arrayListSafe2 = Collections.synchronizedList(new ArrayList<Object>()); 
```
3,使用JUC中的CopyOnWriteArrayList类进行替换。


## Map
 Map用于保存具有映射关系的数据，Map里保存着两组数据：key和value，它们都可以使任何引用类型的数据，但key不能重复。所以通过指定的key就可以取出对应的value。
### Map中定义的方法
``` java
int size();
boolean isEmpty();
boolean containsKey(Object key);// 查询Map中是否包含指定的key，如果包含则返回true
boolean containsValue(Object value); // 查询Map中是否包含指定value，如果包含则返回true
V get(Object key); //获取对象
V put(K key, V value); //设置对象
V remove(Object key); //移除对象
void putAll(Map<? extends K, ? extends V> m);
void clear(); //清空集合
Set<K> keySet(); //返回所有key组成的set集合
Collection<V> values(); // 返回Map中所有的values 
Set<Map.Entry<K, V>> entrySet(); 返回set集合，里面是当前map集合中的entry
...
```

### Map 常用的实现类
Map 接口 键值对的集合 （双列集合）
├———Hashtable 接口实现类， 同步， 线程安全
├———HashMap 接口实现类 ，没有同步， 线程不安全-
│—————–├ LinkedHashMap 双向链表和哈希表实现
│—————–└ WeakHashMap
├ ——–TreeMap 红黑树对所有的key进行排序
└———IdentifyHashMap
————————————————


#### HashMap
jdk1.7 底层是数组+链表
jdk1.8 底层是数组+链表+红黑树实现
非线程安全

//todo  画出hashmap.put的流程图





#### HashTable




StringBuilder是非线程安全的，StringBuffer是线程安全的。
 部分内容参考自 ：https://blog.csdn.net/feiyanaffection/article/details/81394745