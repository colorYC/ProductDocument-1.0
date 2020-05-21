macOS
=========================

.. highlight:: objective-c

**集成桌面或窗口共享功能前，请确保您已经集成了基础的多方音视频通话。**

桌面或窗口共享可以让您和频道中的其他成员一起分享设备里的精彩内容，您可以在频道中利用桌面或窗口共享的功能进行文档演示、在线教育演示、视频会议以及游戏过程分享等。

开启桌面或窗口共享前需要先设置要共享的桌面或窗口id，具体通过 JCMediaDevice 类的 screenshareWindow 属性进行设置。
::

    /// 屏幕分享的窗口
    @property (nonatomic, strong) JCMediaDeviceWindow* __nullable screenshareWindow;


设置要共享的桌面或窗口id
------------------------

首先通过下面的属性获取桌面列表或者窗口列表
::

    /// 桌面列表
    @property (nonatomic, readonly) NSArray<JCMediaDeviceWindow*>* __nonnull desktops;

    /// 窗口列表
    @property (nonatomic, readonly) NSArray<JCMediaDeviceWindow*>* __nonnull windows;


其中，JCMediaDeviceWindow 类有以下两个属性
::

    /// 窗口id
    @property (nonatomic, readonly) NSString* __nonnull windowId;

    /// 窗口名字
    @property (nonatomic, readonly) NSString* __nonnull windowName;


示例代码
::

    NSArray<JCMediaDeviceWindow*>* desktops = JCManager.shared.mediaDevice.desktops;
    NSArray<JCMediaDeviceWindow*>* windows = JCManager.shared.mediaDevice.windows;

列表获取后，可以选取要共享的窗口或桌面id
::

    mediaDevice.screenshareWindow = [desktops objectAtIndex:0];
    windowId = mediaDevice.screenshareWindow.windowId;


屏幕共享采集属性设置
------------------------

您可以调用 JCMediaDevice 类中的 setScreenCaptureProperty 方法设置屏幕共享采集属性，包括采集的高度、宽度和帧速率。
::

    /**
     *  @breif              设置屏幕共享采集属性
     *  @param width        采集宽度，默认640
     *  @param height       采集高度，默认360
     *  @param framerate    帧速率，默认10
     */
    - (void)setScreenCaptureProperty:(int)width height:(int)height framerate:(int)framerate;

.. note:: 该方法可以在开启屏幕共享前调用，也可以在屏幕共享中调用；如果在屏幕共享中调用，则设置的采集属性要在下次屏幕共享开启时生效。


开启或关闭屏幕共享
------------------------

开启或关闭屏幕共享需要根据当前的屏幕共享状态进行判断，具体通过 screenUserId 进行判断。当 screenUserId 不为空时说明当前存在屏幕共享，不能再发起屏幕共享，只有当 screenUserId 为空时才可以发起屏幕共享。

屏幕共享状态是否变化通过 JCMediaChannelPropChangeParam 对象的 screenShare 属性判断。如果屏幕共享状态发生了改变会触发 onMediaChannelPropertyChange 回调
::

    /**
     *  @brief 属性变化回调，目前主要关注屏幕共享状态的更新
     *  @param changeParam 变化标识集合
     */
    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam;

如果当前不存在屏幕共享或者自己发起了屏幕共享，可以调用下面的方法开启或关闭屏幕共享
::

    /**
     * @brief 开关屏幕共享
     * @param enable 是否开启屏幕共享
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)enableScreenShare:(bool)enable;


请求屏幕共享的视频流
------------------------

如果频道中有成员开启了屏幕共享，其他成员将收到 onMediaChannelPropertyChange 的回调，并通过 screenUserId 属性获得发起屏幕共享的用户标识。

获得发起屏幕共享的用户标识后，可以调用 requestScreenVideo 方法请求屏幕共享的视频流
::

    /**
     *  @brief 请求屏幕共享的视频流
     *  @param screenUri 屏幕分享uri
     *  @param pictureSize  视频请求尺寸类型
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常
     *  @see JCMediaChannelPictureSize
     *  @warning 当 pictureSize 为 JCMediaChannelPictureSizeNone 表示关闭请求
     */
    -(bool)requestScreenVideo:(NSString* __nonnull)screenUri pictureSize:(JCMediaChannelPictureSize)pictureSize;


示例代码::

    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam {
        if (changeParam.screenShare) {
            if (mediaChannel.screenUserId = nil) {
                // 开启屏幕共享
                [mediaChannel enableScreenShare:true];
                // 请求屏幕共享的视频流
                JCMediaDeviceVideoCanvas *screen = [mediaDevice startVideo:mediaChannel.screenRenderId renderType:JCMediaDeviceRenderFullContent];
                [mediaChannel requestScreenVideo:mediaChannel.screenRenderId pictureSize:JCMediaChannelPictureSizeLarge];
            } else if (mediaChannel.screenUserId != nil && "自己开启了屏幕共享") {
                // 关闭屏幕共享
                [mediaChannel enableScreenShare:false];
            }
        }
    }
