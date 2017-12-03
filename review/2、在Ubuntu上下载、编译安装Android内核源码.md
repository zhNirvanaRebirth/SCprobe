**知识点：在Ubuntu上下载、编译和安装Android内核源码  
知识来源：老罗博客-[在Ubuntu上下载、编译和安装Android最新内核源代码（Linux Kernel）](http://blog.csdn.net/luoshengyang/article/details/6564592)  
记录内容：在实际操作中遇到的问题和处理方法**  
***  
之前我们在ubuntu上下载Android源码，并进行编译和安装，是不包括linux kernel的，在我们启动模拟器并加载内核时，默认使用的是prebuilts/qemu-kernel/arm目录下的kernel-qemu文件，也就是说这个内核是预先编译好的，若我们想制作自己的内核，就需要进行下列的操作  
***  
## 步骤：  
### 下载Linux Kernel源码  
1. 仓库克隆：使用git工具，进行kernel源码的克隆，在Android源码目录下（我的目录：home/zhwilson/Android/aosp）执行下列操作：  
*mkdir kernel*  
*cd kernel*  
*git clone http://android.googlesource.com/kernel/goldfish.git*  
上述操作依然需要科学上网，并且需要等待，完成之后再kernel目录下回有一个goldfish目录  
2. 查看下载的内核代码版本：进入goldfish目录，使用以下命令查看当前的内核版本：  
*git branch*
3. 选择适用于模拟器的内核：我们可能需要选择不同的内核版本来使用（我使用的是android-goldfish-3.4），依次执行下列操作即可：  
*查看所有可使用的内核版本：git branch -a*  
*checkout需要的版本：git checkout remotes/origin/android-gldfish-3.4*  
### 编译内核源码  
1. 设置交叉编译工具环境变量：这里依然是修改/etc/profile,上一篇笔记中有说到，我的交叉编译工具目录：home/zhwilson/Android/aosp/prebuilts/gcc/linux-x86/arm/arm-eabi-4.8/bin  
2. 修改kernel/goldfish目录下的Makefile文件：  
    ARCH		?= $(SUBARCH)  
	CROSS_COMPILE	?= $(CONFIG_CROSS_COMPILE:"%"=%)  
	修改为：  
	ARCH		?= arm  
	CROSS_COMPILE	?= arm-eabi-  
3. 开始编译：这里我们首先需要进行下配置，由于我的android源码版本使用的是5.1.1，需要在依次执行以下操作：  
*make goldfish_armv7_defconfig*  
*make*  
编译成功后会看的“Kernel: arch/arm/boot/zImage is ready”  
### 在模拟器中运行编译好的内核  
1. 运行模拟器：前面已经说过在执行emulator命令执行，需要先执行以下操作：  
*source build/envsetup.sh*  
*lunch 1*  
*启动模拟器并指定使用的内核文件：emulator -kernel ./kernel/goldfish/arch/arm/boot/zImage &*  
2.使用adb工具链接模拟器，查看内核信息：一次执行下列操作：  
*链接模拟器：adb shell*  
*进入模拟器的proc目录：cd proc*  
*查看内核信息：cat version*  
可通过内核信息看出模拟器使用的内核是不是我们编译出来的内核  
##遗留问题：  
* 交叉编译工具那样设置的原因？  
ARCH ?= arm 是设置编译体系结构为arm  
CROSS_COMPILE     ?= arm-eabi-   设置交叉编译工具的前缀，如使用gcc编译，最终会找到prebuilts/gcc/linux-x86/arm/arm-eabi-4.8/bin目录下的arm-eabi-gcc  
* 在内核目录下使用make命令进行编译，具体干了啥？  
---
**这里都是一些笔记，由于知识有限，肯定有错误的地方，希望不要误导阅读此文的同学，如果有错误的地方，还希望指正**