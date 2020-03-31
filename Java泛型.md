# 泛型
## 为什么设计泛型
因为我们不想要为聚集不同的类设计不同的代码（虽然可以用继承实现，但是继承不能对代码进行行错误检查），同时使代码容易阅读。简单来说就是泛型会使代码具有更高的安全性和可读性。

----

## 泛型类
引入一个类型变量T，使用<>括起来并放在类名后面（如下所示），泛型类可以有多个变量写法如：class Pair<U,S>{...}
```
class Pair<T> {
  private T data;
  public T getData(){
    return data;
  }
  public void setData(T data){
    this.data = data;
  }
}

public static void main(String[] args){
  Pair pair = new Pair<String>();
  pair.setData("this is data");
  System.out.println(pair.getData());
}
```

----

## 泛型方法
类型变量放在修饰符后面，返回类型之前（如下）。可以定义在普通类中也可以定义在泛型类中
```
class ArrayTool{
  public static <T> T getMiddleData(T... t){
      return t[t.length/2];
  }
}

public static void main(String[] args){
  ArrayTool.<String>getMiddleData("one","two","three");
  //以下是简写效果一样
  ArrayTool.getMiddleData("one","two","three");
}
```

----

## 类型变量的限定
有时候我们需要对类型变量加以约束，用于缩小类型变量的取值范围达到某些操作如compareTo。添加限定的方法是使用extends关键字，一个类型变量或通配符可以有多个限定写法如：T extends Comparable & Serializable
```
class ArrayTool{
  public static <T extends Comparable> T getMin(T... t){
    if(t == null||t.length<=0)return null;
    T min = t[0];
    for(int i=0;i<t.length;i++){
      if(min.compareTo(t[i])>0){
        min = t[i];
      }
    }
    return min;
  }
}
```

----

## 泛型代码和虚拟机
对于虚拟机来时所有的类都是普通类，是没有泛型类型对象的。所以无论何时定义了一个泛型类型，都会自动提供一个原始类型。原始类型就是删去类型参数后的泛型类型名，擦除类型变量，替换为原始类型变量。
如 class Pair<T> 的原始类型如下,因为T是个无限定的变量，所以直接用Object替换
```
class Pair {
  private Object data;
  public Object getData(){
    return data;
  }
  public void setData(Object data){
    this.data = data;
  }
}
```
如果使用了类型变量限定符
```
class Pair<T extends Comparable> {
  private T data;
  public T getData(){
    return data;
  }
  public void setData(T data){
    this.data = data;
  }
}
```
原始类型就变成如下
```
class Pair {
  private Comparable data;
  public Comparable getData(){
    return data;
  }
  public void setData(Comparable data){
    this.data = data;
  }
}
```

### 翻译泛型表达式
当程序调用泛型方法时，如果擦除返回类型，编译器插入强制类型转换
```
Pair<Employee> buddies = new Pair<>();
//getData返回的擦除类型时Object，编译器会自动插入强制转换类型Employee
buddies.getData();
//若data是共有的也会自动强转
buddies.data
```

### 翻译泛型方法
* 虚拟机中没有泛型，只有普通类和方法  
* 所有的类型参数都有他们的限定类型替换
* 桥方法被合成来保持多态
* 为保持类型安全性，必要时插入强制类型转换

### 调用遗留代码
* 将一个泛型对象赋值给遗留的代码。
* 由一个遗留代码得到一个原始对象，将这个原始对象赋值给参数化的类型变量。  

以上都会看到一个警告，查看警告之后可以使用注解使之消失

----

## 约束与局限性
### 不能用基本类型实例化类型参数
  因为对象擦除之后的类型是Object，而Object不能存储基本类型。

### 运行时类型查询只适用于原始类型
  * 虚拟机中的对象总有一个特定的非泛型类型，因此所有的类型查询只产生原始类型
  ```
    if(a instanceof Pair<String>)//ERROR
    if(a instanceof Pair<T>)//ERROR
    Pair<String> p = (Pair<String>)a;//WARNING 只能转换为Pair
  ```
  * getClass总是返回原始类型

### 不能创建参数化类型的数组
  因为数组会记住它的元素类型，如果存储其他元素，会抛出一个ArrayStoreException。若使用泛型，擦除之后会使数组这个机制无效。

