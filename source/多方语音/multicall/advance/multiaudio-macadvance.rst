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

.. _音视频录制(mac):

服务器音频录制
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

    /**
     *  @brief 录制分辨率，参见 JCMediaChannelRecordResolution
     */
    @property (nonatomic) JCMediaChannelRecordResolution resolution;

    /// 录制字符串
    @property (nonatomic, strong) NSString* __nonnull recoredString;


示例代码::

    // 设置录制参数
    JCMediaChannelJoinParam *joinParam = [[JCMediaChannelJoinParam alloc] init];
    joinParam.record = [[JCMediaChannelRecordParam alloc] init];
    joinParam.record.recoredString = [JCMediaChannelRecordParam buildQiniuRecordParam:false bucketName:@"QiNiuBucketName" secretKey:@"QiNiuSecretKey" accessKey:@"QiNiuAccessKey" fileName:@"QiNiuFilename"];
    // 加入频道
    [mediaChannel join:@"channelId" joinParam:joinParam];

.. note:: 

       音频录制时，需要将构造方法中的 video 值设为 false。


获取录制状态
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

录制参数设置好后，需要根据目前的录制状态来判断是否启音视频录制。其中录制状态可通过 recordState 属性获得。

recordState 有：
::

    /// 无法进行音频录制
    JCMediaChannelRecordStateNone,
    /// 可以开启音频录制
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


开启或关闭音频录制
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

录制状态获取后，即可根据录制状态调用下面的接口开启或关闭音频录制
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
            // 根据音频录制状态判断是否开启音频录制
            if (mediaChannel.recordState == JCMediaChannelRecordStateNone) {
                // 无法进行音频录制
            } else if (mediaChannel.recordState == JCMediaChannelRecordStateReady) {
                // 可以开启音频录制
                [mediaChannel enableRecord:true recordParam:nil];
            } else if (mediaChannel.recordState == JCMediaChannelRecordStateRunning) {
                // 音频录制中，可以关闭音频录制
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

