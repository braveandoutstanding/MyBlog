```
引入fastjson的依赖即可
<!-- https://mvnrepository.com/artifact/com.alibaba/fastjson -->
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.41</version>
</dependency>
```
```
import com.alibaba.fastjson.JSON;
import com.alibaba.fastjson.TypeReference;
/**
* @copyright create by XuYongJie on 2018/1/24 12:43 
* @description 本工具类实现了深复制、将任意两个字段一致的对象进行转换
* @version 1.0.0
*/
@SuppressWarnings("unchecked")
public class SourceToTargetUtils<T,SOURCE,TARGET>{
    public static <T> T deepClone(T t,TypeReference<T> typereference){//深复制
        return (T)JSON.parseObject(JSON.toJSONString(t),typereference);
    }
    public static <SOURCE,TARGET> TARGET sourceToTarget(SOURCE source,TypeReference<TARGET> typereference){//将任意两个字段一致的对象进行转换
        return (TARGET)JSON.parseObject(JSON.toJSONString(source),typereference);
    }
}
```
效果演示：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-8c52997511b7fa20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-ffbc46841af816ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
