## 数据结构



### 数据结构概要

该页面只讲解它是什么，更细节的东西，请点击详情。

- Java集合框架的基本接口/类层次结构：

```java
java.util.Collection [I]
    +--java.util.List [I]
       +--java.util.ArrayList [C]    
       +--java.util.LinkedList [C]  
       +--java.util.Vector [C]    //线程安全
          +--java.util.Stack [C]  //线程安全
    +--java.util.Set [I]                   
       +--java.util.HashSet [C]      
       +--java.util.SortedSet [I]    
          +--java.util.TreeSet [C]    
    +--Java.util.Queue[I]
        +--java.util.Deque[I]   
        +--java.util.PriorityQueue[C]  
        
java.util.Map [I]
    +--java.util.SortedMap [I]
       +--java.util.TreeMap [C]
    +--java.util.Hashtable [C]   //线程安全
    +--java.util.HashMap [C]
    +--java.util.LinkedHashMap [C]
    +--java.util.WeakHashMap [C]

[I]：接口
[C]：类
```

> 数组 List

数组是相同数据类型的元素按一定顺序排列的集合，是一块连续的内存空间。

- 优点：get和set操作时间上都是O(1)的；
- 缺点：add和remove操作时间上都是O(N)的。

Java中，Array就是数组，此外，ArrayList使用了数组Array作为其实现基础,它和一般的Array相比，最大的好处是，我们在添加元素时不必考虑越界，元素超出数组容量时，它会自动扩张保证容量。

Vector和ArrayList相比，主要差别就在于多了一个线程安全性，但是效率比较低下。如今java.util.concurrent包提供了许多线程安全的集合类（比如 LinkedBlockingQueue），所以不必再使用Vector了。

```java
int[] ints = new int[10];
ints[0] = 5;//set
int a = ints[2];//get
int len = ints.length;//数组长度
```

> 链表

链表是一种非连续、非顺序的结构，数据元素的逻辑顺序是通过链表中的指针链接次序实现的，链表由一系列结点组成。

- 优点：add和remove操作时间上都是O(1)的；
- 缺点：get和set操作时间上都是O(N)的，而且需要额外的空间存储指向其他数据地址的项。

查找操作对于未排序的数组和链表时间上都是O(N)。

Java中，LinkedList 使用链表作为其基础实现。

```java
LinkedList<String> linkedList = new LinkedList<>();
linkedList.add("addd");//add
linkedList.set(0,"s");//set，必须先保证 linkedList中已经有第0个元素
String s =  linkedList.get(0);//get
linkedList.contains("s");//查找
linkedList.remove("s");//删除
//以上方法也适用于ArrayList
```

> 队列

队列是一种特殊的线性表，特殊之处在于它只允许在表的前端进行删除操作，而在表的后端进行插入操作，亦即所谓的先进先出（FIFO）。

Java中，LinkedList实现了Deque，可以做为双向队列（自然也可以用作单向队列）。另外PriorityQueue实现了带优先级的队列，亦即队列的每一个元素都有优先级，且元素按照优先级排序。

```java
Deque<Integer> integerDeque = new LinkedList<>();
// 尾部入队，区别在于如果失败了
// add方法会抛出一个IllegalStateException异常，而offer方法返回false
integerDeque.offer(122);
integerDeque.add(122);
// 头部出队,区别在于如果失败了
// remove方法抛出一个NoSuchElementException异常，而poll方法返回false
int head = integerDeque.poll();//返回第一个元素，并在队列中删除
head = integerDeque.remove();//返回第一个元素，并在队列中删除
// 头部出队，区别在于如果失败了
// element方法抛出一个NoSuchElementException异常，而peek方法返回null。
head = integerDeque.peek();//返回第一个元素，不删除
head = integerDeque.element();//返回第一个元素，不删除
```

> 栈

栈（stack）又名堆栈，它是一种运算受限的线性表。其限制是仅允许在表的一端进行插入和删除运算。这一端被称为栈顶，相对地，把另一端称为栈底。它体现了后进先出（LIFO）
的特点。

Java中，Stack实现了这种特性，但是Stack也继承了Vector，所以具有线程安全线和效率低下两个特性，最新的JDK8中，推荐用Deque来实现栈，比如：

