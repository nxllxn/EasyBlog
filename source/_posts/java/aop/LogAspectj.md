---
title: AOP+ExceptionHandler+ControllerAdvice实现全局异常处理与日志打印
tags:
     - java
     - AOP
     - AspectJ
     - ExceptionHandler
     - ControllerAdvice
     - 全局异常处理
     - 日志
---

### 需求缘起

在一般的业务系统开发过程中异常处理和日志打印都是很常见也很必要的部分。
* 首先是异常处理，你可以通过在代码中加上无数的try-catch代码块儿来完成这件事情，但是这样写出的代码非常不美观，而且大多数都是重复性劳动。举个例子，比如接口接收参数处理这件事情，假设你需要一个long类型的id来完成一次以id为准的查询，同时我们也假设这个调用接口的客户端非常“不负责任”，在调用过程中任何可能的值都可能被传递过来，比如根本就没有传指定字段，空字符串，null,一段非数字的文本（“张三123”）,一个负数等等。当你想要将这个入参字符串转换为一个数值id时，可能发生各种异常，比如空指针啊，数字格式化异常等等。这时肯定不能直接让程序崩溃掉，然后给前端一个500吧？这时不可避免的就需要进行异常处理，你可能会说，“很简单啊，我加一个try-catch块就好了”，对，你这的确可以解决问题，但是新的问题来了，如果有10个20个甚至100个接口，每个接口你都做一遍这种重复性的劳动，岂不是很sb。所以笔者还是推荐使用全局的异常处理机制，在不影响系统功能或者说系统流程执行的情况下，尽量将捕获到的异常向外层调用者抛出，然后在最外层的ExceptionHandler中进行处理。
* 其次我们说一下关于日志打印，你可能又会说了，程序写到那里，如果需要打印日志直接打印就好了。那么我想申明我的几个观点：1）你有必要在整个执行流程里面加上这些日志打印的逻辑吗，这么做和单步调试有什么分别，为什么要用日志来做调试的工作呢，感觉和用System.out.println()的如出一辙。2）如果发生了系统故障，凭借输入，输出，异常堆栈应该完全可以定位到异常代码的位置了，如果此时还不能定位，只能说明代码设计有问题。3）日志从接口调用的输入，异常处理（不是必要的），输出最好能够在一起被打印出来，这很好理解了，因为业务系统运行起来基本是多线程的，如果输入输出打印不同步，日志阅读起来将非常困难。此外，日志是为了帮助程序员在出异常的时候能够快速定位到异常代码的位置。换句话说这些东西并不能看作业务代码的一部分，好的设计应该尽可能减少这些“不相关的代码”对真正的业务代码的侵入性。所以我们也希望实现全局的日志打印。

<!--more-->

### 写在编码之前
前面详细阐述了需求，现在简单描述一下实现思路：
* 全局异常处理：异常抛出+ControllerAdvice+ExceptionHandler
* 全局日志打印，分两种情况：
    * 存在异常，直接在ExceptionHandler注解标注的方法中进行日志打印，包括请求url，参数，对应的异常情况下的响应。
    * 不存在异常，直接在Spring Aop中进行日志打印，spring around注解标注的方法中可以拿到controller中接口返回的响应，先打印日志，再返回结果。
    
### 注意点
实际的设计可能与上述设计有一些出入，主要原因如下：
* 我们定义的切面在Controller的接口上面，但是利用@RequestBody注解（对应Post方法）和@RequestParam注解（对应Get方法）进行参数绑定的过程发生在aop之前。而参数绑定过程中如果Post请求请求体body为空，将直接抛出HttpMessageNotReadableException异常，如果Get请求参数缺失将直接抛出MissingServletRequestParameterException异常。如果抛出异常，请求实体HttpServletRequest以及对应的Exception对象将直接交付给由ExceptionHandler注解的方法进行处理，而不会再进入aop中，所以这些请求的日志打不出来。那么这部分“遗漏的请求”必须在由ExceptionHandler注解的方法中进行日志打印。
* 在实际编码过程中我们也遇到另一个问题，就是HttpServletRequest对象在ExceptionHandler中无法取出参数，InputStream为空，调用getReader抛出流已经被打开的异常。但是在aop中可以通过@Around注解标记的方法的入参ProceedingJoinPoint.getArgs获取到。所以我们决大多数日志都在aop中进行打印。
* 会不会有遗漏参数的情况呢，其实是有的，HttpMessageNotReadableException对应Post请求请求体body为空，这是没有参数可取，也就没有遗漏一说。MissingServletRequestParameterException对应Get请求参数缺失。这时，我们还需要利用HttpServletRequest.getQueryString将请求参数打印出来。

### 代码实现

