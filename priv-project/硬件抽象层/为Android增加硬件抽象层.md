/*
	content��ΪAndroid����Ӳ������㣨HAL��ģ�飬���ڷ���linux�ں������������ϲ��ṩ�����ں˵Ľӿ�
	author��zhwilson
*/

/*
	AndroidӲ�������淶��
		1������ģ��ID
		2������Ӳ��ģ��ṹ��
		3������Ӳ���ӿڽṹ��
		
	�ҵ�Android�ں�Ŀ¼��/home/zhwilson/Android/aosp/
*/

//---------------------------step---------------------------
1������ͷ�ļ�������ģ��ID��Ӳ��ģ��ṹ���Ӳ���ӿڽṹ�壨��~/hardware/libhardware/include/hardware/Ŀ¼�´���ͷ�ļ���
2������ģ��Ŀ¼�Լ�Դ�ļ�����~/hardware/libhardware/modules/Ŀ¼�´���ģ��Ŀ¼��
3���߼�ʵ��
	3.1������ģ�鷽�����Լ���Ӧ����(�豸�򿪡��رյ�)
	3.2������ģ��ʵ������
	3.3������ʵ��
	
ע�������Լ��������豸�ļ�����ֻ��root�ɶ�д�������������뵽system/core/rootdirĿ¼��������һ����Ϊueventd.rc�ļ������������һ�У��������豸�ļ���/dev/hello��
      /dev/hello 0666 root root
//---------------------------code---------------------------
//ͷ�ļ�
#ifndef ANDROID_INFO_INTEFACE_H
#define ANDROID_INFO_INTEFACE_H
#include<hardware/hardware.h>

__BEGIN_DECLS

//����ģ��ID
#define INFO_HAREWARE_MODULE_ID "zhwilson_info"

//����ģ��ṹ��
struct info_module_t {
	struct hw_module_t common;
};

//����ӿڽṹ��
struct info_device_t {
	struct hw_device_t common;
	int fd;//�豸�ļ�����������Ӧ����Ҫ������豸�ļ����硰dev/info��
	//�����ṩ�Ľӿڷ���
	int (*set_age)(struct info_device_t *dev, int age);
	int (*get_age)(struct info_device_t *dev, int *age);
};

__END_DECLS

#endif

//Դ�ļ�
#include<hardware/hardware.h>
#include<hardware/info.h>//�Զ����ͷ�ļ�
#include<fcntl.h>
#include<errno.h>
#include<cutils/log.h>
#define MODULE_NAME "Persion_info"
#define DEVICE_NAME "/dev/info"
#define MODULE_AUTHOR "zhwilson"

//ģ�鷽��
static int info_device_open(const struct hw_module_t *module, const char *name, struct hw_device_t **device);
static int info_device_close(struct hw_device_t *device);

//�豸���ʷ���
static int info_set_age(struct info_device_t *dev, int age);
static itn info_get_age(struct info_device_t *dev, int *age);

//ģ�鷽����
static struct hw_module_methods_t info_module_methods = {
	open:info_device_open
};

//ģ��ʵ������(���ֱ����ǣ�HAL_MODULE_INFO_SYM�� tag�����ǣ�HARDWARE_MODULE_TAG)
struct info_module_t HAL_MODULE_INFO_SYM = {
	common: {
		tag: HARDWARE_MODULE_TAG,
		version_major:1,
		version_minor:0,
		id:INFO_HAREWARE_MODULE_ID,
		name:MODULE_NAME,
		author:MODULE_AUTHOR,
		methods:&info_module_methods,
	}
};

//ģ�鷽��ʵ��
/*
	open������
		1��Ϊ�ӿڽṹ������ڴ�
		2����ֵ�ӿڽṹ�������뷽����common��fd�� set_age��get_age��
*/
static int info_device_open(const struct hw_module_t *module, const char *name, struct hw_device_t **device){
	struct info_device_t *dev;
	dev = (struct info_device_t *)malloc(sizeof(struct info_device_t));
	if (!dev){
		LOGE("zhwilson:Failed to alloc space");
		return -EFAULT;
	}
	dev->common.tag = HARDWARE_DEVICE_TAG;
	dev->common.varsion = 0;
	dev->common.module = (hw_module_t *)module;
	dev->common.close = info_device_close;
	dev->set_age = info_set_age;
	dev->get_age = info_get_age;
	if((dev->fd = open(DEVICE_NAME, O_RDWR)) == -1){
		SLOGE("zhwilson:Failed to open %s with %s.\n", DEVICE_NAME, strerror(errno));
		free(dev);
		return -EFAULT;
	}
	*device = &(dev->common);
	SLOGI("zhwilson:open %s successfully.\n", DEVICE_NAME);
	return 0;
}

static int info_device_close(struct hw_device_t *device){
	struct info_device_t *dev = (struct info_device_t *)device;
	if (dev){
		close(dev->fd);
		free(dev);
	}
	
	return 0;
}

static int info_set_age(struct info_device_t *device, int age){
	SLOGI("zhwilson:set age %d to device.\n", age);
	write(device->fd, &age, sizeof(age));
	return 0;
}

static int info_get_age(struct info_device_t *device, int *age){
	if (!age) {
		SLOGE("zhwilson:error age pointer.\n");
		return -EFAULT;
	}
	read(device->fd, age, sizeod(*age));
	SLOGI("zhwilson:get age %d from device.\n", *age);
	return 0;
}