/*
	content：Android系统使用的内核字符驱动
	author：zhwilson
*/
//--------------------------step--------------------------
/*
	我的Android源码目录：/home/zhwilson/Android/aosp/
*/
1、在kernel/goldfish/drivers目录下创建对应文件夹
2、在该文件夹下编写源码

//--------------------------structure and method--------------------------
//设备结构
struct zhwilson_dev {
	struct zhwilson_qset *data;//pointer to first quantum set
	int quantum;//current quantum size
	int qset;//current qset size;
	unsigned long size;//amount of data stored here
	struct semaphore sem;//mutual exclusion semaphore
	struct cdev cdev;//char device structure
	...//something more if need
}

//zhwilson qset
struct zhwilson_qset {
	void **data;//pointer to a list whitch is contain elements, the list size is qset, the element size is quantum
	struct zhwilson_qset *next;//next zhwilson_qset
}

//设备号及主、从编号
设备号:struct dev_t dev_t;
主编号:int major;
从编号:int minor;
	根据设备号获取主、从编号
		MAJOR(dev_t);//主编号
		MINOR(dev_t);//从编号
	根据主、从编号获取设备号
		MKDEV(major, minor);
分配和释放设备编号:
	/*
		分配设备编号（知道开始设备号）
		first:开始的设备号（其次编号常常是0）
		count:请求的连续设备编号的总数
		name:链接到这个编号范围的设备的名字（它会出现在/proc/devices/和/sys/fs/文件中）
		return:成功返回0, 错误返回负值
	*/
	int register_chrdev_region(dev_t first, unsigned int count, char *name);
	
	/*
		动态分配设备编号（不知道开始设备号）
		dev:分配成功的第一个设备号
		firstminor:第一个设备号的次编号（常常是0）
		count:请求的连续设备编号的总数
		name:链接到这个编号范围的设备的名字（它会出现在/proc/devices/和/sys/fs/文件中）
		return:成功返回0, 错误返回负值
	*/
	int alloc_chrdev_region(dev_t *dev, unsigned int firstminor, unsigned int count, char *name);
	
	/*
		释放设备编号
		first:分配成功的第一个设备号
		count:请求的连续设备编号的总数
	*/
	void unregister_chrdev_region(dev_t first, unsigned int count);
	
//文件操作(仅限一些驱动使用的属性和方法)
struct file_operation {
	struct module *owner;//拥有这个结构的模块的指针
	ssize_t (*read)(struct file *file, char __user *buf, size_t size, loff_t *f_pos);//从设备读取数据，失败返回负值
	ssize_t (*write)(struct file *file, const char __user *buf, size_t size, loff_t *f_pos);//向设备写数据，失败返回负值
	int (*open)(struct inode *inode, struct file *filep);//打开文件
	int (*release)(struct inode *inode, struct file *filep);//释放文件
}

//文件结构
struct file {
	mode_t f_mode;//文件模式，确定文件是可读还是可写的（FMODE_READ, FMODE_WRITE）
	loff_t f_pos;//当前文件读写位置
	unsigned int f_flags;//文件标志（O_RDONLY, O_NONBLOCK, O_SYNC）
	struct file_operation *f_op;//文件操作
	void *private_data;//私有成员
	struct dentry *f_dentry;//目录入口结构
}

//inode结构
struct inode {
	struct dev_t dev_t;//设备号
	struct cdev cdev;//字符设备
}

//设备注册
1、分配
	获取一个独立的cdev：
		struct cdev zhwilson_cdev = cdev_alloc();
		zhwilson_cdev->ops = &zhwilson_fops;
	将cdev嵌入自己的结构:
		void cdev_init(struct cdev *cdev, struct file_operations *fops);
2、添加
	int cdev_add(struct cdev *cdev, dev_t first, unsigned int count);//成功返回0，失败返回负值
删除设备：void cdev_del(struct cdev *cdev);

//驱动加载步骤及方法
1、动态分配主设备号和从设备号
2、为设备分配内存
3、注册设备(这里需要设计文件操作，即：file_operations)
4、在/sys/class/目录下创建设备类别目录
5、在/dev/目录和/sys/class/设备类型/目录下创建设备文件
6、在/sys/class/设备类型/设备文件/目录下创建属性文件
7、存储驱动中要用到的私有数据：dev_set_drvdata
8、在/proc/目录下创建设备文件

//驱动卸载处理
1、若设备编号分配成功，将其释放
2、若设备内存已分配，将其释放
3、若设备已注册，删除设备
4、若设备类别已创建，删除该设备类别
5、若设备文件已创建，删除设备文件
//--------------------------code--------------------------

//头文件
#ifndef _ZHWILSON_INFO_ANDROID_H_
#define _ZHWILSON_INFO_ANDROID_H_

#include<linux/cdev.h>
#include<linux.semaphore.h>

//几个将要使用的名字
#define ZHWILSON_INFO_NODE_NAME "zhwilson_info"     //设备节点名字
#define ZHWILSON_INFO_CLASS_NAME "zhwilson_info"     //设备类别目录名字
#define ZHWILSON_INFO_FILE_NAME "zhwilson_info"     //设备文件名字

//设备结构
struct zhwilson_dev {
	int age;
	struct semaphore sema;
	struct cdev cdev;
};

#endif

//源代码
#include<linux/init.h>
#include<linux/module.h>
#include<linux/cdev.h>
#include<linux/errno.h>
#include "info.h"

//许可
MODULE_LICENSE("GPL");

int info_major = 0;
int info_minor = 0;

