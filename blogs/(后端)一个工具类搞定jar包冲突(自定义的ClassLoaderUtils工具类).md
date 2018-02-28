>解决同一个项目中同时使用两个互不兼容不同版本jar包的问题，比如Quartz的2.2.1版本的org.quartz.JobDetail是一个接口，而1.7.2版本的org.quartz.JobDetail却是一个类，互不兼容。抽取出来的类加载器如下
```
import java.io.File;
import java.io.IOException;
import java.net.URL;
import java.net.URLClassLoader;

public class ClassLoaderUtils extends URLClassLoader {//原创by胥勇杰,2017年12月31日19:09:52
    public ClassLoaderUtils(String jarDirectoryPath) throws IOException {
        this(ClassLoader.getSystemClassLoader().getParent(),jarDirectoryPath);//设置默认父加载器为ExtClassLoader
    }
    public ClassLoaderUtils(ClassLoader parentClassLoader, String jarDirectoryPath) throws IOException {//父加载器，jar所在的文件夹路径
        super(new URL[] {},parentClassLoader);//调用父类的构造器
        this.addURLClassPath(new File(jarDirectoryPath));//将jar所在的文件夹路径加入到URLClassLoader(父类)的URLClassPath中，这样findClass()会从URLClassLoader查找类
    }
    private void addURLClassPath(File file) throws IOException {//jar所在的文件夹路径
        if (file != null && file.exists() && file.isDirectory()) {
            File[] jarFileArray = file.listFiles();
            if (jarFileArray != null && jarFileArray.length > 0){
                for (File jarFile : jarFileArray) {
                    if (jarFile != null && jarFile.exists()){
                        if (jarFile.isFile() && jarFile.getName().endsWith(".jar")) {//如果是jar包,则将此jar包加入URLClassPath
                            super.addURL(new URL("file", null, jarFile.getCanonicalPath()));// 将指定的URL添加到URL列表中，以便搜索类和资源
                            continue;
                        }
                        if (jarFile.isDirectory()){//如果是文件夹则迭代遍历
                            this.addURLClassPath(jarFile);
                        }
                    }
                }
            }
        }
    }
}
```
>运用示例：
>准备两个有相同全限定名类的jar，将其中一个Add as Library，在线程中修改当前线程所使用的类加载器，而后通过反射调用类中的方法，两jar如下：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-eed260e6ba157222.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-1b282ac6010cf72b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>将lib下的jar包Add as Library
![image.png](http://upload-images.jianshu.io/upload_images/9541455-abc93d52cdb15c64.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-ed641e04259174b0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-b74d3ec4d00f9b7b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
>如上图所示，解决了同一个项目中同时使用两个互不兼容不同版本jar包的问题。


