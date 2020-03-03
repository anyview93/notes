<center><h1>SpringContextUtils</h1></center>

#### 手动获取Spring Bean的工具类，一般使用在工具类中（**@Autowared不生效的情况下**）

```java
package com.thunisoft.sfss.sfss.utils;

import java.util.Map;

import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.lang.Nullable;
import org.springframework.stereotype.Component;

/**
 * SpringContextUtils.java
 *
 * @author shizhiguo
 * @date 2018年11月27日
 */
@Component
public final class SpringContextUtils implements ApplicationContextAware {

    /**
     * 应用程序上下文
     */
    private ApplicationContext applicationContext;

    /**
     * 设置应用程序上下文
     *
     * @param applicationContext
     * @throws BeansException
     */
    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringContextUtilsHolder.INSTANCE.applicationContext = applicationContext;

    }

    /**
     * 获取applicationContext对象
     *
     * @return
     */
    public ApplicationContext getApplicationContext() {
        return this.applicationContext;
    }

    /**
     * 根据bean的id来查找对象
     *
     * @param id
     * @return
     */
    public Object getBeanById(String id) {
        return this.applicationContext.getBean(id);
    }

    /**
     * 根据bean的class来查找对象
     *
     * @param c
     * @return
     */
    public <T> T getBeanByClass(@Nullable Class<T> c) {
        return this.applicationContext.getBean(c);
    }

    /**
     * 根据bean的class来查找所有的对象(包括子类)
     *
     * @param c
     * @return
     */
    public <T> Map<String, T> getBeansByClass(@Nullable Class<T> c) {
        return this.applicationContext.getBeansOfType(c);
    }

    /**
     * 构造方法私有化
     *
     * @param applicationContext
     */
    private SpringContextUtils() {
        super();
    }

    /**
     * 获取SpringContentUtils实例
     *
     * @return SpringContextUtils
     */
    public static final SpringContextUtils getInstance() {
        return SpringContextUtilsHolder.INSTANCE;
    }

    /**
     * SpringContextUtils.java
     * <p>
     * 登记式/静态内部类实现单例
     * 这种方式能达到双检锁方式一样的功效，但实现更简单
     *
     * @author shizhiguo
     * @date 2018年12月3日
     */
    private static class SpringContextUtilsHolder {
        private static final SpringContextUtils INSTANCE = new SpringContextUtils();
    }
}

```