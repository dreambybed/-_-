# Tecent_GameDevelop_OpenClass  
## 第一课  
### 1.UE源码编译  
在Epic中绑定github账号，加入EpicGames组织，下载源码  
运行源码中的Setup.bat，下载依赖文件  
运行GenerateProjectFiles.bat生成工程文件  
使用VS2022打开UE5.sln，编译工程  
编译完成后在Engine\Binaries\Win64目录下找到UnrealEditor打开软件  
### 2.Android环境配置  
下载Android Studio，并在sdk manager中找到SDK Components Setup，安装组件  
在sdk manager中找到SDK Tools，安装Android SDK Command-line Tools  
关闭UE5和Android Studio，在UE5文件夹Engine\Extras\Android下找到SetupAndroid.bat，让UE5关联Android SDK相关路径  
*在https://services.gradle.org/distributions/下载Gradle对应版本，将压缩包放在C:\Users\user\.gradle\wrapper\dists\gradle-7.5-all(对应版本)\6qsw290k5lz422uaf8jf6m7co\目录下*  
### 3.android打包和真机调试  
打开UE5，新建一个fps模板项目，在上方菜单的平台选项中选择Android平台并打包   
在手机中打开开发者选项并打开USB调试，连接至电脑，此时UE5会自动识别设备，在平台选项中选择移动设备，经过编译后就可以在手机上运行游戏项目
