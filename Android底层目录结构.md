环境：vmware中装的ubuntu系统
Ubuntu版本：14.04.1（查看命令：uname -a）
Android版本：android-goldfish-3.4（查看命令：在Android内核目录下（如：~/Android/aosp/kernel/goldfish/）git branch; 查看所有goldfish版本：git branch -a）
系统用户：zhwilson
最终Android内核目录：home/zhwilson/Android/aosp/kernel/goldfish/



交叉编译工具目录：home/zhwilson/Android/aosp/prebuilts/gcc/linux-x86/arm/arm-eabi-4.8/bin
ANDROID_PRODUCT_OUT目录：home/zhwilson/Android/aosp/out/target/product/generic
模拟器目录：home/zhwilson/Android/aosp/out/host/linux-x86/bin
编译后生成的Android系统根目录：home/zhwilson/Android/aosp/out/target/product/generic/root
Android系统内置拓展程序源码（c，cpp）目录：home/zhwilson/Android/aosp/external




注：因为版本的不同，目录结构可能会有差别，但大体还是相差不多的。在Ubuntu上下载和编译Android源码，请参照老罗的博客：http://blog.csdn.net/luoshengyang/article/details/6564592