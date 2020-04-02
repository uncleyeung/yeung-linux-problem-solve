<p align="center">
  <a href="https://github.com/uncleyeung">
   <img alt="Uncle-Yeong-Logo" src="https://raw.githubusercontent.com/uncleyeung/uncleyeung.github.io/master/web/img/logo1.jpg">
  </a>
</p>

<p align="center">
  为简化开发工作、提高生产率而生
</p>

<p align="center">
  
  <a href="https://github.com/996icu/996.ICU/blob/master/LICENSE">
    <img alt="996icu" src="https://img.shields.io/badge/license-NPL%20(The%20996%20Prohibited%20License)-blue.svg">
  </a>

  <a href="https://www.apache.org/licenses/LICENSE-2.0">
    <img alt="code style" src="https://img.shields.io/badge/license-Apache%202-4EB1BA.svg?style=flat-square">
  </a>
</p>

# uncleyeung's Repo For Cydia
> * Source: https://github.com/uncleyeung/uncleyeung.github.io/
> * Twitter: https://twitter.com/uncle_yeung
> * Tumblr: https://www.tumblr.com/blog/uncleyeung
# cpu标高检查过程及其解决方案
#### 前言
+ 通过NIO读取文件内容模拟cpu持续跑高问题
+ 结果是已经确定的,通过cpu跑高反向检查异常线程栈
---
模拟CPU跑高代码, [详情参考见](https://github.com/uncleyeung/yeung-java-controller-test/blob/master/src/main/java/com/uncle/controller/nio/FileNioTest.java)


```java
/**
 * @author 杨戬
 * @className FileNioTest
 * @email uncle.yeung.bo@gmail.com
 * @date 20-4-2 10:50
 */
public class FileNioTest {
    public static void main(String[] args) {
        while (true) {
            try {
                RandomAccessFile rdf = new RandomAccessFile("/home/uncle/tmp/hello.txt", "rw");
                //利用channel中的FileChannel来实现文件的读取
                FileChannel inChannel = rdf.getChannel();
                //设置缓冲区容量为10
                ByteBuffer buf = ByteBuffer.allocate(10);
                //从通道中读取数据到缓冲区，返回读取的字节数量
                int byteRead = inChannel.read(buf);
                //数量为-1表示读取完毕。
                while (byteRead != -1) {
                    //切换模式为读模式，其实就是把postion位置设置为0，可以从0开始读取
                    buf.flip();
                    //如果缓冲区还有数据
                    while (buf.hasRemaining()) {
                        //输出一个字符
                        System.out.print((char) buf.get());
                    }
                    //数据读完后清空缓冲区
                    buf.clear();
                    //继续把通道内剩余数据写入缓冲区
                    byteRead = inChannel.read(buf);
                }
                //关闭通道
                rdf.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
            //Thread.sleep(1000);
        }
    }
}
``` 
+ 运行main方法
---
+ 现在先来分析一下cpu的情况,可以看到pid:8953和pid:15268 cpu的占用率达到了106.3%和98.3%,此时已经达到了我们的预期cpu高位骚走.

![GitHub](img/top-1.png "top 情况")

---
+  现在通过[top -H -p 8953] 或者 [ps -mp 8953 -o THREAD,time,tid] 查看当前线程组的情况,可以看到pid:31866/9000/32347持续跑高,怎么定位呢?怎么知道是哪个java的方法的哪个线程吗?


![GitHub](img/top-2.png "top -H -p 情况")
---
稍后在做更新....