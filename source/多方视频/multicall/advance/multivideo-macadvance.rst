macOS 进阶
=========================

.. highlight:: objective-c

**集成进阶功能前，请确保您已经集成了基础的音视频通话功能**

.. _查询频道(mac):

查询频道
---------------------------

如需查询频道相关信息，例如频道名称、是否存在、成员名、成员数，可以调用 query 接口进行查询操作
::

    /**
     *  @brief              查询媒体频道相关信息，例如是否存在，人数等
     *  @param channelId    媒体通道标识
     *  @return             返回操作id，与 onQuery 回调中的 operationId 对应
     */
    -(int)query:(NSString* __nonnull)channelId;

示例代码::

    [mediaChannel query:@"channelId"];

查询操作发起后，UI 通过以下方法监听回调查询的结果：
::

    /**
     *  @brief 查询媒体通道信息结果回调
     *  @param operationId 操作id，由 query 接口返回
     *  @param result true 表示查询成功，false 表示查询失败
     *  @param reason  查询失败原因 当 result 为 false 时该值有效
     *  @param queryInfo 查询到的信息，当失败时只需关注 channelId
     *  @see JCMediaChannelReason
     */
    -(void)onQuery:(int)operationId result:(bool)result reason:(JCMediaChannelReason)reason queryInfo:(JCMediaChannelQueryInfo*)queryInfo;

