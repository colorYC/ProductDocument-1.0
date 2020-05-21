Windows 进阶
=========================

.. highlight:: csharp

互动连麦
------------------------------

在语音直播中，观众可以向主播发送语音连麦互动请求，主播也可以对加入直播的观众发送语音连麦请求。

语音连麦的流程如下图所示：

.. image:: audioConnect.png

发送连麦请求
>>>>>>>>>>>>>>>>>>>>>>>>>>>>

通过发送消息功能实现连麦请求的发送，接口如下：
::

        /// <summary>
        /// 频道中发送消息，当 toUserId 不为 null 时，content 不能大于 4k
        /// </summary>
        /// <param name="type">消息类型</param>
        /// <param name="content">消息内容</param>
        /// <param name="toUserId">接收方成员的userid，值为null发送给所有人</param>
        /// <returns>是否发送成功</returns>
        public bool sendMessage(string type, string content, string toUserId)


示例代码
::

    mediaChannel.sendMessage("request_video", "消息内容", "主播id");


接受连麦
>>>>>>>>>>>>>>>>>>>>>>>>>>>>

连麦消息发送后，主播会收到 onMessageReceive 回调

::

        /// <summary>
        /// 接收频道消息的回调
        /// </summary>
        /// <param name="type">消息类型</param>
        /// <param name="content">消息内容</param>
        /// <param name="fromUserId">消息发送成员userId</param>
        void onMessageReceive(string type, string content, string fromUserId);


此时，主播可以根据消息类型（开发者可以自定义消息类型字符串）以及消息发送的对象进行对应的处理。

如果是语音连麦，则主播同意后，连麦成员会收到成员状态更新的回调，此时请求连麦的成员需要调用 enableUploadAudioStream 接口上传本地音频流。

主播同意连麦后，成员状态变为连麦互动状态。此时需要调用“设置状态”接口改变成员的状态。


设置状态
>>>>>>>>>>>>>>>>>>>>>>>>>>>>

当成员与主播进行音频连麦、停止连麦以及连麦成员离开时，均需要调用下面的接口设置成员状态

::

        /// <summary>
        /// 设置成员状态
        /// </summary>
        /// <param name="customState"></param>
        /// <param name="participant"></param>
        public void setCustomState(int customState, JCMediaChannelParticipant participant)


状态值可以根据 JCMediaChannelCustomState 枚举值自定义，JCMediaChannelCustomState 有以下几种
::

        /// <summary>
        /// 自定义状态0
        /// </summary>
        CUSTOM_STATE_0 = 1 << 12,
        /// <summary>
        /// 自定义状态1
        /// </summary>
        CUSTOM_STATE_1 = 1 << 13,
        /// <summary>
        /// 自定义状态2
        /// </summary>
        CUSTOM_STATE_2 = 1 << 14,
        /// <summary>
        /// 自定义状态3
        /// </summary>
        CUSTOM_STATE_3 = 1 << 15

例如
::

    //自定义音频连麦状态
    JCMediaChannelCustomState STATE_AUDIO_JOIN = JCMediaChannelConstants.CUSTOM_STATE_0;


状态设置后，可以通过下面的方法获取自定义状态值
::

        /// <summary>
        /// 获取自定义状态
        /// </summary>
        /// <returns></returns>
        public int getCustomState()


示例代码

::

    //自定义音频连麦状态
    JCMediaChannelCustomState STATE_AUDIO_JOIN = JCMediaChannelConstants.CUSTOM_STATE_0;
    //收到连麦请求信息
    public void mediaChannelMessage(NSNotification note) {
      //取出消息类型
      //取出消息发送者id
      //获取发送者对象
      JCMediaChannelParticipant partp = mediaChannel.getParticipant("消息发送者id");
      if (// 主播同意音频连麦) { 
          if partp != nil {
              //设置成员状态
              mediaChannel.setCustomState(STATE_AUDIO_JOIN, partp);
          }
      }
    }


成员状态设置后会收到 onParticipantUpdate 回调，该回调会上报变化的成员以及变化的属性
::

        /// <summary>
        /// 成员更新回调
        /// </summary>
        /// <param name="participant">成员对象</param>
        /// <param name="changeParam">更新标识类</param>
        void onParticipantUpdate(JCMediaChannelParticipant participant, JCMediaChannelParticipant.ChangeParam changeParam);


示例代码
::

    public void onParticipantUpdate(JCMediaChannelParticipant participant, JCMediaChannelParticipant.ChangeParam changeParam) {
        if (participant.isSelf && participant.isAudience && participantChangeParam.customState) {
            if (participant.isAudioJoiner) { //音频连麦成员
                mediaChannel.enableUploadVideoStream(false);
                mediaChannel.enableUploadAudioStream(true);
            } else { //普通成员
                mediaChannel.enableUploadVideoStream(false);
                mediaChannel.enableUploadAudioStream(false);
            }
        }


踢出成员
------------------------

直播中，主播可以调用下面的接口踢出成员
::

        /// <summary>
        /// 踢出成员
        /// </summary>
        /// <param name="participant"></param>
        /// <returns></returns>
        public bool kick(JCMediaChannelParticipant participant)


示例代码
::

    mediaChannel.kick(partp);


.. _发送消息(iOS):

发送消息
----------------------

如果想在直播中发送消息，可以调用下面的接口
::

        /// <summary>
        /// 频道中发送消息，当 toUserId 不为 null 时，content 不能大于 4k
        /// </summary>
        /// <param name="type">消息类型</param>
        /// <param name="content">消息内容</param>
        /// <param name="toUserId">接收方成员的userid，值为null发送给所有人</param>
        /// <returns>是否发送成功</returns>
        public bool sendMessage(string type, string content, string toUserId)


其中，消息类型（type）为自定义类型。


示例代码::
    
    public void sendMessage() {
        // 发送给所有成员
        mediaChannel.sendMessage("text", "content", null);
        // 发送给某个成员
        mediaChannel.sendMessage("text", "content", "接收者id");
    }


当频道中的其他成员收到消息时，会收到 onMessageReceive 回调
::

        /// <summary>
        /// 接收频道消息的回调
        /// </summary>
        /// <param name="type">消息类型</param>
        /// <param name="content">消息内容</param>
        /// <param name="fromUserId">消息发送成员userId</param>
        void onMessageReceive(string type, string content, string fromUserId);