###Spring boot自动装配原理
####@SpringBootApplication
- 启动类配置注解，是一个组合注解

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

#####@SpringBootConfiguration
- 组合注解之一，表名该类是Spring的一个配置

#####@EnableAutoConfiguration
- 开启**自动配置**的注解

```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({EnableAutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
    String ENABLED_OVERRIDE_PROPERTY = "spring.boot.enableautoconfiguration";

    Class<?>[] exclude() default {};

    String[] excludeName() default {};
}
```
   - @AutoConfigurationPackage注解是为了给Spring容器导入一个组件`Registrar.class`,说白了就是讲主配置类所在的包及子包里面的组件扫描加载到Spring容器中。
   - @Import({EnableAutoConfigurationImportSelector.class})，`EnableAutoConfigurationImportSelector.class`类继承自`AutoConfigurationImportSelector`，其中有一个方法：`String[] selectImports(AnnotationMetadata annotationMetadata)`会将需要导入的组件以全类名的方式返回，并将这些组件添加到容器中；

   ```
   public String[] selectImports(AnnotationMetadata annotationMetadata) {
        if (!this.isEnabled(annotationMetadata)) {
            return NO_IMPORTS;
        } else {
            try {
                AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
                AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
                List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
                configurations = this.removeDuplicates(configurations);
                configurations = this.sort(configurations, autoConfigurationMetadata);
                Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
                this.checkExcludedClasses(configurations, exclusions);
                configurations.removeAll(exclusions);
                configurations = this.filter(configurations, autoConfigurationMetadata);
                this.fireAutoConfigurationImportEvents(configurations, exclusions);
                return (String[])configurations.toArray(new String[configurations.size()]);
            } catch (IOException var6) {
                throw new IllegalStateException(var6);
            }
        }
    }
   ```
   其中`getCandidateConfigurations(annotationMetadata, attributes)`方法会去加载`META-INF/spring.factories`中的信息，并将这些值导入到容器中，其中`META-INF/spring.factories`文件位于`spring-boot-autoconfigure`包中。
   

[引用](https://www.cnblogs.com/hhcode520/p/9450933.html)