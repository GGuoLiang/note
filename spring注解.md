# spring注解

## 组件注册

### @Bean

给容器注册一个组件 

```Java
 @Bean(value = "person01") // 往容器中注册组件
    public Person person(){
        return  new Person("李四",19);
    },useDefaultFilters=flase
```

  ~~~xml
  <bean id="person" class="com.guoliang.entity.Person">
        <property name="age" value="18"></property>
        <property name="name" value="张三"></property>
    </bean>
  ~~~

上面俩种方式 注册组件

### @Configuration  

告诉spring的容器的这是一个注解类

### @ComponentScan

包扫描注解  **@Controller @Service @Repository  @Component** 会被扫描到容器

######xml方式

```xml
 <context:component-scan base-package="com.guoliang"/>
```

###### 注解

~~~java 
@ComponentScan(value = "com.guoliang.*")

//需要的排除
@ComponentScan(value = "com.guoliang.*",excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Controller.class, Service.class})
})

//指定扫描的
@ComponentScan(value = "com.guoliang.*",includeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, value = {Controller.class, Service.class})
},useDefaultFilters = false)

~~~

##### @ComponentScans

~~~Java
@ComponentScans(value = {
        @ComponentScan(value = "com.guoliang.*",excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,value = {Controller.class, Service.class})
        })
})
~~~

##### 获取IOC容器的俩种方式

~~~java
// xml配置文件获取容器 
ClassPathXmlApplicationContext xmlApplicationContext = new ClassPathXmlApplicationContext("bean.xml");
        String[] beanDefinitionNames = xmlApplicationContext.getBeanDefinitionNames();
        for (String definitionName : beanDefinitionNames) {
            System.out.println(definitionName);
        }

// 注解获取容器
 AnnotationConfigApplicationContext configApplicationContext = new AnnotationConfigApplicationContext(MainTestConfig.class);
        String[] definitionNames = configApplicationContext.getBeanDefinitionNames();
        for (String definitionName : definitionNames) {
            System.out.println(definitionName);
        }
~~~

