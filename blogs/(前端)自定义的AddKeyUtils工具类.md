>本工具类自动给传入的对象或数组中的每一个元素加一个key。

>React需要我们给每一个元素加一个key，不然会报如下警告：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-b54ef51555512317.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```
//胥勇杰,2018年2月3日18:29:15
//本工具类自动给传入的对象或数组中的每一个元素加一个key。
export const AddKeyUtils = {
  addKey: function (object){
    if (object instanceof Array) {
      for (let i = 0, length = object.length; i < length; i++) {
        object[i].key = i;
      }
    }else{
      object.key = 0;
    }
    return object;
  }
};
```
>效果截图：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-a35a12b822485373.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-e30d901c2d4c5a72.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-c5b6c863c64c084c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-a9e0e72993a7ae1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-88d886414efd27da.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>参考博客：
React中key的必要性与使用
https://www.jianshu.com/p/0218ff2591ec
