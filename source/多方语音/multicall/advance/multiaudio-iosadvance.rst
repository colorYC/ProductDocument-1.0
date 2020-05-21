iOS 进阶
=========================

.. highlight:: objective-c

**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCMediaDevice *mediaDevice = [JCMediaDevice create:client callback:self];
    JCMediaChannel *mediaChannel = [JCMediaChannel create:client mediaDevice:mediaDevice callback:self];

.. note:: SDK 不支持模拟器运行，请使用真机。

.. _查询频道(iOS):

查询频道
---------------------------

如需查询频道相关信息，例如频道名称、是否存在、成员名、成员数，可以调用 query 接口进行查询操作
::

    /**
     *  @brief              查询媒体频道相关信息，例如是否存在，人数等
     *  @param channelId    媒体通道标识
     *  @return             返回操作id，与 onQuery 回调中的 operationId 对应
     */
    -(int)query:(NSString*)channelId;

示例代码::

    [mediaChannel query:@"channelId"];

查询操作发起后，UI 通过以下方法监听回调查询的结果：
::

    /**
     *  @brief                  查询媒体频道信息结果回调
     *  @param operationId      操作id，由 query 接口返回
     *  @param result           true 表示查询成功，false 表示查询失败
     *  @param reason           查询失败原因 当 result 为 false 时该值有效
     *  @param queryInfo        查询到的信息
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

.. _音频录制(iOS):

服务器音频录制
----------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在 加入频道之前 通过 JCMediaChannelJoinParamRecord 关键字设置录制参数，然后在加入频道的时候传入录制参数。

示例代码::

    // 设置录制参数
    NSDictionary *paramDic = @{@"Protocol" : @"qiniu",
                                   @"AccessKey" : accessKey,
                                   @"SecretKey" : secretKey,
                                   @"BucketName" : bucketName,
                                   @"FileKey" : fileKey};
    NSDictionary *storageDic = @{@"MtcConfIsVideoKey" : @NO, @"Storage" : paramDic};
    [dic setObject:storageDic forKey:JCMediaChannelJoinParamRecord];
    // 加入频道
    [mediaChannel join:@"channelId" params:paramDic];


.. note:: 
    
       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       **如果进行音频录制，需要将 MtcConfIsVideoKey 值设为 NO**。即：NSDictionary *storageDic = @{@"MtcConfIsVideoKey" : @NO, @"Storage" : paramDic};


录制参数设置好后，需要根据目前的录制状态来判断是否启音频录制。其中录制状态可通过 recordState 属性获得。

recordState 有：
::

    /// 无法进行录制
    JCMediaChannelRecordStateNone,
    /// 可以开启录制
    JCMediaChannelRecordStateReady,
    /// 音频录制中
    JCMediaChannelRecordStateRunning,

录制状态的变化通过 onMediaChannelPropertyChange 回调上报
::

    /**
     *  @brief 属性变化回调，目前主要关注屏幕共享状态的更新
     *  @param changeParam 变化标识集合
     */
    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam;

录制状态获取后，即可调用下面的接口开启或关闭音频录制
::

    /**
     *  @brief 开关视频录制
     *  @param enable 是否开启屏幕录制
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)enableRecord:(bool)enable;


示例代码::

    -(void)onMediaChannelPropertyChange:(JCMediaChannelPropChangeParam *)changeParam {
        if (changeParam.recordState) { // 录制状态变化
            // 根据音视频录制状态判断是否开启音视频录制
            if (mediaChannel.recordState == JCMediaChannelRecordStateNone) {
                // 无法进行音视频录制
            } else if (mediaChannel.recordState == JCMediaChannelRecordStateReady) {
                // 可以开启音视频录制
                [mediaChannel enableRecord:true];
            } else if (mediaChannel.recordState == JCMediaChannelRecordStateRunning) {
                // 音视频录制中，可以关闭音视频录制
                [mediaChannel enableRecord:false];
            }
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送消息(iOS):


发送消息
----------------------

如果想在频道中给其他成员发送消息，可以调用下面的接口
::

    /**
     * @brief 发送消息
     *
     * @param type       消息类型
     * @param content    消息内容，当 toUserId 不为 nil 时，content 不能大于 4k
     * @param toUserId   接收者id，null则发给频道所有人员
     * @return           返回 true 表示成功，false表示失败
     */
    -(bool)sendMessage:(NSString *)type content:(NSString *)content toUserId:(NSString *)toUserId;

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
     * @brief                接收频道消息的回调
     *
     * @param type           消息类型
     * @param content        消息内容
     * @param fromUserId     消息发送成员的userId
     */
    -(void)onMessageReceive:(NSString *)type content:(NSString *)content fromUserId:(NSString *)fromUserId;
