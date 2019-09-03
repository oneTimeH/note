# Spring + Struts + Hibernate + maven整合

## struts工作流程

（1） 客户端（Client）向Action发送一个请求（Request）
（2） Container通过web.xml映射请求，并获得控制器（Controller）的名字
（3） 容器（Container）调用控制器（StrutsPrepareAndExecuteFilter或FilterDispatcher）。在Struts2.1以前调用FilterDispatcher，Struts2.1以后调用StrutsPrepareAndExecuteFilter
（4） 控制器（Controller）通过ActionMapper获得Action的信息
（5） 控制器（Controller）调用ActionProxy
（6） ActionProxy读取struts.xml文件获取action和interceptor stack的信息。
（7） ActionProxy把request请求传递给ActionInvocation
（8） ActionInvocation依次调用action和interceptor
（9） 根据action的配置信息，产生result
（10） Result信息返回给ActionInvocation
（11） 产生一个HttpServletResponse响应
（12） 产生的响应行为发送给客服端。 



1、客户端初始化一个指向Servlet容器（例如Tomcat）的请求

2、这个请求经过一系列的过滤器（Filter）（这些过滤器中有一个叫做ActionContextCleanUp的可选过滤器，这个过滤器对于Struts 2和其他框架的集成很有帮助，例如：SiteMesh Plugin）

3、接着FilterDispatcher被调用，FilterDispatcher询问ActionMapper来决定这个请是否需要调用某个Action

FilterDispatcher是控制器的核心，就是mvc中c控制层的核心。下面粗略的分析下我理解的FilterDispatcher工作流程和原理：FilterDispatcher进行初始化并启用核心doFilter

## 1、配置流程 

### 1.1、先搭建maven依赖(Pom.xml)

~~~ xml
<dependencies>
  
  	<dependency>
	    <groupId>javax.servlet</groupId>
	    <artifactId>servlet-api</artifactId>
	    <version>2.5</version>
	    <scope>provided</scope>
	</dependency>
  	<!-- spring3 需要的包 -->
  	<!-- spring核心包 -->
  	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-core</artifactId>
	    <version>3.1.2.RELEASE</version>
	</dependency>
  	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-context</artifactId>
	    <version>3.1.2.RELEASE</version>
	</dependency>
	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-jdbc</artifactId>
	    <version>3.1.2.RELEASE</version>
	</dependency>
	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-beans</artifactId>
	    <version>3.1.2.RELEASE</version>
	</dependency>
	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-web</artifactId>
	    <version>3.1.2.RELEASE</version>
	</dependency>
	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-orm</artifactId>
	    <version>3.1.2.RELEASE</version>
	</dependency>
	<!-- junit -->
	<dependency>
	    <groupId>junit</groupId>
	    <artifactId>junit</artifactId>
	    <version>4.10</version>
	    <scope>test</scope>
	</dependency>	
	<!-- log4j -->
	<dependency>
	    <groupId>log4j</groupId>
	    <artifactId>log4j</artifactId>
	    <version>1.2.17</version>
	</dependency>
	
	<!-- struts2 -->
	<!-- struts2核心包 -->
	<dependency>
	    <groupId>org.apache.struts</groupId>
	    <artifactId>struts2-core</artifactId>
	    <version>2.3.24</version>
	    <!-- 组合hibernate时会冲突，-->
	    <exclusions>
	    	<exclusion>
	    		<groupId>javassit</groupId>
	    		<artifactId>javassit</artifactId>
	    	</exclusion>
	    </exclusions>
	     
	</dependency>
	<!-- 将struts2 action的类在web容器启动的时候加入到spring上下文中 -->
	<dependency>
	    <groupId>org.apache.struts</groupId>
	    <artifactId>struts2-spring-plugin</artifactId>
	    <version>2.3.24.1</version>
	</dependency>
	<!-- 使用struts2注解导入的包 -->
	<dependency>
	    <groupId>org.apache.struts</groupId>
	    <artifactId>struts2-convention-plugin</artifactId>
	    <version>2.3.24.1</version>
	</dependency>
	<!-- hibernate 4 -->
	<dependency>
	    <groupId>org.hibernate</groupId>
	    <artifactId>hibernate-core</artifactId>
	    <version>4.1.7.Final</version>
	</dependency>
	<!-- AspectJ（也就是AOP） -->
	<dependency>
	    <groupId>org.aspectj</groupId>
	    <artifactId>aspectjweaver</artifactId>
	    <version>1.7.0</version>
	</dependency>
	<!-- 阿里巴巴数据源jar包 -->
	<dependency>
	    <groupId>com.alibaba</groupId>
	    <artifactId>druid</artifactId>
	    <version>1.0.0</version>
	</dependency>
	<!-- mysql数据库驱动 -->
	<dependency>
	    <groupId>mysql</groupId>
	    <artifactId>mysql-connector-java</artifactId>
	    <version>5.1.21</version>
	</dependency>
	<!-- jstl -->
	<dependency>
	    <groupId>javax.servlet</groupId>
	    <artifactId>jstl</artifactId>
	    <version>1.2</version>
	</dependency>
	<!-- 缺少jsp-api jar包 添加进去
		依赖的域范围要改成provided
	-->
	<dependency>
		<groupId>javax.servlet.jsp</groupId>
		<artifactId>jsp-api</artifactId>
		<version>2.2</version>
		<scope>provided</scope>
	</dependency>
	<!-- 缺少servlet-api jar包 添加进去
		依赖的域范围要改成provided
	-->
	<dependency>
	    <groupId>javax.servlet</groupId>
	    <artifactId>javax.servlet-api</artifactId>
	    <version>3.0.1</version>
	    <scope>provided</scope>
	</dependency>
	<!-- 阿里巴巴开发的json插件 -->
	<dependency>
	    <groupId>com.alibaba</groupId>
	    <artifactId>fastjson</artifactId>
	    <version>1.2.0</version>
	</dependency>
	<!-- ehcache缓存 -->
	<!-- 
	<dependency>
	    <groupId>net.sf.ehcache</groupId>
	    <artifactId>ehcache</artifactId>
	    <version>2.7.4</version>
	</dependency>
	 -->
  </dependencies
