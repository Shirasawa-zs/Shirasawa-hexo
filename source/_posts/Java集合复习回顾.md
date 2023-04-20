---
title: Java集合回顾
date: 2023-03-15 09:25:00
author: scy
img: https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315144706.png
top: true
hide: false
cover: true
toc: false
summary: Java集合泛型
categories: 后端
tags:
  - java
---

# Collection学习前瞻

## 1.Collection一些常用的方法

collection是一个接口，要使用他的方法要使用他的实现类实现

```java
public interface Collection<E> extends Iterable<E> {
    boolean contains(Object o);
    boolean isEmpty();
    boolean add(E e);
    boolean remove(Object o);
    void clear();
    Object[] toArray();
    ......
}






//Test
 public static void main(String[] args) {
        Collection<String> collection = new ArrayList();

        //add
        collection.add("scy");
        collection.add("test");
        System.out.println(collection); //[scy, test]


        //remove
        collection.remove("scy");
        System.out.println(collection);//[test]

        //isEmpty
        System.out.println(collection.isEmpty()); //false

        //contains
        System.out.println(collection.contains("test")); //true

        //toArray
        System.out.println(collection.toArray()); //奇奇怪怪的
    }
```





## 2.遍历集合的三种方式

### 2.1迭代器遍历(所有单列)

**迭代器就是Iterator的简称，专门用来对Collection进行遍历用的，学习迭代器就是为了遍历集合**

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315154141.png)

```java
//Collectin接口及成立Iterable接口
public interface Collection<E> extends Iterable<E> {
       Iterator<E> iterator();
}




//Iterator接口
public interface Iterator<E> {
    //1.
    boolean hasNext();
    //2.
    E next();
    //3.
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
    //4.
    default void forEachRemaining(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        while (hasNext())
            action.accept(next());
    }
}


//Test
 public static void main(String[] args) {
        Collection<String> collection1 = new ArrayList();
        collection1.add("scy");
        collection1.add("test");
        collection1.add("zjj");
        Iterator<String> iterator = collection1.iterator();
        while(iterator.hasNext()){
            String str = iterator.next();
            System.out.println(str);
        }
    }
// 输出 ”scy“,"test","zjj"
```

迭代器注意事项：

1.当迭代器迭代元素完成后，不能继续next获取元素，否则会报错:NoSuchElementException

2.当迭代器在使用过程中，不能使用金额对象增删元素，否则会报错CocurrentModificationException.如果要删除可以使用迭代器来删除

```java
public interface ListIterator<E> {
    boolean hasNext();

    E next() throws SAXException, JAXBException;
}

```

### 2.2增强for遍历(所有单列)

```java
/*
 格式：
  	for(集合中的元素类型 变量名 : 单列集合或者数组){
  	变量名代表集合或数组中每一个元素
  	}
  	
  	优：方便遍历
  	劣：没有索引，遍历的目标不能为null
  	底层仍然是Iterator
*/

public static void main(String args[]){
      Collection<String> collection1 = new ArrayList();
        collection1.add("scy");
        collection1.add("test");
        collection1.add("zjj");
        for (String s : collection1) {
            System.out.println(s);
        }
    
    
    
      int[] arr = {1, 2, 4, 4,};
        for (int i : arr) {
            System.out.println(i);
        }
}

```

### 2.3普通for遍历(只有List)



## 3.泛型介绍

> 泛型的定义

**泛型是一种类型参数，专门用来保存类型用的。**

最早接触泛型是在ArrayList<E>，这个E就是所谓的泛型了。使用ArrayList时，只要给E指定某一个类型，里面所有用到泛型的地方都会被指定对应的类型。

> 泛型带来的好处

**不用泛型带来的问题:**

集合若不指定泛型，默认就是0bject。存储的元素类型自动提升为0bject类型。获取元素时得到的都是Object，若要调用特有方法需要转型，给我们编程带来麻烦.

**使用泛型带来的好处:**

可以在编译时就对类型做判断，避免不必要的类型转换操作，精简代码，也避免了运行时期因为类型转换导致的错误。

> 泛型的注意事项

