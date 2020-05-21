Android
=========================

.. highlight:: java

**集成屏幕共享功能前，请确保您已经集成了多方音视频通话。**

.. note:: SDK 不支持模拟器运行，请使用真机。

.. _屏幕共享(android):


屏幕共享可以让您和频道中的其他成员一起分享设备里的精彩内容，您可以在频道中利用屏幕共享的功能进行文档演示、在线教育演示、视频会议以及游戏过程分享等。

.. note:: 发起屏幕共享需要 Android 5.0 及以上。

屏幕共享采集属性设置
------------------------------

您可以调用 JCMediaDevice 类中的 setScreenCaptureProperty 方法设置屏幕共享采集属性，包括采集的高度、宽度和帧速率。
::

    /**
     * 设置屏幕共享采集属性
     * @param width     采集宽度，默认640
     * @param height    采集高度，默认360
     * @param frameRate 采集帧速率，默认10
     */
    public abstract void setScreenCaptureProperty(int width, int height, int frameRate);

.. note:: 该方法可以在开启屏幕共享前调用，也可以在屏幕共享中调用；如果在屏幕共享中调用，则设置的采集属性要在下次屏幕共享开启时生效。


开启或关闭屏幕共享
------------------------------

开启或关闭屏幕共享需要根据当前的屏幕共享状态进行判断，具体通过 screenUserId 进行判断。当 screenUserId 不为空时说明当前存在屏幕共享，不能再发起屏幕共享，只有当 screenUserId 为空时才可以发起屏幕共享。

屏幕共享状态是否变化通过 screenShare 判断。如果屏幕共享状态发生了改变会触发 onMediaChannelPropertyChange 回调
::

    /**
     * 属性变化回调，目前主要关注屏幕共享状态的更新
     *
     * @param propChangeParam 变化标识集合
     */
    void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);

如果当前不存在屏幕共享或者自己发起了屏幕共享，可以调用下面的方法开启或关闭屏幕共享
::

    /**
     * 开关屏幕分享
     * @param enable 是否开启屏幕分享
     *
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableScreenShare(boolean enable);

.. note::
      
       屏幕共享发送方需要在 manifest 文件中做以下声明，否则无法发送本地视频桌面的视频流::

           <activity
                   android:name = "com.justalk.cloud.zmf.ZmfActivity"
                   android:theme = "@android:style/Theme.Dialog"/>



请求屏幕共享的视频流
----------------------------

如果频道中有成员开启了屏幕共享，其他成员将收到 onMediaChannelPropertyChange 的回调，并通过 getScreenUserId 属性获得发起屏幕共享的用户标识。

获得发起屏幕共享的用户标识后，可以调用 requestScreenVideo 方法请求屏幕共享的视频流
::

    /**
     * 请求屏幕共享的视频流
     * 当 pictureSize 为 JCMediaChannelPictureSizeNone 表示关闭请求
     *
     * @param screenUri     屏幕分享uri
     * @param pictureSize   视频请求尺寸类型
     * @return              返回 true 表示正常执行调用流程，false 表示调用异常
     * @see JCMediaChannel.PictureSize
     */
    public abstract boolean requestScreenVideo(String screenUri, @PictureSize int pictureSize);



示例代码::

    public void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam) {
        if (propChangeParam.screenShare) {
            if (mediaChannel.screenUserId = nil) {
                // 开启屏幕共享
                mediaChannel.enableScreenShare(true);
                // 请求屏幕共享的视频流
                JCMediaDeviceVideoCanvas screenShare = mediaDevice.startVideo(mediaChannel.getScreenRenderId(), JCMediaDevice.RENDER_FULL_CONTENT);
                mediaChannel.requestScreenVideo(mediaChannel.getScreenRenderId(),JCMediaChannel.PICTURESIZE_LARGE);
            } else if (mediaChannel.screenUserId != nil && "自己开启了屏幕共享") {
                // 关闭屏幕共享
                mediaChannel.enableScreenShare(false);
            }
        }
    }
