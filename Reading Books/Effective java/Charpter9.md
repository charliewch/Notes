#第9章:异常
[TOC]
```
本章目的:异常的作用非常强大,可以提高程序可读性和可靠性,可维护性.本章将介绍有效使用异常指导原则
```
###第57条:只针对异常的情况才使用异常
- 举例：
```
如果有一天你看到下面的代码，请与写这段代码的程序员保持工作距离。
```
```java
// Horrible abuse of exception. Don't ever do this
try{
	int i =0;
    while(true)
    	range[i++].climb();
}catch(ArrayIndexOutOfBoundsException e){
}
```
Note:
```
这段代码写的很糟糕。这段代码使用数组抛出异常的手段来企图终止无限循环的目的，是非常愚蠢的想法。
这里的Excpetion并不具有真正被使用的原因(55)
```
```
写这段代码的人，可能企图使用Java错误判断机制来提高性能。避免VM对每次数组访问检查越界情况。但是结果适得其反
```

#######这样做的三个缺点
```
1. 异常机制的设计初衷是用于不正常的情形，所以VM可能并不会不断的对其进行优化。
现代的JVM实现，基于异常的模式比标准模式要慢得多。
```
```
2. 把代码放在try-catch块中，反而阻止了现代JVM实现本来可能要执行的某些特定优化
```
```
3. 可读性大大的下降，也增加了调试复杂性。同时对数组进行遍历的标准模式并不会导致冗余检查。
```

######从案例中得到警告
```
异常应该只用于异常的情况下，它们永远也不应该用于正常的控制流。
```
```
作为标准程序员，应该优先使用标准的，容易理解的模式，而不是那么声称可以提高性能的，弄巧成拙的方法。
```

#####对API设计的启发
```
设计良好的API不应该强迫它的客户端为了正常的控制流而使用异常。
```
```
如果类具有“状态相关”的方法，即只有在特定的不可预知的条件下被调用的方法，类往往也会提供一个单独的“状态测试”方法。
比如：Iterator接口有一个“状态相关”的next方法，和相应的状态测试方法hasNext().使用传统的for循环遍历代码
```
```java
for(Iterator< Foo > i = collection.iterator();i.hasNext();){
	Foo foo = i.next();
    ....
}
```
```
如果Iterator缺少hasNext方法，客户端使用可能被迫改为使用异常来结束代码
```
```java
// Do not use this hideous code for iteration over a colleation!
try{
	Iterator< Foo > i = collection.iterator();
    while(true){
    	Foo foo = i.next();
        ....
    }
}catch(NoSuchElementException e){

}
```
Note:
```
这段代码就和我们开始举的例子差不多了。是段非常糟糕的代码
```

#####总结
```
异常是为了在异常情况下使用而设计的。不要将它们用于普通的控制流，也不要编写迫使客户端必须这么做的API。
```

###第58条：对可恢复的情况使用受检异常，对编程错误使用运行时异常
#####Java语言提供三种可抛出结构
```
1. 受检的异常
```
```
2. 运行时异常
```
```
3. 错误
```
#####根据一般性原则选择三种抛出结构
#######受检异常或未受检异常选择原则
```
主要原则：如果期望调用者能够适当地恢复，请使用受检的异常。
```
```
通过抛出受检的异常，强迫调用者在一个catch子句中处理改异常，或者将它传播出去。
API设计者让API用户面对受检的异常，强制用户从这个异常中恢复
```
#######未受检的可抛出结构：运行是异常和错误
```
如果程序抛出未受检异常或者错误，往往属于不可恢复的，继续执行下去有害无益。
```
```
运行时异常表明编程错误。大多数运行时异常都表示前提违例。也就是说API的客户没有遵守API规范建立的约定。
比如:ArrayIndexOutOfBoundsException表明数组下标值超出0~(N-1)
```
```
错误的发生，往往表示资源不足，约束失败，或者其他使程序无法继续执行的条件。请不要在实现任何新的Error子类
```
```
对于可以实现所有的未受检的抛出结构都应该直接或者间接是RuntimeException的子类
```