泛型在代码运行时，泛型会被擦除。后面学习反射的时候，可以实现在代码运行的过程中添加其他类型的数据到集合。



```JAVA
        ArrayList<String> list1 = new ArrayList<>();
        ArrayList<Integer> list2 = new ArrayList<>();
        //没有指定默认是Object
        ArrayList list3 = new ArrayList<>(); //Object


===============================================
    
		//泛型没有指定类型，默认就是0bject
        ArrayList list = new ArrayList();
        list.add( "He1lo");
        list.add( "wor1d");
        list.add( 100);
        list.add(false);
        //集合中的数据就比较混乱,会给获取数据带来麻烦
        for (Object obj : list) {
            String str = (String) obj;
            //当遍历到非String类型数据，就会报异常出错
            System.out.println(str +"长度为:" + str.length());}
        }
```

### 3.1自定义泛型类

使用时机：类中定义属性不知道具体类型时，就可以使用泛型

泛型类中确定时机：创建对象的时候可以指定具体类型

```java
//定义方式如下
public class Student<X, Y> {
    X xObj;
}

//Test
 Student<Integer, String> student = new Student();
```

### 3.2自定义泛型接口

使用时间：定义接口时，内部方法中其参数类型，返回类型不确定时

接口格式：如下

泛型接口中泛型的确定时机：

​	1.子类可以确定类型，在实现接口时，直接确定类型

​	2.子类如果不确定类型，回到泛型类的使用

```JAVA
public interface MyCollection<E> {
    public abstract void add(E e);
    public abstract void remove(E e);
}

//1.如果泛型没有只当，那么是Object类型
class MyCollectionImpl1 implements MyCollection{
    @Override
    public void add(Object o) {}
    @Override
    public void remove(Object o) {}
}

//可以在实现接口时，确定接口泛型类型
class MyCollectionImpl2 implements MyCollection<String>{
    @Override
    public void add(String s) {}
    @Override
    public void remove(String s) {}
}

//3.如果实现类接口不指定，就变成了泛型的使用
class MyCollectionImpl3<A> implements MyCollection<A>{
    @Override
    public void add(A a) {}
    @Override
    public void remove(A a) {}
}
```

### 3.3自定义泛型方法

泛型格式： 修饰符<T> 返回值类型 方法名（方法参数）

public static <E> void getElements(E e){

}

```JAVA
 	//ArrayList类型字符串，转成数组
    // <T> T[] toArray(T[] a);
    public void test(){
        ArrayList<String> list = new ArrayList<>();
        list.add("scy");
        list.add("zjj");
        list.add("ours");
        String[] str = new String[list.size()];
        String[] strings = list.toArray(str);
        System.out.println(Arrays.toString(strings));
    }
```



### 3.4通配符

> 通配符介绍

当我们对泛型的类型确定不了，而想要表达的可以是任意类型，可以使用泛型通配符给定。符号就是一个问号:?表示任意类型，用来给泛型指定的一种通配值。如下:

```JAVA
public static void shuffle(List<?> list) {
       。。。
}
```

> 泛型通配符的使用注意

1）泛型通配符搭配集合使用一般在方法的参数中比较常见

2）在集合中泛型是不支持多态的，如果为了匹配任意类型，我们就会使用泛型通配符了。

3）方法中的参数是一个集合,集合如果携带了通配符,集合的类型会提升为Object类型。集合不能进行添加和修改操作，可以删除和获取

```JAVA
public class Student<X, Y> {
    @Test
    //ArrayList类型字符串，转成数组
    // <T> T[] toArray(T[] a);
    public void test(){
        ArrayList<String> list1 = new ArrayList<>();
        ArrayList<Object> list2 = new ArrayList<>();
        ArrayList<Integer> list3 = new ArrayList<>();
        userList(list1);
        userList(list2);
        userList(list3);
    }

    public static void userList(ArrayList<?> list){
            //不能添加修改，只能获取和删除
            // list.add("sdf");
            // list.set()
    }
}
```

> 上下限类型

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315170219.png)

```java
public static void show1(ArrayList<? super Number>){
}

public static void show1(ArrayList<? extends Number>){
}
```

## 4.数据结构简单概念介绍

