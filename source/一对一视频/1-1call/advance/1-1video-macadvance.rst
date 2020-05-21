macOS 进阶
=========================

.. highlight:: objective-c

**集成进阶功能前，请确保您已经集成了基础的音视频通话功能**

.. _通话录音(mac):

通话录音
-----------------------------

可以在通话中进行录音，开启或关闭录音需要根据当前的录音状态（audioRecord）来决定。如果正在录制或者通话被挂起或者挂起的情况下，不能进行音频录制。录音状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_ 对象获取。

开启或关闭录音接口如下
::

    /**
     * 语音通话录音，通过 JCCallItem 对象中的audioRecord状态来决定开启关闭录音
     *
     * @param item              JCCallItem 对象
     * @param enable            开启关闭录音
     * @param filePath          录音文件路径
     * @return                  返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)audioRecord:(JCCallItem* __nonnull)item enable:(bool)enable filePath:(NSString* __nullable)filePath;


示例代码::

    // 语音录制
    - (void)audioRecord:(JCCallItem* __nonnull)item {
        if (item.audioRecord) { // 正在录制中
           //可以做录音结束的处理
           [call audioRecord:item enable:false filePath:@"your filePath"];
            ...
        } else { // 未在录制中
            // 创建录音文件
            NSString *filePath; // 录音文件的绝对路径，SDK会自动创建录音文件
            if (filePath != nil) {
               // 开始录音
               [call audioRecord:item enable:true filePath:filePath];
                ...
            } else {
                // 录音失败的处理
            }
        } 
    }


开启或关闭录音时，录音状态会发生改变，并通过 onCallItemUpdate 回调上报 
::

    /**
     *  @brief 通话状态更新回调（当上层收到此回调时，可以根据 JCCallItem 对象获得该通话的所有信息及状态，从而更新该通话相关UI）
     *  @param item JCCallItem 对象
     *  @param changeParam 更新标识类
     */
    -(void)onCallItemUpdate:(JCCallItem* __nonnull)item changeParam:(JCCallChangeParam * __nullable)changeParam;


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _视频通话录制(mac):

视频通话录制
----------------------------

视频通话录制提供 **本地视频源录制和远端视频源录制**。

开启或者关闭视频录制需要根据本端视频源录制状态（localVideoRecord）和远端视频源录制状态（remoteVideoRecord）来决定。视频录制状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_ 对象获得。还可以对录制视频的高度和宽度以及存储路径进行设置，接口如下：
::

    /**
     * 视频通话录制，通过 JCCallItem 对象中的 localVideoRecord 状态来决定开启关闭录制
     *
     * @param item              JCCallItem 对象
     * @param enable            开启关闭录制
     * @param remote            是否为远端视频源
     * @param width             录制视频宽像素
     * @param height            录制视频高像素
     * @param filePath          录制视频文件存储路径
     * @param bothAudio         是否录制两端音频，false 表示只录制视频端音频
     * @return                  返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)videoRecord:(JCCallItem* __nonnull)item enable:(bool)enable remote:(bool)remote width:(int)width height:(int)height filePath:(NSString* __nullable)filePath bothAudio:(bool)bothAudio;

.. note:: 参数 remote 值为 true 时为远端视频源录制，值为 false 时为本地视频源录制。

示例代码::

    // 视频录制
    - (void)videoRecord {
        JCCallItem *item = [call getActiveCallItem];
        if (item.localVideoRecord) { //如果正在录制本地视频
            // 停止录制本地视频
            [call videoRecord:item enable:false remote:false width:0 height:0 filePath:@"your filePath" bothAudio:false];
             ...
        } else if (item.remoteVideoRecord) { // 如果正在录制远端视频
            // 停止录制远端视频
            [call videoRecord:item enable:false remote:true width:0 height:0 filePath:@"your filePath" bothAudio:false];
            ...
        } else {
            // 创建视频录制文件保存路径
            NSString *filePath; // 视频录制文件的绝对路径，SDK会自动创建视频录制文件
            if (filePath != nil) {
                // 远端视频录制
                [call videoRecord:item enable:true remote:true width:640 height:360 filePath:filePath bothAudio:false];
                // 本端视频录制
                [call videoRecord:item enable:true remote:false width:640 height:360 filePath:filePath bothAudio:false];
                ...
            } 
        }
    }


开启或关闭视频录制时，录制状态会发生改变，并通过 onCallItemUpdate 回调上报 
::

    /**
     *  @brief 通话状态更新回调（当上层收到此回调时，可以根据 JCCallItem 对象获得该通话的所有信息及状态，从而更新该通话相关UI）
     *  @param item JCCallItem 对象
     *  @param changeParam 更新标识类
     */
    -(void)onCallItemUpdate:(JCCallItem* __nonnull)item changeParam:(JCCallChangeParam * __nullable)changeParam;