#####总结
```
对于可恢复的情况，使用受检的异常；对于程序错误，使用运行时异常。当然某些情况可能不好区分
```
- 举例：
```
某程序分配一块不合理的过大的数组，导致资源不足，如果资源短缺是由于临时需要太大，则这种情况是可以恢复的，
API设计者允许其恢复，则使用受检的异常。如果不允许，使用运行时异常。
如果不清楚是否可恢复，最好使用未受检的异常(59)
```

```
异常是一个有意义的对象，可以为它定义任意的方法，这些方法主要用途是为了捕捉异常的代码而提供额外的信息。
特别是引发这个异常条件的信息。
```
```
受检方法往往指明可恢复的条件，所以，对于这样的异常，提供一些辅助方法是非常重要的，通过这些方法，调用者可以获得有助于恢复的信息。
```
- 举例：
```
用于支付商品时，由于余额不足，发生受检异常，则这个受检异常应该提供客户查询所缺的费用的余额方法。
```

###第59条：避免不必要地使用受检的异常
```
受检异常是Java程序设计语言的一项很好的特性，强迫程序员程序处理异常的条件，大大增加可靠性。
```
```
但是，过分使用受检异常，会使API使用起来非常不方便。所以，请避免不必要的受检异常
```

#####受检的异常使用原则
#######使用条件
```
如果下面两个条件有任何一个不成立,则更适合使未受检的异常
```
```
1. 正确地使用API并不能保证这种异常条件的产生
```
```
2. 并且一旦产生异常，使用API程序员的程序可以理解采取有用的动作.这种负担被认为是正当的
```
-  举例:
```java
} catch(TheCheckedException e){
	throw new AssertionError(); // Can't happen
}
```
```java
} catch(TheCheckedException e){
	e.printStackTrace(); // Oh well, we close
    System.exit(1);
}
```
Note:
```
在实践中,catch块几乎总是具有断言(assertion)失败的特征.异常受检的本质并没有为程序员提供任何好处,反而需要付出努力,让程序变的复杂.
```

#####受检的异常后果
```
被一个方法单独抛出的受检异常,会给程序员带来非常高的负担.如果这个方法中还有其他的受检异常,对使用方法的程序员来说,
每个异常可能都要加一个catch块.
```

#####将受检的异常变成未受检的异常
```
把这个抛出异常的方法分成两个方法,其中一个方法返回一个boolean,表明是否应该抛出异常.
```
```java
// Invocation with checked exception
try{
	obj.action(args);
}catch(TheCheckedException e){
	// handle exceptional condition
}
```
```
重构成
```
```java
// Invocation with state-testing method and unchecked exception
if(obj.actionPermitted(args)){
	obj.action(args);
}else{
	//handle exceptional condition
}
```
Note:
```
这种重构可能并不总是恰当的.但是如果可行,会使API使用舒服.
```

#####总结
```
本条目提到的重构,可能在本质上等同于"状态测试方法"(57),并且,同样的告诫依然有效:
如果对象在缺少外部同步的情况下被并发访问,或者可被外界改变状态,这种重构方式将是不恰当的.
```
```
如果单独的actionPermitted方法必须重复action方法的工作,处于性能,这种API重构不值得做.
```

###第60条:优先使用标准的异常
```
讨论常见的可重用异常
```
```
Java平台类库提供了一组基本的未受检异常,满足了大多数API的异常抛出需求.
```

#####专家级程序员与缺乏经验的程序员最大的区别
```
专家追求并且通常也能实现高度的代码重用.代码重用是值得提倡的,异常代码也不例外.
```
#####重用现有的异常的好处
```
1. 它会使你的API更加容易学习和使用,你的API属于习惯用法
```
```
2. 对于使用这些API的程序员而言,可读性会更好,因为它不会出现很多程序员不熟悉的异常.
```

#####Java类库中的异常