```java
Deque<Integer> stack = new ArrayDeque<Integer>();
stack.push(12);//尾部入栈
stack.push(16);//尾部入栈
int tail = stack.pop();//尾部出栈，并删除该元素
tail = stack.peek();//尾部出栈，不删除该元素
```

> 集合

集合是指具有某种特定性质的具体的或抽象的对象汇总成的集体，这些对象称为该集合的元素，其主要特性是元素不可重复。

- HashSet 体现了这种数据结构，而HashSet是在MashMap的基础上构建的。
- LinkedHashSet继承了HashSet，使用HashCode确定在集合中的位置，使用链表的方式确定位置，所以有顺序。
- TreeSet实现了SortedSet 接口，是排好序的集合（在TreeMap 基础之上构建），因此查找操作比普通的Hashset要快（log(N)）；插入操作要慢（log（N））,因为要维护有序。

```java
HashSet<Integer> integerHashSet = new HashSet<>();
integerHashSet.add(12121);//添加
integerHashSet.contains(121);//是否包含
integerHashSet.size();//集合大小
integerHashSet.isEmpty();//是否为空
```

> 散列表

散列表也叫哈希表，是根据关键键值(Keyvalue)进行访问的数据结构，它通过把关键码值映射到表中一个位置来访问记录，以加快查找的速度，这个映射函数叫做散列函数。

Java中

- HashMap实现了散列表，而Hashtable比它多了一个线程安全性，但是由于使用了全局锁导致其性能较低，所以现在一般用ConcurrentHashMap.
- ConcurrentHashMap来实现线程安全的HashMap（类似的，以上的数据结构在最新的java.util.concurrent的包中几乎都有对应的高性能的线程安全的类）。
- TreeMap实现SortMap接口，能够把它保存的记录按照键排序。
- LinkedHashMap保留了元素插入的顺序。
- WeakHashMap是一种改进的HashMap，它对key实行“弱引用”，如果一个key不再被外部所引用，那么该key可以被GC回收，而不需要我们手动删除。

```java
HashMap<Integer,String> hashMap = new HashMap<>();
hashMap.put(1,"asdsa");//添加
hashMap.get(1);//获得
hashMap.size();//元素个数
```

> 树

树（tree）是包含n（n>0）个节点的有穷集合，其中：

- 每个元素称为节点（node）；
- 有一个特定的节点被称为根节点或树根（root）。
- 除根节点之外的其余数据元素被分为m（m≥0）个互不相交的结合T1，T2，……Tm-1，其中每一个集合Ti（1<=i<=m）本身也是一棵树，被称作原树的子树（subtree）。

树这种数据结构在计算机世界中有广泛的应用，比如操作系统中用到了红黑树，数据库用到了B+树，编译器中的语法树，内存管理用到了堆（本质上也是树），信息论中的哈夫曼编码等等等等，在Java中TreeSet和TreeMap用到了树来排序（二分查找提高检索速度），不过一般都需要程序员自己去定义一个树的类，并实现相关性质，而没有现成的API。下面就用Java来实现各种常见的树。

> 二叉树

二叉树是一种基础而且重要的数据结构，其每个结点至多只有二棵子树，二叉树有左右子树之分，第i层至多有2^(i-1)个结点（i从1开始）；深度为k的二叉树至多有2^(k)-1)个结点，对任何一棵二叉树，如果其终端结点数为n0，度为2的结点数为n2，则n0=n2+1。

二叉树的性质：

1) 在非空二叉树中，第i层的结点总数不超过2^(i-1), i>=1;

2) 深度为h的二叉树最多有2^h-1个结点(h>=1)，最少有h个结点;

3) 对于任意一棵二叉树，如果其叶结点数为N0，而度数为2的结点总数为N2，则N0=N2+1;

4) 具有n个结点的完全二叉树的深度为log2(n+1);

5)有N个结点的完全二叉树各结点如果用顺序方式存储，则结点之间有如下关系：

