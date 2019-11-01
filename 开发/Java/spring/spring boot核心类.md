# spring boot

# 1 WebMvcConfigurer

​		spring mvc配置器，实现该接口可以注册资源映射器、拦截器、视图控制器、视图解析器、（请求和返回）参数处理器等等

~~~java
@Configuration
public class MvcConfig implements WebMvcConfigurer {

    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        if(!registry.hasMappingForPattern("/static/**")){
            registry.addResourceHandler("/static/**").addResourceLocations("classpath:/static/");
        }
    }

    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/login.html").setViewName("toLogin");
    }

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        String[] excludePathPatterns = {"/index.html", "/toLogin", "/index", "/login", "/css/**", "/js/**", "/img/**", "/err", "/error"};
        registry.addInterceptor(new LoginInterceptor()).addPathPatterns("/**")
                .excludePathPatterns(excludePathPatterns);
    }
}
~~~

## 1.1  HandlerInterceptor

​		spring mvc拦截器，对访问系统的请求进行拦

~~~java
public class LoginInterceptor implements HandlerInterceptor {

    private final Logger LOG = LoggerFactory.getLogger(LoginInterceptor.class);
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        Object user = request.getSession().getAttribute("user");
        if (null == user){
            LOG.info("redirect to login.html");
            request.getRequestDispatcher("/toLogin").forward(request,response);
            return false;
        }
        return true;
    }
}
~~~

## 1.2 ThymeleafViewResolver

​		Thymeleaf视图处理处理器，定义视图返回时如何解析（redirect和forward）

~~~java
@Override
    protected View createView(final String viewName, final Locale locale) throws Exception {
        // First possible call to check "viewNames": before processing redirects and forwards
        if (!this.alwaysProcessRedirectAndForward && !canHandle(viewName, locale)) {
            vrlogger.trace("[THYMELEAF] View \"{}\" cannot be handled by ThymeleafViewResolver. Passing on to the next resolver in the chain.", viewName);
            return null;
        }
        // Process redirects (HTTP redirects)
        if (viewName.startsWith(REDIRECT_URL_PREFIX)) {
            vrlogger.trace("[THYMELEAF] View \"{}\" is a redirect, and will not be handled directly by ThymeleafViewResolver.", viewName);
            final String redirectUrl = viewName.substring(REDIRECT_URL_PREFIX.length(), viewName.length());
            final RedirectView view = new RedirectView(redirectUrl, isRedirectContextRelative(), isRedirectHttp10Compatible());
            return (View) getApplicationContext().getAutowireCapableBeanFactory().initializeBean(view, viewName);
        }
        // Process forwards (to JSP resources)
        if (viewName.startsWith(FORWARD_URL_PREFIX)) {
            // The "forward:" prefix will actually create a Servlet/JSP view, and that's precisely its aim per the Spring
            // documentation. See http://docs.spring.io/spring-framework/docs/4.2.4.RELEASE/spring-framework-reference/html/mvc.html#mvc-redirecting-forward-prefix
            vrlogger.trace("[THYMELEAF] View \"{}\" is a forward, and will not be handled directly by ThymeleafViewResolver.", viewName);
            final String forwardUrl = viewName.substring(FORWARD_URL_PREFIX.length(), viewName.length());
            return new InternalResourceView(forwardUrl);
        }
        // Second possible call to check "viewNames": after processing redirects and forwards
        if (this.alwaysProcessRedirectAndForward && !canHandle(viewName, locale)) {
            vrlogger.trace("[THYMELEAF] View \"{}\" cannot be handled by ThymeleafViewResolver. Passing on to the next resolver in the chain.", viewName);
            return null;
        }
        vrlogger.trace("[THYMELEAF] View {} will be handled by ThymeleafViewResolver and a " +
                        "{} instance will be created for it", viewName, getViewClass().getSimpleName());
        return loadView(viewName, locale);
    }
~~~

## 1.3 BasicErrorController

​		默认错误处理逻辑，根据请求类型返回页面或者json数据

~~~java
@RequestMapping(
        produces = {"text/html"}
    )
    public ModelAndView errorHtml(HttpServletRequest request, HttpServletResponse response) {
        HttpStatus status = this.getStatus(request);
        Map<String, Object> model = Collections.unmodifiableMap(this.getErrorAttributes(request, this.isIncludeStackTrace(request, MediaType.TEXT_HTML)));
        response.setStatus(status.value());
        ModelAndView modelAndView = this.resolveErrorView(request, response, status, model);
        return modelAndView != null ? modelAndView : new ModelAndView("error", model);
    }



    @RequestMapping
    public ResponseEntity<Map<String, Object>> error(HttpServletRequest request) {
        Map<String, Object> body = this.getErrorAttributes(request, this.isIncludeStackTrace(request, MediaType.ALL));
        HttpStatus status = this.getStatus(request);
        return new ResponseEntity(body, status);
    }
~~~

### 1.3.1自定义错误信息

添加自定义错误信息，重点在**注解**和**状态码**

~~~java
@ControllerAdvice
public class MyExceptionHandler {

    @ExceptionHandler(Exception.class)
    public String handleException(Exception e, HttpServletRequest request){
        String message = e.getMessage();
        request.setAttribute("javax.servlet.error.status_code",400);
        HashMap<String, Object> map = new HashMap<>();
        map.put("code", 500);
        map.put("message",message);
        request.setAttribute("ext",map);
        return "forward:/error";
    }
}
~~~

获取添加的错误信息返回给页面

~~~java
@Component
public class MyErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(WebRequest webRequest, boolean includeStackTrace) {
        Map<String, Object> errorAttributes = super.getErrorAttributes(webRequest, includeStackTrace);
        Object ext = webRequest.getAttribute("ext", RequestAttributes.SCOPE_REQUEST);
        errorAttributes.put("ext", ext);
        return errorAttributes;
    }
}
~~~