####### 大多数的方法错误调用,都可以使用异常:IllegalArgumentException,IllegalStateException
```
1. 非法参数异常:IllegalArgumentException

当调用这传递的参数不合适时,产生这个异常.比如:一个参数代表"某个动作执行次数",但程序却出入一个负数.
```
```
2. 非法状态异常:IllegalStateException

如果对象的状态使用非法,产生这个异常.比如:在某个对象被正确初始化之前,调用者企图使用这个对象,则应该抛出这个异常.
```

####### 还有一些标准异常被用于特定情况下的非法参数和非法状态
```
1. 如果某个方法不允许传入null值的参数,习惯抛出NullPointException,而不是IllegalArgumentException
```
```
2. 如果序列的下标参数传递越界只,习惯抛出IndexOutOfBoundsException,而不是IllegalArgumentException
```

#######其他常用的异常
```
1. 并发状态异常:ConcurrentModificationException

如果某个对象被设计成单线程或者与外部同步机制配合使用,一旦发现被并发修改,则应该抛出ConcurrentModificationException异常
```
```
2. 不支持操作的异常:UnsupportedOperationException

绝大多数对象都会支持它们实现的所有方法.但是可能由于继承某个接口,不得不实现这个接口的某些方法,但是又不想被调用,可以使用
```

#####最常见的可重用异常
|异常|使用场合|
|:---:|:---:|
|IllegalArgumentException|非null参数值的不正确|
|IllegalStateException|对方法调用而言,对象状态不合适|
|NullPointerException|禁止使用null的情况下参数值为null|
|IndexOutOfBoundsException|下标参数值越界|
|ConcurrentModificationException|禁止并发修改的情况下,检查到对象并发修改|
|UnsupportedOperationException|对象不支持用户请求的方法|
|IllegalAccessException|不允许方法的方法,非法访问异常|
|InvalidObjectException|不合理对象异常|

#####其他异常类
```
在条件许可的情况下,其他的异常也可以被重用
```
```
如果希望稍微增加更多的失败-捕获(failure-capture)信息(63),可以放心的将现有的异常类子类化
```
- 举例:
```
实现诸如复数或者有理数之类的算术对象,也可以重用ArithmeticException和NumberFormatException.
```

#####总结
```
可能选择重用哪个异常并不总是那么精确,可能使用场合互相排斥.
```

###第61条:抛出与抽象相对应的异常
```
如果方法抛出的异常与它的执行任务没有明显的联系,这种情况下可能会让人不知所措.
```
- 举例:
```
当方法传递有低层抽象抛出的异常时,往往可能污染了更高层的PAI.可能高层的实现在未来的版本中发生变化,需要记得处理低层异常.
```
#######解决方法
```
异常转译:更高层的实现应该捕获低层的异常,同时抛出可以按照高层抽象进行解释的异常.
```
```java
// Exception Translation
try{
//Use lower-level abstraction to do our bidding
}catch(LowerLevelException e){
	throw new HigherLevelException();
}
```

#####异常转译举例
```
异常转译例子取自于AbstractSequentialList类.List接口的一个骨架实现(18)
```
```
List< E > 接口中get方法的规范要求,异常转译是必须的.
```
```java
 /**
     * Returns the element at the specified position in this list.
     *
     * @param index index of the element to return
     * @return the element at the specified position in this list
     * @throws IndexOutOfBoundsException if the index is out of range
     *         (<tt>index &lt; 0 || index &gt;= size()</tt>)
     */
    public E get(int index){
    	ListIterator< E > i = listIterator(index);
        try{
        	return i.next();
        }catch(NoSuchElementException e){
        	throw new IndexOutOfBoundsException("Index :" + index );
        }
    }
```

