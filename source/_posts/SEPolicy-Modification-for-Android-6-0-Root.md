title: Android 6.0 Root 必备：修改 sepolicy
date: 2015-12-26 16:48:57
categories:
- 教程
tags:
- 教程
- sepolicy
- root
---

从Android 6.0开始，Root变得没有以前那么容易了。6.0之前，对于HTC手机，Root其实很简单，只需要官方解锁，刷入第三方Recovery（如TWRP Recovery），然后通过Recovery刷入SuperSU包即可。

然而，到了6.0，如果还是这么做，手机会卡第一屏。这是由于Google对Android 6.0修改了新的安全机制导致的。解决卡屏的方法，SuperSU的作者已经给出，就是给boot.img里ramdisk下的sepolicy打补丁。

虽然，最新测试版的SuperSU已经试图加入自动修改sepolicy，但现在还处于早期，可能会存在不稳定因素。而且2.6x的SuperSU不再将su和SuperSU.apk放在/system下，而是转而放到/data下，这样会导致一旦wipe data，Root就没了。

（这里提供2.6x版本SuperSU的主帖，想研究的可以去看下：[http://forum.xda-developers.com/apps/supersu/wip-android-6-0-marshmellow-t3219344](http://forum.xda-developers.com/apps/supersu/wip-android-6-0-marshmellow-t3219344)）

因此，在SuperSU新版进入正式阶段，现在比较好的方法还是SuperSU 2.52配修改版boot.img（即sepolicy打过补丁的boot.img）。

下面就来讲下sepolicy的修改方法。

<!-- more -->

首先提供之后会用到的工具：

Android Image Kitchen：[http://forum.xda-developers.com/showthread.php?t=2073775](http://forum.xda-developers.com/showthread.php?t=2073775)

SuperSU 2.52：[http://forum.xda-developers.com/apps/supersu/2014-09-02-supersu-v2-05-t2868133](http://forum.xda-developers.com/apps/supersu/2014-09-02-supersu-v2-05-t2868133)

adb工具：这个不提供了下载地址了，到处都是

你还需要另一台Android手机，系统版本4.4+，且已经使用SuperSU 2.50或更高版本获取了Root权限。

下面是具体步骤：

1、使用Android Image Kitchen解包想要Root的系统里的boot.img

2、在ramdisk文件夹下获取sepolicy文件，并将该文件放到adb工具所在文件夹中

3、将已经Root的手机连接电脑，并打开USB调试

4、电脑端运行CMD，并cd到adb所在目录，执行：

``` sh
adb push sepolicy /data/local/tmp/sepolicy
```

如果提示没有权限，也可以手动把sepolicy文件拷贝到手机里，然后通过RE管理器等软件移动到`/data/local/tmp/sepolicy`或其他路径，如果在其他路径，后面执行的命令也需要对应修改

5、继续执行：

``` sh
adb shell su -c "supolicy --file /data/local/tmp/sepolicy /data/local/tmp/sepolicy_out"

adb shell su -c "chmod 0644 /data/local/tmp/sepolicy_out"
```

6、再执行：

``` sh
adb pull /data/local/tmp/sepolicy_out sepolicy_out
```

或者去该目录下将其拷贝出来

7、获取的sepolicy_out文件即为修改好的sepolicy，将其改名成sepolicy，替换到ramdisk下

8、通过Android Image Kitchen打包boot.img

至此，修改完成。接下来，你只需要将修改好的boot.img和SuperSU 2.52刷入手机即可获取Root权限了。