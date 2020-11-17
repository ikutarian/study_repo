```java
/**
 * Run the Spring application, creating and refreshing a new
 * {@link ApplicationContext}.
 * @param args the application arguments (usually passed from a Java main method)
 * @return a running {@link ApplicationContext}
 */
```

方法的的注释解释了方法的作用：`run` 方法用于启动 Spring 应用，创建和刷新一个新的 `ApplicationContext`。传入 `main` 方法的参数，返回 `ApplicationContext`

方法内容如下，重点的部分加上了注释

```java
public ConfigurableApplicationContext run(String... args) {
   StopWatch stopWatch = new StopWatch();   // 创建计时器   
   stopWatch.start();  // 启动计时器
   ConfigurableApplicationContext context = null;
   Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
   configureHeadlessProperty();  // 配置Headless属性
   SpringApplicationRunListeners listeners = getRunListeners(args);  // 获取监听器列表
   listeners.starting();  // 调用监听器的starting()方法，表示SpringBoot应用正在启动
   try {
      // 解析main方法的args参数，生成ApplicationArguments对象
      ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
      // 属性配置
      ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
      configureIgnoreBeanInfo(environment);
      // 打印banner
      Banner printedBanner = printBanner(environment);
      // 创建容器
      context = createApplicationContext();
      // 获取异常报告器
      exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
            new Class[] { ConfigurableApplicationContext.class }, context);
      // 准备容器
      prepareContext(context, environment, listeners, applicationArguments, printedBanner);
      // 刷新容器
      refreshContext(context);
      // 刷新容器之后的操作，目前为空
      afterRefresh(context, applicationArguments);
      // 停止计时器
      stopWatch.stop();
      // 打印启动日志
      if (this.logStartupInfo) {
         new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
      }
      // 监听器通知SpringBoot应用启动完毕
      listeners.started(context);
      // 调用ApplicationRunner和CommandLineRunner的run方法
      callRunners(context, applicationArguments);
   }
   catch (Throwable ex) {
      // 异常处理
      handleRunFailure(context, ex, exceptionReporters, listeners);
      throw new IllegalStateException(ex);
   }

   try {
      // 监听器通知SpringBoot应用正在运行
      listeners.running(context);
   }
   catch (Throwable ex) {
      // 异常处理
      handleRunFailure(context, ex, exceptionReporters, null);
      throw new IllegalStateException(ex);
   }
   return context;
}
```

## SpringApplicationRunListener 监听器

### 获取监听器实现类列表

在 `run` 方法中通过这一行代码可以获取监听器实现类列表

```java
SpringApplicationRunListeners listeners = getRunListeners(args);
```

`SpringApplicationRunListeners` 是一个容器类，用来存放监听器实现类列表，并且提供了一些和 `SpringApplicationRunListener`  同名的方法，由容器类作为中介去循环调用监听器实现类的方法，比如

```java
class SpringApplicationRunListeners {

	// 省略...
	
	void starting() {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.starting();
		}
	}

	// 省略...
}
```

获取监听器实现类列表如下

```java
private SpringApplicationRunListeners getRunListeners(String[] args) {
   Class<?>[] types = new Class<?>[] { SpringApplication.class, String[].class };
   return new SpringApplicationRunListeners(logger,
         getSpringFactoriesInstances(SpringApplicationRunListener.class, types, this, args));
}
```

通过 `SpringFactoriesLoader.loadFactoryNames` 方法获取 `META-INF/spring.factories` 中配置的实现类，然后把它们实例化，并返回。目前只有一个实现类 `EventPublishingRunListener`

```properties
# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener
```

### SpringApplicationRunListener  源码解析

`SpringApplicationRunListener` 是一个接口，提供了一些 Spring Boot 生命周期相关的方法

```java
public interface SpringApplicationRunListener {

    /**
	 * Called immediately when the run method has first started. Can be used for very
	 * early initialization.
	 *
	 * 当run方法第一次被执行十被调用，可用于非常早期的初始化工作
	 */
	default void starting() {
	}
	
    /**
	 * Called once the environment has been prepared, but before the
	 * {@link ApplicationContext} has been created.
	 * @param environment the environment
	 *
	 * 当environment被准备完成，ApplicationContext被创建之前，该方法被调用
	 */
	default void environmentPrepared(ConfigurableEnvironment environment) {
	}
	
    /**
	 * Called once the {@link ApplicationContext} has been created and prepared, but
	 * before sources have been loaded.
	 * @param context the application context
	 *
	 * ApplicationContext被创建并准备好，但是资源还没有被加载时，该方法被调用
	 */
	default void contextPrepared(ConfigurableApplicationContext context) {
	}
	
    /**
	 * Called once the application context has been loaded but before it has been
	 * refreshed.
	 * @param context the application context
	 *
	 * ApplicationContext被刷新之前调用
	 */
	default void contextLoaded(ConfigurableApplicationContext context) {
	}
	
    /**
	 * The context has been refreshed and the application has started but
	 * {@link CommandLineRunner CommandLineRunners} and {@link ApplicationRunner
	 * ApplicationRunners} have not been called.
	 * @param context the application context.
	 * @since 2.0.0
	 *
	 * ApplicationContext已刷新，并且应用已经启动，但是CommandLineRunners和ApplicationRunners还没有被调用，该方法被调用
	 */
	default void started(ConfigurableApplicationContext context) {
	}
	
    /**
	 * Called immediately before the run method finishes, when the application context has
	 * been refreshed and all {@link CommandLineRunner CommandLineRunners} and
	 * {@link ApplicationRunner ApplicationRunners} have been called.
	 * @param context the application context.
	 * @since 2.0.0
	 *
	 * 所有工作都做完了，run方法执行完毕之前，该方法被调用
	 */
	default void running(ConfigurableApplicationContext context) {
	}
	
    /**
	 * Called when a failure occurs when running the application.
	 * @param context the application context or {@code null} if a failure occurred before
	 * the context was created
	 * @param exception the failure
	 * @since 2.0.0
	 *
	 * 应用出错时被调用
	 */
	default void failed(ConfigurableApplicationContext context, Throwable exception) {
	}
}
```

### 监听器的实现类 EventPublishingRunListener

`EventPublishingRunListener` 是 Spring Boot 中 `SpringApplicationRunListener` 接口的唯一实现。其内部是通过 `SimpleApplicationEventMulticaster` 来广播生命周期的相关方法。