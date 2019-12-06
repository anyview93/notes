## <center>静态字段值从取配置文件获取</center>

## 1. 工具类代码

~~~java
package com.example.ehcache.controller;

import java.io.IOException;
import java.io.InputStream;
import java.util.Properties;

import org.springframework.core.io.ClassPathResource;

/**
 * 静态文件工具类，采用单例模式将配置文件中的值注入字段中，字段采用volatile修饰，修改完成后立刻被其他线程感知到
 * 因为是单例模式，实际上该值只会被初始化一次，达到和类字段同样的效果
 * @author shizhiguo
 * @version 1.0.
 * @Description
 * @Date 2019/12/6
 */
public final class SingletonProperties {

    private static SingletonProperties instance;

    private volatile String ip;

    public SingletonProperties(String ip) {
        this.ip = ip;
    }

    /**
     *  使用双重校验锁的单例模式获取该实例
     * @return
     * @throws IOException
     */
    public static SingletonProperties getInstance() throws IOException {
        if(instance == null){
            synchronized (SingletonProperties.class){
                if(instance == null) {
                    synchronized (SingletonProperties.class) {
                        //读取classpath路径下的文件
                        ClassPathResource classPathResource = new ClassPathResource("props.properties");
                        InputStream in = classPathResource.getInputStream();
                        Properties properties = new Properties();
                        properties.load(in);
                        instance = new SingletonProperties(properties.getProperty("ip"));
                    }
                }
            }
        }
        return instance;
    }

    public String getIp() {
        return ip;
    }

    @Override
    public String toString() {
        return "SingletonProperties{" +
                "ip='" + ip + '\'' +
                '}';
    }
}

~~~

## 2. 配置文件

### 2.1 配置文件位置

maven项目的resource目录下：resource/props.properties

### 2.2 配置文件内容

```properties
ip=192
```

## 3. 测试方法

```java
package com.example.ehcache.controller;

import java.io.IOException;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

/**
 * @author 
 * @version 1.0.
 * @Description
 * @Date 2019/12/6
 */
@RestController
public class SingletonController {

    @GetMapping("/getSingleton")
    public Object getSingleton() throws IOException {
        SingletonProperties instance = SingletonProperties.getInstance();
        return instance;
    }
}

```

打成jar包之后也可以正常访问