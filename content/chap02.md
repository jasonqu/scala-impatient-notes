### 2.8 默认参数和带名参数

java中的函数重载可以使用默认参数轻松实现：

	def decorate(str: String, left: String = "[", right: String = "]") =
		left + str + right
	
	scala> decorate("Hello")
	res10: String = [Hello]
	
	scala> decorate("Hello", "<<<", ">>>")
	res11: String = <<<Hello>>>

	scala> decorate("Hello", ">>>[")
	res12: String = >>>[Hello]

甚至可以在参数上带上名字：

	scala> decorate(left = "<<<", str = "Hello", right = ">>>")
	res13: String = <<<Hello>>>

### 2.9 变长参数

对变长参数，序列可以使用:_*来转换成参数序列

	def sum(args: Int*) = {
	  for(arg <- args) print(arg + ";") 
	}
	
	scala> sum(1,2,3)
	1;2;3;
	
	scala> sum(1 to 5 :_*)
	1;2;3;4;5;

### 2.11 lazy val

val 声明为 lazy的时候，初始化将被延至首次被调用的时候

	val words = io.Source.fromFile("/usr/share/dict/words").mkString
	// 定义时取值 Evaluated as soon as words is defined
	lazy val words = io.Source.fromFile("/usr/share/dict/words").mkString
	// 首次使用时取值，开销是每次调用都会有一个检查
	def words = io.Source.fromFile("/usr/share/dict/words").mkString
	// 每次使用时取值
