## 基础

### 基础语法

#### hashCode()与equals()

1. 如果根据equals(Object)方法，两个对象相等，则hashCode一定相同
2. 两个对象有相同的hashcode，也不一定相等
3. equals方法被重写，hashCode方法也必须被重写

##### equals()特性

自反性；对称性；传递性；一致性；x.equals(null)=false

#### ==与equals的区别

对于基本类型，==比较的是值

对于引用类型（包括包装类型），如果没有被重写，==比较的是两个引用是否指向同一个对象地址；如果被重写，比较的是地址里的内容

### 包装类

#### String

