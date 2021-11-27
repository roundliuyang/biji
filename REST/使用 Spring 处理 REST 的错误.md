# 使用 Spring 处理 REST 的错误

## **1. 概述**



本教程将说明**如何使用 Spring 为 REST API 实现异常处理。**我们还将获得一些历史概述，并了解不同版本引入了哪些新选项。

**在 Spring 3.2 之前，在 Spring MVC 应用程序中处理异常的两种主要方法是HandlerExceptionResolver或@ExceptionHandler注释。**两者都有一些明显的缺点。

**从 3.2 开始，我们使用@ControllerAdvice注释**来解决前两个解决方案的局限性，并在整个应用程序中促进统一的异常处理。

现在**Spring 5 引入了 ResponseStatusException 类**——一种在我们的 REST API 中进行基本错误处理的快速方法。

所有这些确实有一个共同点：它们很好地处理了**关注点分离**。应用程序可以正常抛出异常以指示某种失败，然后将单独处理。



最后，我们将看到 Spring Boot 带来了什么，以及我们如何配置它以满足我们的需求。

## **2. 方案一：控制器级@ExceptionHandler**



第一个解决方案适用于*@Controller*级别。我们将定义一个方法来处理异常并使用*@ExceptionHandler 对其进行*注释：

```java
public class FooController{
    
    //...
    @ExceptionHandler({ CustomException1.class, CustomException2.class })
    public void handleException() {
        //
    }
}
```

这种方法有一个很大的缺点：T**他@ExceptionHandler注解的方法才会被激活针对特定的控制器**，而不是全局为整个应用程序。当然，将它添加到每个控制器使其不太适合通用异常处理机制。

我们可以通过让**所有控制器扩展一个基本控制器类**来解决这个限制**。**

但是，对于无论出于何种原因都无法实现的应用程序，此解决方案可能是一个问题。例如，控制器可能已经从另一个基类扩展而来，该基类可能在另一个 jar 中或不可直接修改，或者它们本身可能不可直接修改。



接下来，我们将研究解决异常处理问题的另一种方法——一种全局性的方法，不包括对现有工件（如控制器）的任何更改。

## **3. 方案二：HandlerExceptionResolver**



第二种解决方案是定义一个*HandlerExceptionResolver。*这将解决应用程序抛出的任何异常。它还允许我们在 REST API 中实现**统一的异常处理机制**。

在使用自定义解析器之前，让我们回顾一下现有的实现。

### **3.1. ExceptionHandlerExceptionResolver**



这个解析器是在 Spring 3.1 中引入的，默认情况下在*DispatcherServlet 中*启用。这实际上是前面介绍的*@ExceptionHandler*机制如何工作的核心组件。

### **3.2. DefaultHandlerExceptionResolver**



这个解析器是在 Spring 3.0 中引入的，它在*DispatcherServlet 中*默认启用。



