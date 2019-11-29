###Spring AOP的简单使用
####Maven配置

```
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-aop</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>${aspectj.version}</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjtools</artifactId>
            <version>${aspectj.version}</version>
        </dependency>
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjrt</artifactId>
            <version>${aspectj.version}</version>
            <scope>runtime</scope>
        </dependency>
        <dependency>
            <groupId>cglib</groupId>
            <artifactId>cglib</artifactId>
            <version>3.1</version>
        </dependency>
```

####appContext.xml配置

```
<!--进行类对象代理，使用cglib进行代理-->
<aop:aspectj-autoproxy proxy-target-class="true"/>
<!--告诉spring去扫瞄这个路径下-->
<context:component-scan base-package="com.minhua.Zeus"/>
```

####切面程序

```
@Slf4j
@Component
@Aspect
public class MyAopLogTest {
    @Around(value = "execution(* com.minhua.Zeus.service.impl.PayChannelServiceImpl.getPaychannel2(..))")
    public Object myAop(ProceedingJoinPoint point) {
        log.info("成功调用切面了");
        Object obj = null;
        try {
            obj =  point.proceed();
        }catch (Throwable ex) {
            ex.printStackTrace();
        }
        log.info("尾部切面");
        return obj;
    }

}
```

注意：
- 必须用@Component注解该类
- 返回返回值为Object,其中point.proceed()指代被切方法，如有返回值，通知是需要返回对象的
- 被切面修饰的方法必须是public或protected


     
####基于自定注解的通知

```
@Around(value = "@annotation(com.minhua.Zeus.aop.annocation.MyLogerAop)")
    public Object myAop2(ProceedingJoinPoint point) {
        log.info("成功调用切面了");
        Object obj = null;
        try {
            obj =  point.proceed();
        }catch (Throwable ex) {
            ex.printStackTrace();
        }
        log.info("尾部切面");
        return obj;
    }
```

自定义注解

```
package com.minhua.Zeus.aop.annocation;

import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * @author: M.H.
 * @date: 2019/8/5
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyLogerAop {
}
```