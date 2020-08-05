## Scala

### 面向对象编程（高级）

#### 静态属性和静态方法

##### 基本介绍

Scala 语言是完全面向对象的语言，所以并没有静态的操作（即在 Scala 中没有静态的概念）。但是为了能够和 Java 语言交互（因为 Java 中有静态概念），就产生了一种特殊的对象来模拟类对象，我们称之为类的**伴生对象**。这个类的所有**静态内容**都可以放置在它的伴生对象中声明和调用

##### 快速入门

```scala
object AccompanyObject { 
    def main(args: Array[String]): Unit = {
		println(ScalaPerson.sex) //true 在底层等价于 ScalaPerson$.MODULE$.sex() 
        ScalaPerson.sayHi()//在底层等价于 ScalaPerson$.MODULE$.sayHi()
	}
}

// 说明
// 1. class ScalaPerson 称为伴生类，将非静态的内容写到该类中
// 2. object ScalaPerson 称为伴生对象，将静态的内容写入到该伴生对象中
// 3. class ScalaPerson 编译后底层生成 ScalaPerson 类 ScalaPerson.class
// 4. object ScalaPerson 编译后底层生成 ScalaPerson$ 类 ScalaPerson$.class
// 5. 对于伴生对象的内容，我们可以直接通过 ScalaPerson.属性或者方法

// 伴生类
class ScalaPerson { // 
    var name : String = _
}

// 伴生对象
object ScalaPerson { //
    var sex : Boolean = true
    def sayHi(): Unit = { 
        println("object ScalaPerson sayHI~~")
	}
}
```

##### 小结

1. Scala 中伴生对象采用 object 关键字声明，伴生对象中声明的全是**静态内容**，可以**通过伴生对象名称直接调用**
2. 伴生对象对应的类称为伴生类，伴生对象的名称应该和伴生类名一致
3. 伴生对象中的属性和方法都可以通过伴生对象名（类名）直接调用访问
4. 从语法角度讲，所谓的伴生对象其实就是类的静态方法和成员的集合
5. 从技术角度讲，Scala 还是没有生成静态的内容，只不过是将伴生对象生成了一个新的类，实现属性和方法的调用
6. 从底层原理看，伴生对象实现静态特性是依赖于 public static final MODULE$ 实现的
7. 伴生对象的声明应该和伴生类的声明在同一个源码文件中（如果不在同一个文件中会运行错误）但是如果没有伴生类，也就没有所谓的伴生对象了
8. 如果 class A 独立存在，那么 A 就是一个类，如果 object A 独立存在，那么 A 就是一个**静态**性质的对象【即类对象】，在 object A 中声明的属性和方法可以通过 A.属性 和 A.方法来实现调用

#### 接口

Scala 语言中，采用特质 trait 来代替接口的概念

##### trait 的声明

```scala
trait 特质名 {
    trait 体
}
```

##### Scala 中 trait 的使用

1. 没有父类

   ```scala
   class 类名 extends 特质1 with 特质2 with 特质3
   ```

2. 有父类

   ```scala
   class 类名 extends 父类 with 特质1 with 特质2 with 特质3
   ```

   

### 模式匹配

#### match

Scala 中的模式匹配类似于 Java 中的 switch 语法

```scala
// 入门案例
object MatchDemo01 { 
    def main(args: Array[String]): Unit = { 
        val oper = '-' 
        val n1 = 20 
        val n2 = 10
		var res = 0
        // 1. match(类似 java switch) 和 case 是关键字
        // 1.1 可以在 match 中使用其他类型，不仅仅是字符
        // 2. 如果匹配成功则执行 => 后面的代码块
        // 3. 匹配的顺序是从上到下，匹配到一个就执行对应的代码
        // 4. => 后面的代码块，不用写 break，会自动的退出 match
        // 5. 如果一个都没有匹配到，则执行 case _ 后面的代码块
        // 6. 如果所有 case 都不匹配，又没有写 case _ 分支，则会抛出 MatchError
        oper match {
            case '+' => res = n1 + n2 
            case '-' => res = n1 - n2 
            case '*' => res = n1 * n2 
            case '/' => res = n1 / n2
			case _ => println("oper error")
        }
        println("res=" + res)
    }
}
```

#### 守卫

如果想要表达匹配某个范围的数据，就需要在模式匹配中增加条件守卫