#### AOP日志打印
```
@Aspect   //定义一个切面
@Configuration
public class LogAspect {
    private static final String TAG = "LogAspect";

    // 定义切点Pointcut
    @Pointcut("execution(* com.nxllxn.controller.*Controller.*(..))")
    public void executeService() {
    }

    @Around("executeService()")
    public Object executeService(ProceedingJoinPoint pjp) throws Throwable {
        RequestAttributes ra = RequestContextHolder.getRequestAttributes();
        ServletRequestAttributes sra = (ServletRequestAttributes) ra;
        HttpServletRequest request = sra.getRequest();

        String url = request.getRequestURL().toString();
        String method = request.getMethod();

        String requestParamStr = null;
        if (method.equalsIgnoreCase("GET")) {
            requestParamStr = request.getQueryString();
        } else if (method.equalsIgnoreCase("POST")) {
            requestParamStr = Arrays.toString(pjp.getArgs());
        }

        // result的值就是被拦截方法的返回值
        Object result;
        try {
            result = pjp.proceed();
        } catch (Exception ex) {
            //如果发生异常，那么将其转换为用户友好的接口调用返回信息
            String code = getCodeByException(ex);
            String msd = getMsgByException(ex);

            JSONObject resultJsonObj = new JSONObject();
            resultJsonObj.put("code", code);
            resultJsonObj.put("msg", msg);

            Log.i(TAG, String.format("[%s] [%s] [%s] [%s]", method.toUpperCase(), url, requestParamStr, resultJsonObj));

            return resultJsonObj.toJSONString();
        }

        Log.i(TAG, String.format("[%s] [%s] [%s] [%s]", method.toUpperCase(), url, requestParamStr, result));

        return result;
    }
}

```

代码很简单，就几个关键点提一下：
* 首先我们定义了PointCut。就是当前切面在controller包下面的所有Controller中所有接口之前执行
* 我们从@Around注解标记的方法的入参ProceedingJoinPoint对象中取出了请求对象HttpServletRequest
* 从HttpServletRequest中取出请求方法，url
* 如果是Get请求，那么用request.getQueryString()取出参素
* 如果是Post请求，利用ProceedingJoinPoint.getArgs()取出参数
* ProceedingJoinPoint.proceed()方法执行时会具体调用我们要调用的接口并得到返回值result。
* 接下来对异常进行处理，按照规范为异常定义统一的异常码，以及异常提示语句。
* 打印日志，格式为`[请求方法] [请求URL] [请求参数] [返回结果]`

#### ExceptionHandler异常处理
```
@ControllerAdvice
public class ExceptionHandler{
    private static final String TAG = "ExceptionHandler";

    @org.springframework.web.bind.annotation.ExceptionHandler
    public String exceptionHandler(HttpServletRequest request,Exception e) throws IOException {
        String url = request.getRequestURL().toString();
        String method = request.getMethod();

        //为什么这里还需要处理呢，那是因为HttpMessageNotReadableException MissingServletRequestParameterException
        //这两个异常发生在aop进入切面之前的参数绑定阶段（spring自动完成的），如果参数缺失，将无法在aop（LogAspect）中打印出日志

        //为什么这里只处理get的情况呢，因为
        //如果是Post请求，有参数的话，即使为null也会进入aop（LogAspect）。如果完全没有参数（即传递的是空字符串，那么requestParamStr就是空字符串，没有处理一说）
        //如果是Get请求，不缺少单数，进入aop（LogAspect）打印日志，如果缺少参数在此处打印日志
        String requestParamStr = null;
        if (method.equalsIgnoreCase("GET")) {
            requestParamStr = request.getQueryString();
        }

        String code = getCodeByException(ex);
        String msd = getMsgByException(ex);
        
        JSONObject resultJsonObj = new JSONObject();
        resultJsonObj.put("code", code);
        resultJsonObj.put("msg", msg);

        Log.i(TAG, String.format("[%s] [%s] [%s] [%s]", method.toUpperCase(), url, requestParamStr, resultJsonObj.toJSONString()));

        return resultJsonObj.toJSONString();
    }
}
```

代码描述：
* 首先用ControllerAdvice注解注解用于全局异常处理的类
* 其次用ExceptionHandler注解注解用于全局异常处理的方法,此方法拥有签名String exceptionHandler(HttpServletRequest,Exception)
* 如果是GET请求，需要做一点额外处理，就是取出参数，Post不用，因为这种情况下肯定是HttpMessageNotReadableException异常，换句话说，请求参数为空
* 接下来对异常进行处理，按照规范为异常定义统一的异常码，以及异常提示语句。
* 打印日志，格式为`[请求方法] [请求URL] [请求参数] [返回结果]`


### 结束语
至此我们的全局异常处理以及全局日志打印就实现完成了。