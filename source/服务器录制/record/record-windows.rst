Windows
==========================

.. _音视频录制(windows):

.. highlight:: csharp

**集成服务器音视频录制功能前，请确保您已经集成了基础的音视频通话功能。**


设置录制参数
-------------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前通过 JOIN_PARAM_RECORD 关键字设置录制参数，然后在加入频道的时候传入录制参数。

示例代码
::

    // 设置录制参数
    JObject json = new JObject();
    json.Add("MtcConfIsVideoKey", true);
    JObject storageObj = new JObject();
    storageObj.Add("Protocol", "qiniu");
    storageObj.Add("BucketName", bucketName);
    storageObj.Add("SecretKey", secretKey);
    storageObj.Add("AccessKey", accessKey);
    storageObj.Add("FileKey", fileName);
    json.Add("Storage", storageObj);

    Dictionary<string, string> joinparams = new Dictionary<string, string>();
    joinparams.Add(JCMediaChannelConstants.JOIN_PARAM_RECORD, json.ToString());
    // 加入频道
    mediaChannel.join("channelId", joinparams);


.. note:: 
    
       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       如果进行音频录制，需要将 MtcConfIsVideoKey 值设为 false。


获取录制状态
--------------------------

录制参数设置好后，需要根据目前的录制状态来判断是否启音视频录制。其中录制状态可通过 recordState 属性获得。

recordState 有::

    /// 无法进行视频录制
    None,
    /// 可以开启视频录制
    Ready,
    /// 视频录制中
    Running

录制状态的变化通过 onMediaChannelPropertyChange 回调上报
::

        /// <summary>
        /// 属性变化回调，目前主要关注屏幕共享和窗口共享状态的更新
        /// </summary>
        void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);


开启或关闭音视频录制
-------------------------

录制状态获取后，即可根据录制状态调用下面的接口开启或关闭音视频录制
::

    /// <summary>
    /// 开关视频录制，内部根据当前状态决定是否开启
    /// <param name="enable">是否开启屏幕录制</param>
    /// </summary>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableRecord(bool enable)
   

示例代码
::

    public void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam) {
        if (changeParam.recordState) { // 录制状态变化 {
            // 根据音视频录制状态判断是否开启音视频录制
            if (mediaChannel.recordState == JCMediaChannelRecordState.None) {
                // 无法进行音视频录制
            } else if (mediaChannel.recordState == JCMediaChannelRecordState.Ready) {
                // 可以开启音视频录制
                mediaChannel.enableRecord(true);
            } else if (mediaChannel.recordState == JCMediaChannelRecordState.Running) {
                // 音视频录制中，可以关闭音视频录制
                mediaChannel.enableRecord(false);
            }
        }
    }

