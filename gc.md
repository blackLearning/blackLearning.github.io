# garbage collection

## 什么是垃圾

常见的编程语言中，对使用的内存分为 stack 和 heap 两种，
stack 内存被用作线程的执行，当一个函数被调用，就会有一块栈内存被分配用来存储局部变量等，当该函数被返回的时候，所有该函数分配的栈内存都会被释放，通常认为，程序中的每种语句对栈都是零副作用的，也就是当每个语句执行完成之后，stack 栈的长度应该和执行语句之前的长度一致。

与栈相对应的是，堆内存通常用来存放动态的持久的内存使用，例如闭包变量，全局变量，对象等。

通常在 C/C++等语言中，程序员需要手动的去分配和释放内存，如果在内存分配了之后没有及时清除，就有可能造成内存泄漏，相反，如果一个对象的内存在仍然有可能被使用的时候被释放了，就会造成空指针的情况。

什么决定一个对象是否是垃圾对象呢，通常的原则是如果一个对象可经由某个被定义为活跃对象的对象，通过某个指针链所访问，则它就是活跃的。其他的都被视为垃圾。

1. 所有的根对象都是活跃对象
2. 任何活跃对象可以引用到的对象都是活跃对象

常见的根对象：栈中的对象，
全局变量，函数调用帧，被活跃对象引用的闭包对象等等。

常见的垃圾对象：例如未被引用的闭包对象，临时使用的字符串对象例如`var a = "hello" + "world";`，弱引用的对象有没有其他对象应用的对象比如 weakMap 的键值对象。

## 垃圾回收时机的选择

为了保证程序执行的时候不会对对象的状态进行修改，因此垃圾回收的时候程序会将全暂停执行(stop-the-world)，因此一个垃圾回收器执行的时间和频率很大程度上影响了用户的代码的性能，衡量一个垃圾回收性能的标准有很多，比如：

1. 吞吐量

吞吐量是总的代码的时间/垃圾回收占用的时间，例如跑了一个程序花费的时间是 10s, 而垃圾回收所占有的时间是 1s,那么我们的吞吐量是 90%. 从性能的角度来看，用户希望吞吐量越高越好。

2. 延迟量

延迟量通常是当垃圾回收发生时用户代码最长的暂停时间，同样的跑了一个程序花费的时间是 10s, 而垃圾回收所占有的时间是 1s, 如果统一把垃圾回收放在一次执行，这样程序的延迟量是 1s, 而如果垃圾回收分 5 次执行，每次执行的时间是 1/5 秒，这样延迟量就是 0.5s, 在一些金融业务或者对延迟量要求比较高的业务中，延迟量越低就代表了性能越好。

这意味着一个好的垃圾回收器应该在两者之间达到平衡，即既不能执行的太频繁导致吞吐量减少，因为此时可能并没有很多的垃圾对象需要回收，垃圾回收器可能在浪费时间，也不能执行的太过于迟钝，这样会导致积累的垃圾过多而单次需要回收的时间很长，从而增加了延迟量

## 常见的垃圾回收算法

### 引用计数

引用计数垃圾回收的策略是，对每一个内存中的对象持有一个引用数字。例如：

```js
var a = {}; // 此时该对象的引用为1
var b = a; // 此时对象的引用变为2, 因为有了一个新的变量引用到了该对象

var c = {};
b = null; // b的引用被删除，因此该对象重新变为1
a = c; // a的引用指向了其他对象，因此该对象的引用计数重新变为了1，此时如果垃圾回收运行，则该对象的内存可以被回收
```

引用计数的问题之一在于不能处理循环引用的情况：

例如：

```ts
class A {
  private b: B;
  setB(b: B) {
    this.b = b;
  }
}
class B {
  private a: A;
  setA(a: A) {
    this.a = a;
  }
}

// 此时内存中分配了两个对象的内存，他们的引用计数为1
var one = new A();
var two = new B();

// 将其互相进行引用，他们的引用计数为变为2
one.setB(two);
two.setA(one);

// 此时即使我们将两个变量对其的引用删除，垃圾回收始终无法将其回收，因为他们的引用计数不为0
one = null;
two = null;
```

### 标记-清除

标记-清除法顾名思义有两个阶段：

1. 标记：

