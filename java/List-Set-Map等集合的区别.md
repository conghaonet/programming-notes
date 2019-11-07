# List-Set-Map等集合的区别.md
* List , Set, Map都是接口，前两个继承至Collection接口，Map为独立接口
* Set下有HashSet，LinkedHashSet，TreeSet
* List下有ArrayList，Vector，LinkedList
* Map下有Hashtable，LinkedHashMap，HashMap，TreeMap
* Collection接口下还有个Queue接口，有PriorityQueue类

## Connection接口
### List 有序,可重复
* ArrayList
  * 优点: 底层数据结构是数组，查询快，增删慢。
  * 缺点: 线程不安全，效率高
* Vector
  * 优点: 底层数据结构是数组，查询快，增删慢。
  * 缺点: 线程安全，效率低
* LinkedList
  * 优点: 底层数据结构是链表，查询慢，增删快。
  * 缺点: 线程不安全，效率高

### Set 无序,唯一
* HashSet
  * 底层数据结构是哈希表。(无序,唯一)
  * 如何来保证元素唯一性?
    1. 依赖两个方法：hashCode()和equals()
* LinkedHashSet
  * 底层数据结构是链表和哈希表。(FIFO插入有序,唯一)
    1. 由链表保证元素有序
    2. 由哈希表保证元素唯一
* TreeSet
  * 底层数据结构是红黑树。(唯一，有序)
    1. 如何保证元素排序的呢?
        * 自然排序
        * 比较器排序
    2. 如何保证元素唯一性的呢?
        * 根据比较的返回值是否是0来决定

### Map接口
Map接口有三个比较重要的实现类，分别是HashMap、TreeMap和HashTable。
* TreeMap是有序的，HashMap和HashTable是无序的。
* Hashtable的方法是同步的，HashMap的方法不是同步的。这是两者最主要的区别。

这就意味着:
* Hashtable是线程安全的，HashMap不是线程安全的。
* HashMap效率较高，Hashtable效率较低。
如果对同步性或与遗留代码的兼容性没有任何要求，建议使用HashMap。 查看Hashtable的源代码就可以发现，除构造函数外，Hashtable的所有 public 方法声明中都有 synchronized关键字，而HashMap的源码中则没有。
* Hashtable不允许null值，HashMap允许null值（key和value都允许）
* 父类不同：Hashtable的父类是Dictionary，HashMap的父类是AbstractMap