### Varatgs警告
  在可变参数的方法中使用泛型，会抛出一个警告，可以通过添加@SupporWarning到调用方法的函数或@SafeVarargs直接修饰方法消除警告。

### 不能实例化类型变量
  不能使用类似new T(...),new T[....]或T.class这样的表达式，必须像下面这样的API可以支配Class对象
  ```
    public static <T> Pair<T> makePair(Class<T> cls){
        try{
          return new Pair<>(cls.newInstance(),cls.newInstance());
        }catch(Exception e){
          return null;
        }
    }

    Pair<String> pairString = makePair(String.class);
  ```

### 泛型类的静态上下文中类型变量无效

禁止使用带有类型变量的静态域或方法
```
public class Singleton<T>{
  private static T singleInstance;
  public static T getInstance(){
    if(singleInstance == null){
      //构造器创建一个对象
    }
    return singleInstance;
  }
}
```
以上代码是不能运行的，擦除类型之后只剩下Singlton类，他只包含一个singleInstance域。

### 不能抛出或者捕获泛型类的实例
以下行为是不可实现的
```
//泛型扩展Throwable，例如
public class Problem<T> extends Exception{}
```

```
//catch子句中不能适用类型变量
public static <T extends Throwable> void doWork(Class<T> t){
  try{
    //doWork
  }catch(T e){

  }
}
```
在异常规范中使用类型变量是被允许的
```
public static <T extends Throwable> void doWork(T t) throws Exception{
  try{
    //doWork
  }catch(Throwable e){
    t.initCase(e);
    throw t;
  }
}
```

### 可以消除对受检异常的检查

### 注意擦除后的冲突
1. 当类型擦除之后方法重载问题，例如

```
public class Person<T>{
  //方法将不能创建，因为擦除后该方法和Object的equals(Object obj)相同
  public boolean equals(T t){
    //do something
  }
}
```

2. 类型变量不能实现多个是同一接口不同参数话的两个接口

```
class Person implements Comparable<String>{}
//下面这个类将不被允许
class Manager extends Person implements Comparable<Employee>{

}
```

但下面的实现确实合法的

```
class Person implements Comparable{}
//下面这个类却被允许
class Manager extends Person implements Comparable{

}
```
----

## 泛型类型的继承规则
1. 假设Manager是Person的子类，但是Pair<Manger>并不是Pair<Person>的子类。
2. 永远可以将参数化类型转换为一个原始类型。但是转换之后会出现类型错误。

  ```
    Pair<Manager> pairType = new Pair<>();
    Pair pair = pairType;

  ```

3. 泛型类可以扩展和实现其他的泛型类。
----

## 通配符类型（？）
通配符类型中，允许参数类型变化。
### 通配符的子类型限定 （extends）
```
//例子
 Pair<? extends Employee> pair = new Pair<>();
//这时pair的set和get方式类似以下
//不能调用，因为编译器知道需要某个Employee的子类型，但不知道具体类型。
setFirst(? extends Employee){......}
//可以调用，将返回值赋值给Employee的引用是合法的。
? extends Employee getFirst()

```

### 通配符的超类型限定 （super）
```
//这个通配符限制为Manager的所有超类
Pair<? super Manager> pair = new Pair<>();
//这时pair的set和get方法类似
//不能调用，因为编译器不知道具体类型,不能传Employee或者Object只能传Manager，或者某个子类型。
setFirst(? super Manager){......}
//不能保证返回值类型，只能将其赋值给Object。
? super Manager getFirst()

```

```

```

### 无限定通配符
```
//与原始类型的本质区别是可以用任意 Object 对象调用原始 Pair 类的 setObject
方法
Pair<?> pair = new Pair<>();
//这时pair的set和get方法类似
//不能调用，甚至不能传Object。可以调用setFirst(null)
void setFirst(?){......}
//不能保证返回值类型，只能将其赋值给Object。
? Manager getFirst()
```

### 通配符捕获
```

public static void swap(Pair<?> pair){
    swapHelper(pair);
}
//这时T捕获通配符‘？’，
public static <T> void swapHelper(Pair<T> pair){
    T t = pair.getFirst();
    pair.setFirst(pair.getSecond());
    pair.setSecond(t);
}
```

----

## 反射和泛型

----
