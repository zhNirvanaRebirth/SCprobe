/*
	content��ΪAndroidӲ�������ģ���дJNI�������ṩJava����Ӳ������ӿ�
	author��zhwilson
*/



















more��

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
	
	
Android.mk��https://developer.android.google.cn/ndk/guides/android_mk.html
	LOCAL_PATH := $(call my-dir) //��ʾԴ�ļ��ڿ������е�λ�ã�my-dir������ǰĿ¼·��
	include $(CLEAR_VARS)  //CLEAR_VARS����ָ�������GNU Makefile,��������LOCAL_XXX(LOCAL_PATH����)
	LOCAL_MODULE := hello-jni  //LOCAL_MODULE�洢����ģ������ƣ�����ϵͳ�����ɹ����ʱ�����Զ����ǰ׺�ͺ�׺���磺libhello-jni.so�����������Ѿ���lib��ͷ���򲻻��ٴ����libǰ׺��
	LOCAL_SRC_FILES := hello-jni.c  //ָ��Դ�ļ������Դ�ļ��Կո�ָ�
	include $(BUILD_SHARED_LIBRARY)  //BUILD_SHARED_LIBRARYָ��GNU Makefile�ű��������ռ����include��LOCAL_XXX�����е���Ϣ����ȷ��Ҫ���������ݺͲ�������
	
	
���⣺
1��������Android��Ӧ�ò����Ӳ���������֮��ģ����һֱ��loading���棬�޷�����
	ԭ�򣬲���ֱ��ִ���磺mmm packages/app/Hello,�����ᷢ����out/target/product/generic/Ŀ¼�µ��ļ������˱仯���ܶණ����ʧ�ˣ�����ģ�����޷���������������ֱ�ӽ���make���������±���AndroidԴ�룬�ǵ�ÿ��make��ִ��make snod