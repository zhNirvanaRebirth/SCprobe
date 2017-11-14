/*
	content��Androidϵͳʹ�õ��ں��ַ�����
	author��zhwilson
*/
//--------------------------step--------------------------
/*
	�ҵ�AndroidԴ��Ŀ¼��/home/zhwilson/Android/aosp/
*/
1����kernel/goldfish/driversĿ¼�´�����Ӧ�ļ���
2���ڸ��ļ����±�дԴ��

//--------------------------structure and method--------------------------
//�豸�ṹ
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

//�豸�ż������ӱ��
�豸��:struct dev_t dev_t;
�����:int major;
�ӱ��:int minor;
	�����豸�Ż�ȡ�����ӱ��
		MAJOR(dev_t);//�����
		MINOR(dev_t);//�ӱ��
	���������ӱ�Ż�ȡ�豸��
		MKDEV(major, minor);
������ͷ��豸���:
	/*
		�����豸��ţ�֪����ʼ�豸�ţ�
		first:��ʼ���豸�ţ���α�ų�����0��
		count:����������豸��ŵ�����
		name:���ӵ������ŷ�Χ���豸�����֣����������/proc/devices/��/sys/fs/�ļ��У�
		return:�ɹ�����0, ���󷵻ظ�ֵ
	*/
	int register_chrdev_region(dev_t first, unsigned int count, char *name);
	
	/*
		��̬�����豸��ţ���֪����ʼ�豸�ţ�
		dev:����ɹ��ĵ�һ���豸��
		firstminor:��һ���豸�ŵĴα�ţ�������0��
		count:����������豸��ŵ�����
		name:���ӵ������ŷ�Χ���豸�����֣����������/proc/devices/��/sys/fs/�ļ��У�
		return:�ɹ�����0, ���󷵻ظ�ֵ
	*/
	int alloc_chrdev_region(dev_t *dev, unsigned int firstminor, unsigned int count, char *name);
	
	/*
		�ͷ��豸���
		first:����ɹ��ĵ�һ���豸��
		count:����������豸��ŵ�����
	*/
	void unregister_chrdev_region(dev_t first, unsigned int count);
	
//�ļ�����(����һЩ����ʹ�õ����Ժͷ���)
struct file_operation {
	struct module *owner;//ӵ������ṹ��ģ���ָ��
	ssize_t (*read)(struct file *file, char __user *buf, size_t size, loff_t *f_pos);//���豸��ȡ���ݣ�ʧ�ܷ��ظ�ֵ
	ssize_t (*write)(struct file *file, const char __user *buf, size_t size, loff_t *f_pos);//���豸д���ݣ�ʧ�ܷ��ظ�ֵ
	int (*open)(struct inode *inode, struct file *filep);//���ļ�
	int (*release)(struct inode *inode, struct file *filep);//�ͷ��ļ�
}

//�ļ��ṹ
struct file {
	mode_t f_mode;//�ļ�ģʽ��ȷ���ļ��ǿɶ����ǿ�д�ģ�FMODE_READ, FMODE_WRITE��
	loff_t f_pos;//��ǰ�ļ���дλ��
	unsigned int f_flags;//�ļ���־��O_RDONLY, O_NONBLOCK, O_SYNC��
	struct file_operation *f_op;//�ļ�����
	void *private_data;//˽�г�Ա
	struct dentry *f_dentry;//Ŀ¼��ڽṹ
}

//inode�ṹ
struct inode {
	struct dev_t dev_t;//�豸��
	struct cdev cdev;//�ַ��豸
}

//�豸ע��
1������
	��ȡһ��������cdev��
		struct cdev zhwilson_cdev = cdev_alloc();
		zhwilson_cdev->ops = &zhwilson_fops;
	��cdevǶ���Լ��Ľṹ:
		void cdev_init(struct cdev *cdev, struct file_operations *fops);
2�����
	int cdev_add(struct cdev *cdev, dev_t first, unsigned int count);//�ɹ�����0��ʧ�ܷ��ظ�ֵ
ɾ���豸��void cdev_del(struct cdev *cdev);

//�������ز��輰����
1����̬�������豸�źʹ��豸��
2��Ϊ�豸�����ڴ�
3��ע���豸(������Ҫ����ļ�����������file_operations)
4����/sys/class/Ŀ¼�´����豸���Ŀ¼
5����/dev/Ŀ¼��/sys/class/�豸����/Ŀ¼�´����豸�ļ�
6����/sys/class/�豸����/�豸�ļ�/Ŀ¼�´��������ļ�
7���洢������Ҫ�õ���˽�����ݣ�dev_set_drvdata
8����/proc/Ŀ¼�´����豸�ļ�

//����ж�ش���
1�����豸��ŷ���ɹ��������ͷ�
2�����豸�ڴ��ѷ��䣬�����ͷ�
3�����豸��ע�ᣬɾ���豸
4�����豸����Ѵ�����ɾ�����豸���
5�����豸�ļ��Ѵ�����ɾ���豸�ļ�
//--------------------------code--------------------------

//ͷ�ļ�
#ifndef _ZHWILSON_INFO_ANDROID_H_
#define _ZHWILSON_INFO_ANDROID_H_

#include<linux/cdev.h>
#include<linux.semaphore.h>

