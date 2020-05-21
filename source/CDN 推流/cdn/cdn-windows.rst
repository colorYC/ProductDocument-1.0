Windows
==========================

.. _CDN 推流(windows):

.. highlight:: csharp

**集成 CDN 推流功能前，请确保您已经集成了基础的多方音视频通话功能。**

CDN 推流服务适用于各类音视频直播场景，如企业级音视频会议、赛事、游戏直播、在线教育、娱乐直播等。

CDN 推流集成简单高效，开发者只需调用相关 API 即可将 CDN 推流无缝对接到自己的业务应用中。

推流地址设置
--------------------------

如要开启 CDN 推流，需在 **加入频道前** 进行 CDN 推流地址的设置。具体为通过 JOIN_PARAM_CDN 关键字进行配置

示例代码

::

    // 设置 CDN 推流地址
    Dictionary<string, string> joinparams = new Dictionary<string, string>();
    joinparams.Add(JCMediaChannelConstants.JOIN_PARAM_CDN, "your cdnurl");
    // 加入频道
    mediaChannel.join("channelId", joinparams);


CDN 状态获取
-----------------------------

开启 CDN 推流前需要判断 CDN 的状态，通过 cdnState 属性获取推流器状态。只有 CDN 当前状态不为 JCMediaChannelCdnStateNone 时才可以进行 CDN 推流。其中，CDN 推流状态有以下几种：
::

    /// 无法进行CDN推流
    None,
    /// 可以开启CDN推流
    Ready,
    /// CDN推流中
    Running


CDN 状态的变化通过 onMediaChannelPropertyChange 回调上报
::

        /// <summary>
        /// 属性变化回调，目前主要关注屏幕共享和窗口共享状态的更新
        /// </summary>
        void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);


开启或关闭 CDN 推流
--------------------------

CDN 状态获取后，即可根据 CDN 的状态开启或关闭 CDN 推流，接口如下
::

    /// <summary>
    /// 开关Cdn推流，内部根据当前状态决定是否开启
    /// 在收到onMediaChannelPropertyChange回调时检查cdnState
    /// </summary>
    /// <param name="enable">是否开启cdn推流</param>
    /// <param name="keyInterval">推流关键帧间隔(毫秒)，当 enable 为 true 时有效，-1表示使用默认值(5000毫秒)</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableCdn(bool enable, int keyInterval)

示例代码

::

    public void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam) {
        if (propChangeParam.cdnState) { // CDN 状态变化
            // 根据CDN推流状态判断是否开启推流
            if (mediaChannel.cdnState == JCMediaChannelCdnState.None) {
                // 无法使用 CDN 推流
            } else if (mediaChannel.cdnState == JCMediaChannelCdnState.Ready) {
                // 可以开启 CDN 推流
                mediaChannel.enableCdn(true);
            } else if (mediaChannel.cdnState == JCMediaChannelCdnState.Running) {
                // CDN 推流中，可以关闭 CDN 推流
                mediaChannel.enableCdn(false);
            }
        }
    }