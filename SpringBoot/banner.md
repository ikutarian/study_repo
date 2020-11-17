banner演示
banner获取原理
banner输出原理
总结

# 什么是banner

启动SpringBoot应用的时候，会在控制台打印一个串文字

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.1.RELEASE)
```

这个就是banner

# banner的相关配置

有以下几个相关的配置。其他的配置可以打开 `spring-configuration-metadata.json` 文件，以 `banner` 为关键字进行搜索

```
spring.banner.charset
spring.main.banner-mode
spring.banner.location
spring.banner.image.location

省略...
```

# 如何自定义banner

## 文字banner

在 `resources` 文件夹下添加一个 `banner.txt` 文本文件，然后填写文字内容即可。比如

```
////////////////////////////////////////////////////////////////////
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//             佛祖保佑          永无故障         永不修改            //
////////////////////////////////////////////////////////////////////
```

## 图片banner

也可以在 `resources` 文件夹下添加一个 `banner.jpg` 图片文件，比如

![](./banner.jpg)

控制台就会输出

```
          &&&&&&&&&&                                       &&                 
        &&&       :                                        &&                 
       &&*                                                 &&                 
      &&&                8888888*    *******     &&&&&&&&: &&   8888888       
      &&&      &&&&&&&& 88&    888  **     ***  &&     &&: &&  88:    88      
       &&           && 888      88 ***      ** &&.     &&: && :8888888        
        &&&        &&&  88      88 ***     **: &&&     &&: &&  88.            
         8&&&&&&&&&&    .88888888   *********   &&&&&&&&&: &&   88888888      
             :&&.           88         .**         o&  &&*        .88         
                                               .&&     &&                     
                                                 &&&&&&&               
```

## 自定义banner文件名

banner文件不一定要叫 `banner.txt` 和 `banner.jpg`，也可以自己指定名称

```properties
# 文字banner
spring.banner.location=favorite.txt
# 图片banner
spring.banner.image.location=favorite.jpg
```

## 兜底banner

如果文本 banner 和图片 banner都不设置，而是使用 Java 代码来设置 banner。这种 banner 在 SpringBoot 中叫做兜底 banner（fallback banner）

### 文本banner

```java
import org.springframework.boot.ResourceBanner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.io.ClassPathResource;

@SpringBootApplication
public class FullstackApplication {

    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(FullstackApplication.class);
        springApplication.setBanner(new ResourceBanner(new ClassPathResource("favorite.txt")));
        springApplication.run(args);
    }
}
```

### 图片banner

```java
import org.springframework.boot.ImageBanner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.core.io.ClassPathResource;

@SpringBootApplication
public class FullstackApplication {

    public static void main(String[] args) {
        SpringApplication springApplication = new SpringApplication(FullstackApplication.class);
        springApplication.setBanner(new ImageBanner(new ClassPathResource("favorite.jpg")));
        springApplication.run(args);
    }
}
```

现在知道了如何设置 banner，那么 SpringBoot 框架是如何获取与打印 banner 的呢？

# SpringBoot框架是从哪里获取到banner的？

查看 `SpringApplication` 的 `run()` 方法

```java
/**
 * Run the Spring application, creating and refreshing a new
 * {@link ApplicationContext}.
 * @param args the application arguments (usually passed from a Java main method)
 * @return a running {@link ApplicationContext}
 */