它用于将标准 Spring 异常解析为相应的[HTTP 状态代码](https://www.baeldung.com/cs/http-status-codes)，即客户端错误*4xx*和服务器错误*5xx*状态代码。[这是](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-ann-rest-spring-mvc-exceptions)它处理的 Spring 异常[的](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-ann-rest-spring-mvc-exceptions)[**完整列表**](http://static.springsource.org/spring/docs/3.2.x/spring-framework-reference/html/mvc.html#mvc-ann-rest-spring-mvc-exceptions)以及它们如何映射到状态代码。

虽然它确实正确设置了响应的状态代码，但一个**限制是它没有为响应的正文设置任何内容。**对于 REST API——状态码真的不足以提供给客户端的信息——响应也必须有一个主体，以允许应用程序提供有关失败的附加信息。

这可以通过通过*ModelAndView*配置视图分辨率和渲染错误内容来解决，但解决方案显然不是最优的。这就是为什么 Spring 3.2 引入了一个更好的选项，我们将在后面的部分中讨论。

### **3.3. ResponseStatusExceptionResolver**



这个解析器也在 Spring 3.0 中引入，默认情况下在*DispatcherServlet 中*启用。

它的主要职责是使用可用于自定义异常的*@ResponseStatus*注释并将这些异常映射到 HTTP 状态代码。

这样的自定义异常可能如下所示：

```java
@ResponseStatus(value = HttpStatus.NOT_FOUND)
public class MyResourceNotFoundException extends RuntimeException {
    public MyResourceNotFoundException() {
        super();
    }
    public MyResourceNotFoundException(String message, Throwable cause) {
        super(message, cause);
    }
    public MyResourceNotFoundException(String message) {
        super(message);
    }
    public MyResourceNotFoundException(Throwable cause) {
        super(cause);
    }
}
```

与*DefaultHandlerExceptionResolver*相同，这个解析器在处理响应主体的方式上受到限制——它确实将状态代码映射到响应上，但主体仍然为*空。*

### **3.4. SimpleMappingExceptionResolver和AnnotationMethodHandlerExceptionResolver**



该*的SimpleMappingExceptionResolver*已经有相当长的一段时间。它来自较旧的 Spring MVC 模型，与**REST 服务不太相关。**我们基本上使用它来将异常类名映射到视图名称。



该*AnnotationMethodHandlerExceptionResolver*在春3.0中引入处理过的异常*@ExceptionHandler*注释但已被弃用*ExceptionHandlerExceptionResolver*如春3.2。

### **3.5. 自定义HandlerExceptionResolver**



*DefaultHandlerExceptionResolver*和*ResponseStatusExceptionResolver*的组合对于为 Spring RESTful 服务提供良好的错误处理机制大有帮助。缺点是，如前所述，**无法控制响应的主体。**

理想情况下，我们希望能够输出 JSON 或 XML，具体取决于客户端要求的格式（通过*Accept*标头）。

仅此一项就证明创建**一个新的自定义异常解析器**是合理的：

```java
@Component
public class RestResponseStatusExceptionResolver extends AbstractHandlerExceptionResolver {

    @Override
    protected ModelAndView doResolveException(
      HttpServletRequest request, 
      HttpServletResponse response, 
      Object handler, 
      Exception ex) {
        try {
            if (ex instanceof IllegalArgumentException) {
                return handleIllegalArgument(
                  (IllegalArgumentException) ex, response, handler);
            }
            ...
        } catch (Exception handlerException) {
            logger.warn("Handling of [" + ex.getClass().getName() + "] 
              resulted in Exception", handlerException);
        }
        return null;
    }

    private ModelAndView 
      handleIllegalArgument(IllegalArgumentException ex, HttpServletResponse response) 
      throws IOException {
        response.sendError(HttpServletResponse.SC_CONFLICT);
        String accept = request.getHeader(HttpHeaders.ACCEPT);
        ...
        return new ModelAndView();
    }
}
```

这里要注意的一个细节是我们可以访问*请求*本身，因此我们可以考虑客户端发送的*Accept*标头的值。

例如，如果客户端请求*application/json*，那么在出现错误情况的情况下，我们希望确保我们返回一个用*application/json*编码的响应正文。

另一个重要的实现细节是**我们返回一个 ModelAndView——这是响应的主体，**它将允许我们在其上设置任何必要的内容。

这种方法是用于 Spring REST 服务的错误处理的一致且易于配置的机制。



但是，它确实有局限性：它与低级*HtttpServletResponse 交互*并适合使用*ModelAndView*的旧 MVC 模型，因此仍有改进的空间。

## **4. 方案三：@ControllerAdvice**



Spring 3.2 支持**带有** **@ControllerAdvice****注释****的全局@ExceptionHandler。**

这启用了一种脱离旧 MVC 模型并利用*ResponseEntity*以及*@ExceptionHandler*的类型安全性和灵活性的*机制*：

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler 
  extends ResponseEntityExceptionHandler {

    @ExceptionHandler(value 
      = { IllegalArgumentException.class, IllegalStateException.class })
    protected ResponseEntity<Object> handleConflict(
      RuntimeException ex, WebRequest request) {
        String bodyOfResponse = "This should be application specific";
        return handleExceptionInternal(ex, bodyOfResponse, 
          new HttpHeaders(), HttpStatus.CONFLICT, request);
    }
}
```

该*@ControllerAdvice*注释使我们能够**巩固我们的多，散@ExceptionHandler期从之前到一个单一的，全球性的错误处理组件。**

实际的机制非常简单，但也非常灵活：

- 它使我们可以完全控制响应的主体以及状态代码。
- 它提供了多个异常到同一方法的映射，以便一起处理。
- 它很好地利用了较新的 RESTful *ResposeEntity*响应。

这里要记住的一件事是将**用****@ExceptionHandler****声明的异常与****用作方法参数的异常**进行**匹配。**

如果这些不匹配，编译器不会抱怨——它没有理由应该——并且 Spring 也不会抱怨。

但是，当在运行时实际抛出异常时，**异常解析机制将失败**：



```bash
java.lang.IllegalStateException: No suitable resolver for argument [0] [type=...]
HandlerMethod details: ...
```

## **5.解决方案4：ResponseStatusException（Spring 5及以上）**



Spring 5 引入了*ResponseStatusException*类。

我们可以创建它的一个实例，提供*HttpStatus*和可选的*原因*和*原因*：

```java
@GetMapping(value = "/{id}")
public Foo findById(@PathVariable("id") Long id, HttpServletResponse response) {
    try {
        Foo resourceById = RestPreconditions.checkFound(service.findOne(id));

        eventPublisher.publishEvent(new SingleResourceRetrievedEvent(this, response));
        return resourceById;
     }
    catch (MyResourceNotFoundException exc) {
         throw new ResponseStatusException(
           HttpStatus.NOT_FOUND, "Foo Not Found", exc);
    }
}
```

使用*ResponseStatusException 有*什么好处？

- 非常适合原型设计：我们可以非常快速地实现基本解决方案。
- 一种类型，多种状态代码：一种异常类型可以导致多种不同的响应。**与@ExceptionHandler相比，这减少了紧密耦合。**
- 我们不必创建那么多的自定义异常类。
- 我们可以**更好地控制异常处理，**因为可以通过编程方式创建异常。

那么权衡呢？

- 没有统一的异常处理方式：与提供全局方法的*@ControllerAdvice 相比*，强制执行一些应用程序范围的约定更加困难。
- 代码重复：我们可能会发现自己在多个控制器中复制代码。

我们还应该注意到，可以在一个应用程序中组合不同的方法。

**例如，我们可以全局实现@ControllerAdvice，但也可以 在本地实现 ResponseStatusException。**

但是，我们需要小心：如果可以通过多种方式处理同一个异常，我们可能会注意到一些令人惊讶的行为。一种可能的约定是始终以一种方式处理一种特定类型的异常。

有关更多详细信息和更多示例，请参阅我们[关于*ResponseStatusException*](https://www.baeldung.com/spring-response-status-exception)的[教程](https://www.baeldung.com/spring-response-status-exception)。



## **6、处理Spring Security中拒绝访问**



当经过身份验证的用户尝试访问他没有足够权限访问的资源时，会发生拒绝访问。

### **6.1. MVC - 自定义错误页面**



首先我们看一下解决方案的MVC风格，看看如何自定义拒绝访问的错误页面。

XML 配置：

```xml
<http>
    <intercept-url pattern="/admin/*" access="hasAnyRole('ROLE_ADMIN')"/>   
    ... 
    <access-denied-handler error-page="/my-error-page" />
</http>
```

和Java配置：

```java
@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/admin/*").hasAnyRole("ROLE_ADMIN")
        ...
        .and()
        .exceptionHandling().accessDeniedPage("/my-error-page");
}
```

当用户在没有足够权限的情况下尝试访问资源时，他们将被重定向到*“/my-error-page”*。

### **6.2. 自定义AccessDeniedHandler**



接下来，让我们看看如何编写我们的自定义*AccessDeniedHandler*：

```java
@Component
public class CustomAccessDeniedHandler implements AccessDeniedHandler {

    @Override
    public void handle
      (HttpServletRequest request, HttpServletResponse response, AccessDeniedException ex) 
      throws IOException, ServletException {
        response.sendRedirect("/my-error-page");
    }
}
```

现在让我们使用**XML 配置来**配置它：

```xml
<http>
    <intercept-url pattern="/admin/*" access="hasAnyRole('ROLE_ADMIN')"/> 
    ...
    <access-denied-handler ref="customAccessDeniedHandler" />
</http>
```

0r 使用 Java 配置：



```java
@Autowired
private CustomAccessDeniedHandler accessDeniedHandler;

@Override
protected void configure(HttpSecurity http) throws Exception {
    http.authorizeRequests()
        .antMatchers("/admin/*").hasAnyRole("ROLE_ADMIN")
        ...
        .and()
        .exceptionHandling().accessDeniedHandler(accessDeniedHandler)
}
```

请注意，在我们的*CustomAccessDeniedHandler 中*，我们可以通过重定向或显示自定义错误消息来根据需要自定义响应。

### **6.3. REST 和方法级安全性**



最后，让我们看看如何处理方法级安全性*@PreAuthorize*、*@PostAuthorize*和*@Secure* Access Denied。

当然，我们也将使用我们之前讨论过的全局异常处理机制来处理*AccessDeniedException*：

```java
@ControllerAdvice
public class RestResponseEntityExceptionHandler 
  extends ResponseEntityExceptionHandler {

    @ExceptionHandler({ AccessDeniedException.class })
    public ResponseEntity<Object> handleAccessDeniedException(
      Exception ex, WebRequest request) {
        return new ResponseEntity<Object>(
          "Access denied message here", new HttpHeaders(), HttpStatus.FORBIDDEN);
    }
    
    ...
}
```

## **7. Spring Boot 支持**



**Spring Boot 提供了一个 ErrorController实现来以合理的方式处理错误。**

简而言之，它为浏览器提供后备错误页面（又名白标错误页面），并为 RESTful 非 HTML 请求提供 JSON 响应：

```plaintext
{
    "timestamp": "2019-01-17T16:12:45.977+0000",
    "status": 500,
    "error": "Internal Server Error",
    "message": "Error processing the request!",
    "path": "/my-endpoint-with-exceptions"
}
```

像往常一样，Spring Boot 允许使用属性配置这些功能：

- *server.error.whitelabel.enabled*：可用于禁用 Whitelabel 错误页面并依赖 servlet 容器提供 HTML 错误消息
- *server.error.include-stacktrace* :*总是* 值；在 HTML 和 JSON 默认响应中包含堆栈跟踪
- *server.error.include-message：* 从2.3版本开始，Spring Boot在响应中隐藏了 *message*字段，避免泄露敏感信息；我们可以使用带有*always* 值的属性 来启用它

除了这些属性之外，**我们还可以为 /错误提供我们自己的视图解析器映射， 覆盖白标签页面。**

我们还可以通过在上下文中包含一个*ErrorAttributes*  bean 来自定义我们想要在响应中显示的属性 。我们可以扩展Spring Boot 提供的 *DefaultErrorAttributes*类，让事情变得更简单：



```java
@Component
public class MyCustomErrorAttributes extends DefaultErrorAttributes {

    @Override
    public Map<String, Object> getErrorAttributes(
      WebRequest webRequest, ErrorAttributeOptions options) {
        Map<String, Object> errorAttributes = 
          super.getErrorAttributes(webRequest, options);
        errorAttributes.put("locale", webRequest.getLocale()
            .toString());
        errorAttributes.remove("error");

        //...

        return errorAttributes;
    }
}
```

如果我们想进一步定义（或覆盖）应用程序将如何处理特定内容类型的错误，我们可以注册一个 *ErrorController*  bean。

同样，我们可以利用Spring Boot 提供的默认 *BasicErrorController* 来帮助我们。

例如，假设我们想要自定义我们的应用程序如何处理在 XML 端点中触发的错误。我们所要做的就是使用*@RequestMapping*定义一个公共方法 ，并声明它产生*application/xml*媒体类型：

```java
@Component
public class MyErrorController extends BasicErrorController {

    public MyErrorController(
      ErrorAttributes errorAttributes, ServerProperties serverProperties) {
        super(errorAttributes, serverProperties.getError());
    }

    @RequestMapping(produces = MediaType.APPLICATION_XML_VALUE)
    public ResponseEntity<Map<String, Object>> xmlError(HttpServletRequest request) {
        
    // ...

    }
}
```

注意：这里我们仍然依赖于*server.error.**可能已经在我们的项目中定义的引导属性，这些属性绑定到*ServerProperties*  bean。

## 8.统一异常实践

### 案例一

@RestControllerAdvice = @ControllerAdvice + @ResponseBody

```java
/**
 * 异常数据封装
 */
@ControllerAdvice
@Slf4j
public class AdminControllerAdvice {

    @ExceptionHandler(BaleFireBizException.class)
    @ResponseBody
    public BaseResult<String> BaleFireBizExceptionException(BaleFireBizException e) {
        log.info("自定义业务异常 ", e);
        return BaseResult.fail(e.getResultCode(), e.getResultMsg());
    }
    @ExceptionHandler(MethodArgumentNotValidException.class)
    @ResponseBody
    public BaseResult<String> httpMethodArgumentNotValidException(MethodArgumentNotValidException e) {
        log.info("请求参数错误 ", e);
        String resultMsg = "请求参数错误";
        try {
            resultMsg = e.getBindingResult().getFieldError().getDefaultMessage();
        } catch (Exception ex) {
            log.info("获取请求错误 {}", ex);
        }
        return BaseResult.fail(ResponseCodeEnum.REQUEST_PARAMS_ERROR.getCode(), resultMsg);
    }

    @ExceptionHandler(HttpMessageNotReadableException.class)
    @ResponseBody
    public BaseResult<String> httpMessageNotReadableException(HttpMessageNotReadableException e) {
        log.info("请求参数错误 ", e);
        return BaseResult.fail(ResponseCodeEnum.REQUEST_PARAMS_ERROR.getCode(), "请求参数错误", e.getLocalizedMessage());
    }

    @ExceptionHandler(HttpMediaTypeNotSupportedException.class)
    @ResponseBody
    public ResultBase<String> httpMediaTypeNotSupportedException(HttpMediaTypeNotSupportedException e) {
        log.info("content-type 设置错误 ", e);
        return ResultBase.fail(ResponseCodeEnum.REQUEST_PARAMS_ERROR.getCode(), "content-type 设置错误", e.getLocalizedMessage());
    }

    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    @ResponseBody
    public ResultBase<String> httpRequestMethodNotSupportedException(HttpRequestMethodNotSupportedException e) {
        log.info("不支持的方法", e);
        return ResultBase.fail(ResponseCodeEnum.REQUEST_PARAMS_ERROR.getCode(), "该方法不支持", e.getLocalizedMessage());
    }

    @ExceptionHandler(MissingServletRequestParameterException.class)
    @ResponseBody
    public ResultBase<String> missingServletRequestParameterException(MissingServletRequestParameterException e) {
        log.info("缺少必要的参数", e);
        return ResultBase.fail(ResponseCodeEnum.REQUEST_PARAMS_ERROR.getCode(), "缺少必要请求的参数", e.getLocalizedMessage());
    }

    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    @ResponseBody
    public ResultBase<String> methodArgumentTypeMismatchException(MethodArgumentTypeMismatchException e) {
        log.info("参数匹配错误", e);
        return ResultBase.fail(ResponseCodeEnum.REQUEST_PARAMS_ERROR.getCode(), "参数匹配错误", e.getLocalizedMessage());
    }
}
```



### 案例二

在spring 3.2中，新增了@ControllerAdvice 注解，可以用于定义@ExceptionHandler、@InitBinder、@ModelAttribute，并应用到所有@RequestMapping中。参考：@ControllerAdvice 文档

#### 一、介绍

创建 MyControllerAdvice，并添加 @ControllerAdvice注解。

```java
package com.sam.demo.controller;

import org.springframework.ui.Model;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.*;
import java.util.HashMap;
import java.util.Map;

/**
 * controller 增强器
 * @author sam
 * @since 2017/7/17
 */
@ControllerAdvice
public class MyControllerAdvice {

    /**
     * 应用到所有@RequestMapping注解方法，在其执行之前初始化数据绑定器
     * @param binder
     */
    @InitBinder
    public void initBinder(WebDataBinder binder) {}

    /**
     * 把值绑定到Model中，使全局@RequestMapping可以获取到该值
     * @param model
     */
    @ModelAttribute
    public void addAttributes(Model model) {
        model.addAttribute("author", "Magical Sam");
    }

    /**
     * 全局异常捕捉处理
     * @param ex
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public Map errorHandler(Exception ex) {
        Map map = new HashMap();
        map.put("code", 100);
        map.put("msg", ex.getMessage());
        return map;
    }

}
```

启动应用后，被 @ExceptionHandler、@InitBinder、@ModelAttribute 注解的方法，都会作用在 被 @RequestMapping 注解的方法上。
@ModelAttribute：在Model上设置的值，对于所有被 @RequestMapping 注解的方法中，都可以通过 ModelMap 获取，如下：

```java
@RequestMapping("/home")
public String home(ModelMap modelMap) {
    System.out.println(modelMap.get("author"));
}

//或者 通过@ModelAttribute获取

@RequestMapping("/home")
public String home(@ModelAttribute("author") String author) {
    System.out.println(author);
}
```

@ExceptionHandler 拦截了异常，我们可以通过该注解实现自定义异常处理。其中，@ExceptionHandler 配置的 value 指定需要拦截的异常类型，上面拦截了 Exception.class 这种异常。

#### 二、自定义异常处理（全局异常处理）

spring boot 默认情况下会映射到 /error 进行异常处理，但是提示并不十分友好，下面自定义异常处理，提供友好展示。

##### 1、编写自定义异常类：

```java
package com.sam.demo.custom;

/**
 * @author sam
 * @since 2017/7/17
 */
public class MyException extends RuntimeException {

    public MyException(String code, String msg) {
        this.code = code;
        this.msg = msg;
    }

    private String code;
    private String msg;

    // getter & setter
}

```

注：spring 对于 RuntimeException 异常才会进行事务回滚。

##### 2、编写全局异常处理类

创建 MyControllerAdvice.java，如下：

```java
package com.sam.demo.controller;

import org.springframework.ui.Model;
import org.springframework.web.bind.WebDataBinder;
import org.springframework.web.bind.annotation.*;

import java.util.HashMap;
import java.util.Map;

/**
 * controller 增强器
 *
 * @author sam
 * @since 2017/7/17
 */
@ControllerAdvice
public class MyControllerAdvice {

    /**
     * 全局异常捕捉处理
     * @param ex
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = Exception.class)
    public Map errorHandler(Exception ex) {
        Map map = new HashMap();
        map.put("code", 100);
        map.put("msg", ex.getMessage());
        return map;
    }
    
    /**
     * 拦截捕捉自定义异常 MyException.class
     * @param ex
     * @return
     */
    @ResponseBody
    @ExceptionHandler(value = MyException.class)
    public Map myErrorHandler(MyException ex) {
        Map map = new HashMap();
        map.put("code", ex.getCode());
        map.put("msg", ex.getMsg());
        return map;
    }

}

```

##### 3、controller中抛出异常进行测试。

```java
@RequestMapping("/home")
public String home() throws Exception {

//        throw new Exception("Sam 错误");
    throw new MyException("101", "Sam 错误");

}
```

> 启动应用，访问：<http://localhost:8080/home> ，正常显示以下json内容，证明自定义异常已经成功被拦截。
>
> {"msg":"Sam 错误","code":"101"}
>
> * 如果不需要返回json数据，而要渲染某个页面模板返回给浏览器，那么MyControllerAdvice中可以这么实现：

```java
@ExceptionHandler(value = MyException.class)
public ModelAndView myErrorHandler(MyException ex) {
    ModelAndView modelAndView = new ModelAndView();
    modelAndView.setViewName("error");
    modelAndView.addObject("code", ex.getCode());
    modelAndView.addObject("msg", ex.getMsg());
    return modelAndView;
}
```

在 templates 目录下，添加 error.ftl（这里使用freemarker） 进行渲染：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>错误页面</title>
</head>
<body>
    <h1>${code}</h1>
    <h1>${msg}</h1>
</body>
</html>
```

重启应用，http://localhost:8080/home 显示自定的错误页面内容。
补充：如果全部异常处理返回json，那么可以使用 @RestControllerAdvice 代替 @ControllerAdvice ，这样在方法上就可以不需要添加 @ResponseBody。