1.栈：进栈出栈，栈顶元素，栈底元素，先进后出

2.队列：入队出队，先进先出

3.数组：查询数据通过地址值和索引定位，**查询速度快**

​			  删除数据要将原始数据删除，后面的数据后移，**删除效率低**

​			 添加数据添加位置的每个数据后移，再添加数据，**添加效率极低**

4.链表：**增删快，查询慢**，类比c语言的链表                                                                                                                                

**5.树（二叉树，二叉平衡树，红黑树）**

**5.1二叉树**

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315173520.png)



**5.2二叉查找树**

**大的存右边，小的存左  边，一样的不存，但可能出现线性表，所以需要进一步考虑**



**5.3二叉平衡树**

1.二叉树任意节点的左右子树高度差不超过1

2.任意节点的左右两个子树都是一颗平衡二叉树

触发时机:

​	当添加一个节点后，这个树不是一颗平衡二叉树

​	左旋 vs 右旋

​	左左，左右，右左，右右

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315195141.png)

**建议结合别人的博客理解**

6.哈希表(数组 + 链表 + 红黑树)





![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315201933.png)

## 5.List集合

### 5.1List接口

> 特点

List集合是Collection集合子类型，继承了所有Collection中功能，同时List增加了带索引的功能

1.元素的存取是有序的【有序】

2.元素具备索引【有索引】

3.元素可以重复存储【可重复】

> 实现子类

ArrayList:底层结构就是数组【查询快，增删慢】

Vector:底层结构也是数组（线程安全，同步安全的，低效，用的就少)

LinkedList:底层是链表结构（双向链表〉【查询慢，增删快】

```java
public interface List<E> extends Collection<E> {
    E get(int index);
    E set(int index, E element);
    void add(int index, E element);
    E remove(int index);
}

//Test

 		List<String> lists = new ArrayList<>();
        lists.add("zjj");
        lists.add("scy");
        lists.add("zhy");
        System.out.println(lists);
        System.out.println(lists.remove(2));
        System.out.println(lists.set(2, "scy"));
        System.out.println(lists.get(0));
```

### 5.2LinkedList

> LinkedList介绍

LinkedList底层结构是双向链表。每个节点有三个部分的数据，第一个是保存元素数据，第二个是保存前一个节点的地址，第三个是保存后一个节点的地址。可以双向查询,效率会比单向链表高。

> 介绍

一些自己的独特方法和头尾相关，自己可以进源码看看，看一个遍历的优点意思的

```java
Node<E> node(int index) {
    // assert isElementIndex(index);

    if (index < (size >> 1)) {
        Node<E> x = first;
        for (int i = 0; i < index; i++)
            x = x.next;
        return x;
    } else {
        Node<E> x = last;
        for (int i = size - 1; i > index; i--)
            x = x.prev;
        return x;
    }
```

## 6.Set

> Set介绍

Set集合也是Collection集合的子类型，没有特有方法。Set比Collection定义更严谨，Set集合要求:

1.元素不能保证添加和取出顺序（无序)

2．元素是没有索引的(无索引)

3.元素唯一(元素唯一)

> 常用主要子类介绍

**HashSet**:底层结构 哈希表结构。

具有特点︰去重，无索引，无序特点。

哈希表结构的集合，操作效率会非常高。



**LinkedIlashSet**:底层结构 链表加哈希表结构。

具有哈希表表结构的特点，也具有链表的特点。

链表结构:是为了保证插入顺序

哈希表结构:是为了去重



**TreeSet**:底层数据结构红黑树。

具有特点∶去重，排序的功能

### 6.1HashSet

没什么好介绍的，无序无索引去重，自己测试一下就行，需要注意的一点就是对象的去重判断需要重写equals方法和hashcode方法

### 6.2哈希值

```JAVA
public static void main(String[] args) {
        //1.同一个对象多次调用的hashcode是相同的
      	//2.默认情况下，不同的hash值是不同的，重写hash可以让他们一样
        Teacher teacher = new Teacher();
        Teacher teacher1 = new Teacher();
      	System.out.println(teacher.hashCode());
        System.out.println(teacher.hashCode());
        System.out.println(teacher1.hashCode());

        //3.不同字符串的hashcode可能一样
        System.out.println("通话".hashCode());//1179395
        System.out.println("重地".hashCode());//1179395
    }

public class Teacher {
    String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Teacher)) return false;
        Teacher teacher = (Teacher) o;
        return Objects.equals(name, teacher.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```