public ConfigurableApplicationContext run(String... args) {
	StopWatch stopWatch = new StopWatch();
	stopWatch.start();
	ConfigurableApplicationContext context = null;
	Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
	configureHeadlessProperty();
	SpringApplicationRunListeners listeners = getRunListeners(args);
	listeners.starting();
	try {
		ApplicationArguments applicationArguments = new DefaultApplicationArguments(args);
		ConfigurableEnvironment environment = prepareEnvironment(listeners, applicationArguments);
		configureIgnoreBeanInfo(environment);
		Banner printedBanner = printBanner(environment);
		context = createApplicationContext();
		exceptionReporters = getSpringFactoriesInstances(SpringBootExceptionReporter.class,
				new Class[] { ConfigurableApplicationContext.class }, context);
		prepareContext(context, environment, listeners, applicationArguments, printedBanner);
		refreshContext(context);
		afterRefresh(context, applicationArguments);
		stopWatch.stop();
		if (this.logStartupInfo) {
			new StartupInfoLogger(this.mainApplicationClass).logStarted(getApplicationLog(), stopWatch);
		}
		listeners.started(context);
		callRunners(context, applicationArguments);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, listeners);
		throw new IllegalStateException(ex);
	}

	try {
		listeners.running(context);
	}
	catch (Throwable ex) {
		handleRunFailure(context, ex, exceptionReporters, null);
		throw new IllegalStateException(ex);
	}
	return context;
}
```

方法体内与 banner 有关的代码只有两行：

- `Banner printedBanner = printBanner(environment);` 
- `prepareContext(context, environment, listeners, applicationArguments, printedBanner);`

现在按照顺序来分析这两行代码

## Banner printedBanner = printBanner(environment);

```java
private Banner printBanner(ConfigurableEnvironment environment) {    
	if (this.bannerMode == Banner.Mode.OFF) {
		return null;
	}
	ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader
			: new DefaultResourceLoader(getClassLoader());
	SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
	if (this.bannerMode == Mode.LOG) {
		return bannerPrinter.print(environment, this.mainApplicationClass, logger);
	}
	return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
}
```

### org.springframework.boot.Banner 接口

这个方法会返回一个 `Banner` 类型的值。 `Banner`  是一个接口。接口中有一个打印 banner 的方法，以及一个 Mode 枚举

```java
package org.springframework.boot;

import java.io.PrintStream;
import org.springframework.core.env.Environment;

/**
 * Interface class for writing a banner programmatically.
 *
 * @author Phillip Webb
 * @author Michael Stummvoll
 * @author Jeremy Rickard
 * @since 1.2.0
 */
@FunctionalInterface
public interface Banner {

	/**
	 * Print the banner to the specified print stream.
	 * @param environment the spring environment
	 * @param sourceClass the source class for the application
	 * @param out the output print stream
	 */
	void printBanner(Environment environment, Class<?> sourceClass, PrintStream out);

	/**
	 * An enumeration of possible values for configuring the Banner.
	 */
	enum Mode {

		/**
		 * Disable printing of the banner.
		 */
		OFF,

		/**
		 * Print the banner to System.out.
		 */
		CONSOLE,

		/**
		 * Print the banner to the log file.
		 */
		LOG
	}
}
```

`Banner` 接口有几个主要的实现类

- `org.springframework.boot.ImageBanner` 图片 banner
- `org.springframework.boot.ResourceBanner` 文字 banner
- `org.springframework.boot.SpringBootBanner` 默认的 banner

`printBanner()` 方法体内首先判断是否要打印 banner，如果不打印就返回 null

```java
if (this.bannerMode == Banner.Mode.OFF) {
    return null;
}
```

### 初始化 SpringApplicationBannerPrinter 对象

接着初始化 `SpringApplicationBannerPrinter` 对象，传入 `SpringApplicationBannerPrinter` 构造方法的的 `this.banner` 参数就是兜底 banner

```java
ResourceLoader resourceLoader = (this.resourceLoader != null) ? this.resourceLoader
			: new DefaultResourceLoader(getClassLoader());