若I为结点编号则 如果I>1，则其父结点的编号为I/2；
如果2I<=N，则其左儿子（即左子树的根结点）的编号为2I；若2I>N，则无左儿子；
如果2I+1<=N，则其右儿子的结点编号为2I+1；若2I+1>N，则无右儿子。
　　　　

6)给定N个节点，能构成h(N)种不同的二叉树，其中h(N)为卡特兰数的第N项，h(n)=C(2*n, n)/(n+1)。

7)设有i个枝点，I为所有枝点的道路长度总和，J为叶的道路长度总和J=I+2i。

满二叉树、完全二叉树

满二叉树：除最后一层无任何子节点外，每一层上的所有结点都有两个子结点；

完全二叉树：若设二叉树的深度为h，除第 h 层外，其它各层 (1～(h-1)层) 的结点数都达到最大个数，第h层所有的结点都连续集中在最左边，这就是完全二叉树；

满二叉树是完全二叉树的一个特例。

> 二叉查找树

二叉查找树，又称为是二叉排序树（Binary Sort Tree）或二叉搜索树。二叉排序树或者是一棵空树，或者是具有下列性质的二叉树：

1) 若左子树不空，则左子树上所有结点的值均小于它的根结点的值；
2) 若右子树不空，则右子树上所有结点的值均大于或等于它的根结点的值；
3) 左、右子树也分别为二叉排序树；
4) 没有键值相等的节点。

二叉查找树的性质：对二叉查找树进行中序遍历，即可得到有序的数列。

二叉查找树的时间复杂度：它和二分查找一样，插入和查找的时间复杂度均为O(logn)，但是在最坏的情况下仍然会有O(n)的时间复杂度。原因在于插入和删除元素的时候，树没有保持平衡。我们追求的是在最坏的情况下仍然有较好的时间复杂度，这就是平衡二叉树设计的初衷。

二叉查找树可以这样表示：

```
public class BST<Key extends Comparable<Key>, Value> {
    private Node root;             // 根节点

    private class Node {
        private Key key;           // 排序的间
        private Value val;         // 相应的值
        private Node left, right;  // 左子树，右子树
        private int size;          // 以该节点为根的树包含节点数量

        public Node(Key key, Value val, int size) {
            this.key = key;
            this.val = val;
            this.size = size;
        }
    }
    public BST() {}

    public int size() {//获得该二叉树节点数量
        return size(root);
    }

    private int size(Node x) {获得以该节点为根的树包含节点数量
        if (x == null) return 0;
        else return x.size;
    }
}
```

查找：

```
public Value get(Key key) {
    return get(root, key);
}

private Value get(Node x, Key key) {//在以x节点为根的树中查找key
    if (x == null) return null;
    int cmp = key.compareTo(x.key);
    if      (cmp < 0) return get(x.left, key);//递归左子树查找
    else if (cmp > 0) return get(x.right, key);//递归右子树查找
    else              return x.val;//找到了
}
```

插入：

```
public void put(Key key, Value val) {
    root = put(root, key, val);
}

private Node put(Node x, Key key, Value val) {在以x节点为根的树中查找key，val
    if (x == null) return new Node(key, val, 1);
    int cmp = key.compareTo(x.key);
    if      (cmp < 0) x.left  = put(x.left,  key, val);//递归左子树插入
    else if (cmp > 0) x.right = put(x.right, key, val);//递归右子树插入
    else              x.val   = val;
    x.size = 1 + size(x.left) + size(x.right);
    return x;
}
```

删除：

```
public Key min() {
    return min(root).key;
} 
private Node min(Node x) { 
    if (x.left == null) return x; 
    else                return min(x.left); 
} 

public void deleteMin() {
    root = deleteMin(root);
}
private Node deleteMin(Node x) {//删除以x为根节点的子树最小值
    if (x.left == null) return x.right;
    x.left = deleteMin(x.left);
    x.size = size(x.left) + size(x.right) + 1;
    return x;
}

public void delete(Key key) {
     root = delete(root, key);
}
private Node delete(Node x, Key key) {
    if (x == null) return null;

    int cmp = key.compareTo(x.key);
    if      (cmp < 0) x.left  = delete(x.left,  key);//递归删除左子树
    else if (cmp > 0) x.right = delete(x.right, key);//递归删除右子树
    else { //该节点就是所要删除的节点
        if (x.right == null) return x.left;//没有右子树，把左子树挂在原节点父节点上
        if (x.left  == null) return x.right;//没有左子树，，把右子树挂在原节点父节点上
        Node t = x;//用右子树中最小的节点来替代被删除的节点，仍然保证树的有序性
        x = min(t.right);
        x.right = deleteMin(t.right);
        x.left = t.left;
    } 
    x.size = size(x.left) + size(x.right) + 1;
    return x;
} 
```

