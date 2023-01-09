# Java 集合框架



**集合也叫容器 ** 

Java 集合， 也叫作容器，主要是由两大接口派生而来：一个是 `Collecton`接口，主要用于存放单一元素；另一个是 `Map` 接口，主要用于存放键值对 。对于`Collection` 接口，下面又有三个主要的子接口：`List`、`Set` 和  ` Queue ` 



![img](https://i.loli.net/2021/08/30/tLZY4RQfz3barCl.png)



**List , Set , Queue , Map 四者的区别 **

+ `List`:   存储的元素是有序的、可重复的。
+ `Set`:     存储的元素是无序的、不可重复的。
+ `Queue`: 按特定的排队规则来确定先后顺序，存储的元素是有序的、可重复的。
+ `Map`:    使用键值对（key-value）存储



## **集合框架底层数据结构**

**Collection接口下**

 

**List** 

+ ArrayList：基于动态数组实现，支持随机访问   

+ Vector：和 ArrayList 类似，但它是线程安全的  

+ `LinkedList`：双向链表(JDK1.6 之前为循环链表，JDK1.7 取消了循环)  LinkedList 还可以用作栈 、队列和 双向队列  



**Set** 

+ HashSet：基于哈希表实现，支持快速查找，元素是无序的 
+ LinkedHashSet：具有 HashSet 的查找效率，且内部使用双向链表维护元素的插入顺序  
+ TreeSet：基于红黑树实现，支持有序性操作，例如：根据一个范围查找元素的操作。  

floorEntry        找到第一个小于或等于指定key的Map.Entry

ceilingEntry()  大于或等于给定键元素(ele)的最小键元素链接的键值对

**Queue** 

+ `PriorityQueue`: `Object[]` 数组实现的二叉堆
+ `ArrayQueue`: `Object[]` 数组 + 双指针  
+ `LinkedList`：可以用它来实现双向队列 



**Map **  

+ `HashMap`： JDK1.8 之前 `HashMap` 由数组+链表组成的，数组是 `HashMap` 的主体，链表则是主要为了解决哈希冲突而存在的（"拉链法" 解决冲突）。JDK1.8 以后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为 8）（将链表转换成红黑树前会判断，如果当前数组的长度小于 64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间

+ LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。**LinkedHashMap**  可以按插入有序(购物车) ，后续修改不会改变位置；也可以实现按访问有序，用来实现 **LRU**

+ `Hashtable`： 数组 + 链表组成的，数组是 `Hashtable` 的主体，链表则是主要为了解决哈希冲突而存在的

+ `TreeMap`： 红黑树（自平衡的排序二叉树） 



**如何选用集合** 

主要根据集合的特点来选用，比如我们需要根据键值获取到元素值时就选用 `Map` 接口下的集合，需要排序时选择 `TreeMap`，不需要排序时就选择 `HashMap`,需要保证线程安全就选用 `ConcurrentHashMap`。

当我们只需要存放元素值时，就选择实现`Collection` 接口的集合，需要保证元素唯一时选择实现 `Set` 接口的集合比如 `TreeSet` 或 `HashSet`，不需要就选择实现 `List` 接口的比如 `ArrayList` 或 `LinkedList`，然后再根据实现这些接口的集合的特点来选用。



**为什么要使用集合** 

当我们需要保存一组类型相同的数据的时候我们可以数组，数组是定长的，但是我们实际开发中存储的数据的类型是多种多样的，而且个数也不固定，所以需要使用集合(容器)，集合可以存储不同类型的对象(声明为Object)，还可以动态扩容。



## Collection 子接口之 List  

**ArrayList  和 Vector 的区别**  

+ `ArrayList` 是 `List` 的主要实现类，底层使用 `Object[ ]`存储，适用于频繁的查找工作，线程不安全 ；
+ `Vector` 是 `List` 的古老实现类，底层使用`Object[ ]` 存储，线程安全的。



**ArrayList 与 LinkedList 区别** 

1. **是否保证线程安全：** `ArrayList` 和 `LinkedList` 都不保证线程安全；
2. **底层数据结构：** `Arraylist` 底层使用的是 **`Object` 数组**；`LinkedList` 底层使用的是 **双向链表** 数据结构
3. **插入和删除是否受元素位置的影响：** 
   + ArrayList     尾插O(1)，     平均时间复杂度 O(n)
   + LinkedList   头尾插 O(1)    平均时间复杂度 O(n)
4. **是否支持快速随机访问**  `LinkedList` 不支持高效的随机元素访问，`ArrayList` 支持
5. **内存空间占用**   ArrayList 的空 间浪费主要体现在在 List 列表的结尾会预留一定的容量空间，而 LinkedList 的空间花费则体现在它的每一个元素要多存前驱和后继元素

**RandomAccess 接口有何作用**  

+ 标识实现这个接口的类具有随机访问功能，**只起标识作用**



**ArrayList 的扩容机制** 

**int newCapacity = oldCapacity + (oldCapacity >> 1)，所以 ArrayList 每次扩容之后容量都会变为原来的 1.5 倍左右（oldCapacity 为偶数就是 1.5 倍，否则是 1.5 倍左右）** 奇偶不同，比如 ：10+10/2 = 15, 33+33/2=49。如果是奇数的话会丢掉小数 

- 当我们要 add 进第 1 个元素到 ArrayList 时，elementData.length 为 0 （因为还是一个空的 list），因为执行了 `ensureCapacityInternal()` 方法 ，所以 minCapacity 此时为 10。此时，`minCapacity - elementData.length > 0`成立，所以会进入 `grow(minCapacity)` 方法 ，elementData.length 变为10；
- 当 add 第 2 个元素时，minCapacity 为  2  (size +1)，此时 elementData.length(容量) 在添加第一个元素后扩容成 10 了。此时，`minCapacity - elementData.length > 0` 不成立，所以不会进入 （执行）`grow(minCapacity)` 方法 
- 添加第 3、4 ··· 到第 10 个元素时，依然不会执行 grow 方法，数组容量都为 10。

+ 直到添加第 11 个元素，minCapacity(为 11) 比 elementData.length（为 10）要大。进入 grow 方法进行扩容。





## Collection 子接口之 Set 

**comparable 和 Comparator 的区别**

+ `comparable` 接口实际上是出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序
+ `comparator`接口实际上是出自 java.util 包它有一个`compare(Object obj1, Object obj2)`方法用来排序 



**Set  存储无序性和不可重复性的含义是什么**

+ 无序性         指存储的数据在底层数组中的位置是根据数据的哈希值决定的

+ 不可重复性  保证添加的元素按照equals() 判断时，不能返回 true 




**HashSet、LinkedHashSet 和 TreeSet 三者的异同**

+ `HashSet` 是 `Set` 接口的主要实现类 ，`HashSet` 的底层是 `HashMap`，线程不安全的
+ `LinkedHashSet` 是 `HashSet` 的子类，能够按照添加的顺序遍历
+ `TreeSet` 底层使用红黑树，能够按照添加元素的顺序进行遍历，排序的方式有自然排序和定制排序。 **不支持存储 `NULL` 和 `non-comparable` 的对象 **





**HashSet 的实现原理** 

HashSet 的实现是依赖于 HashMap 的， HashSet 的值都是存储在 HashMap 中的。在 HashSet 的构造法中会初
始化一个 HashMap 对象， HashSet 不允许值重复。因此， HashSet 的值是作为 HashMap 的 key 存储在
HashMap 中的，当存储的值已经存在时返回 **false** 





**HashSet 怎么保证元素不重复的 **

HashMap 的 key 是不能重复的，而HashSet 的元素作为了 map 的 key，所以不会重复。



## Collection 子接口之 Queue

**Queue 与 Deque 的区别**

+ `Queue` 是单端队列，只能从一端插入元素，另一端删除元素，实现上一般遵循 **先进先出（FIFO）** 规则。
+ `Queue` 扩展了 `Collection` 的接口，根据 **因为容量问题而导致操作失败后处理方式的不同** 可以分为两类方法: 一种在操作失败后会抛出异常，另一种则会返回特殊值。

| `Queue` 接口 | 抛出异常  | 返回特殊值 |
| ------------ | --------- | ---------- |
| 插入队尾     | add(E e)  | offer(E e) |
| 删除队首     | remove()  | poll()     |
| 查询队首元素 | element() | peek()     |

+ `Deque` 是双端队列，在队列的**两端**均可以插入或删除元素。 
+ `Deque` 扩展了 `Queue` 的接口, 增加了在队首和队尾进行插入和删除的方法，同样根据失败后处理方式的不同分为两类：  

| `Deque` 接口 | 抛出异常      | 返回特殊值      |
| ------------ | ------------- | --------------- |
| 插入队首     | addFirst(E e) | offerFirst(E e) |
| 插入队尾     | addLast(E e)  | offerLast(E e)  |
| 删除队首     | removeFirst() | pollFirst()     |
| 删除队尾     | removeLast()  | pollLast()      |
| 查询队首元素 | getFirst()    | peekFirst()     |
| 查询队尾元素 | getLast()     | peekLast()      |

记忆窍门：强制的直接报错，比较委婉的返回 null 

+ `Deque` 还提供有 `push()` 和 `pop()` 等其他方法，可用于模拟**栈**。



**ArrayDeque 与 LinkedList 的区别**

`ArrayDeque` 和 `LinkedList` 都实现了 `Deque` 接口

+ `ArrayDeque` 和 `LinkedList` 都实现了 `Deque` 接口
+ `ArrayDeque` **不支持存储 `NULL` 数据**，但 `LinkedList` 支持  
+ `ArrayDeque` 插入时可能存在扩容过程，均摊后的插入操作依然为 O(1)，`LinkedList` 不需要扩容，每次插入数据时均需要申请新的堆空间，均摊性能相比更慢。 

**`ArrayDeque` 来实现队列要比 `LinkedList` 更好。此外，`ArrayDeque` 也可以用于实现栈。**



**PriorityQueue**

+ `PriorityQueue` 利用了二叉堆的数据结构来实现的，底层使用可变长的数组来存储数据
+ `PriorityQueue` 通过堆元素的上浮和下沉，实现了在 O(logn) 的时间复杂度内插入元素和删除堆顶元素。
+ `PriorityQueue` 是非线程安全的，且**不支持存储 `NULL` 和 `non-comparable` 的对象**。
+ `PriorityQueue` 默认是小顶堆，但可以接收一个 `Comparator` 作为构造参数，从而来自定义元素优先级的先后。



## Map 接口

**HashMap 和 Hashtable 的区别** 

+ **线程是否安全：**`HashMap` 是非线程安全的，`HashTable` 是线程安全的,因为 `HashTable` 内部的方法基本都经过`synchronized` 修饰 

+ **对NULL的支持** `HashMap` 可以存储 null 的 key 和 value，但 null 作为键只能有一个，null 作为值可以有多个HashTable 不允许有 null 键和 null 值

+ **效率 ** 

+ **初始容量大小和每次扩充容量大小的不同**  

  ① 创建时如果不指定容量初始值，`Hashtable` 默认的初始大小为 11，之后每次扩充，容量变为原来的 2n+1。`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍

  ② 创建时如果给定了容量初始值，那么 Hashtable 会直接使用你给定的大小，而 `HashMap` 会将其扩充为 2 的幂次方大小（`HashMap` 中的`tableSizeFor()`方法保证 ) 

+ **底层数据结构** Hashtable 永远是 数组+链表



**HashMap 和 HashSet 区别** 

+ `HashSet` 底层就是基于 `HashMap` 实现的，一个实现了Map接口且存储键值对，一个实现了Set接口且存储对象。



**HashMap 和 TreeMap 区别** 

+ **相比于`HashMap`来说 `TreeMap` 主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力 **



**HashSet 如何检查重复**  

+ 当把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals() 方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。 
+ 无论`HashSet`中是否已经存在了某元素，`HashSet`都会直接插入，只是会在`add()`方法的返回值处告诉我们插入前是否存在相同元素。有重复返回false，没重复返回null 



**如何遍历HashMap**

1. 使用迭代器（Iterator）EntrySet 的方式进行遍历；

2. 使用迭代器（Iterator）KeySet 的方式进行遍历；

   

# **HashMap ** 

HashMap 的实现原理理/底层数据结构 ?  JDK1.7  和  JDK1.8  

+ JDK1.7： Entry数组 + 链表  
+ JDK1.8： Node 数组 + 链表/红黑树，当链表上的元素个数超过 8 个并且数组长度 >= 64 时自动转化成红黑树，节点变成树节点，以提高搜索效率和插入效率到 O(logN) 。 Entry 和 Node 都包含 key、 value、 hash、 next 属性。
+ 小于等于临界值6个，则退化成链表

 

HashMap 的 put 方法的执行过程  

+ ![img](G:\markdown图片\8JZqinjDdUoMblF.png)



HashMap 的 get 方法的执行过程 

+ ![image-20210824230302280](G:\markdown图片\GZjO5SJqhDUFmvw.png)

  




HashMap 的 resize 方法的执行过程 

1. 第一次调⽤用 HashMap 的 put 方法时，会调用 resize 方法对 table 数组进行初始化，如果不传入指定值，默认大小为 16  
2. 扩容时会调用 resize，即 size > threshold 时， table 数组大小翻倍  

每次扩容之后容量都是翻倍，扩容后要将原数组中的所有元素找到在新数组中合适的位置  

当我们把 table[i] 位置的所有 Node 迁移到 newtab 中去的时候：这里面的 node 要么在 newtab 的 i 位置(不变)，要么在 newtab 的 i + n 位置。也就是我们可以这样处理：把 table[i] 这个桶中的 node 拆分为两个链表 l1和 l2：如果 hash & n == 0，那么当前这个 node 被连接到 l1 链表；否则连接到 l2 链表。这样下来，当遍历完table[i] 处的所有 node 的时候，我们得到两个链表 l1 和 l2，这时我们令 newtab[i] = l1， newtab[i + n] = l2，这就完成了了 table[i] 位置所有 node 的迁移(rehash)，这也是 HashMap 中容量一定的是 2 的整数次幂带来的方便之处。  **n为扩容前的容量 ** 



HashMap 的 size 为什么必须是 2 的整数次方  

1. 这样做总是能够保证 HashMap 的底层数组长度为 2 的 n 次方。当 length 为 2 的 n 次方时， h & ( length - 1)
   就相当于对 length 取模，而且速度比直接取模快得多，这是 HashMap 在速度上的一个优化 。而且每次扩容
   时都是翻倍  
   
2. 如果 length 为 2 的次幂，则 length - 1 转化为二进制必定是 11111……的形式，在与 h 的二进制进行与操作
   时效率会非常的快，而且空间不浪费。  不为2的n次幂，会导致增加碰撞的几率，减慢了查询的效率 。  
   
   

HashMap 线程安全

- 在jdk1.7中，在多线程环境下，扩容时会造成环形链或数据丢失 

- 在jdk1.8中，在多线程环境下，会发生数据覆盖的情况 

  

HashMap 的 get 方法能否判断某个元素是否在 map 中    

+ HashMap 的 get 函数的返回值不能判断一个 key 是否包含在 map 中，因为 get 返回 null 有可能是不包含该
  key，也有可能该 key 对应的 value 为 null。因为 HashMap 中允许 key 为 null，也允许 value 为 null 





# ConcurrentHashMap 



**ConcurrentHashMap 和 Hashtable 的区别**

`ConcurrentHashMap` 和 `Hashtable` 的区别主要体现在实现线程安全的方式上不同 

+ **底层数据结构：** JDK1.7 的 `ConcurrentHashMap` 底层采用 **分段的数组+链表** 实现，JDK1.8 采用的数据结构跟 `HashMap1.8` 的结构一样，数组+链表/红黑二叉树；Hashtable 是 数组+链表

+ **实现线程安全的方式（重要）**

  ① **在 JDK1.7 的时候，`ConcurrentHashMap`（分段锁）** 对整个桶数组进行了分割分段(`Segment`)，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。 **到了 JDK1.8 的时候已经摒弃了 `Segment` 的概念，而是直接用 `Node` 数组+链表+红黑树的数据结构来实现，并发控制使用 `synchronized` 和 CAS 来操作。**

  ② **`Hashtable`(同一把锁)** :使用 `synchronized` 来保证线程安全，效率非常低下。

JDK1.8 的 `ConcurrentHashMap` 不再是 **Segment 数组 + HashEntry 数组 + 链表**，而是 **Node 数组 + 链表 / 红黑树**。不过，Node 只能用于链表的情况，红黑树的情况需要使用 **`TreeNode`**。当冲突链表达到一定长度时，链表会转换成红黑树。



**ConcurrentHashMap 线程安全的具体实现方式/底层具体实现** 

**1.7** 

数据分为一段一段的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。  

**`ConcurrentHashMap` 是由 `Segment` 数组结构和 `HashEntry` 数组结构组成**。

Segment 实现了 `ReentrantLock`，所以 `Segment` 是一种可重入锁，扮演锁的角色。`HashEntry` 用于存储键值对数据。

一个 `ConcurrentHashMap` 里包含一个 `Segment` 数组 。`Segment` 的结构和 `HashMap` 类似，是一种数组和链表结构，一个 `Segment` 包含一个 `HashEntry` 数组，每个 `HashEntry` 是一个链表结构的元素，每个 `Segment`  守护着一个 `HashEntry` 数组里的元素，当对 `HashEntry` 数组的数据进行修改时，必须首先获得对应的 `Segment` 的锁 。

![JDK1.7的ConcurrentHashMap](https://i.loli.net/2021/08/30/k4eon7ga1suWQlb.png)



**1.8**

`ConcurrentHashMap` 取消了 `Segment` 分段锁，采用 CAS 和 `synchronized` 来保证并发安全。数据结构跟 HashMap1.8 的结构类似，数组+链表/红黑二叉树。Java 8 在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为 O(N)）转换为红黑树（寻址时间复杂度为 O(log(N))） 



`synchronized` 只锁定当前链表或红黑二叉树的首节点，只要 hash 不冲突，效率就很高

![Java8 ConcurrentHashMap 存储结构（图片来自 javadoop）](https://i.loli.net/2021/08/30/TN3XOuvZeoIg7lb.png)





## LinkedHashMap 的原理    

LinkedHashMap 是基于HashMap实现的，是HashMap子类，重写了recordAccess，recordRemoval，父类中为空

**1.7** 

private final boolean accessOrder   可以指定按插入或访问有序，默认按插入顺序

private transient Entry<K,V> header  header节点表示双向链表的头，他是单独存在的，类型是Entry，他继承了HashMap的Entry，并且增加了 before，after 指向链表的前驱和后继   有两个核心方法，recordAccess，recordRemoval  



按访问顺序 

+ put 方法  key已经存在，会调整节点到末尾。 如果是新键，则会加入链表尾也加入哈希表，删除最久未访问的数据
+ put 在HashMap中操作后会调用 recordAccess，remove  会调用 recordRemoval，这两个方法在HashMap中为空



# Collections 工具类

1. 排序
2. 查找，替换操作
3. 同步控制





## 如何对一个集合排序 

+ Collections.sort(List list)   

  自然排序，参与排序的对象需实现comparable接口，重写其compareTo()方法，方法体中实现对象的比较大小规则

+ Collections.sort (List list,Comparator c)  

  定制排序，需编写匿名内部类，先new一个Comparator接口的比较器对象c，同时实现compare()其方法
  





# 集合注意事项



### Arrays.asList() 使用

`Arrays.asList()`将数组转换为集合后，底层其实还是数组 (这个ArrayList  是Arrays的内部类，并不是我们常用的ArrayList，无法进行 add 等操作  )，传入的数组必须是**对象**数组  **此方法作为基于数组和基于集合的API之间的桥梁**

### 如何正确的将数组转换为ArrayList  

Arrays.asList() 作为桥梁 

```java
String[] myArray = {"Apple", "Banana", "Orange"};
List<String> myList = Arrays.asList(myArray);

List list = new ArrayList<>(Arrays.asList("a", "b", "c"))
```



### Collections.toArray()  如何反转数组

Arrays.asList()将数组转化成List，然后使用 Collections.reverse(list) ，后使用 .toArray(xxx)，转换成数组 

xxx未指定返回的是Object数组，xxx为空会报错   



### 不要在 foreach 循环里进行元素的  remove / add 操作

+ 除非通过迭代器自身`remove/add`方法，否则会产生 **fail-fast 机制**。

+ 如果并发操作，需要对 `Iterator` 对象加锁

# ————————————

## Collection 和 Collections 有什么区别  

+ Collection：是最基本的集合接口，一个 Collection 代表一组 Object，即 Collection 的元素。它的直接继承接口有List， Set 和 Queue   
+ Collections：是不属于 Java 的集合框架的，它是集合类的一个工具类/帮助类。此类不能被实例例化， 服务于 Java的 Collection 框架。它包含有关集合操作的静态多态方法，实现对各种集合的搜索、排序、线程安全(装饰器模式)等操作。  







## 迭代器

集合实现Iterable接口，代表可迭代，并实现 iterator() 方法，返回一个实现了Iterator接口的对象，他有hasNext，next，remove方法，可以用它来遍历数据结构和修改数据结构。**迭代器表示的是一种关注点分离的思想，将数据结构与数据迭代遍历相分离。 **  



**Iterator 和 ListIterator 有什么区别**    

Iterator 可用来遍历 Set 和 List 集合，但是 ListIterator 只能用来遍历 List。 Iterator 对集合只能是前向遍历，
ListIterator 既可以前向也可以后向。ListIterator 实现了 Iterator 接口，并包含其他的功能，比如：增加元素，替
换元素(set)，获取前一个和后一个元素的索引等 。



## fail-fast  

**在迭代时，如果发现 该集合数据 结构被改变 (`modCount != expectedModCount`)，就会 抛出 `ConcurrentModificationException`**



**fail-fast 与 fail-safe 有什什么区别**

`java.util` 包下的 属于 `fail-fast` , **快速失败**~ 😝

`java.util.concurrent` 包下的 属于 `fail-safe` ，**安全失败**~ 😝

`java.util` 包下的属于`fail-fast` ， 它的特点是 不允许并发修改，如果并发修改的话会导致在迭代过程中抛出 `ConcurrentModificationException`***  ，触发点是  `modCount != expectedModCount`😝

`java.util.concurrent` 包下的 属于  `fail-safe`   ， 它的特点是 **允许并发修改，但是 无法保证在迭代过程中获取到最新的值 。** 😋 而且 `concurrentHashMap`  修改数据时 是直接通过 **UNSAFE** 类 直接**从主内存中获取数据，或者直接更新数据到主内存，remove 底层就是把当前位置设置为next ~** ，  `CopyOnWriteArrayList`  或者 `CopyOnWriteArraySet`  ，就直接 **复制原来的集合，然后在复制出来的集合上进行操作** ，是不同的操作~