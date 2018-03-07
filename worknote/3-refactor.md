####Duplicated Code(重复代码)

常见情况以及建议的处理方式：

1. **同一个类的两个函数含有相同的表达式。**做法：Extract Method 提炼出重复的代码，让两个地方都调用提炼出来的代码；
2. **两个互为兄弟的子类包含相同表达式，**做法：1. 对两个类使用Extract Method，然后再对被提炼出来的代码使用Pull up Method，将它推入超类中。
3. **两个毫不相关的类出现Duplicated Code，**考虑其中一个使用Extract Class，将重复代码提炼到一个独立类中，然后在另一个类内使用这个新类。

####Long Method(过长函数)

1.程序愈长遇难理解。你应该积极的分解函数。每当感觉需要用注释来说明点什么的时候，就需要把说明的东西写进一个独立函数中，并以其用途命名。
2.关键不在于函数的长度，而在于函数“做什么”和“如何做”之间的语义距离。

如何确定提炼哪一段代码？
> 1. **寻找注释。**它通常能之处代码用途和实现手法之间的语义距离。如果代码前方有一行注释，就是在提醒你：可以将这段代码替换成一个函数，而且可以在注释的基础上给这个函数命名。
> 2. **条件表达式和循环常常也是提炼的信号。**至于循环，你应该将循环和其内的代码提炼到一个独立函数中。

####Large Class（过大的类）

如果想利用单个类做太多事情，其内往往就会出现太多实例变量。解决方案：运用Extract Class将几个变量一起提炼至新类中。提炼时应该选择类内彼此相关的变量，将他们放在一起。

一个类如果有太多代码，往往适合使用Extract Class 和 Extract Subclass。解决方案：先确定客户端如何使用他们，然后运用Extract Interface为每一种使用方式提炼出一个接口。

如果是一个大的GUI类，可以将数据和行为移到一个独立领域对象中去。

####Long Parameter List(过长参数列表)

太长的参数列表难以理解，太多参数会造成前后不一致、不易使用，而且一旦你需要更多数据，就不得不修改它。解决方案：将对象传递给函数，大多数修改都将没有必要，只需要在对象中添加get/set方法，就能得到更多数据。

####Divergent Change(发散式变化)


####Shotgun Surgery(霰弹式修改)
####Feature Envy(依恋情结)
####Data Clumps(数据泥团)
####Primitive Obsession(基本类型偏执)
####Switch Statements(switch惊悚现身)
####Parallel Inheritance Hierarchies(平行继承体系)
####Lazy Class(冗赘类)
####Speculative Generality （夸夸其谈未来性）
####Temporary Field(令人迷惑的暂时字段)
####Message Chains(过度耦合的消息链)
####Middle Man(中间人)
####Inappropriate Intimacy (狎昵关系)
####Alternative Classes with Different Interfaces(异曲同工的类)
####Incomlete Library Class(不完美的类库)
####Data Class(纯稚的数据类)
####Refused Bequest(被拒绝的遗赠)
####Comments(过多的注释)


