```
import java.util.List;
import java.util.Map;

/**
 * xuyj,工具类,List或Map对象不为空且长度大于0则返回true,否则返回false
 */
public class CollectionUtils {
    public static boolean notNullAndNotEmpty(List list){
        return list != null && list.size() > 0;
    }
    public static boolean notNullAndNotEmpty(Map map){
        return map != null && map.size() > 0;
    }
}
```
