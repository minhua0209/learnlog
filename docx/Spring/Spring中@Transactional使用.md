##Spring中@Transactional使用
###基本步骤
- XML添加事物配置信息

```
<tx:annotation-driven />
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource" />
</bean>
```
- 将`@Transactional`注解添加到合适的方法上，并设置合适的属性信息
   - name:当在配置文件中有多个 TransactionManager , 可以用该属性指定选择哪个事务管理器
   - propagation:事务的传播行为，默认值为 REQUIRED
   - isolation:事务的隔离度，默认值采用 DEFAULT
   - timeout:事务的超时时间，默认值为-1。如果超过该时间限制但事务还没有完成，则自动回滚事务
   - read-only:指定事务是否为只读事务，默认值为 false；为了忽略那些不需要事务的方法，比如读取数据，可以设置 read-only 为 true
   - rollback-for:用于指定能够触发事务回滚的异常类型，如果有多个异常类型需要指定，各类型之间可以通过逗号分隔
   - no-rollback- for:抛出 no-rollback-for 指定的异常类型，不回滚事务

除此以外，@Transactional 注解也可以添加到类级别上。当把@Transactional 注解放在类级别时，表示所有该类的公共方法都配置相同的事务属性信息。见清单 2，EmployeeService 的所有方法都支持事务并且是只读。当类级别配置了@Transactional，方法级别也配置了@Transactional，应用程序会以方法级别的事务属性信息来管理事务，换言之，方法级别的事务属性信息会覆盖类级别的相关配置信息

###Spring注解方式的事物实现机制
默认使用AOP代理，在代码运行时生成一个代理对象，根据`@Transactional`的属性配置信息，这个代理对象决定该声明`@Transactional`的目标方法是否由拦截器`TransactionInterceptor`来使用拦截，在`TransactionInterceptor`拦截时，会在在目标方法开始执行之前创建并加入事务，并执行目标方法的逻辑, 最后根据执行情况是否出现异常，利用抽象事务管理器`AbstractPlatformTransactionManager`操作数据源`DataSource`提交或回滚事务，其中Spring AOP 代理有`CglibAopProxy`和`JdkDynamicAopProxy`两种

###注意事项
####正确设置 @Transactional 的 propagation 属性
- 以下三种配置事务将不会回滚
   - `TransactionDefinition.PROPAGATION_SUPPORTS`：如果当前存在事务，则加入该事务；如果当前没有事务，则以非事务的方式继续运行
   - `TransactionDefinition.PROPAGATION_NOT_SUPPORTED`：以非事务方式运行，如果当前存在事务，则把当前事务挂起
   - `TransactionDefinition.PROPAGATION_NEVER`：以非事务方式运行，如果当前存在事务，则抛出异常 
   
####正确的设置@Transactional 的 rollbackFor 属性
- 默认情况下，如果在事务中抛出了未检查异常（继承自`RuntimeException`的异常）或者`Error`，则 Spring 将回滚事务；除此之外，Spring 不会回滚事务
- 如果期望Spring能回滚其他类型的异常，可指定`rollbackFor`。

```
@Transactional(propagation= Propagation.REQUIRED,rollbackFor= MyException.class)
```
若在目标方法中抛出的异常是 rollbackFor 指定的异常的子类，事务同样会回滚

####@Transactional 只能应用到 public 方法才有效
####避免 Spring 的 AOP 的自调用问题
- 若同一类中的其他没有`@Transactional`注解的方法内部调用有`@Transactional`注解的方法，有`@Transactional`注解的方法的事务被忽略，不会发生回滚;

例：

```
@Service
public class OrderService {
    private void insert() {
insertOrder();
}
@Transactional
    public void insertOrder() {
        //insert log info
        //insertOrder
        //updateAccount
       }
}
```
Spring默认使用代理类来对方法添加事物，同Spring boot 的`@Async`注解一样，自调用时，调用者和被注释对象都在同一个类中，调用时会直接进行调用，不经过代理类调用，因此不产生效果


####AspectJ 取代 Spring AOP 代理
- 注解只应用到 public 方法和自调用问题，是由于使用 Spring AOP 代理造成的，可以使用AspectJ 取代 Spring AOP 代理

配置

```
<tx:annotation-driven mode="aspectj" />
<bean id="transactionManager"
class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
<property name="dataSource" ref="dataSource" />
</bean>
</bean
class="org.springframework.transaction.aspectj.AnnotationTransactionAspect"
factory-method="aspectOf">
<property name="transactionManager" ref="transactionManager" />
</bean>
```
同时在 Maven 的 pom 文件中加入 spring-aspects 和 aspectjrt 的 dependency 以及 aspectj-maven-plugin

```
<dependency>
<groupId>org.springframework</groupId>
<artifactId>spring-aspects</artifactId>
<version>4.3.2.RELEASE</version>
</dependency>
<dependency>
<groupId>org.aspectj</groupId>
<artifactId>aspectjrt</artifactId>
<version>1.8.9</version>
</dependency>
<plugin>
<groupId>org.codehaus.mojo</groupId>
<artifactId>aspectj-maven-plugin</artifactId>
<version>1.9</version>
<configuration>
<showWeaveInfo>true</showWeaveInfo>
<aspectLibraries>
<aspectLibrary>
<groupId>org.springframework</groupId>
<artifactId>spring-aspects</artifactId>
</aspectLibrary>
</aspectLibraries>
</configuration>
<executions>
<execution>
<goals>
<goal>compile</goal>
<goal>test-compile</goal>
</goals>
</execution>
</executions>
</plugin>
```
   
