#	为SprintBoot 添加拦截器

##	声明拦截器

​	集成`HandlerInterceptorAdapter`类实现拦截器功能,有3个主要方法可以重写`preHandle`,`postHandle`,`afterComplete`,三个方法分别是请求过程中的声明周期.

​	主要重写`preHandle`方法,存在`boolean`类型的返回值,方法返回`false`时则拦截请求,再通过对**HttpServletResponse**进行操作返回需要返回的值

```java
@Component
public class AuthorizationInterceptor extends HandlerInterceptorAdapter {

    @Autowired
    private TokenManager manager;

    /**
     * @return 若要主动拦截,则需要返回false
     */
    public boolean preHandle(HttpServletRequest request,
                             HttpServletResponse response, Object handler) throws Exception {
        //从header中得到token
        String authorization = request.getHeader(Constants.AUTHORIZATION);
        System.out.println("authorization  >>> " + authorization);
        //验证token
        TokenModel model = manager.getToken(authorization);
        if (manager.checkToken(model)) {
            //如果token验证成功，将token对应的用户id存在request中，便于之后注入
            request.setAttribute(Constants.CURRENT_USER_ID, model.getUserId());
            return true;
        } else {
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
            return false;
        }
    }
}
```

---

##	注册拦截器

​	声明一个类集成`WebMvcConfigurerAdapter`,添加`Configuration`告诉Spring这是一个配置类	

​	重写`addInterceptors`来注册自定义的拦截器`addPathPatterns`方法传入需要进行拦截处理的URL路径, `/**`代表该路径下的所有接口都要经过拦截器处理

```
@Configuration
public class WebConfiguration extends WebMvcConfigurerAdapter {

    @Autowired
    private AuthorizationInterceptor authorizationInterceptor;

    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        InterceptorRegistration interceptor = registry.addInterceptor(authorizationInterceptor);
        //只拦截/auth 之外的路径
        interceptor.addPathPatterns("/v1/story/**")
                .addPathPatterns("/v1/user/**")
                .addPathPatterns("/v1/car/**")
                .addPathPatterns("/v1/carevent/**")
                .addPathPatterns("/v1/comment/**")
                .addPathPatterns("/v1/commemoration/**")
                .addPathPatterns("/v1/couple/**")
                .addPathPatterns("/v1/loved/**")
                .addPathPatterns("/v1/video/**")
                .addPathPatterns("/v1/redpacket/**")
                .addPathPatterns("/v1/auth/logout")// 注销也需要先登录
                .addPathPatterns("/v1/image/**");
//        interceptor.excludePathPatterns("/v1/auth/**");
    }
}
```