SpringApplicationBannerPrinter bannerPrinter = new SpringApplicationBannerPrinter(resourceLoader, this.banner);
```

`SpringApplicationBannerPrinter` 是一个很重要的类，就是由它来负责打印 banner 的工作。根据 banner 的打印方式（LOG 或者 CONSOLE）调用不同的 `print()` 方法

```java
if (this.bannerMode == Mode.LOG) {
	return bannerPrinter.print(environment, this.mainApplicationClass, logger);
}
return bannerPrinter.print(environment, this.mainApplicationClass, System.out);
```

默认的 `bannerMode` 是 `Mode.CONSOLE`，所以先看一下最后一行代码

```java
Banner print(Environment environment, Class<?> sourceClass, PrintStream out) {
    Banner banner = getBanner(environment);
    banner.printBanner(environment, sourceClass, out);
    return new PrintedBanner(banner, sourceClass);
}
```

### 获取 banner

首先获取 banner

```java
private Banner getBanner(Environment environment) {
    Banners banners = new Banners();
    banners.addIfNotNull(getImageBanner(environment));
    banners.addIfNotNull(getTextBanner(environment));
    if (banners.hasAtLeastOneBanner()) {
        return banners;
    }
    if (this.fallbackBanner != null) {
        return this.fallbackBanner;
    }
    return DEFAULT_BANNER;
}
```

方法体内的 `Banners` 是一个实现 `org.springframework.boot.Banner` 接口的内部类，用来存放 banner 的集合。它的 `printBanner` 方法就是用来遍历 banner 集合，调用 banner 的打印方法

```java
/**
 * {@link Banner} comprised of other {@link Banner Banners}.
 */
private static class Banners implements Banner {

    private final List<Banner> banners = new ArrayList<>();

    void addIfNotNull(Banner banner) {
        if (banner != null) {
            this.banners.add(banner);
        }
    }

    boolean hasAtLeastOneBanner() {
        return !this.banners.isEmpty();
    }

    @Override
    public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
        for (Banner banner : this.banners) {
            banner.printBanner(environment, sourceClass, out);
        }
    }

}
```

实例化 `Banners` 对象之后，通过下面的两行代码尝试添加图片 banner 和文字 banner

- `banners.addIfNotNull(getImageBanner(environment));`
- `banners.addIfNotNull(getTextBanner(environment));`

```java
private Banner getImageBanner(Environment environment) {
    String location = environment.getProperty(BANNER_IMAGE_LOCATION_PROPERTY);
    if (StringUtils.hasLength(location)) {
        Resource resource = this.resourceLoader.getResource(location);
        return resource.exists() ? new ImageBanner(resource) : null;
    }
    for (String ext : IMAGE_EXTENSION) {
        Resource resource = this.resourceLoader.getResource("banner." + ext);
        if (resource.exists()) {
            return new ImageBanner(resource);
        }
    }
    return null;
}
```

首先检查是否有使用 `spring.banner.image.location` 指定了图片 banner 的位置，如果有就创建 `ImageBanner` 对象并返回。如果没有，就检查是否有 `banner.gif`、`banner.jpg` 或者 `banner.png` 的存在，有就生成 `ImageBanner` 对象并返回。如果以上两步都失败了，就说明没有配置图片 banner，于是返回 `null`

```java
private Banner getTextBanner(Environment environment) {
	String location = environment.getProperty(BANNER_LOCATION_PROPERTY, DEFAULT_BANNER_LOCATION);
	Resource resource = this.resourceLoader.getResource(location);
	if (resource.exists()) {
		return new ResourceBanner(resource);
	}
	return null;
}
```

检查是否有使用 `spring.banner.location` 指定文字图片 banner 的位置，如果没有配置就再检查是否存在 `banner.txt` 文件，有就创建 `ResourceBanner` 对象，如果没有就返回 `null`。通过以上这两步，图片 banner 和文字 banner 就添加好了

然后判断 banners 集合中是否有至少一个元素，有就返回

```java
if (banners.hasAtLeastOneBanner()) {
    return banners;
}
```

没有的话，就判断是否有兜底 banner，有就返回

```java
if (this.fallbackBanner != null) {
	return this.fallbackBanner;
}
```

如果还是没有，就返回 `SpringBootBanner`。这个 banner 就是我们常见的 banner

```
  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::        (v2.2.1.RELEASE)
