��Androidϵͳ������C��ִ�г���������Linux��������
1����externalĿ¼�´���cԴ����Ŀ¼hello��make hello
2����д�����ں����������cԴ����
3����external/hello�ļ����´���Android.mk�ļ�������Android.mk�ļ�
4������home/zhwilson/Android/aosp/Ŀ¼
5��ʹ��mmm��������helloģ�飺mmm ./external/hello ������֮ǰִ�����source bulid/envsetup.sh�� ������ɣ�������out/target/product/gerneric/system/binĿ¼�£�������ִ���ļ�hello��
6�����´��Androidϵͳ�ļ�system.img��make snod���ɹ���hello��ִ���ļ��Ͱ�����system.img�ļ����ˣ�
7������ģ������emulator -kernel ./kernel/goldfish/arch/arm/boot/zImage &������֮ǰִ�����lunch��
8��ִ��hello��ִ�г����������ں�������ʹ��adb shellȻ�����system/binִ��./hello


ע�����е����Ŀ¼����home/zhwilson/Android/aosp/Ŀ¼��











���޲��ͣ���Ubuntu��ΪAndroidϵͳ����C��ִ�г������Linux�ں���������http://blog.csdn.net/luoshengyang/article/details/6571210��
note��
���������ײ�����ôʵ�ֵģ�
1��open(DEVICE_NAME, O_RDWR);
2��read(fd, &val, sizeof(val));
�ļ���
1��Android.mk�ļ�����Ҫ��ô���ã���ʾʲô��˼��