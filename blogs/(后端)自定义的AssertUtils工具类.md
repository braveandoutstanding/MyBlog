>将判NULL、判空、字符串是否有内容及定义异常信息精简到极致。
```
import java.util.Collection;

/**
 * @author XuYongJie
 * @description 参照org.springframework.util.Assert,对类型进行判断,如果第一个参数不满足对应的要求,则抛出IllegalArgumentException异常,否则返回传入的一个对象
 */
@SuppressWarnings("unchecked")
public class AssertUtils {
    public static <T> T notNull(T object, String message) {
        if (object == null) {
            throw new IllegalArgumentException(message);
        }
        return object;
    }
    public static <T extends Collection> T notEmpty(Collection<?> collection, String message) {
        if (collection == null || collection.size() == 0) {
            throw new IllegalArgumentException(message);
        }
        return (T)collection;//这里实际上是将collection强转为接收者的类型,反编译一看便知,如: List<String> stringList2 = (List)AssertUtils.notEmpty(stringList, "notEmpty");
    }
    public static <T> T[] notEmpty(T[] objectArray,String message) {
        if (objectArray == null || objectArray.length == 0) {
            throw new IllegalArgumentException(message);
        }
        return objectArray;
    }
    public static String hasText(String text, String message) {
        if (text == null || text.trim().length() == 0) {
            throw new IllegalArgumentException(message);
        }
        return text;
    }
}
```
>效果截图：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-aedfacc005b79365.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-5b50d12396f5e202.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-f9031df3afc601a8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
