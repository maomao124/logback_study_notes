









# logback

## 介绍

Logback继承自log4j。Logback的架构非常的通用，适用于不同的使用场景。



![image-20221031184044757](img/logback学习笔记/image-20221031184044757.png)



logback和Log4j都是slf4j规范的具体实现，我们在程序中直接调用的API其实都是slf4j的api，底层则是真正的日志实现组件---logback或者log4j。

Logback 构建在三个主要的类上：Logger，Appender 和 Layout。这三个不同类型的组件一起作用能够让开发者根据消息的类型以及日志的级别来打印日志。 



**Logger**作为日志的记录器，把它关联到应用的对应的context后，主要用于存放日志对象，也可以定义日志类型、级别。各个logger 都被关联到一个 LoggerContext，LoggerContext负责制造logger，也负责以树结构排列各 logger。

**Appender**主要用于指定日志输出的目的地，目的地可以是控制台、文件、 数据库等。

**Layout** 负责把事件转换成字符串，输出格式化的日志信息。





logback的maven坐标：

```xml
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-core</artifactId>
    <version>1.2.3</version>
</dependency>
```







## logback层级

在 logback中每一个 logger 都依附在 LoggerContext 上，它负责产生 logger，并且通过一个**树状**的层级结构来进行管理。

一个 Logger 被当作为一个实体，它们的命名是大小写敏感的，并且遵循以下规则：

* 如果一个logger的名字加上一个.作为另一个logger名字的前缀，那么该logger就是另一个logger的祖先。如果一个logger与另一个logger之间没有其它的logger，则该logger就是另一个logger的父级。



在logback中有一个root logger，它是logger层次结构的最高层，它是一个特殊的logger，因为它是每一个层次结构的一部分









## logback日志输出等级

logback的日志输出等级分为：TRACE, DEBUG, INFO, WARN, ERROR。

如果一个给定的logger没有指定一个日志输出等级，那么它就会继承离它最近的一个祖先的层级。

为了确保所有的logger都有一个日志输出等级，root logger会有一个默认输出等级 --- DEBUG。











## logback初始化步骤

1. logback会在类路径下寻找名为logback-test.xml的文件
2. 如果没有找到，logback会继续寻找名为logback.groovy的文件
3. 如果没有找到，logback会继续寻找名为logback.xml的文件
4. 如果没有找到，将会在类路径下寻找文件META-INFO/services/ch.qos.logback.classic.spi.Configurator，该文件的内容为实现了Configurator接口的实现类的全限定类名
5. 如果以上都没有成功，logback会通过BasicConfigurator为自己进行配置，并且日志将会全部在控制台打印出来

最后一步的目的是为了保证在所有的配置文件都没有被找到的情况下，提供一个默认的配置。









## logback入门案例

### 案例一



#### 第一步：创建maven工程logback_demo



![image-20221031185158971](img/logback学习笔记/image-20221031185158971.png)