~~~

### 1.2、建立数据连接配置(config.properties)

~~~ properties
##mysql config
jdbc_url=jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=UTF-8&zeroDateTimeBehavior=convertToNull
jdbc_username=root
jdbc_password=123456
jdbc_drivername=com.mysql.jdbc.Driver

##hibernate config
hibernate.dialect=org.hibernate.dialect.MySQLDialect
validationQuery=SELECT 1 FROM DUAL
hibernate.hbm2ddl.auto=update
hibernate.show_sql=true
hibernate.format_sql=true
sessionInfoName=sessionInfo
##设置启用二级缓存
hibernate.cache.use_second_level_cache=true
##设置二级缓存的实现类
hibernate.cache.region.factory_class=org.hibernate.cache.ehcache.EhCacheRegionFactory

~~~

### 1.3、建立日志配置(log4j.properties)

~~~ properties

log4j.rootLogger=INFO,A1,R

log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.Target=System.out
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=[%c]%m%n

log4j.appender.R=org.apache.log4j.RollingFileAppender 
log4j.appender.R.File=Easyuisys.log
log4j.appender.R.MaxFileSize=10MB
log4j.appender.R.Threshold=ALL
log4j.appender.R.layout=org.apache.log4j.PatternLayout
log4j.appender.R.layout.ConversionPattern=[%p][%d{yyyy-MM-dd HH\:mm\:ss,SSS}][%c]%m%n

~~~

### 1.4、spring-hibernate的配置(spring-hibernate.xml)