//������Ҫʹ�õ�����
#define ZHWILSON_INFO_NODE_NAME "zhwilson_info"     //�豸�ڵ�����
#define ZHWILSON_INFO_CLASS_NAME "zhwilson_info"     //�豸���Ŀ¼����
#define ZHWILSON_INFO_FILE_NAME "zhwilson_info"     //�豸�ļ�����

//�豸�ṹ
struct zhwilson_dev {
	int age;
	struct semaphore sema;
	struct cdev cdev;
};

#endif

//Դ����
#include<linux/init.h>
#include<linux/module.h>
#include<linux/cdev.h>
#include<linux/errno.h>
#include "info.h"

//���
MODULE_LICENSE("GPL");

int info_major = 0;
int info_minor = 0;

//�豸����
static struct zhwilson_dev *zhwilson_info_dev = NULL;
//�豸������
static struct class *zhwilson_info_class = NULL;

//�豸��������
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

//��ʼ���豸
static int __info_setup_dev(struct zhwilson_dev *zhwilson_info_dev){
	int err;
	dev_t devno = MKDEV(info_major, info_minor);
	memset(zhwilson_info_dev, 0, sizeof(struct zhwilson_dev));
	//��ʼ��cdev
	cdev_init(&(zhwilson_info_dev->cdev), &zhwilson_info_fops);
	zhwilson_info_dev->cdev.owner = THIS_MODULE;
	zhwilson_info_dev->cdev.ops = &zhwilson_info_fops;
	//ע���豸
	err = cdev_add(&(zhwilson_info_dev->cdev), devno, 1);
	if (err) return err;
	//��ʼ���豸�ź���
	init_MUTEX(&(zhwilson_info_dev->sema));
	//��ʼ���豸����
	zhwilson_info_dev->age = 21;
	
	return 0;
}

//ģ�����
static int __init info_init(void){
	int err = -1;
	dev_t devno = 0;
	struct device *temp = NULL;
	printk(KERN_ALERT "Initializing zhwilson info char dev ...\n");
	
	//��̬�����豸��
	err = alloc_chrdev_region(&devno, 0, 1, ZHWILSON_INFO_NODE_NAME);
	if (err < 0){
		printk(KERN_ALERT "Failed to alloc char dev region!\n");
		goto fail;
	}
	//ͨ���豸�Ż�ȡ�����豸���
	info_major = MAJOR(devno);
	info_minor = MINOR(devno);
	
	//Ϊ�豸�����ڴ�
	zhwilson_info_dev = kmalloc(sizeof(struct zhwilson_dev), GFP_KERNEL);
	if (!zhwilson_info_dev){
		err = -ENOMEM;
		printk(KERN_ALERT "Failed to alloc zhwilson_dev!\n");
		goto unregister;
	}
	
	//ע���豸
	err = __info_setup_dev(zhwilson_info_dev);//zhwilson_info_dev�Ѿ���һ��ָ�������
	if(err) {
		printk(KERN_ALERT "Failed to setup zhwilson_dev!\n");
		goto cleanup;
	}
	
	//��/sys/class/Ŀ¼�´����豸����Ŀ¼
	zhwilson_info_class = class_create(THIS_MODULE, ZHWILSON_INFO_CLASS_NAME);
	if (IS_ERR(zhwilson_info_class)){
		err = PTR_ERR(zhwilson_info_class);
		printk(KERN_ALERT "Failed to create zhwilson_info class!\n");
		goto destory_cdev;
	}
	
	//��/dev/Ŀ¼��/sys/class/�豸��������/Ŀ¼�´����豸�ļ�
	temp = device_create(zhwilson_info_class, NULL, devno, "%s", ZHWILSON_INFO_FILE_NAME);
	if(IS_ERR(temp)){
		err = PTR_ERR(temp);
		printk(KERN_ALERT "Failed to create zhwilson_info device!\n");
		goto destory_class;
	}
	
	//��/sys/class/�豸����/�豸�ļ�/Ŀ¼�´��������ļ�
	err = device_create_file(temp, &dev_attr_age);
	if (err < 0){
		printk(KERN_ALERT "Failed to create attribute age!\n");
		goto detory_device;
	}
	
	/*
		�洢������Ҫ�õ���˽������
		Դ�룺
			static inline void dev_set_drvdata(struct device *dev, void *data)  
			{  
				dev->driver_data = data;  
			} 
	*/
	dev_set_drvdata(temp, zhwilson_info_dev);
	
	//����/sys/�豸�ļ�
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

//ģ��ж��
static void __exit info_exit(void){
	dev_t devno = MKDEV(info_major, info_minor);
	printk(KERN_ALERT "Destory zhwilson_info device.\n");
	//ɾ��/proc/�豸�ļ�
	zhwilson_info_destory_proc();
	
	//ɾ���豸�����豸
	if (zhwilson_info_class){
		device_destory(zhwilson_info_class, devno);
		class_destory(zhwilson_info_class);
	}
	
	//ɾ���ַ��豸���ͷ��ڴ�
	if (zhwilson_info_dev){
		cdev_del(&(zhwilson_info_dev->cdev));
		kfree(zhwilson_info_dev);
	}
	
	//�ͷ��豸��
	unregister_chrdev_region(devno, 1);
}

module_init(info_init);
module_exit(info_exit);
