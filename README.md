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
*在[gradle官网](https://services.gradle.org/distributions/)中下载Gradle对应版本，将压缩包放在C:\Users\user\.gradle\wrapper\dists\gradle-7.5-all(对应版本)\6qsw290k5lz422uaf8jf6m7co\目录下*  
### 3.android打包和真机调试  
打开UE5，新建一个fps模板项目，在上方菜单的平台选项中选择Android平台并打包   
在手机中打开开发者选项并打开USB调试，连接至电脑，此时UE5会自动识别设备，在平台选项中选择移动设备，经过编译后就可以在手机上运行游戏项目  
## 第二课  
### 1.射击命中物体加分  
实现功能：子弹射击命中物体后，物体首先缩放为y倍，再次命中后物体销毁，物体有两种，一种是特殊目标，加分更多，一种是普通目标，加分较少，考虑多人游戏的情况
创建一个新蓝图作为目标，命名为BP_TargetBlock，在Components下新建一个StaticMesh，选择合适的素材和Materials，由于多人游戏需要同步，因此将Class Default中的Replicates和Replicate Movement勾选，此时目标状态将自动同步到客户端上  
在EventHit检测碰撞，使用Cast To BP_FirstPersonProjectile检测碰撞体为子弹  
在多人游戏情况下，需要区分子弹发出者是否是本地玩家，否则子弹会在他人窗口中被物体检测到，从而给他人加分，因此在BP_FirstPersonProjectile中添加Player Controller类型的变量Shoot Player，在检测碰撞时比较本地玩家与Shoot Player是否相同  
添加一个新的Customer Event，命名为HitAddScore，在检测玩家相同后，给玩家加分。在BP_FirstPersonCharacter中添加变量Score，在HitAddScore事件中通过Get Player Character获取BP_FirstPersonCharacter，再通过Cast To BP_FirstPersonCharacter设置分数，由于目标存在特殊目标和普通目标两种，所以在BP_TargetBlock中添加新布尔变量IsSpecial，判断是否为特殊目标，增加分数  
在完成增加分数的逻辑后，需要完成缩放和销毁逻辑，在BP_TargetBlock创建一个新的Customer Event为HandleHitOnServer，将其Replicates设置为Run on Server，让逻辑在服务端触发，在创建一个Event为UpdateHitOnClients，将其Replicates设置为Multicast，使服务端能将状态广播到服务端。为了检查目标是第一次击中还是第二次击中，新建一个布尔变量IsScale，当IsScale为False时，使用Set Relative Scale 3D缩放物体，并将IsScale设为True，当第二次击中时，使用Destroy Actor销毁物体。最后将HandleHitOnServer连接至UpdateHitOnClients  
为了让特殊目标在视觉上有差异，在BP_FirstPersonGameMode中添加逻辑，使用BlockN的整数变量设置特殊目标的个数，通过Get All Actors Of Class 获取所有目标，再通过Shuffle打乱数组，选取前N个作为特殊目标，将IsSpecial设为True  
为了动态加载材质，回到BP_TargetBlock创建新的event，SetSpecialTarget，将其Replicates设置为Run on Server，让逻辑在服务端触发，再创建UpdateVisualsOnClients与SetSpecialTarget连接，将其Replicates设置为Multicast，使用Create Dynamic Material Instance动态添加材质，再在GameMode中调用SetSpecialTarget，从而实现了特殊材质的动态加载与同步。  
### 2.子弹同步  
实现功能：子弹能够在多人游戏中同步显示，并触发正确的逻辑  
在原来的模板中，子弹不能够在其他窗口显示。首先打开BP_FirstPersonProjectile，将Class Default中的Replicates和Replicate Movement勾选，让子弹能够从服务端向其他客户端同步  
打开BP_WeaponComponent，发现其为BP_FirstPersonCharacter的子组件，为了实现同步功能，将子弹生成的逻辑移到BP_FirstPersonCharacter中：新建SpawnProjectileOnServer的Event,将其Replicates设置为Run on Server，使用SpawnActor BP_FirstPersonProjectile生成子弹，其中spawn transform从BP_WeaponComponent中传入，这是因为由于逻辑运行在服务端，如果直接调用BP_FirstPersonCharacter的逻辑，Get Player Camera Manager得到的rotation会是服务端的。在生成子弹时，设置发出子弹的Player为当前窗口的用户，从而使击中目标时能对正确的Player加分。最后在BP_WeaponComponent完成本地逻辑，包括音效和动画。    
### 3.UGM实现UI  
实现功能：能够在左上角显示分数，在计时结束后显示最终得分      
创建Widget蓝图ScoreWidget，添加一个text block并添加绑定，获取BP_FirstPersonCharacter的Score并显示，然后在BP_FirstPersonCharacter的BeginPlay中添加逻辑，使用Create Widget和Add to Viewport将该Widget添加到屏幕上  
创建Widget蓝图TimeOutWidget，添加一个text block并添加绑定，同样获取BP_FirstPersonCharacter的Score并显示到text上，然后在BP_FirstPersonCharacter的BeginPlay最后添加逻辑，使用SetTimerByEvent，在计时结束后添加Widget并暂停游戏
