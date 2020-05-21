Android 进阶
=========================

.. highlight:: java

互动连麦
------------------------

观众可以在直播中向主播发送连麦互动请求，主播也可以对加入直播的观众发送连麦请求。连麦请求包括语音连麦和视频连麦。

连麦的流程如下：

.. image:: interactive.png

发送连麦请求
>>>>>>>>>>>>>>>>>>>>>>>>>>

通过发送消息功能实现连麦请求的发送，接口如下：

::

    /**
     * 发送消息
     *
     * @param type     消息类型
     * @param content  消息内容，当 toUserId 不为 null 时，content 不能大于 4k
     * @param toUserId 接收者id，null则发给频道所有人员
     * @return true表示成功，false表示失败
     */
    public abstract boolean sendMessage(String type, String content, String toUserId);


示例代码
::

    JCLive.get().mediaChannel.sendMessage(JCLiveConstants.MESSAGE_TYPE_MESSAGE, message, null)


接受连麦
>>>>>>>>>>>>>>>>>>>>>>>>>>

连麦消息发送后，主播会收到 onMessageReceive 回调

::

    /**
     * 接收频道消息的回调
     *
     * @param type          消息类型
     * @param content       消息内容
     * @param fromUserId    消息发送成员的userId
     */
    public void onMessageReceive(String type, String content, String fromUserId);

此时，主播可以根据消息类型（语音连麦或者视频连麦请求）以及消息发送的对象进行对应的处理。

如果是语音连麦，则主播同意后，连麦成员状态变为音频连麦状态，连麦成员会收到成员状态更新的回调，此时请求连麦的成员需要调用 enableUploadAudioStream 接口上传本地音频流。

如果是视频连麦，则主播同意后，连麦成员状态变为视频连麦状态，连麦成员会收到成员状态更新的回调，此时请求连麦的成员需要调用 enableUploadAudioStream 接口和 enableUploadVideoStream 接口上传本地音频流和视频流，同时主播需要调用 startVideo 接口和 requestVideo 接口创建连麦成员的视频画面。

主播同意连麦后，成员状态变为连麦互动状态。此时需要调用“设置状态”接口改变成员的状态。


设置状态
>>>>>>>>>>>>>>>>>>>>>>>>>>

当成员与主播进行音视频连麦、停止连麦以及连麦成员离开时，均需要调用下面的接口设置成员状态

::

    /**
     * 设置自定义状态
     *
     * @param customState 自定义状态, 参看 CustomState
     * @param participant 成员，null 则默认设置自己
     */
    public abstract void setCustomState(@CustomState int customState, JCMediaChannelParticipant participant);

状态值可以根据 CustomState 枚举值自定义， CustomState 有以下几种
::

    /**
     * 自定义状态0
     */
    public static final int CUSTOM_STATE_0 = 1<<12;
    /**
     * 自定义状态1
     */
    public static final int CUSTOM_STATE_1 = 1<<13;
    /**
     * 自定义状态2
     */
    public static final int CUSTOM_STATE_2 = 1<<14;
    /**
     * 自定义状态3
     */
    public static final int CUSTOM_STATE_3 = 1<<15;


例如
::

    //自定义音频连麦状态
    int STATE_AUDIO_JOIN = JCMediaChannel.CUSTOM_STATE_0;
    //自定义视频连麦状态
    int STATE_VIDEO_JOIN = JCMediaChannel.CUSTOM_STATE_1;


状态设置后，可以通过下面的方法获取自定义状态值
::

    /**
     * 获得自定义状态
     *
     * @return
     */
    public abstract @CustomState int getCustomState();


示例代码

::

        public void dealMessage(message) {  
            //主播接受音频连麦请求，取出消息类型和消息对象
            JCMediaChannelParticipant p = mediaChannel.getParticipant(message.fromUserId)
            if (p != null) {
                // 设置成员状态
                mediaChannel.setCustomState(STATE_AUDIO_JOIN, p)
            }
        }


成员状态设置后会收到 onParticipantUpdate 回调，该回调会上报变化的成员以及变化的属性
::

    /**
     * 成员更新回调
     *
     * @param participant 成员对象
     * @param changeParam 更新标识类
     */
    void onParticipantUpdate(JCMediaChannelParticipant participant, JCMediaChannelParticipant.ChangeParam changeParam);


示例代码
::

    public void onParticipantUpdate(JCMediaChannelParticipant participant, JCMediaChannelParticipant.ChangeParam changeParam) {
        /** custom state 变化处理自身的音视频上传状态 */
        if (participant.isSelf && participant.isAudience && changeParam.customState) {
            if (participant.isAudioJoiner) { // 音频加入
                mediaChannel.enableUploadVideoStream(false)
                mediaChannel.enableUploadAudioStream(true)
            } else { // 普通观众加入
                mediaChannel.enableUploadVideoStream(false)
                mediaChannel.enableUploadAudioStream(false)
            }
        }
    }


踢出成员
--------------------

直播中，主播可以调用下面的接口踢出成员
::

    /**
     * 将成员踢出会议
     *
     * @param participant 成员
     * @return true表示成功，false表示失败
     */
    public abstract boolean kick(JCMediaChannelParticipant participant);

示例代码
::

    mediaChannel.kick(partp)


.. _发送消息(android):

发送消息
----------------------

如果想在频道中给其他成员发送消息，可以调用下面的接口
::

    /**
     * 发送消息
     *
     * @param type     消息类型
     * @param content  消息内容，当 toUserId 不为 null 时，content 不能大于 4k
     * @param toUserId 接收者id，null则发给频道所有人员
     * @return true表示成功，false表示失败
     */
    public abstract boolean sendMessage(String type, String content, String toUserId);

其中，消息类型（type）为自定义类型。


示例代码::

    public void onJoin(boolean result, @JCMediaChannel.MediaChannelReason int reason, String channelId) {
        // 发送给所有成员
        mediaChannel.sendMessage("text", "content", null);
        // 发送给某个成员
        mediaChannel.sendMessage("text", "content", "userId");
    }


当频道中的其他成员收到消息时会收到 onMessageReceive 回调
::

    /**
     * 接收频道消息的回调
     *
     * @param type          消息类型
     * @param content       消息内容
     * @param fromUserId    消息发送成员的userId
     */
    public void onMessageReceive(String type, String content, String fromUserId);