```scala
object MatchIfDemo01 { 
    def main(args: Array[String]): Unit = {
        for (ch <- "+-3!") { //是对"+-3!" 遍历 
            var sign = 0 
            var digit = 0 
            ch match {
				case '+' => sign = 1
                case '-' => sign = -1 
                // 说明.. 
                // 如果 case 后有 条件守卫即 if ,那么这时的 _ 不是表示默认匹配 
                // 表示忽略 传入 的 ch 
                case _ if ch.toString.equals("3") => digit = 3 				   case _ if (ch > 1110 || ch < 120) => println("ch > 10")
				case _ => sign = 2
            }
            // + 1 0
            // - -1 0
            // 3 0 3
            // ! 2 0
            println(ch + " " + sign + " " + digit)
        }
    }
}
```

#### 模式中的变量

如果在 case 关键字后跟变量名，那么 match 前表达式的值会赋给那个变量

```scala
object MatchVar {
    def main(args: Array[String]): Unit = {
        val ch = 'U'
        ch match { 
            case '+' => println("ok~") 
            // 下面 case mychar 含义是 mychar = ch 
            case mychar => println("ok~" + mychar) 
            case _ => println ("ok~~")
		}
        
        val ch1 = '+'
        val res = ch1 match { 
            case '+' => ch1 + " hello "
            // 下面 case mychar 含义是 mychar = ch
            case _ => println ("ok~~")
		}
        
        println("res=" + res)
        // ok~U
		// res=+ hello
    }
}
```

#### 类型匹配

可以匹配对象的任意类型，这样做避免了使用 isInstanceOf 和 asInstanceOf 方法

```scala
object MatchTypeDemo01 { 
    def main(args: Array[String]): Unit = {
        val a = 8 
        //说明 obj 实例的类型 根据 a 的值来返回 
        val obj = if (a == 1) 1 
        else if (a == 2) "2" 
        else if (a == 3) BigInt(3) 
        else if (a == 4) Map("aa" -> 1) 
        else if (a == 5) Map(1 -> "aa") 
        else if (a == 6) Array(1, 2, 3) 
        else if (a == 7) Array("aa", 1)
		else if (a == 8) Array("aa")
        
        //说明 
        //1. 根据 obj 的类型来匹配
		// 返回值
        val result = obj match {
            case a: Int => a 
            case b: Map[String, Int] => "对象是一个字符串-数字的Map 集合"
			case c: Map[Int, String] => "对象是一个数字-字符串的Map 集合"
            case d: Array[String] => d //"对象是一个字符串数组"
            case e: Array[Int] => "对象是一个数字数组" 
            case f: BigInt => Int.MaxValue
			case _ => "啥也不是"
        }
        
        println(result)
    }
}
```

类型匹配注意事项：

1. 如果 case _ 出现在 match 中间，则表示隐藏变量名，即不使用，而不是表示默认匹配

#### 匹配数组

1. Array(0) 匹配只有一个元素且为 0 的数组
2. Array(x, y) 匹配数组有两个元素，并将两个元素赋值为 x 和 y
3. Array(0, _*) 匹配数组以 0 开始

#### 匹配列表

1. 0 :: Nil 匹配只有一个元素且为 0 的列表
2. x :: y :: Nil 匹配只有两个元素，并且将两个元素赋值为 x 和 y
3. 0 :: tail 匹配以 0 开头的列表
4. x :: Nil 匹配只有一个元素，并且将元素赋值给 x

#### 匹配元组

```scala
object MatchTupleDemo01 { 
    def main(args: Array[String]): Unit = {
        for (pair <- Array((0, 1), (1, 0), (10, 30), (1, 1), (1, 0, 2))) {
            val result = pair match {
                case (0, _) => "0 ..."
                case (y, 0) => y
                case (x, y) => (y, x)
                case _ => "other"
            }
            println(result)
        }
    }
}

// 0 ...
// 1
// (30, 10)
// (1, 1)
// other
```

#### 对象匹配

1. case 中对象的 unapply 方法（对象提取器）返回 Some 集合则为匹配成功
2. 返回 None 集合则为匹配失败

