**֪ʶ�㣺��Ubuntu�����ء�����Ͱ�װAndroid�ں�Դ��  
֪ʶ��Դ�����޲���-[��Ubuntu�����ء�����Ͱ�װAndroid�����ں�Դ���루Linux Kernel��](http://blog.csdn.net/luoshengyang/article/details/6564592)  
��¼���ݣ���ʵ�ʲ���������������ʹ�����**  
***  
֮ǰ������ubuntu������AndroidԴ�룬�����б���Ͱ�װ���ǲ�����linux kernel�ģ�����������ģ�����������ں�ʱ��Ĭ��ʹ�õ���prebuilts/qemu-kernel/armĿ¼�µ�kernel-qemu�ļ���Ҳ����˵����ں���Ԥ�ȱ���õģ��������������Լ����ںˣ�����Ҫ�������еĲ���  
***  
## ���裺  
### ����Linux KernelԴ��  
1. �ֿ��¡��ʹ��git���ߣ�����kernelԴ��Ŀ�¡����AndroidԴ��Ŀ¼�£��ҵ�Ŀ¼��home/zhwilson/Android/aosp��ִ�����в�����  
*mkdir kernel*  
*cd kernel*  
*git clone http://android.googlesource.com/kernel/goldfish.git*  
����������Ȼ��Ҫ��ѧ������������Ҫ�ȴ������֮����kernelĿ¼�»���һ��goldfishĿ¼  
2. �鿴���ص��ں˴���汾������goldfishĿ¼��ʹ����������鿴��ǰ���ں˰汾��  
*git branch*
3. ѡ��������ģ�������ںˣ����ǿ�����Ҫѡ��ͬ���ں˰汾��ʹ�ã���ʹ�õ���android-goldfish-3.4��������ִ�����в������ɣ�  
*�鿴���п�ʹ�õ��ں˰汾��git branch -a*  
*checkout��Ҫ�İ汾��git checkout remotes/origin/android-gldfish-3.4*  
### �����ں�Դ��  
1. ���ý�����빤�߻���������������Ȼ���޸�/etc/profile,��һƪ�ʼ�����˵�����ҵĽ�����빤��Ŀ¼��home/zhwilson/Android/aosp/prebuilts/gcc/linux-x86/arm/arm-eabi-4.8/bin  
2. �޸�kernel/goldfishĿ¼�µ�Makefile�ļ���  
    ARCH		?= $(SUBARCH)  
	CROSS_COMPILE	?= $(CONFIG_CROSS_COMPILE:"%"=%)  
	�޸�Ϊ��  
	ARCH		?= arm  
	CROSS_COMPILE	?= arm-eabi-  
3. ��ʼ���룺��������������Ҫ���������ã������ҵ�androidԴ��汾ʹ�õ���5.1.1����Ҫ������ִ�����²�����  
*make goldfish_armv7_defconfig*  
*make*  
����ɹ���ῴ�ġ�Kernel: arch/arm/boot/zImage is ready��  
### ��ģ���������б���õ��ں�  
1. ����ģ������ǰ���Ѿ�˵����ִ��emulator����ִ�У���Ҫ��ִ�����²�����  
*source build/envsetup.sh*  
*lunch 1*  
*����ģ������ָ��ʹ�õ��ں��ļ���emulator -kernel ./kernel/goldfish/arch/arm/boot/zImage &*  
2.ʹ��adb��������ģ�������鿴�ں���Ϣ��һ��ִ�����в�����  
*����ģ������adb shell*  
*����ģ������procĿ¼��cd proc*  
*�鿴�ں���Ϣ��cat version*  
��ͨ���ں���Ϣ����ģ����ʹ�õ��ں��ǲ������Ǳ���������ں�  
##�������⣺  
* ������빤���������õ�ԭ��  
ARCH ?= arm �����ñ�����ϵ�ṹΪarm  
CROSS_COMPILE     ?= arm-eabi-   ���ý�����빤�ߵ�ǰ׺����ʹ��gcc���룬���ջ��ҵ�prebuilts/gcc/linux-x86/arm/arm-eabi-4.8/binĿ¼�µ�arm-eabi-gcc  
* ���ں�Ŀ¼��ʹ��make������б��룬�������ɶ��  
---
**���ﶼ��һЩ�ʼǣ�����֪ʶ���ޣ��϶��д���ĵط���ϣ����Ҫ���Ķ����ĵ�ͬѧ������д���ĵط�����ϣ��ָ��**