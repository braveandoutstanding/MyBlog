>声明：本工具类是我参考老徐写的ShellUtil进行了适当修改，征得同意才开发源码。

>本工具类实现了直接在本地通过JSCH操作Linux主机，并支持批量操作及快速失败。
```
引入pom依赖：
<!--JSCH是一个纯粹的用java实现SSH功能的java  library. 官方地址为：http://www.jcraft.com/jsch/-->
<dependency>
    <groupId>com.jcraft</groupId>
    <artifactId>jsch</artifactId>
    <version>0.1.54</version>
</dependency>
```
```
package com.eks.utils;

import com.jcraft.jsch.Channel;
import com.jcraft.jsch.ChannelSftp;
import com.jcraft.jsch.JSch;
import com.jcraft.jsch.Session;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.io.*;
import java.util.List;
import java.util.Properties;

public class JschUtils {
    private static final Logger logger = LoggerFactory.getLogger(JschUtils.class);
    private final static Integer DEFAULT_PORT = 22;//默认端口
    private final static Integer DEFAULT_CONNECT_TIMEOUT = 30 * 1000;//默认连接超时时间
    private final static String DEFAULT_CHARSETNAME = "UTF-8";//默认字符编码
    private final static String LINUX_LINE_SEPARATOR = "\n";
    /**
    * @copyright create by XuYongJie on 2018/2/7 16:29
    * @description 执行命令并获取结果
    * @version 1.0.1
    */
    public static String shell(String username,String host,String password,String script) throws Exception {
        return JschUtils.shell(username,host,DEFAULT_PORT,password,script,DEFAULT_CONNECT_TIMEOUT,DEFAULT_CHARSETNAME,DEFAULT_CHARSETNAME);
    }
    public static String shell(String username,String host,String password,List<String> scriptList) throws Exception {
        StringBuffer stringBuffer = new StringBuffer();
        for(int i = 0,size = scriptList.size(),last = size -1; i < size ;i++){
            stringBuffer.append(i == last ? (scriptList.get(i)) : (scriptList.get(i) + " && "));//快速失败
        }
        return JschUtils.shell(username,host,password,stringBuffer.toString());
    }
    public static String shell(String username,String host,Integer port,String password,String script,Integer connectTimeout,String writeCharsetName,String readerCharsetName) throws Exception {
        JSch jsch = new JSch();//创建JSch对象
        Session session = null;
        Channel channel = null;
        try {
            session = jsch.getSession(username,host,port);//根据用户名，主机ip，端口获取一个Session对象
            session.setPassword(password);//设置登录主机的密码
            session.setConfig("StrictHostKeyChecking", "no");//如果设置成“yes”，ssh就不会自动把计算机的密匙加入“$HOME/.ssh/known_hosts”文件，并且一旦计算机的密匙发生了变化，就拒绝连接。
            session.connect(connectTimeout);//设置登录超时时间
            /*
            调用openChannel(String type)
            可以在session上打开指定类型的channel。该channel只是被初始化，使用前需要先调用connect()进行连接。
            Channel的类型可以为如下类型：
            shell - ChannelShell
            exec - ChannelExec
            direct-tcpip - ChannelDirectTCPIP
            sftp - ChannelSftp
            subsystem - ChannelSubsystem
            其中，ChannelShell和ChannelExec比较类似，都可以作为执行Shell脚本的Channel类型。它们有一个比较重要的区别：ChannelShell可以看作是执行一个交互式的Shell，而ChannelExec是执行一个Shell脚本。
            */
            channel = session.openChannel("shell");//打开shell通道
            BufferedWriter bufferedWriter = new BufferedWriter(new OutputStreamWriter(channel.getOutputStream(),writeCharsetName));//写入该流的所有数据都将发送到远程端
            BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(channel.getInputStream(), readerCharsetName));//从远程端到达的所有数据都能从这个流中读取到
            BufferedReader errBufferedReader = new BufferedReader(new InputStreamReader(channel.getExtInputStream(), readerCharsetName));//错误流
            channel.connect(connectTimeout);//建立shell通道的连接

            bufferedWriter.write(script);//写入该流的所有数据都将发送到远程端
            bufferedWriter.newLine();//经测试必须加writer.newLine(),流.read方法是按字节读取，只判断有没有读取到数据，不管内容的，所以换行符也会被读出来,而BufferedReader.readLine是按行读取的，即从当前位置一直读取数据，直到遇到换行符，然后去掉换行符，返回读取到的数据
            bufferedWriter.write("exit");//结束本次交互
            bufferedWriter.newLine();//经测试必须加writer.newLine()
            bufferedWriter.flush();//在调用flush或close以前,数据并没有被写入基础流,而是放在缓存区,调用后才被真正写入

            StringBuffer resultStringBuffer = new StringBuffer("");
            String resultString = "";
            while ((resultString = errBufferedReader.readLine()) != null) {
                logger.error(resultString);
                resultStringBuffer.append(resultString).append(JschUtils.LINUX_LINE_SEPARATOR);
            }
            while ((resultString = bufferedReader.readLine()) != null) {
                logger.error(resultString);
                resultStringBuffer.append(resultString).append(JschUtils.LINUX_LINE_SEPARATOR);
            }
            int status = channel.getExitStatus();
            if (status != 0) {//0表示正常退出
                throw new Exception("错误状态:" + status);
            }
            return resultStringBuffer.toString();
        } catch (Exception e) {
            logger.error("调用远程服务器脚本出现错误:username=" + username + ",host=" + host + ",port=" + script + ",script=" + port, e);
            throw e;
        } finally {
            if (channel != null && channel.isConnected()) {
                channel.disconnect();
            }
            if (session != null && session.isConnected()) {
                session.disconnect();
            }
        }
    }
    /**
    * @copyright create by XuYongJie on 2018/2/7 16:30
    * @description 获取文件的内容
    */
    public static String getFileContent(String username,String host,String password,String filePath) throws Exception {
        JSch jsch = new JSch();
        Session session = null;
        ChannelSftp channelSftp = null;
        try {
            session = jsch.getSession(username, host);
            session.setPassword(password);
            Properties config = new Properties();
            config.put("StrictHostKeyChecking", "no");
            session.setConfig(config);
            session.connect();
            channelSftp = (ChannelSftp)session.openChannel("sftp");
            channelSftp.connect();
            return JschUtils.convertStreamToString(channelSftp.get(filePath));
        }catch (Exception e){
            logger.error("调用远程服务器脚本出现错误:username=" + username + ",host=" + host + ",filePath=" + filePath, e);
            throw e;
        }finally{
            if (channelSftp != null && channelSftp.isConnected()) {
                channelSftp.disconnect();
            }
            if (session != null && session.isConnected()) {
                session.disconnect();
            }
        }
    }
    /**
    * @copyright create by XuYongJie on 2018/2/7 16:31
    * @description 将流转为字符串
    * @version 1.0.0
    */
    public static String convertStreamToString(InputStream inputStream) throws IOException{
        BufferedReader bufferedReader = null;
        try {
            bufferedReader = new BufferedReader(new InputStreamReader(inputStream, JschUtils.DEFAULT_CHARSETNAME));
            StringBuffer stringBuffer = new StringBuffer();
            String line = "";
            while ((line = bufferedReader.readLine()) != null) {
                stringBuffer.append(line).append(JschUtils.LINUX_LINE_SEPARATOR);
            }
            return stringBuffer.toString();
        } catch (IOException e) {
            logger.error("将流转为字符串出现错误", e);
            throw e;
        } finally {
            try {
                inputStream.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if(bufferedReader != null){
                    bufferedReader.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```
>效果截图：
![image.png](http://upload-images.jianshu.io/upload_images/9541455-6dcd47a51850011d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-c7d34e026246eb19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-f21d278c66d10898.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-47d198a545171a19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](http://upload-images.jianshu.io/upload_images/9541455-96bdfdf630dc1b4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