```scala
object MatchObject { 
    def main(args: Array[String]): Unit = {
        // 模式匹配使用：
        val number: Double = Square(5.0)// 25.0 // 
        number match { 
            //说明 case Square(n) 的运行的机制 
            //1. 当匹配到 case Square(n) 
            //2. 调用 Square 的 unapply(z: Double),z 的值就是 number 
            //3. 如果对象提取器 unapply(z: Double) 返回的是 Some(5) ,则表示匹配成功，同时将 5 赋给 Square(n) 的 n 
            //4. 如果对象提取器 unapply(z: Double) 返回的是None ,则表示匹配不成功
            case Square(n) => println("匹配成功 n=" + n) 
            case _ => println("nothing matched")
        }
    }
}

object Square { 
    //说明 
    //1. unapply 方法是对象提取器 
    //2. 接收 z:Double 类型 
    //3. 返回类型是Option[Double] 
    //4. 返回的值是 Some(math.sqrt(z)) 返回 z 的开平方的值，并放入到 Some(x)
	def unapply(z: Double): Option[Double] = {
        println("unapply 被调用 z 是=" + z) 
        //Some(math.sqrt(z))
		None
    }
    def apply(z: Double): Double = z * z
}
```

#### 变量声明中的模式

match 中的每一个 case 都可以单独提取出来，意思是一样的

```scala
object MatchVarDemo { 
    def main(args: Array[String]): Unit = {
		val (x, y, z) = (1, 2, "hello") 
        println("x=" + x) 
        val (q, r) = BigInt(10) /% 3 //说明 q = BigInt(10) / 3 r = BigInt(10) % 3 
        val arr = Array(1, 7, 2, 9) 
        val Array(first, second, _*) = arr // 提出 arr 的前两个元素
        println(first, second)
	}
}
// x=1
// (1,7)
```

#### for 表达式中的模式

```scala
object MatchForDemo { 
    def main(args: Array[String]): Unit = { 
        val map = Map("A" -> 1, "B" -> 0, "C" -> 3) 
        for ((k, v) <- map) { 
            println(k + " -> " + v) // 出来三个 key-value ("A"->1), ("B"->0), ("C"->3)
		} 
        //说明 : 只遍历出 value =0 的 key-value ,其它的过滤掉
        println("--------------(k, 0) <- map-------------------") 
        for ((k, 0) <- map) { 
            println(k + " --> " + 0)
		}
		//说明, 这个就是上面代码的另外写法, 只是下面的用法灵活和强大 
        println("--------------(k, v) <- map if v == 0-------------------") 
        for ((k, v) <- map if v >= 1) { 
            println(k + " ---> " + v)
		} 
    }
}
/*
A -> 1
B -> 0
C -> 3
--------------(k, 0) <- map-------------------
B --> 0
--------------(k, v) <- map if v == 0-------------------
A ---> 1
C ---> 3
*/
```

#### 样例类

```scala
object CaseClassDemo01 { 
    def main(args: Array[String]): Unit = { 							println("hello~~")
	}
}

abstract class Amount
case class Dollar(value: Double) extends Amount //样例类
case class Currency(value: Double, unit: String) extends Amount //样例类
case object NoAmount extends Amount //样例类
//类型(对象) =序列化(serializable)==>字符串(1.你可以保存到文件中【freeze】2.反序列化,2 网络传输)
```

基本介绍：

1. 样例类仍然是类
2. 样例类用 case 关键字进行声明
3. 样例类是为模式匹配而优化的类
4. 构造器中的每一个参数都成为 val——除非它被显式地声明为 var（不建议）
5. 在样例类对应的伴生对象中提供 apply 方法让你不用 new 关键字就能构造出相应的对象
6. 提供 unapply 方法让模式匹配可以工作
7. 将自动生成 toString、equals、hashCode 和 copy 方法

样例类最佳实践：

```scala
object CaseClassDemo02 { 
    def main(args: Array[String]): Unit = {
        //该案例的作用就是体验使用样例类方式进行对象匹配简洁性
        for (amt <- Array(Dollar2(1000.0), Currency2(1000.0, "RMB"), NoAmount2)) {
            val result = amt match {
                // 说明
                case Dollar2(v) => "$" + v // $1000.0
                // 说明
                case Currency2(v, u) => v + " " + u // 1000.0 RMB
                case NoAmount2 => "" // ""
            }
            println(amt + ": " + result)
        }
    }
}

abstract class Amount2
case class Dollar2(value: Double) extends Amount2 //样例类
case class Currency2(value: Double, unit: String) extends Amount2 //样例类
case object NoAmount2 extends Amount2 //样例类
```

#### case 语句中的中置表达式

