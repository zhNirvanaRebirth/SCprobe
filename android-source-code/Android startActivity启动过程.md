# Android startActivity启动过程  
> Activity的startActivity方法作为界面跳转，也就是会启动另一个界面，这里是通过Android源码，搞清楚新界面是怎么被绘制然后显示出来的  
  
## 调用方法  
1. Activity.startActivity  
* Intent  
* Context  
* ComponentName:packageName,className  

2. Activity.startActivityForResult  
3. Activity.startActivityFromChild  
4. Instrumentation.execStartActivity:通过AndroidManifest.xml查找需要启动的Activity的相关信息与配置  
5. ActivityManagerProxy.startActivity
