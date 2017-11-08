在Android系统中内置C可执行程序来访问Linux驱动程序
1、在external目录下创建c源代码目录hello：make hello
2、编写访问内核驱动程序的c源代码
3、在external/hello文件夹下创建Android.mk文件并配置Android.mk文件
4、返回home/zhwilson/Android/aosp/目录
5、使用mmm命令编译此hello模块：mmm ./external/hello （编译之前执行命令：source bulid/envsetup.sh； 编译完成，可以在out/target/product/gerneric/system/bin目录下，看到可执行文件hello）
6、重新打包Android系统文件system.img：make snod（成功后，hello可执行文件就包含在system.img文件中了）
7、运行模拟器：emulator -kernel ./kernel/goldfish/arch/arm/boot/zImage &（运行之前执行命令：lunch）
8、执行hello可执行程序来访问内核驱动：使用adb shell然后进入system/bin执行./hello


注：文中的相对目录均在home/zhwilson/Android/aosp/目录下











老罗博客：在Ubuntu上为Android系统内置C可执行程序测试Linux内核驱动程序（http://blog.csdn.net/luoshengyang/article/details/6571210）
note：
方法：（底层是怎么实现的）
1、open(DEVICE_NAME, O_RDWR);
2、read(fd, &val, sizeof(val));
文件：
1、Android.mk文件（需要怎么配置，表示什么意思）