~~~ xml
<!-- 配置数据源 -->
	<bean name="dataSource" class="com.alibaba.druid.pool.DruidDataSource"
		init-method="init" destroy-method="close">
		<property name="url" value="${jdbc_url}" />
		<property name="username" value="${jdbc_username}" />
		<property name="password" value="${jdbc_password}" />

		<!-- 初始化连接大小 -->
		<property name="initialSize" value="0" />
		<!-- 连接池最大使用连接数量 -->
		<property name="maxActive" value="20" />
		<!-- 连接池最大空闲 -->
		<property name="maxIdle" value="20" />
		<!-- 连接池最小空闲 -->
		<property name="minIdle" value="0" />
		<!-- 获取连接最大等待时间 -->
		<property name="maxWait" value="60000" />

		<property name="validationQuery" value="${validationQuery}" />
		<property name="testOnBorrow" value="false" />
		<property name="testOnReturn" value="false" />
		<property name="testWhileIdle" value="true" />

		<!-- 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒 -->
		<property name="timeBetweenEvictionRunsMillis" value="60000" />
		<!-- 配置一个连接在池中最小生存的时间，单位是毫秒 -->
		<property name="minEvictableIdleTimeMillis" value="25200000" />

		<!-- 打开removeAbandoned功能 -->
		<property name="removeAbandoned" value="true" />
		<!-- 1800秒，也就是30分钟 -->
		<property name="removeAbandonedTimeout" value="1800" />
		<!-- 关闭abanded连接时输出错误日志 -->
		<property name="logAbandoned" value="true" />

		<!-- 监控数据库 -->
		<!-- <property name="filters" value="stat" /> -->
		<property name="filters" value="mergeStat" />
	</bean>

	<!-- ehcache缓存配置 -->
	<!-- <bean id="ehcache" class="org.springframework.cache.ehcache.EhCacheManagerFactoryBean"> 
		<property name="configLocation" value="classpath:ehcache.xml" /> </bean> 
		<bean id="cacheManager" class="org.springframework.cache.ehcache.EhCacheCacheManager"> 
		<property name="cacheManager" ref="ehcache"/> </bean> -->
	<!-- 配置hibernate session工厂 -->
	<bean id="sessionFactory"
		class="org.springframework.orm.hibernate4.LocalSessionFactoryBean">
		<property name="dataSource" ref="dataSource" />
		<property name="hibernateProperties">
			<props>
				<prop key="hibernate.hbm2ddl.auto">${hibernate.hbm2ddl.auto}</prop>
				<prop key="hibernate.dialect">${hibernate.dialect}</prop>
				<prop key="hibernate.show_sql">${hibernate.show_sql}</prop>
				<prop key="hibernate.format_sql">${hibernate.format_sql}</prop>
			</props>
		</property>

		<!-- 自动扫描注解方式配置的hibernate类文件 -->
		<property name="packagesToScan">
			<list>
				<value>com.easyui.model</value>
			</list>
		</property>

		<!-- 自动扫描hbm方式配置的hibernate文件和.hbm文件 -->
		<!-- <property name="mappingDirectoryLocations"> <list> <value>classpath:sy/hbm</value> 
			</list> </property> -->
	</bean>

	<!-- 配置事务管理器 -->
	<bean name="transactionManager"
		class="org.springframework.orm.hibernate4.HibernateTransactionManager">
		<property name="sessionFactory" ref="sessionFactory"></property>
	</bean>

	<!-- 注解方式配置事物 -->
	<!-- 这种方式会为所有方法都配置事物，但有些方法不需要事物 -->
	<!-- <tx:annotation-driven transaction-manager="transactionManager" /> -->

	<!-- 拦截器方式配置事物 -->
	<tx:advice id="transactionAdvice" transaction-manager="transactionManager">
		<tx:attributes>
			<tx:method name="add*" />
			<tx:method name="save*" />
			<tx:method name="update*" />
			<tx:method name="modify*" />
			<tx:method name="edit*" />
			<tx:method name="delete*" />
			<tx:method name="remove*" />
			<tx:method name="repair" />
			<tx:method name="deleteAndRepair" />

			<tx:method name="get*" propagation="SUPPORTS" />
			<tx:method name="find*" propagation="SUPPORTS" />
			<tx:method name="load*" propagation="SUPPORTS" />
			<tx:method name="search*" propagation="SUPPORTS" />
			<tx:method name="datagrid*" propagation="SUPPORTS" />

			<tx:method name="*" propagation="SUPPORTS" />
		</tx:attributes>
	</tx:advice>
	
	<aop:config>
		<aop:pointcut id="transactionPointcut"
			expression="execution(* com.easyui.dao..*Impl.*(..))" />
		<aop:advisor pointcut-ref="transactionPointcut"
			advice-ref="transactionAdvice" />
	</aop:config>
~~~

### 1.5、spring的配置(spring.xml)

