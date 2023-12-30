title: Android 6.0 Root 必备：修改 sepolicy
date: 2015-12-26 16:48:57
categories:
- Tutorial
tags:
- Tutorial
- SEPolicy
- Root
---

从 Android 6.0 开始，Root 变得没有以前那么容易了。6.0 之前，对于 HTC 手机，Root 其实很简单，只需要官方解锁，刷入第三方 Recovery（如 TWRP Recovery），然后通过 Recovery 刷入 SuperSU 包即可。

然而，到了 6.0，如果还是这么做，手机会卡第一屏。这是由于 Google 对 Android 6.0 修改了新的安全机制导致的。解决卡屏的方法，SuperSU 的作者已经给出，就是给 boot.img 里 ramdisk 下的 sepolicy 打补丁。

虽然，最新测试版的 SuperSU 已经试图加入自动修改 sepolicy，但现在还处于早期，可能会存在不稳定因素。而且 2.6x 的 SuperSU 不再将 `su` 和 `SuperSU.apk` 放在 `/system` 下，而是转而放到 `/data` 下，这样会导致一旦 wipe data，Root 就没了。

（这里提供 2.6x 版本 SuperSU 的主帖，想研究的可以去看下：[http://forum.xda-developers.com/apps/supersu/wip-android-6-0-marshmellow-t3219344](http://forum.xda-developers.com/apps/supersu/wip-android-6-0-marshmellow-t3219344)）

因此，在 SuperSU 新版进入正式阶段，现在比较好的方法还是 SuperSU 2.52 配修改版 boot.img（即 sepolicy 打过补丁的 boot.img）。

下面就来讲下 sepolicy 的修改方法。

<!-- more -->

首先提供之后会用到的工具：

- Android Image Kitchen：[http://forum.xda-developers.com/showthread.php?t=2073775](http://forum.xda-developers.com/showthread.php?t=2073775)
- SuperSU 2.52：[http://forum.xda-developers.com/apps/supersu/2014-09-02-supersu-v2-05-t2868133](http://forum.xda-developers.com/apps/supersu/2014-09-02-supersu-v2-05-t2868133)
- adb 工具：这个不提供了下载地址了，到处都是
- 你还需要另一台 Android 手机，系统版本 4.4+，且已经使用 SuperSU 2.50 或更高版本获取了 Root 权限。

下面是具体步骤：

1. 使用 Android Image Kitchen 解包想要 Root 的系统里的 boot.img

2. 在 `ramdisk` 文件夹下获取 `sepolicy` 文件，并将该文件放到 adb 工具所在文件夹中

3. 将已经 Root 的手机连接电脑，并打开 USB 调试

4. 电脑端运行 `CMD`，并 `cd` 到 adb 所在目录，执行：

   ``` sh
   adb push sepolicy /data/local/tmp/sepolicy
   ```

   如果提示没有权限，也可以手动把 `sepolicy` 文件拷贝到手机里，然后通过RE管理器等软件移动到 `/data/local/tmp/sepolicy` 或其他路径，如果在其他路径，后面执行的命令也需要对应修改

5. 继续执行：

   ``` sh
   adb shell su -c "supolicy --file /data/local/tmp/sepolicy /data/local/tmp/sepolicy_out"
   
   adb shell su -c "chmod 0644 /data/local/tmp/sepolicy_out"
   ```

6. 再执行：

   ``` sh
   adb pull /data/local/tmp/sepolicy_out sepolicy_out
   ```

   或者去该目录下将其拷贝出来

7. 获取的 `sepolicy_out` 文件即为修改好的 sepolicy，将其改名成 `sepolicy`，替换到 `ramdisk` 下

8. 通过 Android Image Kitchen 打包 boot.img

至此，修改完成。接下来，你只需要将修改好的 boot.img 和 SuperSU 2.52 刷入手机即可获取 Root 权限了。