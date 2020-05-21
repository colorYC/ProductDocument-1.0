Android
========================

.. highlight:: java

**混淆配置**

如果你的 apk 最终会经过代码混淆，请在 proguard 配置文件中加入以下代码::

    -dontwarn  com.juphoon.*
    -keep class com.juphoon.**{*;}	
    底层sdk
    -dontwarn com.justalk.*
    -keep class com.justalk.**{*;}
    -keepattributes InnerClasses
    -keep class **.R$* {*;}

**初始化问题**

- 如果遇到初始化问题，可以参考 `此贴 <http://developer.juphoon.com/portal/cn/bbs/problem_details.php?t_id=1266>`_ ；
- 检查 AppKey 是否正确；
- 检查 build.gradle 文件中是否增加配置程序文件；
- 检查配置文件 AndroidManifest.xml 是否录入权限相关信息。

**登录**

1. Android 在被登出的时候收到多个 JCLogoutedNotification 的通知，是什么原因？

- 界面注册了多个广播监听导致。

**通话**

1.通话音量如何调节？

::

    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_VOLUME_UP || keyCode == KeyEvent.KEYCODE_VOLUME_DOWN) {
            int direction = keyCode == KeyEvent.KEYCODE_VOLUME_UP ? AudioManager.ADJUST_RAISE : AudioManager.ADJUST_LOWER;
            int flags = AudioManager.FX_FOCUS_NAVIGATION_UP;
            AudioManager am = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
            am.adjustStreamVolume(AudioManager.STREAM_VOICE_CALL, direction, flags);
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }

2. 主叫拨打电话，被叫没有收到 JCCallIncomingd 的通知，可能是什么原因？

- 主叫或者被叫处于断网的状况。

- 主叫调用 JCClient.Login 之后在收到 JCCliNotifyLoginOk 通知之前发起呼叫，这个时候 SDK 的环境还不存在，导致无法呼叫。

3. 一对一通话呼叫时长

- 6分钟。

**会议**

1.会议销毁时间

- 无人加入的会议3分钟超时销毁，会议开始后有人加入，再退出后30s销毁。

**推送问题**

- 小米推送常见问题请参考 `小米推送常见问题汇总 <https://dev.mi.com/console/doc/detail?pId=1292>`_ 。

**视频采集**

1.如何设置视频分辨率？

- 如果是本端视频像素设置，可以在本地视频预览的时候调用 mediaDevice 中的 setCameraProperty 方法进行采集分辨率的设置。

**日志路径**

1.Android 日志地址在哪里看？

- 日志路径如下:
  /sdcard/Android/data/工程包名/files/log/