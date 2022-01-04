# HTTP协议 Part1

为了方便，去更好的观察到每个包得真实内容，最好的方式就是搭建一个小型 `HTTP` 环境。

## 搭建 HTTP 实验环境

实验环境用到得应用软件：

* Wireshark+NPcap
* Chrome/Firefox
* Telnet
* OpenResty

同时需要用到这样一个环境[http_study](https://github.com/chronolaw/http_study)。

构建过程如下：

1. 首先从 [http_study](https://github.com/chronolaw/http_study) 下载相关得源码，在 `Release` 页面里面，将其压缩。
2. 安装 `Wireshark+NPcap` 与 `Chrome/Firefox`，软件，安装过程比较简单，就是找官网下载相关的版本，选择最新的就可，然后一路下一步就可以安装成功了
3. Windows 10自带Telnet，不需要安装，但默认是不启用的，需要你稍微设置一下。打开Windows的设置窗口，搜索“Telnet”，就会找到“启用或关闭Windows功能”，在这个窗口里找到“Telnet客户端”，打上对钩就可以了，可以参考截图。

![img](https://static001.geekbang.org/resource/image/1a/47/1af035861c4fd33cb42005eaa1f5f247.png)

4. 安装 `OpenResty`，去它的[官网](http://openresty.org/)，点击左边栏的“Download”，进入下载页面，下载适合你系统的版本，将压缩包解压到刚才的 `http_study` 文件里面。结果如下：

   ![image-20220104222534119](https://cdn.jsdelivr.net/gh/baici1/img-typora/20220104222534.png)

5. 了能够让浏览器能够使用DNS域名访问我们的实验环境，还要改一下本机的hosts文件，位置在`C:\WINDOWS\system32\drivers\etc`，在里面添加三行本机IP地址到测试域名的映射

   ```
   127.0.0.1       www.chrono.com
   127.0.0.1       www.metroid.net
   127.0.0.1       origin.io
   ```

到这里，安装工作基本上就完成了。下面开始测试，环境是否能正常使用。

首先，我们要启动 `Web` 服务器，也就是 `OpenResty`。在` http_study` 的 “`www`” 目录下有四个批处理文件，分别是：

- start：启动OpenResty服务器；
- stop：停止OpenResty服务器；
- reload：重启OpenResty服务器；
- list：列出已经启动的OpenResty服务器进程。

使用鼠标双击“start”批处理文件，就会启动OpenResty服务器在后台运行，这个过程可能会有Windows防火墙的警告，选择“允许”即可。

运行后，鼠标双击“list”可以查看OpenResty是否已经正常启动，应该会有两个nginx.exe的后台进程，大概是下图的样子。

![img](https://static001.geekbang.org/resource/image/db/1d/dba34b8a38e98bef92289315db29ee1d.png)

接下来运行 `Wireshark`，开始抓包。进行以下配置：

![image-20220104223339422](https://cdn.jsdelivr.net/gh/baici1/img-typora/20220104223339.png)



鼠标双击开始界面里的“ `loopback Adapter`”即可开始抓取本机上的网络数据。

然后我们打开Chrome，在地址栏输入“`http://localhost`”，访问刚才启动的OpenResty服务器，就会看到一个简单的欢迎界面，如下图所示。

![img](https://static001.geekbang.org/resource/image/d7/88/d7f12d4d480d7100cd9804d2b16b8a88.png)

这时再回头去看Wireshark，应该会显示已经抓到了一些数据，就可以用鼠标点击工具栏里的“停止捕获”按钮告诉Wireshark“到此为止”，不再继续抓包。

![img](https://static001.geekbang.org/resource/image/f7/79/f7d05a3939d81742f18d2da7a1883179.png)

到现在实验环境已经搭建成功了，后续会利用这个环境进行分析每个包的内容。

## 键入网址再按下回车会发生什么？

这是一个常见的面试题目！

其实在[网络综合篇](https://www.guide.yangdiy.cn/#/%E8%AE%A1%E7%AE%97%E6%9C%BA%E5%9F%BA%E7%A1%80/%E7%BD%91%E7%BB%9C%E7%BB%BC%E5%90%88%E7%AF%87)已经进行一定的分析，而今天我们需要分析呢，主要分析 HTTP 协议在这个过程中到底干了什么，不能光理论，得实操一下，亲眼见见才行。