^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _截屏(mac):

截图
------------------------------

在视频通话中，如果想对当前的通话界面进行保存，可以使用截图功能，截图分为 **本端视频源截图和远端视频源截图**，接口如下：

::

    /**
     *  @breif           视频通话截图
     *  @param width     截屏宽度像素，-1为视频源像素
     *  @param height    截屏高度像素，-1为视频源像素
     *  @param filePath  文件路径
     */
    -(bool)snapshot:(int)width heigh:(int)height filePath:(NSString*)filePath;

示例代码::

    - (void)snapshot {
        JCCallItem *item = [call getActiveCallItem];
        JCMediaDeviceVideoCanvas *localCanvas = [mediaDevice startCameraVideo:JCMediaDeviceRenderFullContent];
        JCMediaDeviceVideoCanvas *remoteCanvas = [mediaDevice startVideo:item.renderId renderType:JCMediaDeviceRenderFullContent];
        NSString *filePath; // 截图文件的绝对路径，SDK会自动创建截图文件
        // 本端视频源截图
        [localCanvas snapshot:-1 heigh:-1 filePath:filePath];
        // 远端视频源截图
        [remoteCanvas snapshot:-1 heigh:-1 filePath:filePath];
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _文件视频源(mac):

使用文件作为视频输入源
------------------------------

在视频通话中，可以将文件作为视频输入源，典型的场景如在线课堂的文稿显示，文件和摄像头作为视频输入源**只能存在一种**，如果将文件作为视频输入源，而当前摄像头已开启的话，则会关闭摄像头。

开启视频文件作为视频输入源有如下两种接口可供选择，开发者可根据具体的需求选择调用
::

    /**
     *  @brief 向文件视频源逐帧添加视频数据
     *  @param srcFrame 画面二进制数据
     *  @param format 视频像素格式
     *  @param width 宽
     *  @param height 高
     */
    -(void)setVideoFileFrame:(NSData* __nonnull)srcFrame format:(JCMediaDeviceVideoPixelFormat)format width:(int)width height:(int)height;

    /**
     *  @brief 向文件视频源逐帧添加视频数据
     *  @param CVPixelBufferRef 视频数据
     *  @param angle 角度, 为 90 的倍数
     *  @param mirror 是否镜像，0 不镜像，1 镜像
     */
    -(void)setVideoFileFrame:(CVPixelBufferRef __nonnull)srcFrame angle:(int)angle mirror:(int)mirror;


如果想关闭文件视频源，则调用下面的接口
::

    /**
     *  @brief 关闭文件视频源
     *  @return ture表示关闭成功，false表示关闭失败
     */
    -(bool)stopVideoFile;

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

通话人数设置
-----------------------------

发起通话前可以通过 maxCallNum 属性设置通话的最大人数，默认为 1。
::

    call.maxCallNum = 2;

当通话超过最大数呼出会失败，收到来电会自动拒绝。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

发送消息
-----------------------------

调用下面的接口在通话中实现发消息的功能
::

    /**
     *  @brief 通过通话建立的通道发送数据
     *  @param item 需要发送数据的 JCCallItem 对象
     *  @param type 文本消息类型，用户可以自定义，例如text、xml等
     *  @param content 消息内容
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)sendMessage:(JCCallItem * __nonnull)item type:(NSString * __nonnull)type content:(NSString * __nonnull)content;


当通话中收到消息时，会收到 onMessageReceive 回调
::

    /**
     *  @brief 通话中收到消息的回调
     *  @param item JCCallItem 对象
     *  @param type 消息类型
     *  @param content 消息内容
     */
    -(void)onMessageReceive:(JCCallItem * __nonnull)item type:(NSString * __nonnull)type content:(NSString * __nonnull)content;

示例代码::

    [call sendMessage:item type:@"text" content:@"消息内容"];