##### 异常链
```
一种特殊的异常转译形式是异常链.
如果低层的异常对于调试导致高层异常的问题有帮组,请使用异常链,将低层的异常(原因)传到高层的异常.
高层的异常提供访问方法(Throwable.getCause)来获得低层异常
```
```java
// Exception Chaining
try{
	...// Use lower-level abstraction to do our bidding
}catch(LowerLevelException cause){
	throw new HigherLevelException(cause);
}
```
```
高层异常的构造器将原因传到支持链的超级构造器,因此它最终将被传给Throwable的其中一个运行异常链的构造器
```
```java
//Exception with chaining-aware constructor
class HigherLevelException extends Exception{
	HigherLevelException(Throwable cause){
    	supper(cause);
    }
}
```
Note:
```
大多数标准异常都支持链的构造器,对于没有支持链的异常,可以使用Throwable的initCause方法设置原因.
程序可以使用getCause方法得到异常原因,这样就可以将原因的堆栈轨迹集成到更高层的异常中
```

#####不可滥用异常转译
```
异常转译与不加选择地从低层传递异常的做法相比有所改进,但是不可滥用
```
```
处理来自低层的异常的最好做法是,在调用低层方法之前确保会执行成功.所以会检查高层方法的参数的有效性,从而避免低层方法抛出异常.
```
```
如果无法避免低层异常,可以让高层绕开这些异常,不做任何处理.但是最好使用日志的方式,将异常记录下来.
```

#####总结
```
如果不能阻止或者处理来至更低层的异常,一般的做法是使用异常转译,
除非低层方法碰巧可以保证抛出所有异常对高层也合适才可以将异常从低层传播到高层
```
```
异常链对高层和低层异常都提供最佳的功能:它允许抛出适当的高层异常,同时又能补获底层的原因进行失败分析(63)
```

###第62条:每个方法抛出的异常都要有文档
```
异常文档是正确使用这个方法时所需文档的重要组成部分.应该仔细地为每个方法抛出的异常建立文档特别重要
```
```
始终单独地声明受检的异常,并且利用Javadoc的@ throws标记,准备地记录下抛出每个异常的条件.
如果一个方法可能抛出多个异常类,请不要认为抛出这个异常类的某个超类,比如直接抛出"throws Exception",这是一个非常糟糕的做法.
```

#####抛出未受检异常的作用
```
未受检异常,通常代表编程上的错误(58),让程序员里了解这些错误,有助于帮助它们避免这样的错误
```
```
如果将方法可能抛出的未受检异常组织成列表文档,可以有效的描述出这个方法成功执行的前提条件.
```
```
 对于接口中的方法,在文档中记录它可能抛出的未受检异常,这将会成为接口的通用约定,指定该接口的多个实现必须遵循的公共行为.
```
#####Javadoc的@throws标签记录使用
```
可以使用Javadoc的@throws标签记录方法可能抛出的每一个未受检异常,但不要将throws关键字将未受检的异常包含在方法的声明中,
这样方便程序员区分受检异常和未受检异常.
```
#####总结
```
1. 如果一个类的每个方法都可能抛出同一个异常,可以在该类的文档注释中对这个异常进行描述
```
```
2. 要为你编写的每个方法所能抛出的每个异常建立文档.
```
```
3. 对于未受检和受检异常,原则上可以不为未受检异常提供throws子句.但最好都提供,方便程序员使用方法.
```

###第63条:在细节消息中包含能捕获失败的信息
```
当程序发生未被捕获异常时,异常类型的toString方法应该更可能的返回有关失败原因的信息.
也就是说:异常细节消息应该捕获住失败,便于以后分析
```

#####异常的细节信息
```
异常的细节信息应该包含所有"对该异常有贡献"的参数和域的值.
```
- 举例:
```
比如IndexOutOfBoundsException异常的细节消息应该告诉程序员是下界,还是上界,还是不在界内.
```

```
应该提供丰富的堆栈信息,方便程序员结合源码分析代码异常.
```

#####确保异常的细节消息丰富
```
将异常细节消息写入异常的构造器.这样,当发生异常时,构造器根据异常细节自动产生完整的细节消息.
```
- 举例:
```
比如:IndexOutOfBoundsExcepiton构造器并不是一个String构造器.
```

