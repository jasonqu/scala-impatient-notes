### 12.1 作为值的函数

可以这样定义一个函数，并当成参数使用

	val fun = math.ceil _
	fun(3.14)
	Array(3.14, 1.42, 2.0).map(fun)

### 12.3 高阶函数

以函数为参数，或者返回函数的函数：

	scala> def valueAtOneQuarter(f: (Double) => Double) = f(0.25)
	valueAtOneQuarter: (f: Double => Double)Double
	
	scala> def mulBy(factor : Double) = (x : Double) => factor * x
	mulBy: (factor: Double)Double => Double

### 12.6 闭包

	def mulBy(factor : Double) = (x : Double) => factor * x
	
	val triple = mulBy(3)
	val half = mulBy(0.5)
	println(triple(14) + " " + half(14))

### 12.7 single abstract method转换

如java中需要通过按钮修改计数器：

	var counter = 0
	val button = new JButton("Increment")
	button.addActionListener(new ActionListener {
	  override def actionPerformed(event: ActionEvent) {
	    counter += 1
	  }
	})

在scala中可以这样写：

	button.addActionListener((event: ActionEvent) => counter += 1)
	button.addActionListener((event: ActionEvent) => println(counter))
	button.addActionListener((event: ActionEvent) => if (counter > 9) System.exit(0))

不过由于参数类型不同，所以需要增加一个隐式转换，即把所有需要ActionListener的地方转换为(ActionEvent => Unit)：

	implicit def makeAction(action: (ActionEvent) => Unit) =
	  new ActionListener {
	    override def actionPerformed(event: ActionEvent) { action(event) }
	  }

### 12.8 柯里化

corresponds 方法比较两个序列在某个比对条件下是否相同，其定义用到了currying

	def corresponds[B](that: GenSeq[B])(p: (A,B) => Boolean): Boolean
	
	val a = Array("Hello", "World")
	val b = Array("hello", "world")
	a.corresponds(b)(_.equalsIgnoreCase(_))

