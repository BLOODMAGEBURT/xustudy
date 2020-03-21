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
        return 
  
}
```

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



#### 4.类型通配符

#### 5.类型擦除

#### 6.泛型与数组

#### 7.泛型与反射