![](https://miro.medium.com/max/1741/1*_xkq7jGtAKf7SP1R1a8vcA.png)

标记对象：三色标记法

三色标记法是图遍历算法的一种常用辅助方法，在寻找环，拓扑排序方面都有使用。

- 白色：在垃圾回收之前，所有的对象都被视为白色对象，意味着我们根本没有接触过该对象
- 灰色：所有的根对象都被视为灰色对象，在递归遍历时，所有的引用到的对象都标记为灰色对象
- 黑色：当我们取出一个灰色对象，并将其所有的引用对象都标记为灰色的时候，然后我们将其标记为黑色对象。

  1. 找到所有的初始对象
  2. 找到所有的根对象并标记为灰色
  3. while(只要存在灰色对象)

     1. 选择一个灰色对象，将该对象所引用到的任何对象都标记为灰色
     2. 将该灰色对象标记为黑色。这就是为什么标记-清除法不存在循环引用的问题，因为只要一个灰色对象变成黑色了，它就不会再次被标记了

2. 清理

![](https://miro.medium.com/max/1729/1*8b-ANSuneRBXkO1JNtH6LQ.png)

遍历所有的白色对象，然后将其内存回收。

注意在完成垃圾回收之后，需要将所有的黑色对象重新标记为白色对象，以便下一次垃圾回收周期使用。

伪代码：

```js
function collectGarbage() {
  // 1. 标记所有的根对象为灰色
  markRoots();

  // 2. 递归的追踪所有对象的引用, 如果我们的程序支持弱引用，例如weakMap, 即使该对象被引用了，也不应该被进行标记。
  traceRefrences();

  // 3. 回收未被引用的对象
  sweep();
}
```

### 标记-压缩

标记-压缩算法大体和标记-清除算法类似，但是由于垃圾压缩算法会导致大量的不连续的内存片段，压缩算法会在清除垃圾对象之后，将所有的对象重新紧缩在一起。

压缩：

![](https://miro.medium.com/max/1729/1*8b-ANSuneRBXkO1JNtH6LQ.png)

### 分代式

一个垃圾回收器如果话费了大量的时间去访问活跃的对象就会丢失吞吐量，但如果它迟钝的去收集和回收对象，就会导致延迟量的上升。如果能够分辨哪些对象是可能长期存活的，哪些对象是相对短命的，这样垃圾回收器就能够避免的频繁访问长期对象，而是经常去清理短命对象。

在程序中，绝大多数对象的生存期很短，只有某些对象的生存期较长。为利用这一特点，每个新对象起初会被分配在新生区（通常很小，只有 1-8 MB，具体根据行为来进行启发）。当新生区的内存被占满，就会有一次清理（小周期），清理掉新生区中不活跃的死对象（新生代中的对象主要通过 Scavenge 算法进行垃圾回收,在 Scavenge 的具体 实现中,主要采用了 Cheney 算法）。对于活跃超过 2 个小周期(这个周期根据具体的实现而定)的对象，则需将其移动至老生区。老生区在标记－清除或标记－紧缩（大周期）的过程中进行回收。大周期进行的并不频繁。一次大周期通常是在移动足够多的对象至老生区后才会发生。至于足够多到底是多少，则根据老生区自身的大小和程序的动向来定。
我们熟悉的 v8 虚拟机就是采用的这种算法。

> Cheney 算法是一种采用复制的方式实现的垃圾回收算法。它将堆内存一分为二,每一部分空间称为 semispace。在这两个 semispace 空间中,只有一个处于使用中,另一个处于闲置状态。处于使用状态的 semispace 空间称为 From 空间,处于闲置状态的空间称为 To 空间。当我们分配对象时,先是在 From 空间中进行分配。当开始进行垃圾回收时,会检查 From 空间中的存活对象,这 些存活对象将被复制到 To 空间中, 而非存活对象占用的空间将会被释放。完成复制后,From 空 间和 To 空间的角色发生对换。简而言之, 在垃圾回收的过程中, 就是通过将存活对象在两个 semispace 空间之间进行复制。Scavenge 算法通过牺牲空间换时间的算法非常适合生命周期短的新生代

![](https://miro.medium.com/max/1442/1*lxyn657K6Wm8-uSRsjSj-w.png)

![](https://miro.medium.com/max/1350/1*YyFP0Jp9W2W3NhWigeQ0Ig.png)

### refs:

- [1][garbage-collection](http://www.craftinginterpreters.com/garbage-collection.html#when-to-collect)

- [2][a-tour-of-v8](http://jayconrod.com/posts/52/a-tour-of-v8--object-representation)

- [3][understanding-java-vm-memory-management](https://www.pluralsight.com/courses/understanding-java-vm-memory-management)

- [4][js-垃圾回收](https://juejin.im/post/5b1f7e62e51d45068a6cb98f)