#### 第二步：修改pom文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <!--
      -maven项目核心配置文件-
    Project name(项目名称)：logback_demo
    Author(作者）: mao
    Author QQ：1296193245
    GitHub：https://github.com/maomao124/
    Date(创建日期)： 2022/10/31
    Time(创建时间)： 18:52
    -->
    <groupId>mao</groupId>
    <artifactId>logback_demo</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>16</maven.compiler.source>
        <maven.compiler.target>16</maven.compiler.target>
    </properties>
    
    <dependencies>

        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.3.4</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.3.4</version>
        </dependency>

        <!-- 测试框架 -->
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>RELEASE</version>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <finalName>logback_demo</finalName>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <mainClass>Test</mainClass>
                            <!--更改项，主类名-->
                        </manifest>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```







#### 第三步：编写单元测试



```java
package mao;

import ch.qos.logback.classic.Level;
import ch.qos.logback.classic.LoggerContext;
import ch.qos.logback.core.util.StatusPrinter;
import org.junit.jupiter.api.Test;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * Project name(项目名称)：logback_demo
 * Package(包名): mao
 * Class(类名): LogbackTest
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 18:56
 * Version(版本): 1.0
 * Description(描述)： 无
 */


public class LogbackTest
{
    //简单使用
    @Test
    public void test1()
    {
        Logger logger = LoggerFactory.getLogger("mao.logback.HelloWorld");
        logger.debug("debug ...");
    }

    //打印日志内部状态
    @Test
    public void test2()
    {
        Logger logger = LoggerFactory.getLogger("mao.logback.HelloWorld");
        logger.debug("debug ...");
        // 打印内部的状态
        LoggerContext lc = (LoggerContext) LoggerFactory.getILoggerFactory();
        StatusPrinter.print(lc);
    }

    /*
     * 日志输出级别：ERROR > WARN > INFO > DEBUG > TRACE
     * */

    //测试默认的日志输出级别
    @Test
    public void test3()
    {
        Logger logger = LoggerFactory.getLogger("mao.logback.HelloWorld");
        logger.error("error ...");
        logger.warn("warn ...");
        logger.info("info ...");
        logger.debug("debug ...");
        //因为默认的输出级别为debug，所以这一条日志不会输出
        logger.trace("trace ...");
    }

    //设置日志输出级别
    @Test
    public void test4()
    {
        ch.qos.logback.classic.Logger logger = (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("mao.logback.HelloWorld");
        logger.setLevel(Level.WARN);
        logger.error("error ...");
        logger.warn("warn ...");
        logger.info("info ...");
        logger.debug("debug ...");
        logger.trace("trace ...");
    }

    //测试Logger的继承
    @Test
    public void test5()
    {
        ch.qos.logback.classic.Logger logger =
                (ch.qos.logback.classic.Logger) LoggerFactory.getLogger("mao");
        logger.setLevel(Level.INFO);
        logger.error("error ...");
        logger.warn("warn ...");
        logger.info("info ...");
        logger.debug("debug ...");
        logger.trace("trace ...");

        // "mao.logback" 会继承 "mao" 的有效级别
        Logger barLogger = LoggerFactory.getLogger("mao.logback");
        // 这条日志会打印，因为 INFO >= INFO
        barLogger.info("子级信息");
        // 这条日志不会打印，因为 DEBUG < INFO
        barLogger.debug("子级调试信息");
    }

    //Logger获取，根据同一个名称获得的logger都是同一个实例
    @Test
    public void test6()
    {
        Logger logger1 = LoggerFactory.getLogger("mao");
        Logger logger2 = LoggerFactory.getLogger("mao");
        System.out.println(logger1 == logger2);
    }

    //参数化日志
    @Test
    public void test7()
    {
        Logger logger = LoggerFactory.getLogger("mao");
        logger.debug("hello {}", "world");
    }
}
```





![image-20221031185920833](img/logback学习笔记/image-20221031185920833.png)



![image-20221031185930069](img/logback学习笔记/image-20221031185930069.png)





![image-20221031185938957](img/logback学习笔记/image-20221031185938957.png)



![image-20221031185946577](img/logback学习笔记/image-20221031185946577.png)





![image-20221031185953283](img/logback学习笔记/image-20221031185953283.png)



![image-20221031190000702](img/logback学习笔记/image-20221031190000702.png)



![image-20221031190037521](img/logback学习笔记/image-20221031190037521.png)















### 案例二



#### 第一步：创建springboot工程springboot_logback_demo



![image-20221031190900228](img/logback学习笔记/image-20221031190900228.png)







#### 第二步：修改pom文件



```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>mao</groupId>
    <artifactId>springboot_logback_demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>springboot_logback_demo</name>
    <description>springboot_logback_demo</description>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

        <!--logback-->
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
            <version>1.2.3</version>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>

```







#### 第三步：在resources下编写logback配置文件logback-base.xml



```xml
<?xml version="1.0" encoding="UTF-8"?>
<included>
    <contextName>logback</contextName>
    <!--
      name的值是变量的名称，value的值时变量定义的值
      定义变量后，可以使“${}”来使用变量
   -->
    <property name="log.path" value="./logs"/>

    <!-- 彩色日志 -->
    <!-- 彩色日志依赖的渲染类 -->
    <conversionRule
            conversionWord="clr"
            converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
    <conversionRule
            conversionWord="wex"
            converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
    <conversionRule conversionWord="wEx"
                    converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
    <!-- 彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>

    <!--输出到控制台-->
    <appender name="LOG_CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <Pattern>${CONSOLE_LOG_PATTERN}</Pattern>
            <!-- 设置字符集 -->
            <charset>UTF-8</charset>
        </encoder>
    </appender>

    <!--输出到文件-->
    <appender name="LOG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
        <file>${log.path}/logback.log</file>
        <!--日志文件输出格式-->
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!-- 每天日志归档路径以及格式 -->
            <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
            <!--日志文件保留天数-->
            <maxHistory>365</maxHistory>
        </rollingPolicy>
    </appender>
</included>
```





#### 第四步：在resources下编写logback配置文件logback-spring.xml



```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!--引入其他配置文件-->
    <include resource="logback-base.xml"/>
    <!--
    <logger>用来设置某一个包或者具体的某一个类的日志打印级别、
    以及指定<appender>。<logger>仅有一个name属性，
    一个可选的level和一个可选的addtivity属性。
    name:用来指定受此logger约束的某一个包或者具体的某一个类。
    level:用来设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF，
          如果未设置此属性，那么当前logger将会继承上级的级别。
    addtivity:是否向上级logger传递打印信息。默认是true。
     -->

    <!--开发环境-->
<!--    <springProfile name="dev">-->
<!--        <logger name="包名" additivity="false" level="debug">-->
<!--            <appender-ref ref="LOG_CONSOLE"/>-->
<!--        </logger>-->
<!--    </springProfile>-->
<!--    &lt;!&ndash;生产环境&ndash;&gt;-->
<!--    <springProfile name="pro">-->
<!--        <logger name="包名" additivity="false" level="info">-->
<!--            <appender-ref ref="LOG_FILE"/>-->
<!--        </logger>-->
<!--    </springProfile>-->

    <!--
    root节点是必选节点，用来指定最基础的日志输出级别，只有一个level属性
    level:设置打印级别，大小写无关：TRACE, DEBUG, INFO, WARN, ERROR, ALL 和 OFF 默认是DEBUG
    可以包含零个或多个元素，标识这个appender将会添加到这个logger。
    -->
    <root level="info">
        <appender-ref ref="LOG_CONSOLE"/>
        <appender-ref ref="LOG_FILE"/>
    </root>
</configuration>
```





#### 第五步：编写application.yml文件



```yaml
logging:
  config: classpath:logback-spring.xml

spring:
  profiles:
    active: dev
```







#### 第六步：创建并编写UserController



```java
package mao.springboot_logback_demo.controller;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Project name(项目名称)：springboot_logback_demo
 * Package(包名): mao.springboot_logback_demo.controller
 * Class(类名): UserController
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 19:24
 * Version(版本): 1.0
 * Description(描述)： 无
 */

@RestController
@RequestMapping("/user")
public class UserController
{
    private static final Logger log = LoggerFactory.getLogger(UserController.class);

    @GetMapping("/get")
    public String get()
    {
        log.trace("trace...");
        log.debug("debug...");
        log.info("info...");
        log.warn("warn...");
        log.error("error...");
        return "OK";
    }
}
```



#### 第七步：启动并访问



http://localhost:8080/user/get



![image-20221031193632439](img/logback学习笔记/image-20221031193632439.png)





可以看到控制台已经开始输出日志信息。

修改application.yml文件中的开发模式为pro，重启项目这日志输出到了文件中。











## Spring Event

### 介绍

Spring Event是Spring的事件通知机制，可以将相互耦合的代码解耦，从而方便功能的修改与添加。Spring Event是监听者模式的一个具体实现。

监听者模式包含了监听者Listener、事件Event、事件发布者EventPublish，过程就是EventPublish发布一个事件，被监听者捕获到，然后执行事件相应的方法。

Spring Event的相关API在spring-context包中。







### Spring Event入门案例



第一步：创建工程spring_event_demo



![image-20221031194439225](img/logback学习笔记/image-20221031194439225.png)





pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>mao</groupId>
    <artifactId>spring_event_demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>spring_event_demo</name>
    <description>spring_event_demo</description>
    <properties>
        <java.version>11</java.version>
    </properties>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```







第二步：创建OptLogDTO类，用于封装操作日志信息



```java
package mao.spring_event_demo.entity;

/**
 * Project name(项目名称)：spring_event_demo
 * Package(包名): mao.spring_event_demo.entity
 * Class(类名): OptLogDTO
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 20:01
 * Version(版本): 1.0
 * Description(描述)： 无
 */


public class OptLogDTO
{
    /**
     * 操作IP
     */
    private String requestIp;

    /**
     * 日志类型 LogType{OPT:操作类型;EX:异常类型}
     */
    private String type;

    /**
     * 操作人
     */
    private String userName;

    /**
     * 操作描述
     */
    private String description;

    /**
     * Instantiates a new Opt log dto.
     */
    public OptLogDTO()
    {

    }

    /**
     * Instantiates a new Opt log dto.
     *
     * @param requestIp   the request ip
     * @param type        the type
     * @param userName    the user name
     * @param description the description
     */
    public OptLogDTO(String requestIp, String type, String userName, String description)
    {
        this.requestIp = requestIp;
        this.type = type;
        this.userName = userName;
        this.description = description;
    }

    /**
     * Gets request ip.
     *
     * @return the request ip
     */
    public String getRequestIp()
    {
        return requestIp;
    }

    /**
     * Sets request ip.
     *
     * @param requestIp the request ip
     */
    public void setRequestIp(String requestIp)
    {
        this.requestIp = requestIp;
    }

    /**
     * Gets type.
     *
     * @return the type
     */
    public String getType()
    {
        return type;
    }

    /**
     * Sets type.
     *
     * @param type the type
     */
    public void setType(String type)
    {
        this.type = type;
    }

    /**
     * Gets user name.
     *
     * @return the user name
     */
    public String getUserName()
    {
        return userName;
    }

    /**
     * Sets user name.
     *
     * @param userName the user name
     */
    public void setUserName(String userName)
    {
        this.userName = userName;
    }

    /**
     * Gets description.
     *
     * @return the description
     */
    public String getDescription()
    {
        return description;
    }

    /**
     * Sets description.
     *
     * @param description the description
     */
    public void setDescription(String description)
    {
        this.description = description;
    }

    @Override
    public boolean equals(Object o)
    {
        if (this == o)
        {
            return true;
        }
        if (o == null || getClass() != o.getClass())
        {
            return false;
        }

        OptLogDTO optLogDTO = (OptLogDTO) o;

        if (getRequestIp() != null ? !getRequestIp().equals(optLogDTO.getRequestIp()) : optLogDTO.getRequestIp() != null)
        {
            return false;
        }
        if (getType() != null ? !getType().equals(optLogDTO.getType()) : optLogDTO.getType() != null)
        {
            return false;
        }
        if (getUserName() != null ? !getUserName().equals(optLogDTO.getUserName()) : optLogDTO.getUserName() != null)
        {
            return false;
        }
        return getDescription() != null ? getDescription().equals(optLogDTO.getDescription()) : optLogDTO.getDescription() == null;
    }

    @Override
    public int hashCode()
    {
        int result = getRequestIp() != null ? getRequestIp().hashCode() : 0;
        result = 31 * result + (getType() != null ? getType().hashCode() : 0);
        result = 31 * result + (getUserName() != null ? getUserName().hashCode() : 0);
        result = 31 * result + (getDescription() != null ? getDescription().hashCode() : 0);
        return result;
    }

    @Override
    public String toString()
    {
        final StringBuffer stringBuffer = new StringBuffer("OptLogDTO{");
        stringBuffer.append("requestIp='").append(requestIp).append('\'');
        stringBuffer.append(", type='").append(type).append('\'');
        stringBuffer.append(", userName='").append(userName).append('\'');
        stringBuffer.append(", description='").append(description).append('\'');
        stringBuffer.append('}');
        return stringBuffer.toString();
    }
}

```







第三步：创建事件类SysLogEvent



```java
package mao.spring_event_demo.event;

import mao.spring_event_demo.entity.OptLogDTO;
import org.springframework.context.ApplicationEvent;

/**
 * Project name(项目名称)：spring_event_demo
 * Package(包名): mao.spring_event_demo.event
 * Class(类名): SysLogEvent
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 20:03
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class SysLogEvent extends ApplicationEvent
{
    public SysLogEvent(OptLogDTO optLogDTO)
    {
        super(optLogDTO);
    }
}
```





第四步：创建监听器类SysLogListener



```java
package mao.spring_event_demo.listener;

import mao.spring_event_demo.entity.OptLogDTO;
import mao.spring_event_demo.event.SysLogEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.context.event.EventListener;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;

import javax.annotation.PostConstruct;

/**
 * Project name(项目名称)：spring_event_demo
 * Package(包名): mao.spring_event_demo.listener
 * Class(类名): SysLogListener
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 20:05
 * Version(版本): 1.0
 * Description(描述)： 无
 */

@Component
public class SysLogListener
{
    private static final Logger log = LoggerFactory.getLogger(SysLogListener.class);

    @Async
    @EventListener(SysLogEvent.class)
    public void saveSysLog(SysLogEvent event)
    {
        OptLogDTO sysLog = (OptLogDTO) event.getSource();
        long id = Thread.currentThread().getId();
        log.info("监听到日志操作事件：" + sysLog + " 线程id：" + id);
        //将日志信息保存到数据库或者其它地方

    }

    @PostConstruct
    public void init()
    {
        log.info("初始化SysLogListener");
    }
}

```







第五步：创建Controller，用于发布事件



```java
package mao.spring_event_demo.controller;

import mao.spring_event_demo.entity.OptLogDTO;
import mao.spring_event_demo.event.SysLogEvent;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationEvent;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * Project name(项目名称)：spring_event_demo
 * Package(包名): mao.spring_event_demo.controller
 * Class(类名): UserController
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 20:09
 * Version(版本): 1.0
 * Description(描述)： 无
 */

@RestController
@RequestMapping("/user")
public class UserController
{
    @Autowired
    private ApplicationContext applicationContext;

    private static final Logger log = LoggerFactory.getLogger(UserController.class);

    @GetMapping("/getUser")
    public String getUser()
    {
        //构造操作日志信息
        OptLogDTO logInfo = new OptLogDTO();
        logInfo.setRequestIp("127.0.0.1");
        logInfo.setUserName("admin");
        logInfo.setType("OPT");
        logInfo.setDescription("查询用户信息");

        //构造事件对象
        ApplicationEvent event = new SysLogEvent(logInfo);

        //发布事件
        applicationContext.publishEvent(event);

        long id = Thread.currentThread().getId();
        log.info("发布事件,线程id：" + id);
        return "OK";
    }
}

```





第六步：在启动类上添加EnableAsync注解



```java
package mao.spring_event_demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.scheduling.annotation.EnableAsync;

@SpringBootApplication
@EnableAsync
public class SpringEventDemoApplication
{

    public static void main(String[] args)
    {
        SpringApplication.run(SpringEventDemoApplication.class, args);
    }

}
```





第七步：启动程序



```sh

  .   ____          _            __ _ _
 /\\ / ___'_ __ _ _(_)_ __  __ _ \ \ \ \
( ( )\___ | '_ | '_| | '_ \/ _` | \ \ \ \
 \\/  ___)| |_)| | | | | || (_| |  ) ) ) )
  '  |____| .__|_| |_|_| |_\__, | / / / /
 =========|_|==============|___/=/_/_/_/
 :: Spring Boot ::                (v2.7.1)

