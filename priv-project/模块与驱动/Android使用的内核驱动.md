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
		name:链接到这个编号范围的设备的名字（它会出现在/proc/devices和/sysfs文件中）
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

//proc文件系统
struct proc_dir_entry {
	const char *name;//virtual file name
	mode_t mode;//mode permission
	uid_t uid;//file's user id
	gid_t gid;//file's group id
	struct inode_operations *proc_iops;//inode operation functions
	struct file_operations *proc_fops;//file operation functions
	struct proc_dir_entry *parent;//parent directory
	read_proc_t *read_proc;//proc read function
	write_proc_t *write_proc;//proc write function
	void *data;//pointer to private data
	atomic_t count;//use count
}

创建一个虚拟文件：struct proc_dir_entry *create_proc_entry(const char *name, mode_t mode, struct proc_dir_entry *parent);//成功返回proc_dir_entry,失败返回NULL
删除一个虚拟文件：void remove_proc_entry(const char *name, struct proc_dir_entry *parent);
向虚拟文件中写入数据：
/*
	filp 参数实际上是一个打开文件结构（我们可以忽略这个参数）。buff 参数是传递给您的字符串数据。缓冲区地址实际上是一个用户空间的缓冲区，因此我们不能直接读取它。len 参数定义了在 buff 中有多少数据要被写入。data 参数是一个指向私有数据的指针（参见 清单 7）。在这个模块中，我们声明了一个这种类型的函数来处理到达的数据。
	Linux 提供了一组 API 来在用户空间和内核空间之间移动数据。对于 write_proc 的情况来说，我们使用了 copy_from_user 函数来维护用户空间的数据。
*/
int mod_write(struct file *filp, const char __user *buf, unsigned long len, void *data);
从虚拟文件中读取数据：
/*
	page 参数是这些数据写入到的位置，其中 count 定义了可以写入的最大字符数。在返回多页数据（通常一页是 4KB）时，我们需要使用 start 和 off 参数。当所有数据全部写入之后，就需要设置 eof（文件结束参数）。与 write 类似，data 表示的也是私有数据。此处提供的 page 缓冲区在内核空间中。因此，我们可以直接写入，而不用调用 copy_to_user。
*/
int mod_read(char *page, char **start, off_t off, int count, int *eof, void *data);

//驱动加载步骤及方法
1、动态分配主设备号和从设备号
2、为设备分配内存
3、注册设备(这里需要设计文件操作，即：file_operations)
4、在/sys/class/目录下创建设备类别目录
5、在/dev/目录和/sys/class/设备类型/目录下创建设备文件
6、在/sys/class/设备类型/设备文件/目录下创建属性文件
7、存储驱动中要用到的私有数据：dev_set_drvdata
8、在/proc/目录下创建设备文件以及相关操作
9、定义属性并设置属性的访问方法

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
#define ZHWILSON_INFO_PROC_NAME "zhwilson_info"     //设备在/proc/目录下的名字

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
#include<linux/proc_fs.h>
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
static int zhwilson_info_open(struct inode *inode, struct file *filp);
static int zhwilson_info_release(struct inode *inode, struct file *fiep);
static ssize_t zhwilson_info_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos);
static ssize_t zhwilson_info_write(struct file *fiep, const char __user *buf, size_t count, loff_t *f_pos);

static struct file_operations zhwilson_info_fops = {
	.owner = THIS_MODULE,
	.open = zhwilson_info_open,
	.release = zhwilson_info_release,
	.read = zhwilson_info_read,
	.write = zhwilson_info_write,
};

//访问属性的方法
static ssize_t zhwilson_info_show(struct device *dev, struct device_attribute *attr, char *buf);
static ssize_t zhwilson_info_store(struct device *dev, struct device_attribute *attr, const char *buf, size_t count);

//定义设备属性
static DEVICE_ATTR(age, S_IRUGO|S_IUSR, zhwilson_info_show, zhwilson_info_store);

static int zhwilson_info_open(struct inode *inode, struct file *filp){
	struct zhwilson_dev *dev;
	dev = container_of(inode->cdev, struct zhwilson_dev, dev);
	filp->private_data = dev;
	return 0;
}

static int zhwilson_info_release(struct inode *inode, struct file *filp){
	return 0;
}

static ssize_t zhwilson_info_read(struct file *filp, char __user *buf, size_t count, loff_t *f_pos){
	//TODO
}

static ssize_t zhwilson_info_write(struct file *filp, const char __user *buf, size_t count, loff_t *f_pos){
	//TODO
}


static ssize_t __zhwilson_info_get_info(struct zhwilson_dev *dev, char *buf){
	int age = 0;
	if (down_interruptible(&(dev->sema))){
		return -ERESTARTSYS;
	}
	age = dev->age;
	up(&(dev->sema));
	return snprintf(bug, PAGE_SIZE, "%d\n", age);
}

static ssize_t zhwilson_info_set_info(struct zhwilson_dev *dev, const char *buf, size_t count){
	int age = 0;
	//字符串转数字
	age = simple_strtol(bug, NULL, 10);
	if(down_interruptible(&(dev->sema))) return -ERESTARTSYS;
	dev->age = age;
	up(&(dev->sema));
	return count;
}

static ssize_t zhwilson_info_show(struct device *dev, struct device_attribute *attr, char *buf){
	struct zhwilson_dev *info_dev = (struct zhwilson_dev *)dev_get_drvdata(dev);
	return __zhwilson_info_get_info(info_dev, buf);
}

static ssize_t zhwilson_info_store(struct device *dev, struct device_attribute *attr, const char *buf, size_t count){
	struct zhwilson_dev *info_dev = (struct zhwilson_dev *)dev_get_drvdata(dev);
	return __zhwilson_info_set_info(info_dev, buf, count);
}

//虚拟文件读取
static ssize_t zhwilson_info_proc_read(char *page, char **start, off_t off, int count, int *eof, void *data){
	if (off > 0){
		eof = 1;
		return 0;
	}
	
	return __zhwilson_info_get_info(zhwilson_info_dev, page);
}

//虚拟文件数据写入
static ssize_t zhwilson_info_proc_write(struct file *filp, const char __user *buf, unsigned long len, void *data){
	int err = 0;
	char *page = NULL;
	if (len > PAGE_SIZE) {
		printk(KERN_ALERT "The buff is too large:%lu.\n", len);
		return -EFAULT;
	}
	page = (char *)__get_free_page(GFP_KERNEL);
	if (!page){
		printk(KERN_ALERT "Failed to alloc page.\n");
		return -ENOMEM;
	}
	//把数据从缓冲区拷贝到内核缓冲区
	if (copy_from_user(page, buf, len)){
		printk(KERN_ALERT "Failed to copy buff from user.\n");
		err = -EFAULT;
		goto out;
	}
	
	err = __zhwilson_info_set_info(zhwilson_info_dev, page, len);
	
	out:
		free_page((unsigned long)page);
	return err;
}

//在/proc目录下创建虚拟文件
static void zhwilson_info_create_proc(void){
	struct proc_dir_entry *entry;
	entry = create_proc_entry(ZHWILSON_INFO_PROC_NAME, 0, NULL);
	if (entry){
		entry->owner = THIS_MODULE;
		entry->read_proc = zhwilson_info_proc_read;
		entry->write_proc = zhwilson_info_proc_write;
	}
}
//删除虚拟文件
static void zhwilson_info_destory_proc(void){
	remove_proc_entry(ZHWILSON_INFO_PROC_NAME, NULL);
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
