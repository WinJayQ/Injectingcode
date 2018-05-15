题外话：此教程是一篇严肃的学术探讨类文章，仅仅用于学习研究，也请读者不要用于商业或其他非法途径上，笔者一概不负责哟~~
##准备工作
* 非越狱的iPhone手机
* 用PP助手下载： 微信6.6.5(越狱应用)
* MachOView
>MachOView下载地址：[http://sourceforge.net/projects/machoview/](https://link.jianshu.com/?t=http://sourceforge.net/projects/machoview/)
>MachOView源码地址：[https://github.com/gdbinit/MachOView](https://link.jianshu.com/?t=https://github.com/gdbinit/MachOView)
* yololib
>yololib下载地址https://github.com/KJCracks/yololib?spm=a2c4e.11153940.blogcont63256.9.5126420eAJpqBD



##代码注入思路：
dylb会加载Frameworks中所有的动态库，那么在Frameworks中加一个自己的动态库，然后在自己动态库中hook和注入代码

###动态库存放的位置：Frameworks
![image.png](https://upload-images.jianshu.io/upload_images/1013424-161997866a52aec0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###找到可执行文件WeChat
![image.png](https://upload-images.jianshu.io/upload_images/1013424-e4f12b6c6aa5fb93.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

用MachOView打开可执行文件WeChat，在Load Commands里可以查看到动态库
![image.png](https://upload-images.jianshu.io/upload_images/1013424-2f0d040fe1dfc01e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1013424-35f07623f288ed4c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

![image.png](https://upload-images.jianshu.io/upload_images/1013424-3a45631e72661e10.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

##步骤：
###1、根据[iOS逆向之自动化重签名](https://www.jianshu.com/p/30c1059879aa)先编译运行微信，然后新建Framework
TARGETS添加：
![image.png](https://upload-images.jianshu.io/upload_images/1013424-5b58e339a4d28335.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![image.png](https://upload-images.jianshu.io/upload_images/1013424-0403308f4dd9fada.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###2、新建一个WJHook类
![image.png](https://upload-images.jianshu.io/upload_images/1013424-073389c9959af107.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###3、想实现刚加载就要运行，代码就要写在load方法里
![image.png](https://upload-images.jianshu.io/upload_images/1013424-9cefc0037c32ff73.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###4、为工程添加依赖关系
* 在代码注入targets的Build Phases中添加Copy Files
![image.png](https://upload-images.jianshu.io/upload_images/1013424-10e1497a63af702d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 在Copy Files中选择Frameworks
![image.png](https://upload-images.jianshu.io/upload_images/1013424-180775d60ea1a8c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 添加WJHookFrameWork
![image.png](https://upload-images.jianshu.io/upload_images/1013424-2650a1626e662df4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###5、编译一下,在app包的位置查看
![image.png](https://upload-images.jianshu.io/upload_images/1013424-468c439074d611d8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
显示包内容，在Frameworks中查看
![image.png](https://upload-images.jianshu.io/upload_images/1013424-b3f9b9b7ec85601a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
由上可知，WJHookFrameWork已经加入成功

###6、运行，并没有成功，没有执行load里的代码
原因：用MachOView打开可执行文件WeChat，在Load Commands找不到WJHookFrameWork
###7、将WJHookFrameWork写入MachO文件
需要用到工具：yololib
因为经常会用到这个工具，建议将它放到 */usr/local/bin*
![image.png](https://upload-images.jianshu.io/upload_images/1013424-bdf24830d382f7f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>显示隐藏文件，可以使用
$defaults write com.apple.finder AppleShowAllFiles -bool true
$KillAll Finder
这条命令来显示。同时，将 true 改成 false, 就可恢复隐藏状态。
* 解压微信越狱包
![image.png](https://upload-images.jianshu.io/upload_images/1013424-40e5f6c27349f512.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 将WeChat.app显示包内容，找到WeChat可执行文件
增加执行权限：```chmod +x WeChat```


* 写入WeChat可执行文件：
```yololib WeChat Frameworks/WJHookFrameWork.framework/WJHookFrameWork```
>"Frameworks/WJHookFrameWork.framework/WJHookFrameWork"路径是指WJHookFrameWork可执行文件的路径
![image.png](https://upload-images.jianshu.io/upload_images/1013424-9f44607696720666.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
* 检查MachO文件的Load Commands里是否有WJHookFrameWork
![image.png](https://upload-images.jianshu.io/upload_images/1013424-526211d15c0da63d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如图，加入成功。
* 删除原来的微信-6.6.5(越狱应用).ipa，打包Payload
```zip -ry WeChat.ipa Payload```
将WeChat.ipa放入APP目录，删除其他文件夹
![image.png](https://upload-images.jianshu.io/upload_images/1013424-d04a7fe5f6b82d42.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###8、运行，成功！
![image.png](https://upload-images.jianshu.io/upload_images/1013424-9eebd5ab778603b9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

>代码和工具已上传：https://gitee.com/winjayq/ios_reverse_code_injection