2022-10-31 20:28:16.647  INFO 11836 --- [           main] m.s.SpringEventDemoApplication           : Starting SpringEventDemoApplication using Java 16.0.2 on mao with PID 11836 (H:\程序\大四上期\spring_event_demo\target\classes started by mao in H:\程序\大四上期\spring_event_demo)
2022-10-31 20:28:16.649  INFO 11836 --- [           main] m.s.SpringEventDemoApplication           : No active profile set, falling back to 1 default profile: "default"
2022-10-31 20:28:17.281  INFO 11836 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat initialized with port(s): 8080 (http)
2022-10-31 20:28:17.287  INFO 11836 --- [           main] o.apache.catalina.core.StandardService   : Starting service [Tomcat]
2022-10-31 20:28:17.287  INFO 11836 --- [           main] org.apache.catalina.core.StandardEngine  : Starting Servlet engine: [Apache Tomcat/9.0.64]
2022-10-31 20:28:17.357  INFO 11836 --- [           main] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring embedded WebApplicationContext
2022-10-31 20:28:17.358  INFO 11836 --- [           main] w.s.c.ServletWebServerApplicationContext : Root WebApplicationContext: initialization completed in 672 ms
2022-10-31 20:28:17.393  INFO 11836 --- [           main] m.s.listener.SysLogListener              : 初始化SysLogListener
2022-10-31 20:28:17.632  INFO 11836 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2022-10-31 20:28:17.641  INFO 11836 --- [           main] m.s.SpringEventDemoApplication           : Started SpringEventDemoApplication in 1.259 seconds (JVM running for 1.697)
```









第八步：访问



http://localhost:8080/user/getUser



```sh
2022-10-31 20:28:17.393  INFO 11836 --- [           main] m.s.listener.SysLogListener              : 初始化SysLogListener
2022-10-31 20:28:17.632  INFO 11836 --- [           main] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 8080 (http) with context path ''
2022-10-31 20:28:17.641  INFO 11836 --- [           main] m.s.SpringEventDemoApplication           : Started SpringEventDemoApplication in 1.259 seconds (JVM running for 1.697)
2022-10-31 20:28:30.387  INFO 11836 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : Initializing Spring DispatcherServlet 'dispatcherServlet'
2022-10-31 20:28:30.387  INFO 11836 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Initializing Servlet 'dispatcherServlet'
2022-10-31 20:28:30.388  INFO 11836 --- [nio-8080-exec-1] o.s.web.servlet.DispatcherServlet        : Completed initialization in 1 ms
2022-10-31 20:28:30.404  INFO 11836 --- [nio-8080-exec-1] m.s.controller.UserController            : 发布事件,线程id：35
2022-10-31 20:28:30.412  INFO 11836 --- [         task-1] m.s.listener.SysLogListener              : 监听到日志操作事件：OptLogDTO{requestIp='127.0.0.1', type='OPT', userName='admin', description='查询用户信息'} 线程id：53
2022-10-31 20:28:30.572  INFO 11836 --- [nio-8080-exec-2] m.s.controller.UserController            : 发布事件,线程id：36
2022-10-31 20:28:30.572  INFO 11836 --- [         task-2] m.s.listener.SysLogListener              : 监听到日志操作事件：OptLogDTO{requestIp='127.0.0.1', type='OPT', userName='admin', description='查询用户信息'} 线程id：54
2022-10-31 20:28:30.789  INFO 11836 --- [nio-8080-exec-3] m.s.controller.UserController            : 发布事件,线程id：37
2022-10-31 20:28:30.789  INFO 11836 --- [         task-3] m.s.listener.SysLogListener              : 监听到日志操作事件：OptLogDTO{requestIp='127.0.0.1', type='OPT', userName='admin', description='查询用户信息'} 线程id：55
2022-10-31 20:28:31.936  INFO 11836 --- [nio-8080-exec-4] m.s.controller.UserController            : 发布事件,线程id：38
2022-10-31 20:28:31.936  INFO 11836 --- [         task-4] m.s.listener.SysLogListener              : 监听到日志操作事件：OptLogDTO{requestIp='127.0.0.1', type='OPT', userName='admin', description='查询用户信息'} 线程id：56
2022-10-31 20:28:32.369  INFO 11836 --- [nio-8080-exec-5] m.s.controller.UserController            : 发布事件,线程id：39
2022-10-31 20:28:32.370  INFO 11836 --- [         task-5] m.s.listener.SysLogListener              : 监听到日志操作事件：OptLogDTO{requestIp='127.0.0.1', type='OPT', userName='admin', description='查询用户信息'} 线程id：57
```























## 自定义spring boot starter

tools-log的开发步骤为：

1、定义日志操作事件类SysLogEvent

2、定义@SysLog注解，用于在Controller的方法上标注当前方法需要进行操作日志的保存处理

3、定义切面类SysLogAspect

4、在切面类SysLogAspect中定义切点，拦截Controller中添加@SysLog注解的方法

5、在切面类SysLogAspect中定义前置通知，在前置通知方法recordLog中收集操作日志相关信息封装为OptLogDTO对象并保存到ThreadLocal中

6、在切面类SysLogAspect中定义后置通知，在后置通知方法doAfterReturning中通过ThreadLocal 获取OptLogDTO并继续设置其他的操作信息到OptLogDTO

7、在切面类SysLogAspect的后置通知方法doAfterReturning中发布事件SysLogEvent

8、定义监听器SysLogListener，监听日志发布事件SysLogEvent

9、定义配置类LogAutoConfiguration，用于自动配置切面SysLogAspect对象

10、定义starter所需的META-INF/spring.factories文件，并配置自动配置类LogAutoConfiguration





### 开发starter



第一步：初始化项目



创建父工程logback_spring_boot_starter_demo



![image-20221031211830329](img/logback学习笔记/image-20221031211830329.png)







创建子工程tools-log



![image-20221031212026706](img/logback学习笔记/image-20221031212026706.png)





创建子工程use-starter



![image-20221031212411768](img/logback学习笔记/image-20221031212411768.png)







第二步：修改pom文件





父工程logback_spring_boot_starter_demo的pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.1</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>mao</groupId>
    <artifactId>logback_spring_boot_starter_demo</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>logback_spring_boot_starter_demo</name>
    <description>logback_spring_boot_starter_demo</description>
    <packaging>pom</packaging>

    <properties>
        <java.version>11</java.version>
    </properties>

    <dependencies>

    </dependencies>

    <dependencyManagement>
        <dependencies>

        </dependencies>
    </dependencyManagement>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```





