###Java注解
- Java注解用于为Java代码提供元数据<br/>
 **元数据**是指用来描述数据的数据，通俗一点，就是描述代码间关系，或者代码与其它资源（例如数据库表）之间内在联系的数据<br/>
 JDK5.0出来后，Java语言中就有了四种类型，即类class、枚举enum、接口interface、注解@interface，它们处于同一级别，Java就是通过注解来表示元数据的
 
####注解作用
- 提供信息给编译器：编译器可以利用注解来探测错误或警告信息
- 编译阶段时的处理：软件工具可以利用注解信息来生成代码、HTML文档或其它响应处理
- 运行时的处理：某些注解可以在程序运行时接受代码的提取
**值得注意的是**，注解不是代码本身的一部分
 
####元注解
注解到注解上的注解

- `@Retention`<br/>
当`@Retention`应用到注解上的时候，它解释说明了这个注解的生命周期
   - `RetentionPolicy.SOURCE`只在源码阶段保留，在编译器进行编译时它将被丢弃忽视
   - `RetentionPolicy.CLASS`只被保留到编译进行的时候，它并不会被加载到JVM中
   - `RetentionPolicy.RUNTIME`可以保留到程序运行的时候，它会被加载到JVM中
- `@Documented`<br/>
能够将注解中的元素包含到Javadoc中去
- `@Target`<br/>
标明注解运用的地方
   - `ElementType.ANNOTATION_TYPE`可以给一个注解进行注解
   - `ElementType.CONSTRUCTOR`可以给构造方法进行注解
   - `ElementType.FIELD`可以给属性进行注解
   - `ElementType.LOCAL_VARIABLE`可以给局部变量进行注解
   - `ElementType.METHOD`可以给方法进行注解
   - `ElementType.PACKAGE`可以给一个包进行注解
   - `ElementType.PARAMETER`可以给一个方法内的参数进行注解
   - `ElementType.TYPE`可以给一个类型进行注解，比如类、接口、枚举 
- `@Inherited`<br/>
如果一个超类被@Inherited注解过的注解进行注解的话，那么如果它的子类没有被任何注解应用的话，那么这个子类就继承了超类的注解

代码实例：

```
package OSChina.ClientNew;

import java.lang.annotation.ElementType;
import java.lang.annotation.Inherited;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Retention(RetentionPolicy.RUNTIME)//注解可以保留到程序运行时，加载到JVM中
@Target(ElementType.TYPE)//给一个类型进行注解，比如类、接口、枚举
@Inherited //子类继承父类时，注解会起作用
public @interface Desc {
    enum Color {
        White, Grayish, Yellow
    }

    // 默认颜色是白色的
    Color c() default Color.White;
}
```
- `@Repeatable`<br/>
@Repeatable 是 Java 1.8 才加进来的,多次运用的注解

例子：<br/>
1.先声明一个Persons注解

```
@Target(ElementType.TYPE)  
@Retention(RetentionPolicy.RUNTIME)
public @interface Persons {
	Person[] value();
}
```

2.声明一个Person注解

```
@Repeatable(Persons.class)
public @interface Person{
	String role() default "";
}
```
`@Repeatable`括号内的就相当于用来保存该注解内容的容器

3.声明一个Man类，给该类加上一些身份

```
@Person(role="CEO")
@Person(role="husband")
@Person(role="father")
@Person(role="son")
public class Man {
	String name="";
}
```

4.访问

```
if(Man.class.isAnnotationPresent(Persons.class)) {
    Persons p2=Man.class.getAnnotation(Persons.class);
    for(Person t:p2.value()){
        System.out.println(t.role());
    }
 }
```
####注解的属性
注解的属性也叫做成员变量，注解只有成员变量，没有方法。注解的成员变量在注解的定义中以“无参的方法”形式来声明，其方法名定义了该成员变量的名字，其返回值定义了该成员变量的类型

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
    int id();
    String msg();
}
```
调用方式

```
@TestAnnotation(id=3,msg="hello annotation")
public class Test {
}
```


注解中属性可以有默认值，默认值需要用`default`关键值指定

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
public @interface TestAnnotation {
    public int id() default -1;
    public String msg() default "江疏影";
}
```
如果一个注解内仅仅只有一个名字为`value`的属性时，应用这个注解时可以直接接属性值填写到括号内，一个注解没有任何属性时，括号都可以省略

####注解与反射
- 注解通过反射获取。首先可以通过 Class 对象的 isAnnotationPresent() 方法判断它是否应用了某个注解

```
public boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {}
```
或者

```
public Annotation[] getAnnotations() {}
```

前一种方法返回指定类型的注解，后一种方法返回注解到这个元素上的所有注解

实例：

```
package OSChina.ClinetNew1.Annotation;

@TestAnnotation
public class Test {
    public static void main(String[] args) {
        boolean hasAnnotation = Test.class.isAnnotationPresent(TestAnnotation.class);
        if(hasAnnotation){
            TestAnnotation testAnnotation = Test.class.getAnnotation(TestAnnotation.class);
            System.out.println("id:"+testAnnotation.id());
            System.out.println("msg:"+testAnnotation.msg());
        }
    }
}
```
**需要注意的是**，如果一个注解要在运行时被成功提取，那么 `@Retention(RetentionPolicy.RUNTIME)`是必须的