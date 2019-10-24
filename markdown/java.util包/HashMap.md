HashMap中的取余操作：(n - 1) & hash

详解：<https://www.cnblogs.com/gne-hwz/p/10060260.html>

hashmap主要内容讲解,不过还差个红黑树:[https://jayfeng.com/2016/12/28/%E7%90%86%E8%A7%A3HashMap/](https://jayfeng.com/2016/12/28/理解HashMap/)



1. 自己动手画一个流程图出来,新增key,删除key,更新key,获取key
2. 原理解释,获取key的原理解释,node数组,红黑树原理
3. 一些配套的方法,rehash,resize之类的方法
4. java8之后新增的方法,compute,merge之类的方法
5. 与hashtable的区别,与linkedhashmap的区别,与ConcurrentHashMap的区别,不过在说出它们之间的区别的之前要先写关于hashtable和ConcurrentHashMap的博客



1. HashMap继承了AbstractMap抽象了,而AbstractMap实现了Map接口;
2. Map接口里面包含一个接口Entry,这个接口常用语EntrySet,用来遍历map;另外也作为Map的存储单元,因为HashMap中的Node数组就是继承了Entry接口
3. Node类,HashMap中使用Node类来存储键值对
4. put的原理是这样的,HashMap中使用Node数组来存储键值对,通过对key值hash再取余得到该key的数组下标,然后将这个key所对应的value存储到对应的数组位置上,
   若对应的数组位置为null,则直接插入;
   若对应的数组位置不为null并且是一个Node节点,则将value链接到链表的最后一个位置上,若此时链表的个数已经超过了所规定的最大值(默认为8),则将链表转化为红黑树;
   若对应的数组位置不为null并且是一个红黑树节点,则将value插入树中
5. get的原理是这样的,通过对传进来的key值做hash得到一个hash值,再将这个hash值取余等到数组的Node数组的下标,这个下标就是value值所在的位置
   先比对第一个元素是否是要找的元素,具体过程是先比对hash值,再比对当前元素的key值与传进来的key是否一致,若一致则认为是同一个元素,直接将当前元素返回
   若第一个元素不是要找的元素,判断当前元素是否是树节点,若是则通过红黑树的查找算法找出value值(红黑树的查找算法后续再介绍)
   若第一个元素不是要找的元素,判断当前元素是否是树节点,若不是则当前节点是链表节点,直接遍历链表节点即可.
6. HashMap的扩容机制
7. HashMap的hash算法
8. HashMap的取余算法