子工程tools-log的pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>logback_spring_boot_starter_demo</artifactId>
        <groupId>mao</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>tools-log</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>tools-log</name>
    <description>tools-log</description>
    <properties>

    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--logback-->
<!--        <dependency>-->
<!--            <groupId>ch.qos.logback</groupId>-->
<!--            <artifactId>logback-classic</artifactId>-->
<!--            <version>1.2.3</version>-->
<!--        </dependency>-->
<!--        <dependency>-->
<!--            <groupId>ch.qos.logback</groupId>-->
<!--            <artifactId>logback-core</artifactId>-->
<!--            <version>1.2.3</version>-->
<!--        </dependency>-->


        <dependency>
            <groupId>org.lionsoul</groupId>
            <artifactId>ip2region</artifactId>
            <version>1.7.2</version>
        </dependency>
        <dependency>
            <groupId>eu.bitwalker</groupId>
            <artifactId>UserAgentUtils</artifactId>
            <version>1.21</version>
        </dependency>
        <dependency>
            <groupId>commons-io</groupId>
            <artifactId>commons-io</artifactId>
            <version>2.11.0</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aspects</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.1.0</version>
        </dependency>

        <dependency>
            <groupId>com.github.xiaoymin</groupId>
            <artifactId>knife4j-spring-boot-starter</artifactId>
            <version>2.0.1</version>
        </dependency>

        <!--阿里巴巴的FastJson json解析-->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>1.2.79</version>
        </dependency>


        <!--spring boot starter开发依赖-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-autoconfigure</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-configuration-processor</artifactId>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <skip>true</skip>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```



不需要在导入logback了，因为spring-boot-starter-web已经包含了logback



![image-20221101134029698](img/logback学习笔记/image-20221101134029698.png)









工程use-starter的pom文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <artifactId>logback_spring_boot_starter_demo</artifactId>
        <groupId>mao</groupId>
        <version>0.0.1-SNAPSHOT</version>
    </parent>

    <artifactId>use-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>use-starter</name>
    <description>use-starter</description>

    <properties>

    </properties>

    <dependencies>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>

    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>

</project>
```







第三步：编写工具类AddressUtil



```java
package mao.tools_log.utils;

import org.lionsoul.ip2region.DataBlock;
import org.lionsoul.ip2region.DbConfig;
import org.lionsoul.ip2region.DbSearcher;
import org.lionsoul.ip2region.Util;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import cn.hutool.core.io.resource.ResourceUtil;
import cn.hutool.core.util.StrUtil;
import org.apache.commons.io.FileUtils;

import java.io.File;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.Method;


/**
 * Project name(项目名称)：logback_spring_boot_starter_demo
 * Package(包名): mao.tools_log.utils
 * Class(类名): AddressUtil
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 22:17
 * Version(版本): 1.0
 * Description(描述)： 解析ip地址的工具类
 */

public class AddressUtil
{
    private static final String JAVA_TEMP_DIR = "java.io.tmpdir";

    private static final Logger log = LoggerFactory.getLogger(AddressUtil.class);

    static DbConfig config = null;
    static DbSearcher searcher = null;


    /**
     * 根据ip查询地址
     *
     * @param ip ip地址
     * @return {@link String}
     */
    /*public static String getCityInfo(String ip)
    {
        DbSearcher searcher = null;
        try
        {
            String dbPath = AddressUtil.class.getResource("/ip2region/ip2region.db").getPath();
            File file = new File(dbPath);
            if (!file.exists())
            {
                String tmpDir = System.getProperties().getProperty(JAVA_TEMP_DIR);
                dbPath = tmpDir + "ip2region.db";
                file = new File(dbPath);
                String classPath = "classpath:ip2region/ip2region.db";
                InputStream resourceAsStream = ResourceUtil.getStreamSafe(classPath);
                if (resourceAsStream != null)
                {
                    FileUtils.copyInputStreamToFile(resourceAsStream, file);
                }
            }
            DbConfig config = new DbConfig();
            searcher = new DbSearcher(config, file.getPath());
            Method method = searcher.getClass().getMethod("btreeSearch", String.class);
            if (!Util.isIpAddress(ip))
            {
                log.error("Error: Invalid ip address");
            }
            DataBlock dataBlock = (DataBlock) method.invoke(searcher, ip);
            return dataBlock.getRegion();
        }
        catch (Exception e)
        {
            log.error("获取地址信息异常，", e);
            return StrUtil.EMPTY;
        }
        finally
        {
            if (searcher != null)
            {
                try
                {
                    searcher.close();
                }
                catch (IOException e)
                {
                    e.printStackTrace();
                }
            }
        }
    }*/

    /*
     * 初始化IP库
     */
    static
    {
        try
        {
            // 因为jar无法读取文件,复制创建临时文件
//            String tmpDir = System.getProperty("user.dir") + File.separator + "temp";
//            String dbPath = tmpDir + File.separator + "ip2region.db";
//            log.info("init ip region db path [{}]", dbPath);
//            File file = new File(dbPath);
//            FileUtils.copyInputStreamToFile(AddressUtil.class.getClassLoader().getResourceAsStream("ip2region/ip2region.db"), file);
            String dbPath = AddressUtil.class.getResource("/ip2region/ip2region.db").getPath();
            File file = new File(dbPath);
            if (!file.exists())
            {
                String tmpDir = System.getProperties().getProperty(JAVA_TEMP_DIR);
                dbPath = tmpDir + "ip2region.db";
                file = new File(dbPath);
                String classPath = "classpath:ip2region/ip2region.db";
                InputStream resourceAsStream = ResourceUtil.getStreamSafe(classPath);
                if (resourceAsStream != null)
                {
                    FileUtils.copyInputStreamToFile(resourceAsStream, file);
                }
            }
            config = new DbConfig();
            searcher = new DbSearcher(config, dbPath);
            log.info("bean [{}]", config);
            log.info("bean [{}]", searcher);
        }
        catch (Exception e)
        {
            log.error("init ip region error:", e);
        }
    }


    /**
     * 解析ip
     *
     * @param ip ip地址
     * @return {@link String}
     */
    public static String getRegion(String ip)
    {
        try
        {
            //db
            if (searcher == null || StrUtil.isEmpty(ip))
            {
                log.error("DbSearcher is null");
                return StrUtil.EMPTY;
            }
            long startTime = System.currentTimeMillis();
            //查询算法
            int algorithm = DbSearcher.MEMORY_ALGORITYM;
            Method method = null;
            switch (algorithm)
            {
                case DbSearcher.BTREE_ALGORITHM:
                    method = searcher.getClass().getMethod("btreeSearch", String.class);
                    break;
                case DbSearcher.BINARY_ALGORITHM:
                    method = searcher.getClass().getMethod("binarySearch", String.class);
                    break;
                case DbSearcher.MEMORY_ALGORITYM:
                    method = searcher.getClass().getMethod("memorySearch", String.class);
                    break;
            }

            DataBlock dataBlock = null;
            if (!Util.isIpAddress(ip))
            {
                log.warn("warning: Invalid ip address");
            }
            dataBlock = (DataBlock) method.invoke(searcher, ip);
            String result = dataBlock.getRegion();
            long endTime = System.currentTimeMillis();
            log.debug("region use time[{}] result[{}]", endTime - startTime, result);
            return result;

        }
        catch (Exception e)
        {
            log.error("error:", e);
        }
        return StrUtil.EMPTY;
    }

    public static void main(String[] args)
    {
        System.out.println(AddressUtil.getRegion("113.222.142.84"));
        System.out.println(AddressUtil.getRegion("113.221.141.84"));
        System.out.println(AddressUtil.getRegion("113.192.142.84"));
        System.out.println(AddressUtil.getRegion("113.224.142.84"));
        System.out.println(AddressUtil.getRegion("114.222.142.84"));
        System.out.println(AddressUtil.getRegion("115.222.142.84"));
        System.out.println(AddressUtil.getRegion("117.222.142.84"));
        System.out.println(AddressUtil.getRegion("119.222.142.84"));
        System.out.println(AddressUtil.getRegion("13.222.142.84"));
        System.out.println(AddressUtil.getRegion("14.222.142.84"));
        System.out.println(AddressUtil.getRegion("15.222.142.84"));
        System.out.println(AddressUtil.getRegion("16.222.142.84"));
    }
}
```