> 平衡二叉树

平衡二叉树又被称为AVL树，具有以下性质：它是一棵空树或它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一棵平衡二叉树。它的出现就是解决二叉查找树不平衡导致查找效率退化为线性的问题，因为在删除和插入之时会维护树的平衡，使得查找时间保持在O(logn)，比二叉查找树更稳定。

ALLTree 的 Node 由 BST 的 Node 加上 private int height; 节点高度属性即可，这是为了便于判断树是否平衡。

维护树的平衡关键就在于旋转。对于一个平衡的节点，由于任意节点最多有两个儿子，因此高度不平衡时，此节点的两颗子树的高度差2.容易看出，这种不平衡出现在下面四种情况：

1、6节点的左子树3节点高度比右子树7节点大2，左子树3节点的左子树1节点高度大于右子树4节点，这种情况成为左左。

2、6节点的左子树2节点高度比右子树7节点大2，左子树2节点的左子树1节点高度小于右子树4节点，这种情况成为左右。

3、2节点的左子树1节点高度比右子树5节点小2，右子树5节点的左子树3节点高度大于右子树6节点，这种情况成为右左。

4、2节点的左子树1节点高度比右子树4节点小2，右子树4节点的左子树3节点高度小于右子树6节点，这种情况成为右右。

从图2中可以可以看出，1和4两种情况是对称的，这两种情况的旋转算法是一致的，只需要经过一次旋转就可以达到目标，我们称之为单旋转。2和3两种情况也是对称的，这两种情况的旋转算法也是一致的，需要进行两次旋转，我们称之为双旋转。

单旋转是针对于左左和右右这两种情况，这两种情况是对称的，只要解决了左左这种情况，右右就很好办了。图3是左左情况的解决方案，节点k2不满足平衡特性，因为它的左子树k1比右子树Z深2层，而且k1子树中，更深的一层的是k1的左子树X子树，所以属于左左情况。

为使树恢复平衡，我们把k1变成这棵树的根节点，因为k2大于k1，把k2置于k1的右子树上，而原本在k1右子树的Y大于k1，小于k2，就把Y置于k2的左子树上，这样既满足了二叉查找树的性质，又满足了平衡二叉树的性质。

这样的操作只需要一部分指针改变，结果我们得到另外一颗二叉查找树，它是一棵AVL树，因为X向上一移动了一层，Y还停留在原来的层面上，Z向下移动了一层。整棵树的新高度和之前没有在左子树上插入的高度相同，插入操作使得X高度长高了。因此，由于这颗子树高度没有变化，所以通往根节点的路径就不需要继续旋转了。

代码：

```
private int height(Node t){  
    return t == null ? -1 : t.height;  
}     

//左左情况单旋转  
private Node rotateWithLeftChild(Node k2){  
    Node k1 = k2.left;  
    k2.left = k1.right;       
    k1.right = k2;        
    k1.size = k2.size;
    k2.size = size(k2.right)+size(k2.left)+1;
    k2.height = Math.max(height(k2.left), height(k2.right)) + 1;  
    k1.height = Math.max(height(k1.left), k2.height) + 1;         
    return k1;      //返回新的根  
}     
//右右情况单旋转  
private Node rotateWithRightChild(Node k2){  
    Node k1 = k2.right;  
    k2.right = k1.left;  
    k1.left = k2;  
    k1.size = k2.size;
    k2.size = size(k2.right)+size(k2.left)+1;       
    k2.height = Math.max(height(k2.left), height(k2.right)) + 1;  
    k1.height = Math.max(height(k1.right), k2.height) + 1;        
    return k1;      //返回新的根   
}  
```

