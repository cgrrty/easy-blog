### Spring Security核心概念

  * SecurityContextHolder

    用于存储当前应用程序的当前安全环境的细节。默认情况下，SecurityContextHolder使用ThreadLocal存储这些信息，这样，安全环境在同一线程执行的方法是一直有效的。当然，我们也可以在启动时指定上下文保存的方式，比如对于一个单独的应用系统，我们可以使用SecurityContextHolder.MODEL_GLOBAL策略。我可以通过设置系统属性或者调用SecurityContextHolder的静态方法的方式进行设置。

    安全主体和系统交互的信息都保存在SecurityContextHolder中，Spring Security使用一个Authentication对应来表现这些信息。通过Authentication对象获取当前用户信息：

        Object principal = SecurityContextHolder.getContext().getAuthentication().getPrincipal();
        if (principal instanceof UserDetails) {
            String username = ((UserDetails)principal).getUsername();
        } else {
            String username = principal.toString();
        }

可以看到，getContext()返回一个SecurityContext接口的实例，getAuthentication可以从SecurityContext获取一个Authentication对象，从Authentication对象中，可以获得我们的安全主体的对象，该对象大多数情况下，可以强制转换成UserDetails对象。在大多数Spring Security的验证机制中，都返回一个UserDetails的实例为主体。

  * UserDetailsService

    通过UserDetailsService对象中的loadUserByUsername方法，我们可以获取用户信息，这也是我们在框架中常用的方法。

        UserDetails loadUserByUsername(String username) throws UsernameNotFoundException;

    当成功通过验证时，UserDetails会被用来建立Authentication对象，保存到SecurityContext中。无论我们怎么实现UserDetailsService对象，UserDetails都可以通过SecurityContextHolder获得。

  * GrantedAuthority

    Authentication提供的重要方法getAuthorities()可以帮助我们获得GrantedAuthority对象数组。其中，GrantedAuthority是赋予主体的权限。这些权限通常使用角色表示，比如ROLE_ADMINISTRATOR或ROLE_HR_SUPERVISOR。在验证中，这些角色都会被配置。

  * HttpSessionContextIntegrationFilter
    
    为了在不同请求使用,把 SecurityContext保存到HttpSession里

  * 验证流程

    1. 获得用户名密码并在UsernamePasswordAuthenticationToken实例中进行比对。

    2. 这个实例被传递给AuthenticationManager实例，并在校验通过后返回Authentication实例。

    3. 安全环境被建立，通过调用SecurityContextHolder.getContext().setAuthentication(...)，传递到返回的验证对象中。

    这是Spring Security内部的验证流程，我们使用的时候只需要配置上述对象即可。

  * 一个经典的web应用的验证过程
  
    1. 你访问主页，点击链接。

    2. 一个请求发送给服务器，服务器决定你是否在请求一个被保护资源。
  
    3. 如果你还没有授权，服务器发回一个响应，提示你必须登录。响应会是一个HTTP响应代码或者重定向到特定的web页面。

    4. 基于验证机制，你的浏览器会重定向到特殊的web页面，所以你可以填写表单，或者浏览器会验证你的身份。
    
    5. 浏览器会发送回一个响应到服务器。这会是一个HTTP POST包含你填写的表单中的内容，或者一个HTTP头部包含你的验证细节。
    
    6. 服务器决定当前证书是否有效。如果他们有效，下一步会执行。如果它们无效，通常你的浏览器会被询问再试一次。

    7. 你的原始请求会引发验证过程。希望你验证你是否以权限来访问被保护资源。如果你允许访问，则请求成功。否则，你会收到一个HTTP错误代码403，意思是“拒绝访问”。
  
    8. 在Spring Security中，对应上述步骤的主要部分是：ExceptionTranslationFilter，一个AuthenticationEntryPoint和一个“验证机制”。

  * ExceptionTranslationFilter

    ExceptionTranslationFilter 是一个Spring Security过滤器负责检测任何一个 Spring Security 抛出的异常。 这些异常会被AbstractSecurityInterceptor抛出, 这是一个验证服务的主要提供器。它负责返回错误代码403（通过授权但权限不足），或者启动一个AuthenticationEntryPoint。

  * AuthenticationEntryPoint
   
    AuthenticationEntryPoint将用户定向到认证的入口，收集认证信息。认证入口一般是个页面，需要用户输入用户名和密码，也有其他方式的入口。收集到认证信息之后，重新提交认证请求。认证信息会再次通过过滤器，由AuthenticationManager认证。AuthenticationEntryPoint是用户提供凭证的入口，真正的认证是由过滤器来完成。

  * 验证机制

    In Spring Security we have a special name for the function of collecting authentication details from a user agent (usually a web browser), referring to it as the “authentication mechanism”.

    在验证机制获得完全的 Authentication 后,它会认为请求合法, 把 Authentication 放 到 SecurityContextHolder 里, 然后AuthenticationManager会进一步进行验证过程。

  * 在两次请求之间，我们要公用SecurityContext，那么需要将SecurityContext保存到session中，这个工作我们通过SecurityContextPersistenceFilter来实现。但是，因为安全原因，我们不应该直接操作HttpSession中的SecurityContext对象，而应该使用SecurityContextHolder来调用。

  * Spring Security中的访问控制

    负责访问控制的主要接口是AccessDecisionMananger，它由一个decide方法，可以获得一个Authentication对象。它决定了主要的访问控制，为对象提供了一个安全对象和一组安全数据属性。

  * Spring Security使用spring的标准aop为方法调用提供了一个环绕advice，使用servlet的标准filter建立了对web请求的环绕advice。

  * 安全对象和AbstractSecurityInterceptor
    
    安全对象在Spring Security中表示任何应用安全机制的对象。最常见的是web调用和方法请求。

    每一个支持的安全对象类型都有一个自己的父类为AbstractSecurityInterceptor的拦截器类型。如果主体是已经通过了验证,在 AbstractSecurityInterceptor 被调用的时候,SecurityContextHolder 将会包􏰀一个有 效的 Authentication。

    AbstractSecurityInterceptor 提供了一套一致的工作流程,来处理对安全对象的请求, 通常是:
    
    1. Look up the “configuration attributes” associated with the present request
    
    2. Submitting the secure object, current Authentication and configuration attributes to the AccessDecisionManager for an authorization decision
    
    3. Optionally change the Authentication under which the invocation takes place
    
    4. Allow the secure object invocation to proceed (assuming access was granted)
    
    5. Call the AfterInvocationManager if configured, once the invocation has returned. If the invocation raised an exception, the AfterInvocationManager will not be invoked.

  * 配置属性

    配置属性是一个被AbstractSecurityInterceptor使用的具有特殊意义的字符串。它们通过框架中的ConfigAttribute表现。

  * AccessDecisionManager
    
    提供访问的decision,适用于 web 以及方法的安全。一个默认的主体会被注册,但是你也可以选择自定义一个,使用正常的 spring bean 语法进行声明。


  * RunAsManager

    通过RunAsManager，用户可能把SecurityContext的Authentication换成另一个Authentication

  * AfterInvocationManager

    与修改请求返回对象相关

    

    
