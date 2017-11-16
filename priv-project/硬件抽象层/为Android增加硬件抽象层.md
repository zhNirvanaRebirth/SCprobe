/*
	content：为Android增加硬件抽象层（HAL）模块，用于访问linux内核驱动程序，向上层提供访问内核的接口
	author：zhwilson
*/

/*
	Android硬件抽象层规范：
		1、定义模块ID
		2、定义硬件模块结构体
		3、定义硬件接口结构体
		
	我的Android内核目录：/home/zhwilson/Android/aosp/
*/

//---------------------------step---------------------------
1、创建头文件，定义模块ID、硬件模块结构体和硬件接口结构体（在~/hardware/libhardware/include/hardware/目录下创建头文件）
2、创建模块目录以及源文件（在~/hardware/libhardware/modules/目录下创建模块目录）
3、逻辑实现
	3.1、创建模块方法表以及对应方法(设备打开、关闭等)
	3.2、定义模块实例变量
	3.3、方法实现
	
注：我们自己创建的设备文件可能只有root可读写，处理方法：进入到system/core/rootdir目录，里面有一个名为ueventd.rc文件，往里面添加一行：（假设设备文件是/dev/hello）
      /dev/hello 0666 root root
//---------------------------code---------------------------
//头文件
#ifndef ANDROID_INFO_INTEFACE_H
#define ANDROID_INFO_INTEFACE_H
#include<hardware/hardware.h>

__BEGIN_DECLS

//定义模块ID
#define INFO_HAREWARE_MODULE_ID "zhwilson_info"

//定义模块结构体
struct info_module_t {
	struct hw_module_t common;
};

//定义接口结构体
struct info_device_t {
	struct hw_device_t common;
	int fd;//设备文件描述符，对应我们要处理的设备文件，如“dev/info”
	//向上提供的接口方法
	int (*set_age)(struct info_device_t *dev, int age);
	int (*get_age)(struct info_device_t *dev, int *age);
};

__END_DECLS

#endif

//源文件
#include<hardware/hardware.h>
#include<hardware/info.h>//自定义的头文件
#include<fcntl.h>
#include<errno.h>
#include<cutils/log.h>
#define MODULE_NAME "Persion_info"
#define DEVICE_NAME "/dev/info"
#define MODULE_AUTHOR "zhwilson"

//模块方法
static int info_device_open(const struct hw_module_t *module, const char *name, struct hw_device_t **device);
static int info_device_close(struct hw_device_t *device);

//设备访问方法
static int info_set_age(struct info_device_t *dev, int age);
static itn info_get_age(struct info_device_t *dev, int *age);

//模块方法表
static struct hw_module_methods_t info_module_methods = {
	open:info_device_open
};

//模块实例变量(名字必须是：HAL_MODULE_INFO_SYM， tag必须是：HARDWARE_MODULE_TAG)
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

//模块方法实现
/*
	open方法：
		1、为接口结构体分配内存
		2、赋值接口结构体属性与方法（common，fd， set_age，get_age）
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