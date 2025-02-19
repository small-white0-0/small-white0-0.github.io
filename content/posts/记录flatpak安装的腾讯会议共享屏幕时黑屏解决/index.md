+++
title = "记录flatpak安装的腾讯会议共享屏幕时黑屏解决"
date = "2025-02-17"

[taxonomies]
tags = ["linux", "flatpak", "wemeet"]
+++

## 情况描述

在debian 12 中使用flatpak 安装腾讯会议后，在共享屏幕后，出现屏幕直接黑屏的情况。

在搜索后，找到[flatpak 安装的腾讯会议为什么共享桌面是黑屏](https://blog.csdn.net/sunyuhua_keyboard/article/details/143877839)内容后，使用`flatpak override`修改了权限。但是依旧黑屏。

然后进入flatpka中腾讯会议的打包的[仓库](https://github.com/flathub/com.tencent.wemeet)中，发现其中描述内嵌了[wemeet-wayland-screenshare](https://github.com/xuwd1/wemeet-wayland-screenshare)，但是我本机使用的是i3，是Xorg环境，而不是wayland环境。所以试着使用`flatpak override --user --unset-env=LD_PRELOAD com.tencent.wemeet`关闭了wemeet-wayland-screenshare。测试下来就好了。

## 总结

问题：

在i3桌面环境，使用flatpak安装wemeet软件。共享屏幕时出现黑屏。

解决：

使用`flatpak override --user --unset-env=LD_PRELOAD com.tencent.wemeet`关闭wemeet-wayland-screenshare。
