# mybaties全局配置文件

官方文档：[https://mybatis.org/mybatis-3/zh/configuration.html]()

##properties（属性）

### 语法

1. 这些属性都是可外部配置且可动态替换的，既可以在典型的 Java 属性文件中配置，亦可通过 properties 元素的子元素来传递。例如：

~~~xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>
~~~

然后其中的属性就可以在整个配置文件中被用来替换需要动态配置的属性值。比如:

~~~xml
<dataSource type="POOLED">
  <property name="driver" value="${driver}"/>
  <property name="url" value="${url}"/>
  <property name="username" value="${username}"/>
  <property name="password" value="${password}"/>
</dataSource>
~~~

> **这个例子中的 username 和 password 将会由 properties 元素中设置的相应值来替换。 driver 和 url 属性将会由 config.properties 文件中对应的值来替换**。这样就为配置提供了诸多灵活选择。

2. 属性也可以被传递到 SqlSessionFactoryBuilder.build()方法中。例如：

```Java
SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, props);

// ... 或者 ...

SqlSessionFactory factory = new SqlSessionFactoryBuilder().build(reader, environment, props);
```

### 配置优先级

- 在 properties 元素体内指定的属性首先被读取。
- 然后根据 properties 元素中的 resource 属性读取类路径下属性文件或根据 url 属性指定的路径读取属性文件，并覆盖已读取的同名属性。
- 最后读取作为方法参数传递的属性，并覆盖已读取的同名属性。

> **因此，通过方法参数传递的属性具有最高优先级，resource/url 属性中指定的配置文件次之，最低优先级的是 properties 属性中指定的属性**。

### 占位符

####默认设置的占位符 （:）

从 MyBatis 3.4.2 开始，你可以为占位符指定一个默认值。例如：

~~~xml
<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${username:ut_user}"/> <!-- 如果属性 'username' 没有被配置，'username' 属性的值将为 'ut_user' -->
</dataSource>
~~~

这个特性默认是关闭的。如果你想为占位符指定一个默认值， 你应该添加一个指定的属性来开启这个特性。例如：

~~~xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.enable-default-value" value="true"/> <!-- 启用默认值特性 -->
</properties>
~~~

#### 自定义设置占位符

如果你已经使用 `":"` 作为属性的键（如：`db:username`） ，或者你已经在 SQL 定义中使用 OGNL 表达式的三元运算符（如： `${tableName != null ? tableName : 'global_constants'}`），你应该通过设置特定的属性来修改分隔键名和默认值的字符。例如：

~~~xml
<properties resource="org/mybatis/example/config.properties">
  <!-- ... -->
  <property name="org.apache.ibatis.parsing.PropertyParser.default-value-separator" value="?:"/> <!-- 修改默认值的分隔符 -->
</properties>
~~~

~~~xml
<dataSource type="POOLED">
  <!-- ... -->
  <property name="username" value="${db:username?:ut_user}"/>
</dataSource>
~~~

## settings（设置）

这是 MyBatis 中极为重要的调整设置，它们会改变 MyBatis 的运行时行为。 下表描述了设置中各项的意图、默认值等。

~~~xml
<settings>
  //全局地开启或关闭配置文件中的所有映射器已经配置的任何缓存。（true | false,默认true）
  <setting name="cacheEnabled" value="true"/>
  //延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 fetchType 属性来覆盖该项的开关状态。(true | false	默认false)
  <setting name="lazyLoadingEnabled" value="true"/>
  //当开启时，任何方法的调用都会加载该对象的所有属性。 否则，每个属性会按需加载（参考 lazyLoadTriggerMethods)。(true | false	默认false （在 3.4.1 及之前的版本默认值为 true）)
  <setting name="aggressiveLazyLoading" value="false"/>
  //是否允许单一语句返回多结果集（需要驱动支持）。(true | false	默认true)
  <setting name="multipleResultSetsEnabled" value="true"/>
  //使用列标签代替列名。不同的驱动在这方面会有不同的表现，具体可参考相关驱动文档或通过测试这两种不同的模式来观察所用驱动的结果。(true | false	默认true)
  <setting name="useColumnLabel" value="true"/>
  //允许 JDBC 支持自动生成主键，需要驱动支持。 如果设置为 true 则这个设置强制使用自动生成主键，尽管一些驱动不能支持但仍可正常工作（比如 Derby）。(true | false	默认False)
  <setting name="useGeneratedKeys" value="false"/>
  //	指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示取消自动映射；PARTIAL 只会自动映射没有定义嵌套结果集映射的结果集。 FULL 会自动映射任意复杂的结果集（无论是否嵌套）。(默认PARTIAL)
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  //指定发现自动映射目标未知列（或者未知属性类型）的行为。(NONE: 不做任何反应;WARNING: 输出提醒日志 ('org.apache.ibatis.session.AutoMappingUnknownColumnBehavior' 的日志等级必须设置为 WARN);FAILING: 映射失败 (抛出 SqlSessionException)) 默认NONE
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  //配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（prepared statements）； BATCH 执行器将重用语句并执行批量更新。(默认SIMPLE)
  <setting name="defaultExecutorType" value="SIMPLE"/>
 //设置超时时间，它决定驱动等待数据库响应的秒数。 (默认为null)
  <setting name="defaultStatementTimeout" value="25"/>
  //为驱动的结果集获取数量（fetchSize）设置一个提示值。此参数只可以在查询设置中被覆盖。(默认为null)
  <setting name="defaultFetchSize" value="100"/>
  //允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。
  <setting name="safeRowBoundsEnabled" value="false"/>
  //是否开启自动驼峰命名规则（camel case）映射，即从经典数据库列名 A_COLUMN 到经典 Java 属性名 aColumn 的类似映射。
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  //	MyBatis 利用本地缓存机制（Local Cache）防止循环引用（circular references）和加速重复嵌套查询。 默认值为 SESSION，这种情况下会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地会话仅用在语句执行上，对相同 SqlSession 的不同调用将不会共享数据。
  <setting name="localCacheScope" value="SESSION"/>
  //	当没有为参数提供特定的 JDBC 类型时，为空值指定 JDBC 类型。 某些驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。
  <setting name="jdbcTypeForNull" value="OTHER"/>
  //指定哪个对象的方法触发一次延迟加载。	用逗号分隔的方法列表。
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
~~~



## typeAliases（类别名）

可以为我们的java类型起别名别名不区分大小写

~~~xml
	<typeAliases>
		<!-- 1、typeAlias:为某个java类型起别名
				type:指定要起别名的类型全类名;默认别名就是类名小写；employee
				alias:指定新的别名
		 -->
		<!-- <typeAlias type="com.atguigu.mybatis.bean.Employee" alias="emp"/> -->
		
		<!-- 2、package:为某个包下的所有类批量起别名 
				name：指定包名（为当前包以及下面所有的后代包的每一个类都起一个默认别名（类名小写），）
		-->
		<package name="com.atguigu.mybatis.bean"/>
		
		<!-- 3、批量起别名的情况下，使用@Alias注解为某个类型指定新的别名 -->
	</typeAliases>
~~~

## environments（环境设置）

environments：环境们，mybatis可以配置多种环境 ,default指定使用某种环境。可以达到快速切换环境。

environment：配置一个具体的环境信息；必须有两个标签；id代表当前环境的唯一标识

transactionManager：事务管理器；

type：事务管理器的类型;JDBC(JdbcTransactionFactory)|MANAGED(ManagedTransactionFactory)自定义事务管理器：实现TransactionFactory接口.type指定为全类名

dataSource：数据源;

type:数据源类型;UNPOOLED(UnpooledDataSourceFactory)|POOLED(PooledDataSourceFactory)
|JNDI(JndiDataSourceFactory)

自定义数据源：实现DataSourceFactory接口，type是全类名

```xml
<environments default="dev_mysql">
		<environment id="dev_mysql">
			<transactionManager type="JDBC"></transactionManager>
			<dataSource type="POOLED">
				<property name="driver" value="${jdbc.driver}" />
				<property name="url" value="${jdbc.url}" />
				<property name="username" value="${jdbc.username}" />
				<property name="password" value="${jdbc.password}" />
			</dataSource>
		</environment>
	
		<environment id="dev_oracle">
			<transactionManager type="JDBC" />
			<dataSource type="POOLED">
				<property name="driver" value="${orcl.driver}" />
				<property name="url" value="${orcl.url}" />
				<property name="username" value="${orcl.username}" />
				<property name="password" value="${orcl.password}" />
			</dataSource>
		</environment>
	</environments>
```

## databaseIdProvider

~~~xml
	<!-- 5、databaseIdProvider：支持多数据库厂商的；
		 type="DB_VENDOR"：VendorDatabaseIdProvider
		 	作用就是得到数据库厂商的标识(驱动getDatabaseProductName())，mybatis就能根据数据库厂商标识来执行不同的sql;
		 	MySQL，Oracle，SQL Server,xxxx
	  -->
	<databaseIdProvider type="DB_VENDOR">
		<!-- 为不同的数据库厂商起别名 -->
		<property name="MySQL" value="mysql"/>
		<property name="Oracle" value="oracle"/>
		<property name="SQL Server" value="sqlserver"/>
	</databaseIdProvider>


~~~

在根据databaseId配置使用：

~~~xml
<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee"
		databaseId="mysql">
		select * from tbl_employee where id = #{id}
	</select>
	<select id="getEmpById" resultType="com.atguigu.mybatis.bean.Employee"
		databaseId="oracle">
		select EMPLOYEE_ID id,LAST_NAME	lastName,EMAIL email 
		from employees where EMPLOYEE_ID=#{id}
	</select>
~~~

##mappers

将sql映射注册到全局配置中：将我们写好的sql映射文件（EmployeeMapper.xml）一定要注册到全局配置文件（mybatis-config.xml）中 

~~~xml
<mappers>
		<!-- 
			mapper:注册一个sql映射 
				注册配置文件
				resource：引用类路径下的sql映射文件
					mybatis/mapper/EmployeeMapper.xml
				url：引用网路路径或者磁盘路径下的sql映射文件
					file:///var/mappers/AuthorMapper.xml
					
				注册接口
				class：引用（注册）接口，
					1、有sql映射文件，映射文件名必须和接口同名，并且放在与接口同一目录下；
					2、没有sql映射文件，所有的sql都是利用注解写在接口上;
					推荐：
						比较重要的，复杂的Dao接口我们来写sql映射文件
						不重要，简单的Dao接口为了开发快速可以使用注解；
		-->
		<!-- <mapper resource="mybatis/mapper/EmployeeMapper.xml"/> -->
		<!-- <mapper class="com.atguigu.mybatis.dao.EmployeeMapperAnnotation"/> -->
		
		<!-- 批量注册： -->
		<package name="com.atguigu.mybatis.dao"/>
	</mappers>
~~~

