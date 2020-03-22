### java基础之泛型

[TOC]

> 泛型概述
>
> jdk5中引入的新特性，泛型的本质就是参数化类型，也就是所操作的类型指定为一个参数 
>
> 编译期的类型检查：ArrayList<String> list = new ArrayList();
>
> 使用数据时减少类型转换
>
> 

#### 1.泛型类

##### 1.1基本语法

```java
class 类名<泛型标识， 泛型标识，...>{

  // 作为变量类型
	private 泛型标识 A;
	
	// 作为函数参数类型
  private void setXXX(泛型标识 xxx){

  }
  
  //作为函数返回值类型
  private 泛型标识 getXXX(){
    return 泛型标识
  }
}

// 例如 ArrayList
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
  
  public E get(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        return (E) elementData[index];
    }
  
  public E set(int index, E element) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));

        E oldValue = (E) elementData[index];
        elementData[index] = element;
        return element;
  
}
```

*在实例化类时，指定泛型的具体类型：ArrayList<String> list = new ArrayList<>();*

##### 1.2常用泛型标识

T E K V

泛型的类型参数只能是类类型，**不能**是基本数据类型（因为中间需要先转化为object类型，然后转换为泛型类型，而基本数据类型**不能**转换为object）

##### 1.3泛型子类的两种情况 

分为两种情况：

```
// 子类也是泛型类，则与父类保持一致
class child<T> extends parent<T>{}
// 子类不是泛型类，则父类需要指定类型
class child extends parent<String>{}
```



#### 2.泛型接口

与泛型类类似

#### 3.泛型方法

调用方法时，指定泛型的具体类型

泛型方法能使方法独立于类而产生变化

如果static方法要想有泛型能力，必须成为泛型方法

```java
// 基本语法
修饰符 <T,E,...> 返回值类型 方法名(形参){

} 

   /**
     * 
     * @param <E>  泛型标识，具体类型由调用时确定
     * @param list 参数
     * @return
     */
    public <E> E getItem(ArrayList<E> list){

        return list.get(new Random().nextInt(list.size()));
    }

		
```



#### 4.类型通配符

<?> 代表任意类型

指定泛型通配符的上限 <? extends 类型> ： 子类及本身

```java
// List<E> 中的addAll方法
boolean addAll(Collection<? extends E> c);
```

指定类型通配符的下限 <? super 类型 >    : 父类及本身

```java
// treeSet中的构造方法
public TreeSet(Comparator<? super E> comparator) {
        this(new TreeMap<>(comparator));
    }
```



#### 5.类型擦除

泛型信息只在编译器有效，在进入JVM之前，与泛型相关的信息会被擦除掉，我们称之为-类型擦除

```java
   public static void show(){
        // 类型擦除
        ArrayList<Integer> intList = new ArrayList<>();
        ArrayList<String> strList = new ArrayList<>();

        System.out.println(intList.getClass().getSimpleName());
        System.out.println(strList.getClass().getSimpleName());
    }
    // 输出的结果都是ArrayList
```

无限制类型擦除：将类型保留为Object

![无限制类型](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20200322152659.png)

有限制类型擦除：将类型保留为父类

![](https://mkdown-1256191338.cos.ap-beijing.myqcloud.com/md/20200322153254.png)

#### 6.泛型与数组

**注意：** 可以创建泛型类型的数组饮用，但是不能创建泛型类型的数组对象

因为数组会一直持有引用，而泛型会在进入JVM之前被擦除掉

~~ArrayList<String> [] listArr = new ArrayList<>[5];~~  不可以，会报错

两种方式创建泛型数组：

ArrayList<String> [] listArr = new ArrayList[5];

Array.newInstance(Class<?> clz, int length);

#### 7.泛型与反射

反射中常用的泛型类

Class<T>

Constructor<T>

```java
public class Main {

    public static void main(String[] args) throws NoSuchMethodException {
	// write your code here

        Class<Main> mainClass = Main.class;

        Constructor<Main> constructor = mainClass.getConstructor();

        ArrayList<String> strings = new ArrayList<>();

    }
}
```





