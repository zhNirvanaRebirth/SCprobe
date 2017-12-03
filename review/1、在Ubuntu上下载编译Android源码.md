**知识点：在Ubuntu上下载、编译和安装Android源码  
知识来源：老罗博客-[在Ubuntu上下载、编译和安装Android最新源代码](http://blog.csdn.net/luoshengyang/article/details/6559955)  
记录内容：在实际操作中遇到的问题和处理方法**  
***  
## 步骤：  
### 环境准备  
1. 磁盘准备：尽量准备大一点，按照博客上的肯定是不行了，我下载到的源码就是几十个G，之后你还要编译，我分配了300G的磁盘，在编译完成之后，看见磁盘占用180多G，所以个人建议还是尽量大一点，内存就4G以上吧，反正尽量高一些。当然，vmware的内存和磁盘大小都是可以改的，但是要是你都在下载源码了，才发现磁盘不够，那你不是又得重新下载么  
2. 安装vmware：这个就比较简单了，根据自己电脑的处理器位数在网上下载一个安装好就是  
3. Ubuntu系统安装：这里我使用的ubuntu系统的版本是14.04，开始使用的是16.04，但是遇到“ninja: no work to do”的问题，所以就将系统改成了14.04  
4. 安装git工具：因为我们需要下载android源码，而android的是使用git工具管理的，在ubuntu上执行下面的命令即可：  
*sudo apt-get install git-core gnupg*  
5. JDK安装：这里我安装老罗的命令执行，在编译过程中提示我说需要openjdk，所以这里就先直接安装openjdk吧，在ubuntu上执行下面的命令即可：[详见](http://www.jianshu.com/p/aeaceda41798)  
*sudo apt-get install openjdk-7-jre*  
*apt-get install openjdk-7-jdk*  
6. 安装相关的依赖包：这里只是一部分，后面遇到其他的需要的依赖包，再安装就是，在ubuntu上执行下面的命令即可：  
*sudo apt-get install flex bison gperf libsdl-dev libesd0-dev libwxgtk2.6-dev build-essential zip curl*  
### 下载Android源码工程  
**需要科学上网，否则你可能需要使用清华大学的镜像[详见](http://www.jianshu.com/p/aeaceda41798)**  
1. 下载repo工具：在ubuntu上依次执行下面的命令即可：
*wget https://dl-ssl.google.com/dl/googlesource/git-repo/repo*  
*chmod 777 repo*  
*cp repo /bin/*  
2. 下载Android的源代码：在ubuntu上依次执行下面的命令即可：  
*mkdir Android*  
*cd Android*  
*repo init -u https://dl-ssl.google.com/dl/googlesource/git-repo/repo*  
*repo sync*  
_这里需要经历漫长的等待，在网速和电脑配置等个人影响因素下，我等了五天！若中途失败，重新执行“repo sync”即可_  
### 编译Android源码  
1. 进入上面创建的Android目录，我们的源码就下载在这里，执行命令：  
*make*  
2. 遇到的问题：  
* *ninja: no work to do*  
上面提到过，我开始装的ubuntu系统是16.04的版本，后面改成14.04的版本就ok了  
* *... asked for an OpenJDK ...*  
编译需要openjdk，所以这里按照上述的安装对应版本的openjdk就ok了  
* *其他*  
比如什么网络问题呀，磁盘分配问题呀，反正很多，就自己查找处理了