### 6.2哈希表介绍

**当数组里面存了16\*0.75 =12个元素的时候，数组就会扩容为原先的两倍 32**

**JDK8之前，底层采用数组+链表实现**

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315205821.png)



**JDK8以后，底层采用数组＋链表/红黑树实现**

底层结构:哈希表。(数组、链表、红黑树的结合体)。当挂在下面的元素过多，那么不利于查询，所以在JDK8以后，当链表长度超过8的时候，自动转换为红黑树。存储流程不变。

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315205922.png)



### 6.3TreeSet

> 特点

1.不包含重复元素的集合

2.没有带索引的方法

3.可以将元素按照规则进行排序

**普通排序测试**

```java
//普通排序测试
 public static void main(String[] args) {
        TreeSet<Integer> ts = new TreeSet<>();
        ts.add(20);
        ts.add(10);
        ts.add(30);
        ts.add(10);
        System.out.println(ts);
    }
//[10, 20, 30]
```



**实现Comparable接口**

```JAVA
//实现Comparable接口

//Stduent1类
public class Student1 implements Comparable<Student1>{
    String username;
    Integer age;

    public Student1(String username, Integer age) {
        this.username = username;
        this.age = age;
    }


    @Override
    public int compareTo(Student1 o) {
        return o.age - this.age;
    }

    @Override
    public String toString() {
        return "Student1{" +
                "username='" + username + '\'' +
                ", age=" + age +
                '}';
    }
}


//Test
public static void main(String[] args) {
        TreeSet<Student1> ts = new TreeSet<>();
        Student1 student1 = new Student1("scy", 20);
        Student1 student2 = new Student1("zj", 21);
        Student1 student3 = new Student1("zj", 23);
        ts.add(student1);
        ts.add(student2);
        ts.add(student3);
        System.out.println(ts);
    }

/*
[Student1{username='zj', age=23}, Student1{username='zj', age=21}, Student1{username='scy', age=20}]
*/
```



**比较器排序**

```JAVA
  public static void main(String[] args) {
        TreeSet<Student1> ts = new TreeSet<>(new Comparator<Student1>() {
            @Override
            public int compare(Student1 o1, Student1 o2) {
                return o1.age - o2.age;
            }
        });
        Student1 student1 = new Student1("scy", 20);
        Student1 student2 = new Student1("zj", 21);
        Student1 student3 = new Student1("zj", 23);
        ts.add(student1);
        ts.add(student2);
        ts.add(student3);
        System.out.println(ts);
    }
```

> 两种比较方式总结

- 自然排序∶自定义类实现Comparable接口，重写compareTo方法，根据返回值进行排序。
- 比较器排序︰创建TreeSet对象的时候传递Comparator的实现类对象，重写compare方法，根据返回值进行排序。
- 如果Java提供好的类已经定义了自然排序排序规则，那么我们可以使用比较器排序进行替换
- 注意:如果自然排序和比较器排序都存在，那么会使用比较器排序

两种方式中，关于返回值的规则:

- ​	如果返回值为负数，表示当前存入的元素是较小值，存左边
-    如果返回值为0，表示当前存入的元素跟集合中元素重复了，不存
- ​	如果返回值为正数，表示当前存入的元素是较大值，存右边

## 7.分析ArrayList和HashSet底层

### 7.1ArrayList

