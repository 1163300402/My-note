# MIT软件构造

---
## 10. 抽象数据类型
目标
> - 理解抽象数据类型（ADT）
> - 理解“表示独立”

### 什么是抽象
- **抽象：** 忽略底层的细节而在高层思考
- **模块化：** 把系统分为每一个模块，每一个都可以单独设计、实现、测试、推倒，并在剩下的开发中可以复用。
- **封装：** 给模块建一堵墙，使得它只对自身的功能负责。系统别处的bug无法影响到它内部的正确性。
- **信息隐藏：** 对模块的实现细节进行隐藏，使得在之后改变模块内部代码时不会影响到外部的代码。
- **功能分离：** 即：一一对应，一个模块仅仅负责一个特性或功能，避免很多模块都有同一个特性，或一个模块有多个特性。

### 类型和操作的分类
对于类型，无论内置还是自定义，都分为两种
- 可改变的（`StringBuilder`就是一种可改变的字符串类型）
- 不可变的（`String`——每一次都是创建一个新对象）

抽象类型的操作符大致分类：
- 创建者creator：
- 生产者producer：
- 观察者obverser：
- 改造者mutator：

### 抽象数据类型的例子
`int`是java中的原始数据类型，不可改变，没有改造者
`List`
`String`
### 抽象类型是通过它的操作定义的

---
## 11. 抽象函数与表示不变量
目标
> - 不变量（invariants）
> - 表⽰暴露（representation exposure）
> - 抽象函数（abstraction functions）
> - 表⽰不变量（representation invariants）

用一种正规的数学思想（抽象函数和表示不变量）去理解抽象数据类型ADT的实现。其中
- **抽象函数** 会让我们清晰的定义对两个ADT判断相等的操作
- **表示不变量** 会让我们更易发现破坏数据结构导致的bug。

### 不变量
**什么设计会产生好的ADT？**

最重要的一点就是它会保护/保留自己的不变量。

不变量是一种属性，它是在程序运行时的一种状态。一旦一个不变类型的对象被创建，它总代表一个不变的值。

当一个ADT能够确保它内部的不变量恒定不变（不受使用者/外部影响），我们就说这个ADT保护/保留自己的不变量。

**好处**
当一个ADT保护/保留自己的不变量时，对代码的分析就会变得简单。例如：
- 能够依赖字符串不变的特点，在分析的时候跳过那些关于字符串的代码；或者当尝试基于字符串建立其他不变量的时候，也会变得简单。
- **相反** ，对于可变的对象，不得不对每一处使用它的地方进行审查。

### 不变性
```java
/**
* This immutable data type represents a tweet from Twitter.
*/
public class Tweet {
    public String author;
    public String text;
    public Date timestamp;
    /**
    * Make a Tweet.
    * @param author Twitter user who wrote the tweet
    * @param text text of the tweet
    * @param timestamp date/time when the tweet was sent
    */
    public Tweet(String author, String text, Date timestamp) {
        this.author = author;
        this.text = text;
        this.timestamp = timestamp;
    }
}
```
针对这个例子，怎么样才能确保Tweet对象是不可变的（一旦被创建，author，message，Date都不能被改变）

#### 使用者可以直接访问Tweet内部的数据，即表示暴露（Rep exposure）
```java
问题
Tweet t = new Tweet("justinbieber",
                    "Thanks to all those beliebers out there inspiring me every day",
                    new Date());
t.author = "rbmllr";

解决方法
public class Tweet {
    private final String author;
    private final String text;
    private final Date timestamp;

    public Tweet(String author, String text, Date timestamp) {
        this.author = author;
        this.text = text;
        this.timestamp = timestamp;
    }
    /** @return Twitter user who wrote the tweet */
    public String getAuthor() {
    return author;
    }
    /** @return text of the tweet */
    public String getText() {
    return text;
    }
    /** @return date/time when the tweet was sent */
    public Date getTimestamp() {
    return timestamp;
    }
    
}
```
给属性加上`private`（表示这个区域只能够同类进行访问），加上`final`（确保该对象的索引不会被改变），使用getAuthor（）等函数返回属性值，**对于不可变的类型来说** ，就是确保了变量的值不可变。
#### 可变对象的索引泄露

