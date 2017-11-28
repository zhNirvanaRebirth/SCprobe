/*
	content：为Android硬件抽象层模块编写JNI方法，提供Java访问硬件服务接口
	author：zhwilson
*/



















more：

Type Signature                Java Type
Z                             boolean
B                             byte
C                             char
S                             short
I                             int
J                             long
F                             float
D                             double
L fully-qualified-class;       fully-qualified-class
[type                         type[]
(arg-types)ret-type           method type

example:
	java: long f (int n, String s, int[] arr);
	c:    (ILjava/lang/String;[I)J
	
	
Android.mk：https://developer.android.google.cn/ndk/guides/android_mk.html
	LOCAL_PATH := $(call my-dir) //表示源文件在开发树中的位置，my-dir返还当前目录路径
	include $(CLEAR_VARS)  //CLEAR_VARS变量指向特殊的GNU Makefile,可清除许多LOCAL_XXX(LOCAL_PATH除外)
	LOCAL_MODULE := hello-jni  //LOCAL_MODULE存储构建模块的名称（构建系统在生成共享库时，会自动添加前缀和后缀，如：libhello-jni.so，但若名称已经是lib开头，则不会再次添加lib前缀）
	LOCAL_SRC_FILES := hello-jni.c  //指定源文件，多个源文件以空格分隔
	include $(BUILD_SHARED_LIBRARY)  //BUILD_SHARED_LIBRARY指向GNU Makefile脚本，用于收集最近include的LOCAL_XXX变量中的信息，以确定要构建的内容和操作方法
	
	
问题：
1、创建在Android的应用层调用硬件服务程序之后，模拟器一直在loading界面，无法启动
	原因，不能直接执行如：mmm packages/app/Hello,这样会发现在out/target/product/generic/目录下的文件发生了变化，很多东西丢失了，导致模拟器无法启动，所以这里直接进行make操作，重新编译Android源码，记得每次make后执行make snod