```JAVA
//构建无参构造，并插入数据
ArrayList<String> arrayList = new ArrayList<>();
arrayList.add("fs");
arrayList.add("fssd");

//构造分析：通过属性和方法可知创建了一个把空对象地址赋给了this.elementData
private static final Object[] EMPTY_ELEMENTDATA = {};
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}


//第一次插入分析
private int size;
public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
}

//进入ensureCapacityInternal(size + 1)
//此时elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA判断为true，minCapacity = 10;
private static final int DEFAULT_CAPACITY = 10;
private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
m
//进入ensureExplicitCapacity(minCapacity); 此时minCapacity = 10;
//minCapacity - 0 > 0
private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

//进入grow方法 此时minCapacity = 10;
 private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8; //20亿左右
 private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
	//Arrays.copyOf(elementData, newCapacity); 给数组赋一段新的空间地址，此时空间地址大小为newCapacity


扩容分析：当oldCapacity大于0时，新创建的数组大小是老容量+老容量的一半，也就是老容量的1.5倍，每次扩容到原来的1.5倍。
```





### 7.2HashSet分析

```JAVA
HashSet<Teacher> teachers = new HashSet<Teacher>;
teachers.add(teacher);
teachers.add(teacher1);

//构造分析，本质是一个HashMap
public HashSet() {
        map = new HashMap<>();
    }

//插入分析
//可分析HashSet用HashMap的键值进行插入
private static final Object PRESENT = new Object();
public boolean add(E e) {
        return map.put(e, PRESENT)==null;
    }

//进入HashMap的put方法
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

//进入hash(key),可以看成对哈希值进行一些处理
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }

```

## 8.Collections工具类

### 8.1shuffle方法

```java
public static void shuffle(List<?> list) {
    Random rnd = r;
    if (rnd == null)
        r = rnd = new Random(); // harmless race.
    shuffle(list, rnd);
}
```

> 注意事项

方法功能：打乱List集合中的数据

使用注意：List<?> list --> 只能打乱List集合；

​					集合中的元素类型任意

### 8.2sort方法

**继承Comparable接口**

类比TreeSet

```JAVA
  public static <T extends Comparable<? super T>> void sort(List<T> list) {
        list.sort(null);
    }
```



**构造方法自定义比较器**

类比TreeSet

```JAVA
public static <T> void sort(List<T> list, Comparator<? super T> c) {
        list.sort(c);
    }
```



### 8.3可变参数

> 可变参数介绍

在JDK5中提供了可变参数，允许在调用方法时传入任意个参数。可变参数原理就是一个数组形式存在格式︰

修饰符 返回值类型 方法名 (数据类型 … 变量名){ }

```java
public void show(int... num):表示当前方法可以接受任意个int类型的数据
```

> 可变参数注意

1．可变参数只能作为方法的最后一个参数，但共前面可以有或没有任何其他参数。

2．可变参数本质上是数组，不能作为方法的重载。如果同时出现相同类型的数组和可变参数方法，是不能编译通过的。

> 可变参数的方法使用

调用可变参数方法，可以给出零到任意多个参数，编译器会将可变参数转化为一个数组，也可以直接传递一个数组。方法内部使用时直接当做数组使用即可。

```JAVA
main:: 
	sum(1, 2, 3 );


public static int sum(int ... num){
    int sum = 0;
    for(int i : num){
        sum += i;
    }
    return sum;
}
```

### 8.4可变参数应用

```JAVA
 public static <T> boolean addAll(Collection<? super T> c, T... elements) {
        boolean result = false;
        for (T element : elements)
            result |= c.add(element);
        return result;
    }
==========================
ArrayList<String> list = new ArrayList<>();
Collections.addAll(list, "a", "b","c","d");
System.out.println(list)
```

## 9.排序算法

### 9.1冒泡排序

```JAVA
int[] arr  = {1, 2 ,3 ,4 ,5 };

for(int i = 0; i < arr.length - 1; i++){
    for(int j = 0; j < arr.length - 1 - i; j++){
        if(arr[i] > arr[i + 1]){
            Arrays.swap(arr[i], arr[i + 1])
        }
    }
}
System.out.println(Arrays.toString(arr));
```



### 9.2选择排序

```JAVA
int[] arr  = {1, 2 ,3 ,4 ,5 };

for(int i = 0; i < arr.length - 1; i++){
    int index = i;
    for(int j = i + 1; j < arr.length - 1; j++){
        if(arr[j] > arr[index]){
            index = j;
        }
    }
    if(i != index){
        Arrays.swap(arr[i], arr[index])
    }
}
System.out.println(Arrays.toString(arr));
```