第四步：编写工具类LogUtil



```java
package mao.tools_log.utils;

import mao.tools_log.annotation.SysLog;
import org.aspectj.lang.JoinPoint;

import java.io.PrintWriter;
import java.io.StringWriter;
import java.lang.reflect.Method;

/**
 * Project name(项目名称)：logback_spring_boot_starter_demo
 * Package(包名): mao.tools_log.utils
 * Class(类名): LogUtil
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 22:16
 * Version(版本): 1.0
 * Description(描述)： 无
 */

public class LogUtil
{
    /***
     * 获取操作信息
     * @param point JoinPoint对象
     * @return String
     */
    public static String getControllerMethodDescription(JoinPoint point)
    {
        try
        {
            // 获取连接点目标类名
            String targetName = point.getTarget().getClass().getName();
            // 获取连接点签名的方法名
            String methodName = point.getSignature().getName();
            //获取连接点参数
            Object[] args = point.getArgs();
            //根据连接点类的名字获取指定类
            Class targetClass = Class.forName(targetName);
            //获取类里面的方法
            Method[] methods = targetClass.getMethods();
            String description = "";
            for (Method method : methods)
            {
                if (method.getName().equals(methodName))
                {
                    Class[] clazzs = method.getParameterTypes();
                    if (clazzs.length == args.length)
                    {
                        description = method.getAnnotation(SysLog.class).value();
                        break;
                    }
                }
            }
            return description;
        }
        catch (Exception e)
        {
            return "";
        }
    }


    /**
     * 获取堆栈信息
     *
     * @param throwable throwable
     * @return {@link String}
     */
    public static String getStackTrace(Throwable throwable)
    {
        StringWriter sw = new StringWriter();
        try (PrintWriter pw = new PrintWriter(sw))
        {
            throwable.printStackTrace(pw);
            return sw.toString();
        }
    }
}
```





第五步：编写工具类NumberHelper



```java
package mao.tools_log.utils;

import java.util.function.Function;


/**
 * 数字类型 帮助类
 */
public class NumberHelper
{

    private static <T, R> R valueOfDef(T t, Function<T, R> function, R def)
    {
        try
        {
            return function.apply(t);
        }
        catch (Exception e)
        {
            return def;
        }
    }

    public static Long longValueOfNil(String value)
    {
        return valueOfDef(value, Long::valueOf, null);
    }

    public static Long longValueOf0(String value)
    {
        return valueOfDef(value, Long::valueOf, 0L);
    }

    public static Long longValueOfNil(Object value)
    {
        return valueOfDef(value, (val) -> Long.valueOf(val.toString()), null);
    }

    public static Long longValueOf0(Object value)
    {
        return valueOfDef(value, (val) -> Long.valueOf(val.toString()), 0L);
    }

    public static Boolean boolValueOf0(Object value)
    {
        return valueOfDef(value, (val) -> Boolean.valueOf(val.toString()), false);
    }

    public static Integer intValueOfNil(String value)
    {
        return valueOfDef(value, Integer::valueOf, null);
    }

    public static Integer intValueOf0(String value)
    {
        return intValueOf(value, 0);
    }

    public static Integer intValueOf(String value, Integer def)
    {
        return valueOfDef(value, Integer::valueOf, def);
    }

    public static Integer intValueOfNil(Object value)
    {
        return valueOfDef(value, (val) -> Integer.valueOf(val.toString()), null);
    }

    public static Integer intValueOf0(Object value)
    {
        return valueOfDef(value, (val) -> Integer.valueOf(val.toString()), 0);
    }

    public static Integer getOrDef(Integer val, Integer def)
    {
        return val == null ? def : val;
    }

    public static Long getOrDef(Long val, Long def)
    {
        return val == null ? def : val;
    }

    public static Boolean getOrDef(Boolean val, Boolean def)
    {
        return val == null ? def : val;
    }

}
```





第六步：编写工具类StrHelper



```java
package mao.tools_log.utils;

import java.net.URLDecoder;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;

import cn.hutool.core.util.StrUtil;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

/**
 * 字符串帮助类
 */

public class StrHelper
{

    private static final Logger log = LoggerFactory.getLogger(StrHelper.class);

    public static String getObjectValue(Object obj)
    {
        return obj == null ? "" : obj.toString();
    }

    public static String encode(String value)
    {
        try
        {
            return URLEncoder.encode(value, StandardCharsets.UTF_8);
        }
        catch (Exception e)
        {
            return "";
        }
    }

    public static String decode(String value)
    {
        try
        {
            return URLDecoder.decode(value, StandardCharsets.UTF_8);
        }
        catch (Exception e)
        {
            return "";
        }
    }

    public static String getOrDef(String val, String def)
    {
        return StrUtil.isEmpty(val) ? def : val;
    }
}
```





第七步：编写类ApplicationLoggerInitializer



```java
package mao.tools_log.init;

import org.springframework.context.ApplicationContextInitializer;
import org.springframework.context.ConfigurableApplicationContext;
import org.springframework.core.env.ConfigurableEnvironment;

/**
 * Project name(项目名称)：logback_spring_boot_starter_demo
 * Package(包名): mao.tools_log.init
 * Class(类名): ApplicationLoggerInitializer
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 21:43
 * Version(版本): 1.0
 * Description(描述)：
 * <p>
 * 通过环境变量的形式注入 logging.file
 * 自动维护 Spring Boot Admin Logger Viewer
 */

public class ApplicationLoggerInitializer implements ApplicationContextInitializer<ConfigurableApplicationContext>
{
    @Override
    public void initialize(ConfigurableApplicationContext applicationContext)
    {
        ConfigurableEnvironment environment = applicationContext.getEnvironment();
        String logBase = environment.getProperty("logging.path", "/data/projects/logs");
        String appName = environment.getProperty("spring.application.name");
        // spring boot admin 直接加载日志
        System.setProperty("logging.file", String.format("%s/%s/root.log", logBase, appName));

        // nacos的日志文件路径
        System.setProperty("nacos.logging.path", String.format("%s/%s", logBase, appName));
        //这里设置了无效，跟启动时，传递 -Dcom.alibaba.nacos.naming.log.level=warn 一样，可能是nacos的bug
//        System.setProperty("com.alibaba.nacos.naming.log.level", "warn");
//        System.setProperty("com.alibaba.nacos.config.log.level", "info");
    }
}
```





第八步：编写接口BaseExceptionCode



```java
package mao.tools_log.exception.code;

/**
 *
 */
public interface BaseExceptionCode
{
    /**
     * 异常编码
     *
     * @return int
     */
    int getCode();

    /**
     * 异常消息
     *
     * @return String
     */
    String getMsg();
}
```





第九步：编写类ExceptionCode