双旋转是针对于左右和右左这两种情况，单旋转不能使它达到一个平衡状态，要经过两次旋转。同样的，这样两种情况也是对称的，只要解决了左右这种情况，右左就很好办了。图4是左右情况的解决方案，节点k3不满足平衡特性，因为它的左子树k1比右子树Z深2层，而且k1子树中，更深的一层的是k1的右子树k2子树，所以属于左右情况。

为使树恢复平衡，我们需要进行两步，第一步，把k1作为根，进行一次右右旋转，旋转之后就变成了左左情况，所以第二步再进行一次左左旋转，最后得到了一棵以k2为根的平衡二叉树树。
代码：

```
//左右情况  
private Node doubleWithLeftChild(Node k3){        
    try{  
        k3.left = rotateWithRightChild(k3.left);  
    }catch(NullPointerException e){  
        System.out.println("k.left.right为："+k3.left.right);  
        throw e;  
    }  
    return rotateWithLeftChild(k3);       
}

//右左情况  
private Node doubleWithRightChild(Node k3){  
    try{  
        k3.right = rotateWithLeftChild(k3.right);  
    }catch(NullPointerException e){  
        System.out.println("k.right.left为："+k3.right.left);  
        throw e;  
    }         
    return rotateWithRightChild(k3);  
}  
```

AVL查找操作与BST相同，AVL的删除与插入操作在BST基础之上需要检查是否平衡，如果不平衡就要使用旋转操作来维持平衡:

```
private Node balance(Node x) {
    if (balanceFactor(x) < -1) {//右边高
        if (balanceFactor(x.right) > 0) {//右左
            x.right = rotateWithLeftChild(x.right);
        }
        x = rotateWithRightChild(x);
    }
    else if (balanceFactor(x) > 1) {//左边高
        if (balanceFactor(x.left) < 0) {//左右
            x.left = rotateWithRightChild(x.left);
        }
        x = rotateWithLeftChild(x);
    }
    return x;
}

private int balanceFactor(Node x) {
    return height(x.left) - height(x.right);
}
```

> 堆

堆是一颗完全二叉树，在这棵树中，所有父节点都满足大于等于其子节点的堆叫大根堆，所有父节点都满足小于等于其子节点的堆叫小根堆。堆虽然是一颗树，但是通常存放在一个数组中，父节点和孩子节点的父子关系通过数组下标来确定。如下图的小根堆及存储它的数组：

值：7,8,9,12,13,11

数组索引：0,1,2,3, 4, 5

通过一个节点在数组中的索引怎么计算出它的父节点及左右孩子节点的索引：

```
public int left(int i) {
     return (i + 1) * 2 - 1;
}

public int right(int i) {
    return (i + 1) * 2;
}

public int parent(int i) {
    // i为根结点
    if (i == 0) {
        return -1;
    }
    return (i - 1) / 2;
}
```

维护大根堆的性质：

```
public void heapify(T[] a, int i, int heapLength) {
    int l = left(i);
    int r = right(i);
    int largest = -1;
    //寻找根节点及其左右子节点，三个元素中的最大值
    if (l < heapLength && a[i].compareTo(a[l]) < 0) {
        largest = l;
    } else {
        largest = i;
    }
    if (r < heapLength && a[largest].compareTo(a[r]) < 0) {
        largest = r;
    }

    // 如果i处元素不是最大的，就把i处的元素与最大处元素交换，使得i处元素变为最大的
    if (i != largest) {
        T temp = a[i];
        a[i] = a[largest];
        a[largest] = temp;
        // 交换元素后，以a[i]为根的树可能不在满足大根堆性质，于是递归调用该方法
        heapify(a, largest, heapLength);
    }
}
```

构造堆：

```
public  void buildHeap(T[] a, int heapLength) {
    //从后往前看lengthParent处的元素是第一个有子节点的元素，所以从它开始，进行堆得维护
    int lengthParent = parent(heapLength - 1);
    for(int i = lengthParent; i >= 0; i--){
        heapify(a, i, heapLength);
    }
}
```

堆的用途：堆排序，优先级队列。此外由于调整代价较小，也适合实时类型的排序与变更。


