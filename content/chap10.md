### Tips

	class ConsoleLogger extends Logger with TimestampLogger with Serializable

这样的代码看上去有些奇怪，不过scala中"Logger with Cloneable with Serializable"是被当做一个整体的。

让特质拥有具体行为存在一个弊端：特质改变后，所有混入该特质的类都必须重新编译

### 10.4 带有特质的对象

scala标准库中有一个Logged特质[@deprecated 将被移除]，有一个log(msg : String)的空实现，可以在类中使用

class SavingsAccount with Logged {
  def withdraw(amount: Double) {
    if (amount > balance) log("Insufficient funds")
    else balance -= amount
  }

  // More methods ...
}

但是这个log什么也不能做，看上去没有意义，但是你可以在构造具体对象的时候混入更好的日志记录器（其中ConsoleLogger <: Logged）：

	val acct2 = new SavingsAccount with ConsoleLogger

### 10.5 叠加特质

对"Logger with TimestampLogger with ShortLogger" 执行顺序是从右向左。

对特质，无法判断super.someMethod会执行哪里的方法，它依赖于使用特质的对象或类的顺序。如果需要控制，可以在方括号中给出名称super[ConsoleLogger].someMethod，但是给出的类型必须是直接超类型。

### 10.6 特质中重写抽象方法

如果Logged中没有给出空实现，则如果TimestampLogger用到super.log的话，该log方法会有编译错误，因为super方法没有实现；所以必须在log前加上"abstract override"

### 10.8 特质中的字段

- 具体字段不是继承，而是简单的加到子类当中
- 抽象字段必须在子类中重写实现，不用加override

### 10.10 特质构造顺序

线性化：串接并去掉重复项，右侧胜出

例如

	class SavingsAccount extends Account with FileLogger with ShortLogger
	lin(SavingsAccount)
	= SavingsAccount >> lin(ShortLogger) >> lin(FileLogger) >> lin(Account)
	= SavingsAccount >> (ShortLogger >> Logger) >> (FileLogger >> Logger) >> Account
	= SavingsAccount >> ShortLogger >> FileLogger >> Logger >> Account

构造顺序就是线性化的反向，所以ShortLogger的super是FileLogger

### 10.11 初始化特质字段

特质只有无参构造函数，所以下面的代码是不能编译的

	val acct2 = new SavingsAccount with FileLogger("myapp.log")

用抽象字段val filename并重写：

	val acct2 = new SavingsAccount with FileLogger {
	  val filename = "myapp2.log"
	} 

但是上面的方案不可行，因为FileLogger先于 { val filename = "myapp2.log" } 构造，所以还是需要“提前定义”。

	val acct2 = new {
	  val filename = "myapp2.log"
	} with SavingsAccount with FileLogger

或者用低效的lazy val

### 10.13 自身类型

scala 特质中用来限制混入类型的做法。如LoggedException只能混入Exception的子类【也可以使用结构类型】


	trait LoggedException extends Logged {
	  this: Exception => // or this: { def getMessage() : String } =>
	    def log() { log(getMessage()) }
	}

TODO 练习