```java
package mao.tools_log.exception.code;


/**
 * 全局错误码 10000-15000
 * <p>
 * 预警异常编码    范围： 30000~34999
 * 标准服务异常编码 范围：35000~39999
 * 邮件服务异常编码 范围：40000~44999
 * 短信服务异常编码 范围：45000~49999
 * 权限服务异常编码 范围：50000-59999
 * 文件服务异常编码 范围：60000~64999
 * 日志服务异常编码 范围：65000~69999
 * 消息服务异常编码 范围：70000~74999
 * 开发者平台异常编码 范围：75000~79999
 * 搜索服务异常编码 范围：80000-84999
 * 共享交换异常编码 范围：85000-89999
 * 移动终端平台 异常码 范围：90000-94999
 * <p>
 * 安全保障平台    范围：        95000-99999
 * 软硬件平台 异常编码 范围：    100000-104999
 * 运维服务平台 异常编码 范围：  105000-109999
 * 统一监管平台异常 编码 范围：  110000-114999
 * 认证方面的异常编码  范围：115000-115999
 */
public enum ExceptionCode implements BaseExceptionCode
{

    //系统相关 start
    SUCCESS(0, "成功"),
    SYSTEM_BUSY(-1, "系统繁忙~请稍后再试~"),
    SYSTEM_TIMEOUT(-2, "系统维护中~请稍后再试~"),
    PARAM_EX(-3, "参数类型解析异常"),
    SQL_EX(-4, "运行SQL出现异常"),
    NULL_POINT_EX(-5, "空指针异常"),
    ILLEGALA_ARGUMENT_EX(-6, "无效参数异常"),
    MEDIA_TYPE_EX(-7, "请求类型异常"),
    LOAD_RESOURCES_ERROR(-8, "加载资源出错"),
    BASE_VALID_PARAM(-9, "统一验证参数异常"),
    OPERATION_EX(-10, "操作异常"),


    OK(200, "OK"),
    BAD_REQUEST(400, "错误的请求"),
    /**
     * {@code 401 Unauthorized}.
     *
     * @see <a href="http://tools.ietf.org/html/rfc7235#section-3.1">HTTP/1.1: Authentication, section 3.1</a>
     */
    UNAUTHORIZED(401, "未经授权"),
    /**
     * {@code 404 Not Found}.
     *
     * @see <a href="http://tools.ietf.org/html/rfc7231#section-6.5.4">HTTP/1.1: Semantics and Content, section 6.5.4</a>
     */
    NOT_FOUND(404, "没有找到资源"),
    METHOD_NOT_ALLOWED(405, "不支持当前请求类型"),

    TOO_MANY_REQUESTS(429, "请求超过次数限制"),
    INTERNAL_SERVER_ERROR(500, "内部服务错误"),
    BAD_GATEWAY(502, "网关错误"),
    GATEWAY_TIMEOUT(504, "网关超时"),
    //系统相关 end

    REQUIRED_FILE_PARAM_EX(1001, "请求中必须至少包含一个有效文件"),
    //jwt token 相关 start

    JWT_TOKEN_EXPIRED(40001, "会话超时，请重新登录"),
    JWT_SIGNATURE(40002, "不合法的token，请认真比对 token 的签名"),
    JWT_ILLEGAL_ARGUMENT(40003, "缺少token参数"),
    JWT_GEN_TOKEN_FAIL(40004, "生成token失败"),
    JWT_PARSER_TOKEN_FAIL(40005, "解析token失败"),
    JWT_USER_INVALID(40006, "用户名或密码错误"),
    JWT_USER_ENABLED(40007, "用户已经被禁用！"),
    //jwt token 相关 end

    ;

    private int code;
    private String msg;

    ExceptionCode(int code, String msg)
    {
        this.code = code;
        this.msg = msg;
    }

    @Override
    public int getCode()
    {
        return code;
    }

    @Override
    public String getMsg()
    {
        return msg;
    }


    public ExceptionCode build(String msg, Object... param)
    {
        this.msg = String.format(msg, param);
        return this;
    }

    public ExceptionCode param(Object... param)
    {
        msg = String.format(msg, param);
        return this;
    }
}
```





第十步：编写接口BaseException



```java
package mao.tools_log.exception;

/**
 * 异常接口类
 */
public interface BaseException
{

    /**
     * 统一参数验证异常码
     */
    int BASE_VALID_PARAM = -9;

    /**
     * 返回异常信息
     *
     * @return String
     */
    String getMessage();

    /**
     * 返回异常编码
     *
     * @return int
     */
    int getCode();

}
```





第十一步：编写类BaseUncheckedException



```java
package mao.tools_log.exception;

/**
 * 非运行期异常基类，所有自定义非运行时异常继承该类
 */
public class BaseUncheckedException extends RuntimeException implements BaseException
{

    private static final long serialVersionUID = -778887391066124051L;

    /**
     * 异常信息
     */
    protected String message;

    /**
     * 具体异常码
     */
    protected int code;

    public BaseUncheckedException(int code, String message)
    {
        super(message);
        this.code = code;
        this.message = message;
    }

    public BaseUncheckedException(int code, String format, Object... args)
    {
        super(String.format(format, args));
        this.code = code;
        this.message = String.format(format, args);
    }


    @Override
    public String getMessage()
    {
        return message;
    }

    @Override
    public int getCode()
    {
        return code;
    }
}
```





第十二步：编写类BizException



```java
package mao.tools_log.exception;


import mao.tools_log.exception.code.BaseExceptionCode;

/**
 * 业务异常
 * 用于在处理业务逻辑时，进行抛出的异常。
 */
public class BizException extends BaseUncheckedException
{

    private static final long serialVersionUID = -3843907364558373817L;

    public BizException(String message)
    {
        super(-1, message);
    }

    public BizException(int code, String message)
    {
        super(code, message);
    }

    public BizException(int code, String message, Object... args)
    {
        super(code, message, args);
    }

    /**
     * 实例化异常
     *
     * @param code    自定义异常编码
     * @param message 自定义异常消息
     * @param args    已定义异常参数
     * @return BizException
     */
    public static BizException wrap(int code, String message, Object... args)
    {
        return new BizException(code, message, args);
    }

    public static BizException wrap(String message, Object... args)
    {
        return new BizException(-1, message, args);
    }

    public static BizException validFail(String message, Object... args)
    {
        return new BizException(-9, message, args);
    }

    public static BizException wrap(BaseExceptionCode ex)
    {
        return new BizException(ex.getCode(), ex.getMsg());
    }

    @Override
    public String toString()
    {
        return "BizException [message=" + message + ", code=" + code + "]";
    }

}
```







第十三步：编写实体类OptLogDTO



