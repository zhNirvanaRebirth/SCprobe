# Android Studio & Gradle  
## Android8.0特性  
* android 8.0对应supportlibrary26  
* android 8.1对应supportlibrary27  
* xml中的字体 font&font family  
* 字体大小控制  
* DynamicAnimation：FlingAnimation，SpringAnimation  

## gradle  

project：

buildScript
	repostories
	dependencies
		classpath 'group:name:version'
allprojects
	repostories
	
	
/////////////////////////////////////////////////////////	
module:

apply plugin: 'com.android.application' //use the project build file 

android
	compileSdkVersion
	buildToolsVersion
	signingConfigs
		config/debug/release
			storeFile file('../xx.jks')
			keyAlias
			keyPassword
			storePassword
	defaultConfig
		applicationId
		minSdkVersion
		targetSdkVersion
		versionCode
		versionName
		testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
		multiDexEnabled
		signingConfig
		//add something to string.xml
		resValue "string" "app_name" "myapp"
		resValue "bool" "is_new" "false"
		//add const value to ConfigField (app/build/source/BuildConfig/dev/packageName/BuildConfig)
		buildConfigField "String" "ENV" '"dev"'
		//replace value in AndroidManifest.xml
		manifestPlaceholders = [CHANNEL_NAME:"xiaomi"]
		
	buildTypes
		debug/release
			minifyEnabled
			proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
			shrinkResources
		
	productFlavors  //打包渠道
		aa/bb/cc {}
		
	dexOptions
		incremental
		javaMaxHeapSize "4g"
		
	lintOptions
		abortOnError
		
		
dependencies
	Compile
	Provided
	APK
	Test compile
	Debug compile
	Release compile
	implementation
	testImplementation
	androidTestImplementation