### 9.3二分算法

```JAVA
private static int binarySearch0(long[] a, int fromIndex, int toIndex,
                                     long key) {
        int low = fromIndex;
        int high = toIndex - 1;

        while (low <= high) {
            int mid = (low + high) >>> 1;
            long midVal = a[mid];

            if (midVal < key)
                low = mid + 1;
            else if (midVal > key)
                high = mid - 1;
            else
                return mid; // key found
        }
        return -(low + 1);  // key not found.
    }
```





## 10.Map

### 10.1基础介绍

因为Map是两个集合在一起，所以也称为双列集合，因为比较复杂，所以采用图来展示。

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315225125.png)

![](https://cdn.jsdelivr.net/gh/Shirasawa-zs/BlogImage@main/img/20230315225241.png)



> Map常用方法

```JAVA
public v put(K key, v value):把指定的键与指定的值添加到Map集合中。 
public v remove(object key):把指定的键所对应的键值对元素在Map集合中删除，返回被删除元素的值。 
public v get(object key）根据指定的键，在Map集合中获取对应的值。  
                  
public set<K> keySet():获取Map集合中所有的键，存储到set集合中。
public boolean containKey(object key):判断该集合中是否有此键。
```





```java
public static void main(String[] args) {
        Map<String, String > map = new HashMap<>();
        map.put("zhang", "shao");
        map.put("zhagn", "tao");
        map.put("hu", "mei");

        System.out.println(map.containsKey("zhang"));
        System.out.println(map.containsValue("shao"));

        //增加修改 put
        System.out.println(map.put("zhang", "shaosdf"));
        System.out.println(map.put("sef","sdf"));
    }


/*
true
true
shao
null
*/
```

### 10.2Map遍历

**keySet()遍历**

```java
 public static void main(String[] args) {
        Map<String, String > map = new HashMap<>();
        map.put("zhang", "shao");
        map.put("zhagn", "tao");
        map.put("hu", "mei");

        Set<String> strings = map.keySet();
        //遍历集合
        for (String string : strings) {
            //根据键找到对应的值
            String value = map.get(string);
            System.out.println(value);
        }
    }
```

**键值对象遍历**

> 简单介绍

键值对对象在Java中用Entry类型表示

要获取所有的键值对Entry对象，需要借助Map中方法:

```JAVA
 public Set<Map.Entry<K,V>> entrySet() {
        Set<Map.Entry<K,V>> es;
        return (es = entrySet) == null ? (entrySet = new EntrySet()) : es;
    }
```

> 遍历步骤

1.调用map集合的entrySet方法获取所有的键值对对象

2.遍历每一个键值对对象（Entry对象)

3.getKey获取键，getValue获取值

```JAVA
  public static void main(String[] args) {
        Map<String, String > map = new HashMap<>();
        map.put("zhang", "shao");
        map.put("zhagn", "tao");
        map.put("hu", "mei");

        Set<Map.Entry<String, String>> entries = map.entrySet();
        for (Map.Entry<String, String> entry : entries) {
            String key = entry.getKey();
            String value = entry.getValue()
        }
    }
```

### 10.3HashMap

> 介绍

HashSet所存储的值，其实底层就放置在HashMap的键当中。底层使用哈希表结构来存储数据，存放数据时会执行hashCode和equals方法来去重。

因此，HashMap使用自定义类型当做键使用时需要注意:自定义类需要重写hashCode和equals方法

### 10.4LinkedHashMap

LinkedHashMap类，底层采用的数据结构∶链表+哈希表特点:

1元素唯一 

2元素有序

### 10.5TreeMap

TreeMap的底层是红黑树实现的，有排序的能力，键去重。

1.可以白然排序（键所在的类要实现Comparable)

2．若自定义类没有自然排序功能，或自然排序功能不满足要求时。可以自定义比较器排序（Comparator)两种排序方式对应了TreeMap的两个构造方法:

```JAVA
public TreeMap()//使用自然排序
    
    
public TreeMap(Comparator<? super K> comparator) //比较器排序
```

## 11.集合嵌套

### 11.1List嵌套List

### 11.2List嵌套Map

### 11.3Map嵌套Map