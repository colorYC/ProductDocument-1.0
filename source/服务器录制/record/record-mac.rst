macOS
=========================

.. highlight:: objective-c

**集成服务器音视频录制功能前，请确保您已经集成了基础的多方音视频通话功能。**

设置录制参数
------------------------

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
------------------------

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
------------------------

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
