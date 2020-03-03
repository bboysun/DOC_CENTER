## ==TensorfFow Windows环境的安装==

一、前言

​	本次安装tensorflow是基于Python的，安装Python的过程不做说明（既然决定按，Python肯定要先了解啊），本次教程是windows下Anaconda安装Tensorflow的过程（cpu版，显卡不支持gpu版的...所以你懂的...）

二、安装环境

​	(1) win7，因为电脑老了，重装了win7旗舰版的操作系统。

​	(2) python3.5.2，目前TensorFlow最好兼容的Python版本是3.5.2，所以一定是3.5.2版本，其他版本会有问题。

​	(3) Anaconda3-4.2.0-Windows-x86_64.exe (windows下安装注意选择windows x86 64位就好)

三、Anaconda安装

​	下载可以去官网上下载，直接搜索找与你电脑对应的版本就好，我个人习惯从国内镜像网站下载，下载快哇（国内清华镜像网站是：https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/）然后一路下一步就okay。（此处就不截图赘述，因为已经安装过了，也没法截图了）

​	验证Anaconda是否安装成功的方法： 命令窗口中输入“conda --version”  ----->得到conda 4.2.0

　　 看到了这个结果，恭喜你，你已经成功的安装上了Anaconda了，那么我们继续。

四、TensorFlow安装

​	(1) 安装Tensorflow时，需要从Anaconda仓库中下载，一般默认链接的都是国外镜像地址，下载肯定很慢啊（跨国呢！），这里我是用国内清华镜像，需要改一下链接镜像的地址。这里，我们打开刚刚安装好的Anaconda中的 Anaconda Prompt，然后输入：

​	conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/ 　　　	conda config --set show_channel_urls yes

　　 这两行代码用来改成连接清华镜像的，对应的文件在C:\Users\Darryl\.condarc，文件内容如下图：

![1552999106567](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1552999106567.png)

​	(2) 接下来，安装TensorFlow，在Anaconda Prompt中输入：conda create -n tensorflow python=3.5.2

​	正常如下图：

![1552999272112](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1552999272112.png)

​	等待，然后输入“y”

![1552999319196](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1552999319196.png)

​	然后：

![1552999348223](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1552999348223.png)

​	最后，看到上面这些activate tensorflow（这么直白的英语，看看是不是很激动，）恭喜你，tensorflow你已经安装成功啦，去激活一下，紧接着输入：“activate tensorflow”就Ok了。

​	(3) 我们要安装的是CPU版本，那么在命令下紧接着输入：pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/https://mirrors.tuna.tsinghua.edu.cn/tensorflow/windows/cpu/tensorflow-1.1.0-cp35-cp35m-win_amd64.whl

 	你也可以自己选择对应的Tensorflow版本，可以在清华镜像中查看。（我装的就是1.1.0 CPU版本）

​	如下图就表明大法完成：

![1552999665128](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1552999665128.png)

五、测试验证：

​	在Anaconda Prompt窗口中输入： python

　　　　进入python后输入：

　　　　import tensorflow as tf

　　　　sess = tf.Session()

　　　　a = tf.constant(10)

　　　　b= tf.constant(12)

　　　　sess.run(a+b)

​	执行如下图：

![1552999911182](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1552999911182.png)

六、备注：

​	如果中途提示说需要pip升级，是因为我们pip的版本问题，我们只需跟着提示将pip版本做对应的升级即可，如下图：

![1553000059738](C:\Users\Darryl\AppData\Roaming\Typora\typora-user-images\1553000059738.png)

​	升级完成后，就可以通过pip对TensorFlow CPU版本进行安装。

==累死老子了，先这样，后续会将我自学到的TensorFlow做逐一的整理，再接再厉咯==