```java
/** @return a tweet that retweets t, one hour later*/
public static Tweet retweetLater(Tweet t) {
    Date d = t.getTimestamp();
    d.setHours(d.getHours()+1);
    return new Tweet("rbmllr", t.getText(), d);
}
```
该函数希望接受一个`Tweet`对象然后修改`Date`后返回一个新的`Tweet`对象
**问题：** `getTimestamp` 调⽤返回⼀个⼀样的`Date` 对象，它会被 `t` .` t.timestamp` 和 `d` 同时索引。所以当我们调⽤` d.setHours()` 后， `t` 也会受到影响,如图所示
![](_v_images/_1524142993_30252.png)

**解决方法：** 通过防御性复制来弥补：在返回的时候复制一个新的对象而不会返回原对象的索引。
```java
public Date getTimestamp(){
    return new Date(timestamp.getTime());
}
```

#### 对Date创建者也进行防御式编程

```java
/** @return a list of 24 inspiring tweets, one per hour today */
public static List<Tweet> tweetEveryHourToday () {
    List<Tweet> list = new ArrayList<Tweet>();
    Date date = new Date();
    for (int i = 0; i < 24; i++) {
        date.setHours(i);
        list.add(new Tweet("rbmllr", "keep it up! you can do it", date));
    }
    return list;
}
```
**问题：** 该程序试图创建24个`Tweet`对象，每一个对象对应一个小时，但是：
![](_v_images/_1524143429_29714.png)
每一个`Tweet`创建时对`Date`对象的索引都一样。
**解决方法：** 对创建者也进行防御式编程。
```java
public Tweet(String author, String text, Date timestamp) {
    this.author = author;
    this.text = text;
    this.timestamp = new Date(timestamp.getTime());
}
```

通常来说，要特别注意ADT操作中的参数和返回值，如果他们中有**可变类型的对象** ，请确保你没有直接使用索引或者直接返回索引。

当然，更好的解决方法是**直接使用不可变类型** ，上面的例子中，如果我们使用的是`Java.time.ZonedDateTime`而不是`java.util.Date`，那么只需要添加`private`和`final`，不用担心表示保留。

### 可变类型的不可变包装

### 表示不变量和抽象函数

![](_v_images/_1524144286_25885.png)

**抽象函数（abstract function）** 是表示值到其对应的抽象值的映射：
> AF: R -> A

快照图中的箭头表示的就是抽象函数，这种映射是满射，但不一定是单射（不一定是双射）。
**表示不变量（rep invariant）** 是表示值到布尔值的映射：
> RI: R -> boolean

对于表⽰值r，当且仅当r被AF映射到了A，RI(r)为真

表⽰不变量和抽象函数都应该在表⽰声明后注释出来：
```java
public class CharSet{
    private String s;
    // Rep invariant:
    //     s contains no repeated characters
    // Abstraction function:
    //     AF(s)= { s[i] | 0<= i < s.length()}
    ...
```

**一个问题：抽象函数和表示不变量似乎是被表示域和抽象域决定的，甚⾄似乎抽象域就可以决定它。如果是这样的话，那么它们的定义似乎没什么⽤。** 

1. 证明：抽象域不能独立决定AF和RI
    - 对于同样的抽象类型可以有多种表⽰⽅法。例如对于⼀个字符集合，我们既可以⽤字符串来表⽰，也可以⽤⽐特向量来表⽰，每⼀个⽐特位对应⼀个可能的字符。显然我们需要两个不同的抽象函数来表⽰这两种不同的映射。
2. 证明：表示域和抽象域也不能决定AF和RI
    - 这里的关键点在于，当我们确定表示域（表示值的空间）时，我们并不能确定哪一些表示是合法的，以及如果是合法的，它是怎样被映射的。

一个ADT的实现不仅选择表示域（规格说明）和抽象域（具体实现），同时也要决定哪一些表示值是合法的（表示不变量），合法表示会被怎么解释/映射（抽象函数）

截至到第13页
```java
public static <L,E> Graph<L,E> empty()
	public void printGraph();//打印图的节点和边
	public Vertex inputTransformVertex(String str);//把命令行输入的信息转换为节点
	public Edge inputTransformEdge(String str);//把命令行输入的信息转换为边
    public abstract class ConcreteGraph implements Graph<Vertex, Edge> 

	public abstract void fillVertexInfo (String[] args);
	abstract public String toString();
```













