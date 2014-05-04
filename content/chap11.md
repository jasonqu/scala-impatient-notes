### 11.3 一元操作符

<code>+ - ！ ~</code> 可作为一元操作符，会转为unary_op 方法

### 11.7 apply & update

	val scores = new scala.collection.mutable.HashMap[String, Int]
	scores("Bob") = 100       // 调用scores.update("Bob", 100)
	val score = scores("Bob") // 调用scores.apply("Bob")
	
### 11.8-10 提取器 unapply

unapply的返回值应该是 Option类型 或boolean类型【用于测试输入】

unapplySeq 返回 Option[Seq[A]]，有了这个方法就可以匹配多个变量了:

	object Name {
	  def unapplySeq(input: String): Option[Seq[String]] =
	    if (input.trim == "") None else Some(input.trim.split("\\s+"))
	}
	
	val author = "Peter van der Linden"
	
	author match {
	  case Name(first, last) => author
	  case Name(first, middle, last) => first + " " + last
	  case Name(first, "van", "der", last) => "Hello Peter!"
	}

TODO 练习