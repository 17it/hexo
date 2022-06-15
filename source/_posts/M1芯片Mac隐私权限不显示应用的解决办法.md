初识M1，确实比较强悍，续航长还静音，但是遇到个问题，飞书和腾讯会议的麦克风和摄像头权限没法开启。

## 表现：

在隐私设置里，选择麦克风和摄像头，里面没有飞书和腾讯会议，并且应用列表下面也没有手动添加应用的 + 号，问了apple客服，说是M1芯片对这种比较敏感的权限不开放给用户手动添加应用的功能。

## 一、进入安全模式，关闭SIP

关机长按电源键，选择选项，然后左上角的终端，输入命令，然后重启。

```bash
csrutil disable
```  

## 二、在权限db里手动插入数据

sql格式如：INSERT or REPLACE INTO access VALUES(权限名,应用包名,0,0,4,1,NULL,NULL,0,'UNUSED',NULL,0,时间戳)

```bash
sudo sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db "INSERT or REPLACE INTO access VALUES('kTCCServiceMicrophone','com.tencent.meeting',0,0,4,1,NULL,NULL,0,'UNUSED',NULL,0,1622199671);"
sudo sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db "INSERT or REPLACE INTO access VALUES('kTCCServiceMicrophone','com.bytedance.macos.feishu',0,0,4,1,NULL,NULL,0,'UNUSED',NULL,0,1622199671);"
sudo sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db "INSERT or REPLACE INTO access VALUES('kTCCServiceCamera','com.tencent.meeting',0,0,4,1,NULL,NULL,0,'UNUSED',NULL,0,1622199671);"
sudo sqlite3 ~/Library/Application\ Support/com.apple.TCC/TCC.db "INSERT or REPLACE INTO access VALUES('kTCCServiceCamera','com.bytedance.macos.feishu',0,0,4,1,NULL,NULL,0,'UNUSED',NULL,0,1622199671);"
```

## 三、常用权限解释

#### 权限：
+ 辅助功能 kTCCServiceAccessibility
+ 摄像头 kTCCServiceCamera
+ 输入监听 kTCCServiceListenEvent
+ 麦克风 kTCCServiceMicrophone
+ 录制屏幕 kTCCServiceScreenCapture
+ 完全磁盘访问权限 kTCCServiceSystemPolicyAllFiles

#### 查看包名：
在访达中找到应用程序文件后（一般位于应用程序），右键选择显示包内容，使用文本编辑器打开Contents/Info.plist，找到<key>CFBundleIdentifier</key>，下面一行在<string>和</string>中间的便是包名


* 参考链接：https://blog.csdn.net/weixin_43917335/article/details/109245795
* 参考链接：https://www.jianshu.com/p/105c88194350
* 参考链接：https://juejin.cn/post/7033792167362035749