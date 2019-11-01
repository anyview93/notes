# 1. 内置Servlet容器配置

​		修改和server有关配置

~~~yaml
# 通用的在servlet.xxx配置
server:
  port: 8080
  servlet:
    context-path: /boot
    # 其他在对应的容器中配置
	tomcat:
   	  uri-encoding: utf-8
~~~

# 2. 注册Servlet，Filter，Listener

## 2.1 ServletRegistrationBean注册自定义Servlet

~~~java
	@Bean
    public ServletRegistrationBean servletRegistrationBean(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean();
        servletRegistrationBean.setServlet(new MyServlet());
        servletRegistrationBean.setUrlMappings(Arrays.asList("/myServlet"));
        return servletRegistrationBean;
    }
~~~

~~~java
@Slf4j
public class MyServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        resp.getWriter().println("this is myServlet");
    }
}
~~~

## 2.2 FilterRegistrationBean注册自定义Filter

~~~java
	@Bean
    public FilterRegistrationBean filterRegistrationBean(){
        FilterRegistrationBean<Filter> filterFilterRegistrationBean = new FilterRegistrationBean<>();
        filterFilterRegistrationBean.setFilter(new MyFilter());
        filterFilterRegistrationBean.setUrlPatterns(Arrays.asList("/myServlet"));
        return filterFilterRegistrationBean;
    }
~~~

~~~java
@Slf4j
public class MyFilter implements Filter {
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }

    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        log.info("myFilter is process......");
        filterChain.doFilter(servletRequest,servletResponse);
    }

    @Override
    public void destroy() {

    }
}
~~~

## 2.3 ServletListenerRegistrationBean注册自定义监听器

~~~java
	@Bean
    public ServletListenerRegistrationBean listenerRegistrationBean(){
        ServletListenerRegistrationBean listenerRegistrationBean = new ServletListenerRegistrationBean();
        listenerRegistrationBean.setListener(new MySessionListener());
        return listenerRegistrationBean;
    }
~~~

~~~java
@Slf4j
public class MySessionListener implements ServletContextListener {
    @Override
    public void contextInitialized(ServletContextEvent sce) {
        log.info("webapp start......");
    }

    @Override
    public void contextDestroyed(ServletContextEvent sce) {
        log.info("webapp distroyed......");
    }
}
~~~

# 3. 使用其他Servlet容器

## 3.1 pom文件修改

​		排除掉spring boot默认的tomcat容器，引入jetty的jar包

​		**tomcat**：适用短链接场景（网页）

​		**jetty**：适用于长连接场景（聊天）

~~~xml
		<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <exclusion>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                    <groupId>org.springframework.boot</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jetty</artifactId>
        </dependency>
~~~

## 3.2 自动配置嵌入式Servlet

​	**EmbeddedWebServerFactoryCustomizerAutoConfiguration类**

~~~java
@Configuration
@ConditionalOnWebApplication
@EnableConfigurationProperties({ServerProperties.class})
public class EmbeddedWebServerFactoryCustomizerAutoConfiguration {
    public EmbeddedWebServerFactoryCustomizerAutoConfiguration() {
    }

    @Configuration
    @ConditionalOnClass({HttpServer.class})
    public static class NettyWebServerFactoryCustomizerConfiguration {
        public NettyWebServerFactoryCustomizerConfiguration() {
        }

        @Bean
        public NettyWebServerFactoryCustomizer nettyWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
            return new NettyWebServerFactoryCustomizer(environment, serverProperties);
        }
    }

    @Configuration
    @ConditionalOnClass({Undertow.class, SslClientAuthMode.class})
    public static class UndertowWebServerFactoryCustomizerConfiguration {
        public UndertowWebServerFactoryCustomizerConfiguration() {
        }

        @Bean
        public UndertowWebServerFactoryCustomizer undertowWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
            return new UndertowWebServerFactoryCustomizer(environment, serverProperties);
        }
    }

    @Configuration
    @ConditionalOnClass({Server.class, Loader.class, WebAppContext.class})
    public static class JettyWebServerFactoryCustomizerConfiguration {
        public JettyWebServerFactoryCustomizerConfiguration() {
        }

        @Bean
        public JettyWebServerFactoryCustomizer jettyWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
            return new JettyWebServerFactoryCustomizer(environment, serverProperties);
        }
    }

    @Configuration
    @ConditionalOnClass({Tomcat.class, UpgradeProtocol.class})
    public static class TomcatWebServerFactoryCustomizerConfiguration {
        public TomcatWebServerFactoryCustomizerConfiguration() {
        }

        @Bean
        public TomcatWebServerFactoryCustomizer tomcatWebServerFactoryCustomizer(Environment environment, ServerProperties serverProperties) {
            return new TomcatWebServerFactoryCustomizer(environment, serverProperties);
        }
    }
}
~~~