//设备变量
static struct zhwilson_dev *zhwilson_info_dev = NULL;
//设备类别变量
static struct class *zhwilson_info_class = NULL;

//设备操作方法
struct int zhwilson_info_open(struct inode *inode, struct file *filp);
struct int zhwilson_info_release(struct inode *inode, struct file *fiep);
struct ssize_t zhwilson_info_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos);
struct ssize_t zhwilson_info_write(struct file *fiep, const char __user *buf, size_t count, loff_t *f_pos);

static struct file_operations zhwilson_info_fops = {
	.owner = THIS_MODULE,
	.open = zhwilson_info_open,
	.release = zhwilson_info_release,
	.read = zhwilson_info_read,
	.write = zhwilson_info_write,
};

static void zhwilson_info_create_proc(void){
	//TODO
}
static void zhwilson_info_destory_proc(void){
	//TODO
}

//初始化设备
static int __info_setup_dev(struct zhwilson_dev *zhwilson_info_dev){
	int err;
	dev_t devno = MKDEV(info_major, info_minor);
	memset(zhwilson_info_dev, 0, sizeof(struct zhwilson_dev));
	//初始化cdev
	cdev_init(&(zhwilson_info_dev->cdev), &zhwilson_info_fops);
	zhwilson_info_dev->cdev.owner = THIS_MODULE;
	zhwilson_info_dev->cdev.ops = &zhwilson_info_fops;
	//注册设备
	err = cdev_add(&(zhwilson_info_dev->cdev), devno, 1);
	if (err) return err;
	//初始化设备信号量
	init_MUTEX(&(zhwilson_info_dev->sema));
	//初始化设备属性
	zhwilson_info_dev->age = 21;
	
	return 0;
}

//模块加载
static int __init info_init(void){
	int err = -1;
	dev_t devno = 0;
	struct device *temp = NULL;
	printk(KERN_ALERT "Initializing zhwilson info char dev ...\n");
	
	//动态分配设备号
	err = alloc_chrdev_region(&devno, 0, 1, ZHWILSON_INFO_NODE_NAME);
	if (err < 0){
		printk(KERN_ALERT "Failed to alloc char dev region!\n");
		goto fail;
	}
	//通过设备号获取主从设备编号
	info_major = MAJOR(devno);
	info_minor = MINOR(devno);
	
	//为设备分配内存
	zhwilson_info_dev = kmalloc(sizeof(struct zhwilson_dev), GFP_KERNEL);
	if (!zhwilson_info_dev){
		err = -ENOMEM;
		printk(KERN_ALERT "Failed to alloc zhwilson_dev!\n");
		goto unregister;
	}
	
	//注册设备
	err = __info_setup_dev(zhwilson_info_dev);//zhwilson_info_dev已经是一个指针变量了
	if(err) {
		printk(KERN_ALERT "Failed to setup zhwilson_dev!\n");
		goto cleanup;
	}
	
	//在/sys/class/目录下创建设备类型目录
	zhwilson_info_class = class_create(THIS_MODULE, ZHWILSON_INFO_CLASS_NAME);
	if (IS_ERR(zhwilson_info_class)){
		err = PTR_ERR(zhwilson_info_class);
		printk(KERN_ALERT "Failed to create zhwilson_info class!\n");
		goto destory_cdev;
	}
	
	//在/dev/目录和/sys/class/设备类型名称/目录下创建设备文件
	temp = device_create(zhwilson_info_class, NULL, devno, "%s", ZHWILSON_INFO_FILE_NAME);
	if(IS_ERR(temp)){
		err = PTR_ERR(temp);
		printk(KERN_ALERT "Failed to create zhwilson_info device!\n");
		goto destory_class;
	}
	
	//在/sys/class/设备类型/设备文件/目录下创建属性文件
	err = device_create_file(temp, &dev_attr_age);
	if (err < 0){
		printk(KERN_ALERT "Failed to create attribute age!\n");
		goto detory_device;
	}
	
	/*
		存储驱动中要用到的私有数据
		源码：
			static inline void dev_set_drvdata(struct device *dev, void *data)  
			{  
				dev->driver_data = data;  
			} 
	*/
	dev_set_drvdata(temp, zhwilson_info_dev);
	
	//创建/sys/设备文件
	zhwilson_info_create_proc();
	
	printk(KERN_ALERT "Succedded to initialize zhwilson_info device!\n");
		
	return 0;
	
	destory_device:
		device_destory(zhwilson_info_class, devno);
	destory_class:
		class_destory(zhwilson_info_class);
	destory_cdev:
		cdev_del(&(zhwilson_info_dev->cdev))
	cleanup:
		kfree(zhwilson_info_dev);
	unregister:
		unregister_chrdev_region(MKDEV(info_major, info_minor), 1);
	fail:
		return err;
}

//模块卸载
static void __exit info_exit(void){
	dev_t devno = MKDEV(info_major, info_minor);
	printk(KERN_ALERT "Destory zhwilson_info device.\n");
	//删除/proc/设备文件
	zhwilson_info_destory_proc();
	
	//删除设备类别和设备
	if (zhwilson_info_class){
		device_destory(zhwilson_info_class, devno);
		class_destory(zhwilson_info_class);
	}
	
	//删除字符设备、释放内存
	if (zhwilson_info_dev){
		cdev_del(&(zhwilson_info_dev->cdev));
		kfree(zhwilson_info_dev);
	}
	
	//释放设备号
	unregister_chrdev_region(devno, 1);
}

module_init(info_init);
module_exit(info_exit);
