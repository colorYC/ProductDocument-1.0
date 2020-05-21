iOS
=========================

.. _音视频录制(iOS):

.. highlight:: objective-c

**集成服务器音视频录制功能前，请确保您已经集成了基础的多方音视频通话功能。**

.. note:: SDK 不支持模拟器运行，请使用真机。


设置录制参数
------------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在 **加入频道之前** 通过 JCMediaChannelJoinParamRecord 关键字设置录制参数，然后在加入频道的时候传入录制参数。

示例代码::

    // 设置录制参数
    NSDictionary *paramDic = @{@"Protocol" : @"qiniu",
                                   @"AccessKey" : accessKey,
                                   @"SecretKey" : secretKey,
                                   @"BucketName" : bucketName,
                                   @"FileKey" : fileKey};
    NSDictionary *storageDic = @{@"MtcConfIsVideoKey" : @YES, @"Storage" : paramDic};
    [dic setObject:storageDic forKey:JCMediaChannelJoinParamRecord];
    // 加入频道
    [mediaChannel join:@"channelId" params:paramDic];


.. note:: 
    
       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       如果进行音频录制，需要将 MtcConfIsVideoKey 值设为 NO。即：NSDictionary *storageDic = @{@"MtcConfIsVideoKey" : @NO, @"Storage" : paramDic};


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