```java
package mao.tools_log.entity;

import java.time.LocalDateTime;


/**
 * Project name(项目名称)：logback_spring_boot_starter_demo
 * Package(包名): mao.tools_log.entity
 * Class(类名): OptLogDTO
 * Author(作者）: mao
 * Author QQ：1296193245
 * GitHub：https://github.com/maomao124/
 * Date(创建日期)： 2022/10/31
 * Time(创建时间)： 21:47
 * Version(版本): 1.0
 * Description(描述)： 无
 */
public class OptLogDTO
{

    private static final long serialVersionUID = 1L;

    /**
     * 操作IP
     */
    private String requestIp;

    /**
     * 日志类型
     * #LogType{OPT:操作类型;EX:异常类型}
     */
    private String type;

    /**
     * 操作人
     */
    private String userName;

    /**
     * 操作描述
     */
    private String description;

    /**
     * 类路径
     */
    private String classPath;

    /**
     * 请求类型
     */
    private String actionMethod;

    /**
     * 请求地址
     */
    private String requestUri;

    /**
     * 请求类型
     * #HttpMethod{GET:GET请求;POST:POST请求;PUT:PUT请求;DELETE:DELETE请求;PATCH:PATCH请求;
     * TRACE:TRACE请求;HEAD:HEAD请求;OPTIONS:OPTIONS请求;}
     */
    private String httpMethod;

    /**
     * 请求参数
     */
    private String params;

    /**
     * 返回值
     */
    private String result;

    /**
     * 异常详情信息
     */
    private String exDesc;

    /**
     * 异常描述
     */
    private String exDetail;

    /**
     * 开始时间
     */
    private LocalDateTime startTime;

    /**
     * 完成时间
     */
    private LocalDateTime finishTime;

    /**
     * 消耗时间
     */
    private Long consumingTime;

    /**
     * 浏览器
     */
    private String ua;

    /**
     * 创建用户
     */
    private Long createUser;


    /**
     * Instantiates a new Opt log dto.
     */
    public OptLogDTO()
    {

    }

    /**
     * Instantiates a new Opt log dto.
     *
     * @param requestIp     the request ip
     * @param type          the type
     * @param userName      the user name
     * @param description   the description
     * @param classPath     the class path
     * @param actionMethod  the action method
     * @param requestUri    the request uri
     * @param httpMethod    the http method
     * @param params        the params
     * @param result        the result
     * @param exDesc        the ex desc
     * @param exDetail      the ex detail
     * @param startTime     the start time
     * @param finishTime    the finish time
     * @param consumingTime the consuming time
     * @param ua            the ua
     * @param createUser    the create user
     */
    public OptLogDTO(String requestIp, String type, String userName, String description,
                     String classPath, String actionMethod, String requestUri,
                     String httpMethod, String params, String result, String exDesc,
                     String exDetail, LocalDateTime startTime, LocalDateTime finishTime,
                     Long consumingTime, String ua, Long createUser)
    {
        this.requestIp = requestIp;
        this.type = type;
        this.userName = userName;
        this.description = description;
        this.classPath = classPath;
        this.actionMethod = actionMethod;
        this.requestUri = requestUri;
        this.httpMethod = httpMethod;
        this.params = params;
        this.result = result;
        this.exDesc = exDesc;
        this.exDetail = exDetail;
        this.startTime = startTime;
        this.finishTime = finishTime;
        this.consumingTime = consumingTime;
        this.ua = ua;
        this.createUser = createUser;
    }

    /**
     * Gets request ip.
     *
     * @return the request ip
     */
    public String getRequestIp()
    {
        return requestIp;
    }

    /**
     * Sets request ip.
     *
     * @param requestIp the request ip
     */
    public void setRequestIp(String requestIp)
    {
        this.requestIp = requestIp;
    }

    /**
     * Gets type.
     *
     * @return the type
     */
    public String getType()
    {
        return type;
    }

    /**
     * Sets type.
     *
     * @param type the type
     */
    public void setType(String type)
    {
        this.type = type;
    }

    /**
     * Gets user name.
     *
     * @return the user name
     */
    public String getUserName()
    {
        return userName;
    }

    /**
     * Sets user name.
     *
     * @param userName the user name
     */
    public void setUserName(String userName)
    {
        this.userName = userName;
    }

    /**
     * Gets description.
     *
     * @return the description
     */
    public String getDescription()
    {
        return description;
    }

    /**
     * Sets description.
     *
     * @param description the description
     */
    public void setDescription(String description)
    {
        this.description = description;
    }

    /**
     * Gets class path.
     *
     * @return the class path
     */
    public String getClassPath()
    {
        return classPath;
    }

    /**
     * Sets class path.
     *
     * @param classPath the class path
     */
    public void setClassPath(String classPath)
    {
        this.classPath = classPath;
    }

    /**
     * Gets action method.
     *
     * @return the action method
     */
    public String getActionMethod()
    {
        return actionMethod;
    }

    /**
     * Sets action method.
     *
     * @param actionMethod the action method
     */
    public void setActionMethod(String actionMethod)
    {
        this.actionMethod = actionMethod;
    }

    /**
     * Gets request uri.
     *
     * @return the request uri
     */
    public String getRequestUri()
    {
        return requestUri;
    }

    /**
     * Sets request uri.
     *
     * @param requestUri the request uri
     */
    public void setRequestUri(String requestUri)
    {
        this.requestUri = requestUri;
    }

    /**
     * Gets http method.
     *
     * @return the http method
     */
    public String getHttpMethod()
    {
        return httpMethod;
    }

    /**
     * Sets http method.
     *
     * @param httpMethod the http method
     */
    public void setHttpMethod(String httpMethod)
    {
        this.httpMethod = httpMethod;
    }

    /**
     * Gets params.
     *
     * @return the params
     */
    public String getParams()
    {
        return params;
    }

    /**
     * Sets params.
     *
     * @param params the params
     */
    public void setParams(String params)
    {
        this.params = params;
    }

    /**
     * Gets result.
     *
     * @return the result
     */
    public String getResult()
    {
        return result;
    }

    /**
     * Sets result.
     *
     * @param result the result
     */
    public void setResult(String result)
    {
        this.result = result;
    }

    /**
     * Gets ex desc.
     *
     * @return the ex desc
     */
    public String getExDesc()
    {
        return exDesc;
    }

    /**
     * Sets ex desc.
     *
     * @param exDesc the ex desc
     */
    public void setExDesc(String exDesc)
    {
        this.exDesc = exDesc;
    }

    /**
     * Gets ex detail.
     *
     * @return the ex detail
     */
    public String getExDetail()
    {
        return exDetail;
    }

    /**
     * Sets ex detail.
     *
     * @param exDetail the ex detail
     */
    public void setExDetail(String exDetail)
    {
        this.exDetail = exDetail;
    }

    /**
     * Gets start time.
     *
     * @return the start time
     */
    public LocalDateTime getStartTime()
    {
        return startTime;
    }

    /**
     * Sets start time.
     *
     * @param startTime the start time
     */
    public void setStartTime(LocalDateTime startTime)
    {
        this.startTime = startTime;
    }

    /**
     * Gets finish time.
     *
     * @return the finish time
     */
    public LocalDateTime getFinishTime()
    {
        return finishTime;
    }

    /**
     * Sets finish time.
     *
     * @param finishTime the finish time
     */
    public void setFinishTime(LocalDateTime finishTime)
    {
        this.finishTime = finishTime;
    }

    /**
     * Gets consuming time.
     *
     * @return the consuming time
     */
    public Long getConsumingTime()
    {
        return consumingTime;
    }

    /**
     * Sets consuming time.
     *
     * @param consumingTime the consuming time
     */
    public void setConsumingTime(Long consumingTime)
    {
        this.consumingTime = consumingTime;
    }

    /**
     * Gets ua.
     *
     * @return the ua
     */
    public String getUa()
    {
        return ua;
    }

    /**
     * Sets ua.
     *
     * @param ua the ua
     */
    public void setUa(String ua)
    {
        this.ua = ua;
    }

    /**
     * Gets create user.
     *
     * @return the create user
     */
    public Long getCreateUser()
    {
        return createUser;
    }

    /**
     * Sets create user.
     *
     * @param createUser the create user
     */
    public void setCreateUser(Long createUser)
    {
        this.createUser = createUser;
    }

    @Override
    public boolean equals(Object o)
    {
        if (this == o)
        {
            return true;
        }
        if (o == null || getClass() != o.getClass())
        {
            return false;
        }

        OptLogDTO optLogDTO = (OptLogDTO) o;

        if (getRequestIp() != null ? !getRequestIp().equals(optLogDTO.getRequestIp()) : optLogDTO.getRequestIp() != null)
        {
            return false;
        }
        if (getType() != null ? !getType().equals(optLogDTO.getType()) : optLogDTO.getType() != null)
        {
            return false;
        }
        if (getUserName() != null ? !getUserName().equals(optLogDTO.getUserName()) : optLogDTO.getUserName() != null)
        {
            return false;
        }
        if (getDescription() != null ? !getDescription().equals(optLogDTO.getDescription()) : optLogDTO.getDescription() != null)
        {
            return false;
        }
        if (getClassPath() != null ? !getClassPath().equals(optLogDTO.getClassPath()) : optLogDTO.getClassPath() != null)
        {
            return false;
        }
        if (getActionMethod() != null ? !getActionMethod().equals(optLogDTO.getActionMethod()) : optLogDTO.getActionMethod() != null)
        {
            return false;
        }
        if (getRequestUri() != null ? !getRequestUri().equals(optLogDTO.getRequestUri()) : optLogDTO.getRequestUri() != null)
        {
            return false;
        }
        if (getHttpMethod() != null ? !getHttpMethod().equals(optLogDTO.getHttpMethod()) : optLogDTO.getHttpMethod() != null)
        {
            return false;
        }
        if (getParams() != null ? !getParams().equals(optLogDTO.getParams()) : optLogDTO.getParams() != null)
        {
            return false;
        }
        if (getResult() != null ? !getResult().equals(optLogDTO.getResult()) : optLogDTO.getResult() != null)
        {
            return false;
        }
        if (getExDesc() != null ? !getExDesc().equals(optLogDTO.getExDesc()) : optLogDTO.getExDesc() != null)
        {
            return false;
        }
        if (getExDetail() != null ? !getExDetail().equals(optLogDTO.getExDetail()) : optLogDTO.getExDetail() != null)
        {
            return false;
        }
        if (getStartTime() != null ? !getStartTime().equals(optLogDTO.getStartTime()) : optLogDTO.getStartTime() != null)
        {
            return false;
        }
        if (getFinishTime() != null ? !getFinishTime().equals(optLogDTO.getFinishTime()) : optLogDTO.getFinishTime() != null)
        {
            return false;
        }
        if (getConsumingTime() != null ? !getConsumingTime().equals(optLogDTO.getConsumingTime()) : optLogDTO.getConsumingTime() != null)
        {
            return false;
        }
        if (getUa() != null ? !getUa().equals(optLogDTO.getUa()) : optLogDTO.getUa() != null)
        {
            return false;
        }
        return getCreateUser() != null ? getCreateUser().equals(optLogDTO.getCreateUser()) : optLogDTO.getCreateUser() == null;
    }

    @Override
    public int hashCode()
    {
        int result1 = getRequestIp() != null ? getRequestIp().hashCode() : 0;
        result1 = 31 * result1 + (getType() != null ? getType().hashCode() : 0);
        result1 = 31 * result1 + (getUserName() != null ? getUserName().hashCode() : 0);
        result1 = 31 * result1 + (getDescription() != null ? getDescription().hashCode() : 0);
        result1 = 31 * result1 + (getClassPath() != null ? getClassPath().hashCode() : 0);
        result1 = 31 * result1 + (getActionMethod() != null ? getActionMethod().hashCode() : 0);
        result1 = 31 * result1 + (getRequestUri() != null ? getRequestUri().hashCode() : 0);
        result1 = 31 * result1 + (getHttpMethod() != null ? getHttpMethod().hashCode() : 0);
        result1 = 31 * result1 + (getParams() != null ? getParams().hashCode() : 0);
        result1 = 31 * result1 + (getResult() != null ? getResult().hashCode() : 0);
        result1 = 31 * result1 + (getExDesc() != null ? getExDesc().hashCode() : 0);
        result1 = 31 * result1 + (getExDetail() != null ? getExDetail().hashCode() : 0);
        result1 = 31 * result1 + (getStartTime() != null ? getStartTime().hashCode() : 0);
        result1 = 31 * result1 + (getFinishTime() != null ? getFinishTime().hashCode() : 0);
        result1 = 31 * result1 + (getConsumingTime() != null ? getConsumingTime().hashCode() : 0);
        result1 = 31 * result1 + (getUa() != null ? getUa().hashCode() : 0);
        result1 = 31 * result1 + (getCreateUser() != null ? getCreateUser().hashCode() : 0);
        return result1;
    }

    @Override
    public String toString()
    {
        final StringBuilder sb = new StringBuilder("OptLogDTO{");
        sb.append("requestIp='").append(requestIp).append('\'');
        sb.append(", type='").append(type).append('\'');
        sb.append(", userName='").append(userName).append('\'');
        sb.append(", description='").append(description).append('\'');
        sb.append(", classPath='").append(classPath).append('\'');
        sb.append(", actionMethod='").append(actionMethod).append('\'');
        sb.append(", requestUri='").append(requestUri).append('\'');
        sb.append(", httpMethod='").append(httpMethod).append('\'');
        sb.append(", params='").append(params).append('\'');
        sb.append(", result='").append(result).append('\'');
        sb.append(", exDesc='").append(exDesc).append('\'');
        sb.append(", exDetail='").append(exDetail).append('\'');
        sb.append(", startTime=").append(startTime);
        sb.append(", finishTime=").append(finishTime);
        sb.append(", consumingTime=").append(consumingTime);
        sb.append(", ua='").append(ua).append('\'');
        sb.append(", createUser=").append(createUser);
        sb.append('}');
        return sb.toString();
    }
}
```