~~~ xml
<!-- 将配置文件加入到spring上下文 -->
	<context:property-placeholder location="classpath:config.properties" />
	
	<!-- 配置spring自动扫包 -->
	<context:component-scan base-package="com.easyui.dao,com.easyui.service" />

	<!-- 启用缓存注解开关 -->
	<!-- <cache:annotation-driven cache-manager="cacheManager"/> -->
~~~

### 1.6、struts的配置(struts.xml)

```xml
<!-- 指定由spring负责action对象的创建 -->
<constant name="struts.objectFactory" value="spring" />
<!-- 所有匹配*.action的请求都由struts2处理  设置访问请求的后缀名 .do  .html 只需在后面加",do"-->
<constant name="struts.action.extension" value="action" />
<!-- 是否启用开发模式 -->
<constant name="struts.devMode" value="true" />
<!-- struts配置文件改动后，是否重新加载 -->
<constant name="struts.configuration.xml.reload" value="true" />
<!-- 设置浏览器是否缓存静态内容 -->
<constant name="struts.serve.static.browserCache" value="false" />
<!-- 请求参数的编码方式 -->
<constant name="struts.i18n.encoding" value="utf-8" />
<!-- 每次HTTP请求系统都重新加载资源文件，有助于开发 -->
<constant name="struts.i18n.reload" value="true" />
<!-- 文件上传最大值 -->
<constant name="struts.multipart.maxSize" value="104857600" />
<!-- 让struts2支持动态方法调用 -->
<constant name="struts.enable.DynamicMethodInvocation" value="true" />
<!-- Action名称中是否还是用斜线 -->
<constant name="struts.enable.SlashesInActionNames" value="false" />
<!-- 允许标签中使用表达式语法 -->
<constant name="struts.tag.altSyntax" value="true" />
<!-- 对于WebLogic,Orion,OC4J此属性应该设置成true -->
<constant name="struts.dispatcher.parametersWorkaround" value="false" />

<!-- 普通方式是直接在里面写action -->
<!-- 
<package name="demo" extends="struts-default" namespace="/">
	<action name="show" method="show" class="com.easyui.action.mainAction">
		<result name="show">/show.jsp</result>
	</action>
</package>
 -->

<!-- 更简单的方式是action类里面写注解 -->
<package name="basePackage" extends="struts-default">
</package>
```
### 1.7、web.xml的配置

~~~ xml
<context-param>
  	<param-name>contextConfigLocation</param-name>
  	<param-value>classpath:spring.xml,classpath:spring-hibernate.xml</param-value>
  </context-param>
  
  <!-- openSessionInview配置 -->
  <filter>
  	<filter-name>openSessionInViewFilter</filter-name>
  	<filter-class>
  		org.springframework.orm.hibernate4.support.OpenSessionInViewFilter
  	</filter-class>
  	<init-param>
  		<param-name>singleSession</param-name>
  		<param-value>true</param-value>
  	</init-param>
  </filter>
  
  <!-- 编码配置 -->
  <filter>
  	<filter-name>encodinFilter</filter-name>
  	<filter-class>com.easyui.filter.EncodeFilter</filter-class>
  </filter>
  
  <!-- struts2 配置 -->
  <filter>
  	<filter-name>struts2</filter-name>
  	<filter-class>
      org.apache.struts2.dispatcher.ng.filter.StrutsPrepareAndExecuteFilter
    </filter-class>
  </filter>
  
  <!-- 编码配置 -->
  <filter-mapping>
  	<filter-name>encodinFilter</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
  
  <!-- openSessionInview -->
  <filter-mapping>
  	<filter-name>openSessionInViewFilter</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
  
  <!-- struts2 请求路径配置 -->
  <filter-mapping>
  	<filter-name>struts2</filter-name>
  	<url-pattern>/*</url-pattern>
  </filter-mapping>
  
  <!-- spring配置监听器 -->
   <listener>
  	<listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  
  <!-- 修复数据库监听器    暂时不启动 -->
  <!-- 
  <listener>
  	<listener-class>com.easyui.listener.RepairListener</listener-class>
  </listener>
   -->
~~~

##### 项目目录

![img](file:///C:\Users\window\AppData\Roaming\Tencent\Users\454447926\QQ\WinTemp\RichOle\AF{8G{M92]7R99[@]BEDDVW.png)