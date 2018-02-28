>直接在方法的参数、对象的属性上加上注解，省去在业务代码中对参数等进行判空等操作。
```
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.validation.beanvalidation.MethodValidationPostProcessor;
/**
* @copyright create by XuYongJie on 2018/1/27 17:32 
* @description JSR和Hibernate validator的校验只能对Object的属性进行校验，不能对单个的参数进行校验，spring 在此基础上进行了扩展，添加了MethodValidationPostProcessor拦截器，可以实现对方法参数的校验
* @version 1.0.0
*/
@Configuration//@Configuration标注在类上，相当于把该类作为spring的xml配置文件中的<beans>，作用为：配置spring容器(应用上下文)
public class MethodValidationPostProcessorConfig{
    //Spring Boot项目由于自带了Hibernate validator 5,即在pom.xml无需再加如下核心的pom依赖
    /**
     <dependency>
     <groupId>org.hibernate</groupId>
     <artifactId>hibernate-validator</artifactId>
     <version>5.3.1.Final</version>
     </dependency>
     */
    @Bean//@Bean标注在方法上(返回某个实例的方法)，等价于spring的xml配置文件中的<bean>，作用为：注册bean对象
    public MethodValidationPostProcessor methodValidationPostProcessor(){//注入校验器到Spring Boot的运行环境
        return new MethodValidationPostProcessor();
    }
}
```

>修改本人《全局异常处理器+统一返回对象+工具类》中的文件
https://www.jianshu.com/p/e7fb7be9edd1

>异常捕获：
```
import com.eks.utils.base.ExceptionUtils2;
import com.eks.utils.base.Result;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.validation.ConstraintViolationException;

/**
* @copyright create by XuYongJie on 2018/1/20 16:13
* @description 全局异常处理器
*/
@ControllerAdvice//该注解是spring2.3以后新增的一个注解，主要是用来Controller的一些公共的需求的低侵入性增强提供辅助，作用于@RequestMapping标注的方法上。
@ResponseBody//如果返回的为json数据或其它对象，添加该注解
public class GlobalExceptionHandler {

    private static final Logger logger = LoggerFactory.getLogger(GlobalExceptionHandler.class);

    @ExceptionHandler(value = Exception.class)//该注解是配合@ExceptionHandler一起使用的注解，自定义错误处理器，可自己组装json字符串，并返回到页面。
    public Result globalExceptionHandler(HttpServletRequest request,Exception exception) throws Exception {
        logger.error(ExceptionUtils2.exceptionString(exception));
        return ExceptionUtils2.exceptionResult(exception);
    }
    @ExceptionHandler(value = ConstraintViolationException.class)//该注解是配合@ExceptionHandler一起使用的注解，自定义错误处理器，可自己组装json字符串，并返回到页面。
    public Result constraintViolationExceptionHandler(HttpServletRequest request,ConstraintViolationException exception) throws Exception {
        logger.error(ExceptionUtils2.exceptionString(exception));
        return ExceptionUtils2.exceptionResult(exception);
    }
    @ExceptionHandler(value = MethodArgumentNotValidException.class)//该注解是配合@ExceptionHandler一起使用的注解，自定义错误处理器，可自己组装json字符串，并返回到页面。
    public Result methodArgumentNotValidExceptionHandler(HttpServletRequest request,MethodArgumentNotValidException exception) throws Exception {
        logger.error(ExceptionUtils2.exceptionString(exception));
        return ExceptionUtils2.exceptionResult(exception);
    }
}
```

>异常处理：

```
import com.eks.utils.StringUtils3;
import org.springframework.validation.BindingResult;
import org.springframework.validation.FieldError;
import org.springframework.web.bind.MethodArgumentNotValidException;

import javax.validation.ConstraintViolation;
import javax.validation.ConstraintViolationException;
import java.io.PrintWriter;
import java.io.StringWriter;
import java.util.List;
import java.util.Set;

/**
 * @copyright create by XuYongJie on 2018/1/20 16:12
 * @description 异常处理工具类
 */
public class ExceptionUtils2 {
    private final static String SEPARATIVE_SIGN = ",";
    public static String exceptionString(Exception exception) {
        StringWriter stringWriter = new StringWriter();
        exception.printStackTrace(new PrintWriter(new StringWriter(),true));
        return stringWriter.toString();
    }
    public static Result exceptionResult(Exception exception) {
        if(exception.getMessage() == null){
            return new Result().setSuccess(false).setErrorMsg(exception.toString());
        }
        return new Result().setSuccess(false).setErrorMsg(exception.getMessage());
    }
    public static Result exceptionResult(ConstraintViolationException exception) {//使用hibernate-validation对参数进行校验的异常处理
        Set<ConstraintViolation<?>> constraintViolationSet = exception.getConstraintViolations();
        StringBuffer stringBuffer = new StringBuffer();
        for (ConstraintViolation<?> constraintViolation : constraintViolationSet) {
            String message = constraintViolation.getMessage();
            String messageTemplate = constraintViolation.getMessageTemplate();
            if(message != null && message.equals(messageTemplate)){
                stringBuffer.append(message);
            }else{
                stringBuffer.append(constraintViolation.toString());
            }
            stringBuffer.append(ExceptionUtils2.SEPARATIVE_SIGN);
        }
        return new Result().setSuccess(false).setErrorMsg(StringUtils3.removeEndString(stringBuffer.toString(),ExceptionUtils2.SEPARATIVE_SIGN));
    }
    public static Result exceptionResult(MethodArgumentNotValidException exception) {//使用hibernate-validation对参数进行校验的异常处理
        BindingResult bindingResult = exception.getBindingResult();
        if(bindingResult != null){
            List<FieldError> fieldErrorList = bindingResult.getFieldErrors();
            StringBuffer stringBuffer = new StringBuffer();
            for(FieldError fieldError : fieldErrorList){
                String defaultMessage = fieldError.getDefaultMessage();
                if(defaultMessage != null && !"".equals(defaultMessage)){
                    stringBuffer.append(defaultMessage);
                }else{
                    stringBuffer.append(fieldError.toString());
                }
                stringBuffer.append(ExceptionUtils2.SEPARATIVE_SIGN);
            }
            return new Result().setSuccess(false).setErrorMsg(StringUtils3.removeEndString(stringBuffer.toString(),ExceptionUtils2.SEPARATIVE_SIGN));
        }
        return new Result().setSuccess(false).setErrorMsg(exception.toString());
    }
}
```

>在需要进行方法参数校验的类上加@Validated注解
在需要进行对象校验的类上加@Valid注解

效果如下：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-f5775d02e0126219.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-09793a1127e4f8b7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-6028718dc24b8751.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-a53bbed9893ca7fd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-8172822701369764.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-d66bc9f9908fdebb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>参考博客：
Java Bean Validation 最佳实践
https://www.cnblogs.com/beiyan/p/5946345.html