第十四步：编写实体类R\<T>



```java
package mao.tools_log.entity;

import java.util.Map;

import com.alibaba.fastjson.JSONObject;
import com.google.common.collect.Maps;


import io.swagger.annotations.ApiModelProperty;
import mao.tools_log.exception.BizException;
import mao.tools_log.exception.code.BaseExceptionCode;


@SuppressWarnings({"AlibabaClassNamingShouldBeCamel"})
public class R<T>
{
    public static final String DEF_ERROR_MESSAGE = "系统繁忙，请稍候再试";
    public static final String HYSTRIX_ERROR_MESSAGE = "请求超时，请稍候再试";
    public static final int SUCCESS_CODE = 0;
    public static final int FAIL_CODE = -1;
    public static final int TIMEOUT_CODE = -2;
    /**
     * 统一参数验证异常
     */
    public static final int VALID_EX_CODE = -9;
    public static final int OPERATION_EX_CODE = -10;
    /**
     * 调用是否成功标识，0：成功，-1:系统繁忙，此时请开发者稍候再试 详情见[ExceptionCode]
     */
    @ApiModelProperty(value = "响应编码:0/200-请求处理成功")
    private int code;

    /**
     * 调用结果
     */
    @ApiModelProperty(value = "响应数据")
    private T data;

    /**
     * 结果消息，如果调用成功，消息通常为空T
     */
    @ApiModelProperty(value = "提示消息")
    private String msg = "ok";

    @ApiModelProperty(value = "请求路径")
    private String path;
    /**
     * 附加数据
     */
    @ApiModelProperty(value = "附加数据")
    private Map<String, Object> extra;

    /**
     * 响应时间
     */
    @ApiModelProperty(value = "响应时间戳")
    private long timestamp = System.currentTimeMillis();

    private R()
    {
        super();
    }

    public R(int code, T data, String msg)
    {
        this.code = code;
        this.data = data;
        this.msg = msg;
    }

    public static <E> R<E> result(int code, E data, String msg)
    {
        return new R<>(code, data, msg);
    }

    /**
     * 请求成功消息
     *
     * @param data 结果
     * @return RPC调用结果
     */
    public static <E> R<E> success(E data)
    {
        return new R<>(SUCCESS_CODE, data, "ok");
    }

    public static R<Boolean> success()
    {
        return new R<>(SUCCESS_CODE, true, "ok");
    }

    /**
     * 请求成功方法 ，data返回值，msg提示信息
     *
     * @param data 结果
     * @param msg  消息
     * @return RPC调用结果
     */
    public static <E> R<E> success(E data, String msg)
    {
        return new R<>(SUCCESS_CODE, data, msg);
    }

    /**
     * 请求失败消息
     *
     * @param msg 消息
     * @return RPC调用结果
     */
    public static <E> R<E> fail(int code, String msg)
    {
        return new R<>(code, null, (msg == null || msg.isEmpty()) ? DEF_ERROR_MESSAGE : msg);
    }

    public static <E> R<E> fail(String msg)
    {
        return fail(OPERATION_EX_CODE, msg);
    }

    public static <E> R<E> fail(String msg, Object... args)
    {
        String message = (msg == null || msg.isEmpty()) ? DEF_ERROR_MESSAGE : msg;
        return new R<>(OPERATION_EX_CODE, null, String.format(message, args));
    }

    public static <E> R<E> fail(BaseExceptionCode exceptionCode)
    {
        return validFail(exceptionCode);
    }

    public static <E> R<E> fail(BizException exception)
    {
        if (exception == null)
        {
            return fail(DEF_ERROR_MESSAGE);
        }
        return new R<>(exception.getCode(), null, exception.getMessage());
    }

    /**
     * 请求失败消息，根据异常类型，获取不同的提供消息
     *
     * @param throwable 异常
     * @return RPC调用结果
     */
    public static <E> R<E> fail(Throwable throwable)
    {
        return fail(FAIL_CODE, throwable != null ? throwable.getMessage() : DEF_ERROR_MESSAGE);
    }

    public static <E> R<E> validFail(String msg)
    {
        return new R<>(VALID_EX_CODE, null, (msg == null || msg.isEmpty()) ? DEF_ERROR_MESSAGE : msg);
    }

    public static <E> R<E> validFail(String msg, Object... args)
    {
        String message = (msg == null || msg.isEmpty()) ? DEF_ERROR_MESSAGE : msg;
        return new R<>(VALID_EX_CODE, null, String.format(message, args));
    }

    public static <E> R<E> validFail(BaseExceptionCode exceptionCode)
    {
        return new R<>(exceptionCode.getCode(), null,
                (exceptionCode.getMsg() == null || exceptionCode.getMsg().isEmpty()) ? DEF_ERROR_MESSAGE : exceptionCode.getMsg());
    }

    public static <E> R<E> timeout()
    {
        return fail(TIMEOUT_CODE, HYSTRIX_ERROR_MESSAGE);
    }


    public R<T> put(String key, Object value)
    {
        if (this.extra == null)
        {
            this.extra = Maps.newHashMap();
        }
        this.extra.put(key, value);
        return this;
    }

    /**
     * 逻辑处理是否成功
     *
     * @return 是否成功
     */
    public Boolean getIsSuccess()
    {
        return this.code == SUCCESS_CODE || this.code == 200;
    }

    /**
     * 逻辑处理是否失败
     *
     * @return 是否失败
     */
    public Boolean getIsError()
    {
        return !getIsSuccess();
    }

    @Override
    public String toString()
    {
        return JSONObject.toJSONString(this);
    }

    //------------------------------------------------------

    public int getCode()
    {
        return code;
    }

    public void setCode(int code)
    {
        this.code = code;
    }

    public T getData()
    {
        return data;
    }

    public void setData(T data)
    {
        this.data = data;
    }

    public String getMsg()
    {
        return msg;
    }

    public void setMsg(String msg)
    {
        this.msg = msg;
    }

    public String getPath()
    {
        return path;
    }

    public void setPath(String path)
    {
        this.path = path;
    }

    public Map<String, Object> getExtra()
    {
        return extra;
    }

    public void setExtra(Map<String, Object> extra)
    {
        this.extra = extra;
    }

    public long getTimestamp()
    {
        return timestamp;
    }

    public void setTimestamp(long timestamp)
    {
        this.timestamp = timestamp;
    }
}
```





第十五步：编写类SysLogEvent