示例代码::

    -(void)onQuery:(int)operationId result:(bool)result reason:(JCMediaChannelReason)reason queryInfo:(JCMediaChannelQueryInfo *)queryInfo {
        // 查询成功
       if (result) {
            // 查询频道标识
            NSString* channelId = queryInfo.channelId;
            // 查询频道号
            int number = queryInfo.number;
            // 查询频道成员数
            int clientCount = queryInfo.clientCount;
            // 查询频道成员列表
            NSMutableArray *members = queryInfo.members;
       } else {
            // 查询失败
       }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^

获取成员
---------------------------

通过 userId 获取频道成员对象
::

    /**
     *  @brief 获得频道成员
     *  @param userId 用户唯一标识
     *  @return 成员对象
     */
    -(JCMediaChannelParticipant* __nullable)getParticipant:(NSString* __nonnull)userId;

^^^^^^^^^^^^^^^^^^^^^^^^^^^

踢出成员
---------------------------

调用下面的方法将成员踢出会议
::

    /**
     * @brief 将成员踢出会议
     *
     * @param participant 成员
     * @return true表示成功，false表示失败
     */
    -(BOOL)kick:(JCMediaChannelParticipant * __nonnull)participant;

示例代码::

    JCMediaChannelParticipant* participant = [mediaChannel getParticipant:@"userId"];
    if (participant != nil) {
        [mediaChannel kick:participant];
    }

^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _屏幕共享(mac):

桌面或窗口共享
----------------------

桌面或窗口共享可以让您和频道中的其他成员一起分享设备里的精彩内容，您可以在频道中利用桌面或窗口共享的功能进行文档演示、在线教育演示、视频会议以及游戏过程分享等。

开启桌面或窗口共享前需要先设置要共享的桌面或窗口id，具体通过 JCMediaDevice 类的 screenshareWindow 属性进行设置。
::

    /// 屏幕分享的窗口
    @property (nonatomic, strong) JCMediaDeviceWindow* __nullable screenshareWindow;


设置要共享的桌面或窗口id
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CDN 推流(mac):


CDN 推流
----------------------

CDN 推流服务适用于各类音视频直播场景，如企业级音视频会议、赛事、游戏直播、在线教育、娱乐直播等。

CDN 推流集成简单高效，开发者只需调用相关 API 即可将 CDN 推流无缝对接到自己的业务应用中。

推流地址设置
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

如要开启 CDN 推流，需在 **加入频道前** 进行 CDN 推流地址的设置。具体为通过 JCMediaChannelJoinParam 对象中的 cdn 属性进行设置
  
示例代码
::

    // 设置 CDN 推流地址
    JCMediaChannelJoinParam *joinParam = [[JCMediaChannelJoinParam alloc] init];
    joinParam.cdn = @"推流地址";
    // 加入频道
    [mediaChannel join:@"channelId" joinParam:joinParam];


CDN 状态获取
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

开启 CDN 推流前需要判断 CDN 的状态，通过 JCMediaChannel 类的 cdnState 属性获取推流器状态。只有 CDN 当前状态不为 JCMediaChannelCdnStateNone 时才可以进行 CDN 推流。其中，CDN 推流状态有以下几种：
::

    /// 无法进行CDN推流
    JCMediaChannelCdnStateNone,
    /// 可以开启CDN推流
    JCMediaChannelCdnStateReady,
    /// CDN推流中
    JCMediaChannelCdnStateRunning,


CDN 状态的变化通过 onMediaChannelPropertyChange 回调上报
::

    /**
     *  @brief 属性变化回调，目前主要关注屏幕共享状态的更新
     *  @param changeParam 变化标识集合
     */
    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam;


开启或关闭 CDN 推流
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

CDN 状态获取后，即可根据 CDN 的状态开启或关闭 CDN 推流，接口如下
::

    /**
     *  @brief              开关Cdn推流
     *  @param enable       是否开启Cdn推流
     *  @param keyInterval  推流关键帧间隔(毫秒)，当 enable 为 true 时有效，-1表示使用默认值(5000毫秒)，有效值需要>=1000
     *  @return             返回 true 表示正常执行调用流程，false 表示调用异常
     *  @warning 在收到 onMediaChannelPropertyChange 回调后检查是否开启
     */
    -(bool)enableCdn:(bool)enable keyInterval:(int)keyInterval;


示例代码
::

    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam {
        if (changeParam.cdnState) {  // CDN 状态变化
           JCMediaChannelCdnState cdnState =  mediaChannel.cdnState;
            // 根据CDN推流状态判断是否开启推流
            if (cdnState == JCMediaChannelCdnStateNone) {
                // 无法使用 CDN 推流
            } else if (cdnState == JCMediaChannelCdnStateReady) {
                // 可以开启 CDN 推流
                [mediaChannel enableCdn:true keyInterval:0];
            } else if (cdnState == JCMediaChannelCdnStateRunning) {
                // CDN 推流中，可以关闭 CDN 推流
                [mediaChannel enableCdn:false keyInterval:0];
            }
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _音视频录制(mac):

服务器音视频录制
----------------------

设置录制参数
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

服务器音频视频录制将录制的文件保存在七牛云上，因此，需要在七牛云注册账号并获取 AccessKey、SecretKey、BucketName、fileKey。

AccessKey、SecretKey、BucketName、fileKey 获取之后，利用 JCMediaChannelRecordParam 对象中的 buildQiniuRecordParam 方法构造七牛录制参数

::

    /**
     * @brief 七牛录制参数构造
     *
     * @param video 是否是视频录制
     * @param bucketName 七牛云 bucketName
     * @param secretKey 七牛云 secretKey
     * @param accessKey 七牛云 accessKey
     * @param fileName 录制文件名
     * @return 录制参数字符串
     */
    +(NSString* __nonnull)buildQiniuRecordParam:(BOOL)video bucketName:(NSString* __nonnull)bucketName secretKey:(NSString* __nonnull)secretKey accessKey:(NSString* __nonnull)accessKey fileName:(NSString* __nonnull)fileName;


录制参数构造之后，**在加入频道之前** 通过 JCMediaChannelJoinParam 对象中的 JCMediaChannelRecordParam 对象传入录制参数。

其中，JCMediaChannelRecordParam 对象有以下属性：

::

    /// 录制分辨率，参见 JCMediaChannelRecordResolution
    @property (nonatomic) JCMediaChannelRecordResolution resolution;

    /// 录制字符串
    @property (nonatomic, strong) NSString* __nonnull recoredString;


JCMediaChannelRecordResolution 枚举值如下
::

    /// 180p
    JCMediaChannelRecordResolution180p = 0x100,
    /// 360p
    JCMediaChannelRecordResolution360p = 0x200,
    /// 720p
    JCMediaChannelRecordResolution720p = 0x300,
    /// 1080p
    JCMediaChannelRecordResolution1080p = 0x400,

示例代码::

    // 设置录制参数
    JCMediaChannelJoinParam *joinParam = [[JCMediaChannelJoinParam alloc] init];
    joinParam.record = [[JCMediaChannelRecordParam alloc] init];
    joinParam.record.resolution = JCMediaChannelRecordResolution360p;
    joinParam.record.recoredString = [JCMediaChannelRecordParam buildQiniuRecordParam:true bucketName:@"QiNiuBucketName" secretKey:@"QiNiuSecretKey" accessKey:@"QiNiuAccessKey" fileName:@"QiNiuFilename"];
    // 加入频道
    [mediaChannel join:@"channelId" joinParam:joinParam];


.. note:: 

       如果进行音频录制，需要将构造方法中的 video 值设为 false。


获取录制状态
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

录制参数设置好后，需要根据目前的录制状态来判断是否启音视频录制。其中录制状态可通过 recordState 属性获得。

recordState 有：
::

    /// 无法进行视频录制
    JCMediaChannelRecordStateNone,
    /// 可以开启视频录制
    JCMediaChannelRecordStateReady,
    /// 视频录制中
    JCMediaChannelRecordStateRunning,

录制状态的变化通过 onMediaChannelPropertyChange 回调上报
::

    /**
     *  @brief 属性变化回调，目前主要关注屏幕共享状态的更新
     *  @param changeParam 变化标识集合
     */
    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam;


开启或关闭音视频录制
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

录制状态获取后，即可根据录制状态调用下面的接口开启或关闭音视频录制
::

    /**
     *  @brief 开关视频录制
     *  @param enable 是否开启屏幕录制
     *  @param recordParam 录制参数，当 enable 为 true 时，可以更改由 join 时传入的录制参数，不需更改则填 nil
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)enableRecord:(bool)enable recordParam:(JCMediaChannelRecordParam* __nullable)recordParam;

.. note::  
      
      recordParam 录制参数，当 enable 为 true 时，可以更改由 join 传入的录制参数，不需更改则填 nil。

示例代码::

    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam {
        if (changeParam.recordState) { // 录制状态变化
            // 根据音视频录制状态判断是否开启音视频录制
            if (mediaChannel.recordState == JCMediaChannelRecordStateNone) {
                // 无法进行音视频录制
            } else if (mediaChannel.recordState == JCMediaChannelRecordStateReady) {
                // 可以开启音视频录制
                [mediaChannel enableRecord:true recordParam:nil];
            } else if (mediaChannel.recordState == JCMediaChannelRecordStateRunning) {
                // 音视频录制中，可以关闭音视频录制
                [mediaChannel enableRecord:false recordParam:nil];
            }
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送消息(mac):


发送消息
----------------------

如果想在频道中给其他成员发送消息，可以调用下面的接口
::

    /**
     * @brief 发送消息
     *
     * @param type 消息类型
     * @param content 消息内容，当 toUserId 不为 nil 时，content 不能大于 4k
     * @param toUserId 接收者id，null则发给频道所有人员
     * @return 返回 true 表示成功，false表示失败
     */
    -(bool)sendMessage:(NSString * __nonnull)type content:(NSString * __nonnull)content toUserId:(NSString * __nullable)toUserId;

其中，消息类型（type）为自定义类型。

示例代码::
    
    -(void)onJoin:(bool)result reason:(JCMediaChannelReason)reason channelId:(NSString*)channelId {
        // 发送给所有成员
        [mediaChannel sendMessage:@"text" content:@"content" toUserId:nil];
        // 发送给某个成员
        [mediaChannel sendMessage:@"text" content:@"content" toUserId:@"接收者id"];
    }


当频道中的其他成员收到消息时，会收到 onMessageReceive 回调
::

    /**
     * @brief 接收频道消息的回调
     *
     * @param type 消息类型
     * @param content 消息内容
     * @param fromUserId    消息发送成员的userId
     */
    -(void)onMessageReceive:(NSString *)type content:(NSString *)content fromUserId:(NSString *)fromUserId;