```

### 打印 banner

经过 `Banner banner = getBanner(environment);` 获取到图片 banner、文字banner的集合或者是 `SpringBootBanner` 之后，就可以把它们打印出来了。具体可以去看这三个类的实现

- `org.springframework.boot.ImageBanner` 图片 banner
- `org.springframework.boot.ResourceBanner` 文字 banner
- `org.springframework.boot.SpringBootBanner` 默认的 banner

比较特殊的是 `org.springframework.boot.ResourceBanner` 文字banner，它的打印方法不仅仅是打印出 banner 的内容，还有做一些额外的操作：**它可以将 banner 文件中的占位符进行替换**

```java
@Override
public void printBanner(Environment environment, Class<?> sourceClass, PrintStream out) {
	try {
		String banner = StreamUtils.copyToString(this.resource.getInputStream(),
				environment.getProperty("spring.banner.charset", Charset.class, StandardCharsets.UTF_8));

        // 替换占位符
		for (PropertyResolver resolver : getPropertyResolvers(environment, sourceClass)) {
			banner = resolver.resolvePlaceholders(banner);
		}
        
		out.println(banner);
	}
	catch (Exception ex) {
		logger.warn(LogMessage.format("Banner not printable: %s (%s: '%s')", this.resource, ex.getClass(),
				ex.getMessage()), ex);
	}
}
```

比如 `application.properties` 中有三个配置项

```properties
user.name=owen
user.age=18
server.port=8080
```

`banner.txt`  的内容如下

```
姓名: ${user.name}
年龄: ${user.age}
服务器端口: ${server.port}

////////////////////////////////////////////////////////////////////
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//             佛祖保佑          永无故障         永不修改            //
////////////////////////////////////////////////////////////////////
```

启动应用，就可以在控制台看到占位符被替换了

```
姓名: owen
年龄: 18
服务器端口: 8080

////////////////////////////////////////////////////////////////////
//                          _ooOoo_                               //
//                         o8888888o                              //
//                         88" . "88                              //
//                         (| ^_^ |)                              //
//                         O\  =  /O                              //
//                      ____/`---'\____                           //
//                    .'  \\|     |//  `.                         //
//                   /  \\|||  :  |||//  \                        //
//                  /  _||||| -:- |||||-  \                       //
//                  |   | \\\  -  /// |   |                       //
//                  | \_|  ''\---/''  |   |                       //
//                  \  .-\__  `-`  ___/-. /                       //
//                ___`. .'  /--.--\  `. . ___                     //
//              ."" '<  `.___\_<|>_/___.'  >'"".                  //
//            | | :  `- \`.;`\ _ /`;.`/ - ` : | |                 //
//            \  \ `-.   \_ __\ /__ _/   .-` /  /                 //
//      ========`-.____`-.___\_____/___.-`____.-'========         //
//                           `=---='                              //
//      ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^        //
//             佛祖保佑          永无故障         永不修改            //
////////////////////////////////////////////////////////////////////
```

上面讲的是 `bannerMode` 为 `Mode.CONSOLE` 的情况，如果是 `Mode.LOG` 的话，调用的是这个方法

```java
Banner print(Environment environment, Class<?> sourceClass, Log logger) {
    Banner banner = getBanner(environment);
    try {
        logger.info(createStringFromBanner(banner, environment, sourceClass));
    }
    catch (UnsupportedEncodingException ex) {
        logger.warn("Failed to create String for banner", ex);
    }
    return new PrintedBanner(banner, sourceClass);
}
```

操作都差不多，先获取 banner，然后在打印，不同的地方就是这句：`logger.info(createStringFromBanner(banner, environment, sourceClass))`。它的内部实现就是将 banner 转换成字符串

```java
private String createStringFromBanner(Banner banner, Environment environment, Class<?> mainApplicationClass)
    throws UnsupportedEncodingException {
    ByteArrayOutputStream baos = new ByteArrayOutputStream();
    banner.printBanner(environment, mainApplicationClass, new PrintStream(baos));
    String charset = environment.getProperty("spring.banner.charset", "UTF-8");
    return baos.toString(charset);
}
```

## prepareContext(context, environment, listeners, applicationArguments, printedBanner)

```java
private void prepareContext(ConfigurableApplicationContext context, ConfigurableEnvironment environment,
		SpringApplicationRunListeners listeners, ApplicationArguments applicationArguments, Banner printedBanner) {
	// 省略...

	if (printedBanner != null) {
		beanFactory.registerSingleton("springBootBanner", printedBanner);
	}
	
	// 省略...
}
```

就是将 banner 对象以 `springBootBanner` 为名加入 bean 工厂