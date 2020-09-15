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

除了在配置文件中设置文本 banner 和图片 banner，还可以使用 Java 代码来设置 banner。这种 banner 在 SpringBoot 中叫做兜底 banner（fallback banner）

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

# SpringBoot框架是从哪里获取到banner的？