```java

public class IndexOutOfBoundsException {
    /**
    * Construct an IndexOutOfBoundsException.
    *
    * @param lowerBound the lowest legal index value.
    * @param upperBound the highest legal index value plus one.
    * @param index the actual index value
    */
    public  IndexOutOfBoundsException(int lowerBound, int upperBound,int index){
      //Generate a detail message tha captures the failure
        super("Lower bound: "+lowerBound +", Upper bound: "
            +", Index: "+index);
        this.lowerBound =lowerBound;
        this.upperBound = upperBound;
        this.index = index;
    }
}
```
Note:
```
可以Java平台类库并没有广泛采取这种做法.但是这种做法值的大力推荐,方便程序员易于捕获程序失败原因.
```

#####总结
```
为异常的"失败捕获"信息提供一些访问方法是合适的(58),提供这样的访问方法对于受检的异常,比对于未受检异常更为重要,
因为,可能提供的捕获信息对从失败中恢复非常游泳.
```

###第64条:努力使失败保持原子性.
```
当对象抛出异常之后,通常我们期望这个对象仍然保持在一种定义良好的可用状态之中.特别是受检的异常而言,应该调用者期望能从这种异常中恢复.
```
#####保持失败原子性
```
失败原子性:一般而言,失败的方法调用应该使对象保持在被调用之前的状态.
```
#####获得失败原子性的办法
```
1. 最简单的办法:设计一个不可变的对象(15).因为对象是不可变的,所以任何的操作失败,都不会改变不可变的对象
```
```
2. 可变对象保证失败原子性常见的办法:在执行操作之前检查参数的有效性(38).这样可使得在对象的状态被修改之前,先抛出适当的异常.
```
- 举例:
```java
public Object pop(){
	if(size == 0){
    	throw new EmptyStackException();
    }
    Object result = elements[--size];
    elements[size] = null; //Eliminate obsolete reference
    return result;
}
```
Note:
```
如果取消对初始大小的检查,这个方法可能企图从空栈中弹出元素时,仍然抛出异常.这将导致size域保持在不一致的状态中(top变成负数).
```
```
pop方法抛出的异常也不适合抽象(61)
```

```
3. 调整计算处理过程的顺序,使得任何可能会失败的计算部分都在对象状态被修改之前.
```
- 举例:
```
考虑TreeMap的情形,它的元素被按照某种特定的顺序做了排序,为了向TreeMap中添加元素,
该元素的类型就必须可以利用TreeMap的排序准则与其他的元素进行比较
```
Note:
```
第三种获得失败原子性的办法远远没有那么常用.做法是编写一段恢复代码,来使对象回滚到操作开始之前的状态上.
```

```
4. 在对象上做一份临时拷贝操作,当操作完成之后,再用临时拷贝的结果替代对象的内容.但这样影响性能的开销.
```

#####总结
```
1. 一般情况下,都希望实现失败原子性.但是并不是所有的情况都可以使用.比如:捕获ConcurrentModificationException异常,往往不可恢复.
```
```
2. 如果失败原子性恢复,显著的增加开销或复杂性.请谨慎.
```
```
3. 方法提供的文档,API文档应该清楚的指明抛出异常后,对象将会处于什么样的状态.
```

###第65条:不要忽略异常
```
当API设计者声明一个方法将抛出某个异常时,就等于正在试图说明某些事情,请不要忽略
```
```
有的程序员,使用空的catch块来忽略异常
```
- 举例:
```java
//Empty catch block ignores exception - Highly suspect !
try{
	//...
}catch(SomeException e){
	//Empty code 
}
```
Note:
```
这样写代码,后患无穷.
```
```
可能在关闭FileInputStream时候,抛出异常允许忽略,但是至少也应该将异常记录下来
```

#####总结:
```
本条目的建议同样适用于受检异常和未受检异常.
```
```
程序抛出异常,如果采用空catch块来忽略,可能在将来执行的某个时刻由于忽略的异常而产生错误,那么,Bug将是非常困难的.
```
