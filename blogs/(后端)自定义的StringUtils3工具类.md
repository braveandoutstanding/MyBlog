>如果string以endString结尾则将去除结尾部分endString后的新string返回
```
/**
* @copyright create by XuYongJie on 2018/1/27 17:21
* @description 如果string以endString则将去除结尾部分endString后的新string返回
* @version 1.0.0
*/
public class StringUtils3 {
    public static String removeEndString(String string,String endString){
        if(string != null && !"".equals(string) && endString != null && !"".equals(endString)){
            if(string.endsWith(endString)){
                return string.substring(0,string.length() - endString.length());
            }
        }
        return string;
    }
}
```
>效果截图：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-d64ecdb41210ec46.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
