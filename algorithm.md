# 一、数据结构

数组

链表

- 单向链表：表头指针，每个node节点包括数据和下一个节点的地址
- 双向链表：表头指针，表尾指针，每个node节点都包括数据、下一个节点的地址和上一个节点的地址

散列表（哈希表）

- 原理：地址和哈希值维护一个映射表，便于增删改查。哈希值一般是key的哈希，但没有key的时候，value自身也可以哈希。因为哈希碰撞的原因，增删改查的效率都是 O(1)<O(hash)<O(n).
- 新建：新建一个数组，数组长度固定。
- 插入：哈希值取余数组长度，将数据插入到对应的序号中。如果取余结果重复（哈希碰撞），解决方法：链式地址法和线性探测法。

- 查询：通过hash值直接找到地址，进而取值（key-value形式有意义，单value形式用作判定是否存在）。因为哈希碰撞原因，可能会得到多个，需要遍历这几个。
- 删除：也是通过hash值直接找到地址。
- 遍历：单value哈希表的主要使用方式。

树

- 原理
- 插入
- 删除
- 查询
- 遍历



栈：后进先出，栈是限定仅在表尾（栈顶，表头称作栈底）进行插入和删除操作的线性表。实现方式：如单项链表，数组

- 数组实现：一个只能增删表尾的限制数组
- 链表实现：一个反着的链表（表尾指针，数据，上一个节点的地址）

队列：先进后出，在一端添加元素（入队），在另一端取出元素（出队）。实现方式，如双向链表

- 数组实现
  - 开辟一个定容的数组，并定义头、尾两个指针。
  - 增：向前移动一格头部指针，当到达终点后循环移动，当尾指针追上头指针时，队列已满，需要扩容。
  - 删：向前移动一格尾部指针，当到达终点后循环移动，当头指针追上尾指针时，队列为空

- 链表实现：双向链表





# 二、java

底层实现仍然是使用数组，链表，哈希表，红黑树，但是暴漏给使用者是按功能划分：set，list，map

## 1、基础

- *Collection*    

  - set 

    - HashSet 无序的，不可重复的。底层：HashMap的封装（key和value是同一个），HashMap<E,Object> map

    - LinkHashSet  有序的，不可重复的。底层：哈希+双向链表，按照哈希的方式存储，每个节点额外存储before和after属性。遍历时不按照数组的顺序遍历，而是根据before和after属性寻找下一个节点，所以变得有序。

    - TreeSet 

  - *list*

    - ArrayList 读快改慢

    - LinkedList 双向链表
    - *Vector*
      - Stack：栈

  - *Queue*
    - Deque
      - LinkedList  双向链表

- *Map*

  - HashMap 无序的(不是插入顺序)、不可重复的(key不能重复)。实现方式：key-value的Set集合，Set<Map.Entry<K,V>> entrySet
    - LinkHashMap  有序的，不可重复的
  - TreeMap：底层红黑树，按照大小排序，所以在需要排序的情况下使用。





## 2、增删查改

| 操作 | set                                      | list                                             | Map                                                          | Stack(Stack是实现类) | Queue(Queue是接口，实现类是LinkedList)                       |
| ---- | ---------------------------------------- | ------------------------------------------------ | ------------------------------------------------------------ | -------------------- | ------------------------------------------------------------ |
| 增   | boolean add(E e)                         | boolean  add(E e)  void  add(int index, E e)     | Object  put(Object key, Object value)  如果key已存在，则返回被替换掉的元素  void  putAll(Map t); | E  push(E item);     | boolean  add(E e);  **boolean offer(E e);**队列为空返回null  在队头插入 |
| 删   | boolean  remove(Object o)  void  clear() | boolean  remove(Object o)  void  clear()         | Object  remove(Object key);  void  clear()                   | E  pop();            | E remove();   **E poll();** 队列为空返回null  获取并删除队头 |
| 查   | boolean  contains(Object o)              | E  get(int index)  boolean  contains(Object o)   | Object  get(Object key);  boolean  containsKey(Object key);  boolean  containsValue(Object value); | E  peek();           | E  element();  **E peek()**; 队列为空返回null  获取但不移除队头 |
| 改   |                                          | E  set(int index, E element)  返回被替换掉的元素 | Object  put(Object key, Object value)  如果key已存在，则返回被替换掉的元素 |                      |                                                              |
| 判空 | boolean  isEmpty()                       | boolean  isEmpty()                               | boolean  isEmpty()                                           | boolean  empty()     | boolean  isEmpty()                                           |
| 大小 | int  size();                             | int  size();                                     | int  size();                                                 |                      |                                                              |
| 遍历 | 一样                                     | 一样                                             | 一样                                                         | 一样                 | 一样                                                         |



## 3、遍历Iterator

```java
public void fun1(Collection c) {
    Iterator i = c.iterator();       //1、定义迭代器，初始位置为-1
    while (i.hasNext()){          //2.（p+1） == null 判断有没有下一个
    if(i.next().equals("aaa")){   //3.p++并返回*p，先++指向下一个，然后在解引用
        i.remove();             //4.删除
      }
    }                          //总结：核心原因是初始为-1，如此循环的思想就是需要的时候再给
}
```

- 当程序中有两个i.next()时,会有两次++，程序每一循环都是+2，后一个就和前一个不一样，后一个也需要先判断是否越界，i.hasNext()，所以只是为了*p的使用，那么先要给出Object o = i.next(); 然后就可以反复使用o，而不用担心多次++



## 4、常用算法Collections  

Collections中的算法只对List实现，因为List是有序的数据结构，且都是静态方法。





















































