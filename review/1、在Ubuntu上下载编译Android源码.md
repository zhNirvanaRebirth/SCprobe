**֪ʶ�㣺��Ubuntu�����ء�����Ͱ�װAndroidԴ��  
֪ʶ��Դ�����޲���-[��Ubuntu�����ء�����Ͱ�װAndroid����Դ����](http://blog.csdn.net/luoshengyang/article/details/6559955)  
��¼���ݣ���ʵ�ʲ���������������ʹ�����**  
***  
## ���裺  
### ����׼��  
1. ����׼��������׼����һ�㣬���ղ����ϵĿ϶��ǲ����ˣ������ص���Դ����Ǽ�ʮ��G��֮���㻹Ҫ���룬�ҷ�����300G�Ĵ��̣��ڱ������֮�󣬿�������ռ��180��G�����Ը��˽��黹�Ǿ�����һ�㣬�ڴ��4G���ϰɣ�����������һЩ����Ȼ��vmware���ڴ�ʹ��̴�С���ǿ��Ըĵģ�����Ҫ���㶼������Դ���ˣ��ŷ��ִ��̲��������㲻���ֵ���������ô  
2. ��װvmware������ͱȽϼ��ˣ������Լ����ԵĴ�����λ������������һ����װ�þ���  
3. Ubuntuϵͳ��װ��������ʹ�õ�ubuntuϵͳ�İ汾��14.04����ʼʹ�õ���16.04������������ninja: no work to do�������⣬���Ծͽ�ϵͳ�ĳ���14.04  
4. ��װgit���ߣ���Ϊ������Ҫ����androidԴ�룬��android����ʹ��git���߹���ģ���ubuntu��ִ�����������ɣ�  
*sudo apt-get install git-core gnupg*  
5. JDK��װ�������Ұ�װ���޵�����ִ�У��ڱ����������ʾ��˵��Ҫopenjdk�������������ֱ�Ӱ�װopenjdk�ɣ���ubuntu��ִ�����������ɣ�[���](http://www.jianshu.com/p/aeaceda41798)  
*sudo apt-get install openjdk-7-jre*  
*apt-get install openjdk-7-jdk*  
6. ��װ��ص�������������ֻ��һ���֣�����������������Ҫ�����������ٰ�װ���ǣ���ubuntu��ִ�����������ɣ�  
*sudo apt-get install flex bison gperf libsdl-dev libesd0-dev libwxgtk2.6-dev build-essential zip curl*  
### ����AndroidԴ�빤��  
**��Ҫ��ѧ�����������������Ҫʹ���廪��ѧ�ľ�������ʹ�õ��廪��ѧ�ľ���[���](http://www.jianshu.com/p/aeaceda41798)**  
1. ����repo���ߣ���ubuntu������ִ�����������ɣ�
*wget https://dl-ssl.google.com/dl/googlesource/git-repo/repo*  
*chmod 777 repo*  
*cp repo /bin/*  
2. �޸�/bin/repo�ļ�����Ϊ/bin/repo������ļ����޷�ֱ���޸ĵģ�����������Ҫ���㸴�Ƶ���һ���ط���Ȼ���޸ģ��ٸ���/bin/repo���޸��������£�  
`REPO_URL = 'https://gerrit.googlesource.com/git-repo'  
�޸�Ϊ��  
REPO_URL = 'https://gerrit-google.tuna.tsinghua.edu.cn/git-repo'`  
3. ����Android��Դ���룺��ubuntu������ִ�����������ɣ�  
*mkdir Android*  
*cd Android*  
*wget -c -t 0 https://mirrors.tuna.tsinghua.edu.cn/aosp-monthly/aosp-latest.tar*  
*tar -vxzf aosp-latest.tar*  
����aospĿ¼  
*repo sync*  
Ȼ��ѡ��汾ͬ��  
*repo init -u https://aosp.tuna.tsinghua.edu.cn/platform/manifest -b android-5.1.1_r1*  
*repo sync*  
_������Ҫ���������ĵȴ��������ٺ͵������õȸ���Ӱ�������£��ҵ������죡����;ʧ�ܣ�����ִ�С�repo sync������_  
### ����AndroidԴ��  
1. �������洴����AndroidĿ¼�����ǵ�Դ������������ִ�����  
*make*  
2. ���������⣺  
* *ninja: no work to do*  
�����ᵽ�����ҿ�ʼװ��ubuntuϵͳ��16.04�İ汾������ĳ�14.04�İ汾��ok��  
* *... asked for an OpenJDK ...*  
������Ҫopenjdk���������ﰴ�������İ�װ��Ӧ�汾��openjdk��ok��  
* *����*  
����ʲô��������ѽ�����̷�������ѽ�������ܶ࣬���Լ����Ҵ�����  
### ��װ����õ�Android����ģ������  
1. ���û���������������Ҫ��emulator�����Լ�Android����Ŀ¼����Ϊ���������������Ҳ��õ����޸�/etc/profile�ļ����ҵ�Android����Ŀ¼����home/zhwilson/Android/aosp/out/target/product/generic����emulatorĿ¼����home/zhwilson/Android/aosp/prebuilts/android-emulator/linux-x86_64�����������ڡ�home/zhwilson/Android/aosp/out/host/linux-x86/bin��Ŀ¼�£���������/etc/profile�ļ�����������´��룺  
*export PATH=$PATH:home/zhwilson/Android/aosp/prebuilts/android-emulator/linux-x86_64:home/zhwilson/Android/aosp/out/target/product/generic*  
2. ����ģ����������ֱ������emulator�������ָ������⣬�Һ��淢�ֻ����������»�������û�����õ��µģ��鿴�����������env | grep PATH��������������Ҫ��ִ��emulator��ִ���������  
*source build/envsetup.sh*  
*lunch*  
*1(�����������Ҫ��ʲô�ܹ���ģ���������о���)*  
��ʱ���ٲ鿴�����������ᷢ�ֶ��˺ܶ���뼰��װAndroidԴ��Ļ�����������ʱִ������emulatorʱ��ģ�����Ϳ�ʼ������  
3. ���������⣺  
*emulator: command not found*  
����emulator��������·�����ô��ˣ��������ᵽ  
*�����������ú�֮������emulator���ǳ��ָ��ִ���*  
�������û�����������ڶ����ᵽ�����������Щ�������ʲô����û��ȥ�о��������ٿ���[���](http://blog.csdn.net/u011913612/article/details/51878356)  
---
**���ﶼ��һЩ�ʼǣ�����֪ʶ���ޣ��϶��д���ĵط���ϣ����Ҫ���Ķ����ĵ�ͬѧ������д���ĵط�����ϣ��ָ��**