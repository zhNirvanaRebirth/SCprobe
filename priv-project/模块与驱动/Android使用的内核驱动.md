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
		name:���ӵ������ŷ�Χ���豸�����֣����������/proc/devices��/sysfs�ļ��У�
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

//proc�ļ�ϵͳ
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

����һ�������ļ���struct proc_dir_entry *create_proc_entry(const char *name, mode_t mode, struct proc_dir_entry *parent);//�ɹ�����proc_dir_entry,ʧ�ܷ���NULL
ɾ��һ�������ļ���void remove_proc_entry(const char *name, struct proc_dir_entry *parent);
�������ļ���д�����ݣ�
/*
	filp ����ʵ������һ�����ļ��ṹ�����ǿ��Ժ��������������buff �����Ǵ��ݸ������ַ������ݡ���������ַʵ������һ���û��ռ�Ļ�������������ǲ���ֱ�Ӷ�ȡ����len ������������ buff ���ж�������Ҫ��д�롣data ������һ��ָ��˽�����ݵ�ָ�루�μ� �嵥 7���������ģ���У�����������һ���������͵ĺ���������������ݡ�
	Linux �ṩ��һ�� API �����û��ռ���ں˿ռ�֮���ƶ����ݡ����� write_proc �������˵������ʹ���� copy_from_user ������ά���û��ռ�����ݡ�
*/
int mod_write(struct file *filp, const char __user *buf, unsigned long len, void *data);
�������ļ��ж�ȡ���ݣ�
/*
	page ��������Щ����д�뵽��λ�ã����� count �����˿���д�������ַ������ڷ��ض�ҳ���ݣ�ͨ��һҳ�� 4KB��ʱ��������Ҫʹ�� start �� off ����������������ȫ��д��֮�󣬾���Ҫ���� eof���ļ��������������� write ���ƣ�data ��ʾ��Ҳ��˽�����ݡ��˴��ṩ�� page ���������ں˿ռ��С���ˣ����ǿ���ֱ��д�룬�����õ��� copy_to_user��
*/
int mod_read(char *page, char **start, off_t off, int count, int *eof, void *data);

//�������ز��輰����
1����̬�������豸�źʹ��豸��
2��Ϊ�豸�����ڴ�
3��ע���豸(������Ҫ����ļ�����������file_operations)
4����/sys/class/Ŀ¼�´����豸���Ŀ¼
5����/dev/Ŀ¼��/sys/class/�豸����/Ŀ¼�´����豸�ļ�
6����/sys/class/�豸����/�豸�ļ�/Ŀ¼�´��������ļ�
7���洢������Ҫ�õ���˽�����ݣ�dev_set_drvdata
8����/proc/Ŀ¼�´����豸�ļ��Լ���ز���
9���������Բ��������Եķ��ʷ���

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
#define ZHWILSON_INFO_PROC_NAME "zhwilson_info"     //�豸��/proc/Ŀ¼�µ�����

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
#include<linux/proc_fs.h>
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

//�������Եķ���
static ssize_t zhwilson_info_show(struct device *dev, struct device_attribute *attr, char *buf);
static ssize_t zhwilson_info_store(struct device *dev, struct device_attribute *attr, const char *buf, size_t count);

//�����豸����
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
	//�ַ���ת����
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

//�����ļ���ȡ
static ssize_t zhwilson_info_proc_read(char *page, char **start, off_t off, int count, int *eof, void *data){
	if (off > 0){
		eof = 1;
		return 0;
	}
	
	return __zhwilson_info_get_info(zhwilson_info_dev, page);
}

//�����ļ�����д��
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
	//�����ݴӻ������������ں˻�����
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

//��/procĿ¼�´��������ļ�
static void zhwilson_info_create_proc(void){
	struct proc_dir_entry *entry;
	entry = create_proc_entry(ZHWILSON_INFO_PROC_NAME, 0, NULL);
	if (entry){
		entry->owner = THIS_MODULE;
		entry->read_proc = zhwilson_info_proc_read;
		entry->write_proc = zhwilson_info_proc_write;
	}
}
//ɾ�������ļ�
static void zhwilson_info_destory_proc(void){
	remove_proc_entry(ZHWILSON_INFO_PROC_NAME, NULL);
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
