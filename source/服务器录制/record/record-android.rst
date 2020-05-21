Android
=========================

.. _音视频录制(android):

.. highlight:: java

**集成服务器音视频录制功能前，请确保您已经集成了基础的音视频通话功能。**

.. note:: SDK 不支持模拟器运行，请使用真机。

设置录制参数
--------------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前通过 JOIN_PARAM_RECORD 关键字设置录制参数，然后在加入频道的时候传入录制参数。

示例代码::

    // 设置录制参数
    Map<String, String> param = new HashMap<>();
    param.put(JCMediaChannel.JOIN_PARAM_RECORD, JCConfUtils.qiniuRecordParam(true, bucketName, secretKey, accessKey, fileName));

.. note:: 

       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       如果想进行语音录制，需要将第一个参数设为 false，即 param.put(JCMediaChannel.JOIN_PARAM_RECORD, JCConfUtils.qiniuRecordParam(false, bucketName, secretKey, accessKey, fileName));


获取录制状态
------------------------

录制参数设置好后，需要根据目前的录制状态来判断是否启音视频录制。其中录制状态可通过 recordState 属性获得。

recordState 有：
::

    /**
     * 无法进行视频录制
     */
    public static final int RECORD_STATE_NONE = 0;
    /**
     * 可以开启视频录制
     */
    public static final int RECORD_STATE_READY = 1;
    /**
     * 视频录制中
     */
    public static final int RECORD_STATE_RUNNING = 2;


录制状态的变化通过 onMediaChannelPropertyChange 回调上报
::

    /**
     * 属性变化回调，目前主要关注屏幕共享状态的更新
     *
     * @param propChangeParam 变化标识集合
     */
    void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);


开启或关闭音视频录制
--------------------------

录制状态获取后，即可根据录制状态调用下面的接口开启或关闭音视频录制
::

    /**
     * 开关视频录制
     * @param enable 是否开启视频录制
     *
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableRecord(boolean enable);


示例代码::


    public void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam) {
        if (changeParam.recordState) { // 录制状态变化
            // 根据音视频录制状态判断是否开启音视频录制
            if (mediaChannel.getRecordState() = JCMediaChannel.RECORD_STATE_NONE) {
                // 无法进行音视频录制
            } else if (mediaChannel.getRecordState() = JCMediaChannel.RECORD_STATE_READY) {
                // 可以开启音视频录制
                mediaChannel.enableRecord(true);
            } else if (mediaChannel.getRecordState() = JCMediaChannel.RECORD_STATE_RUNNING) {
                // 音视频录制中，可以关闭音视频录制
                mediaChannel.enableRecord(false);
            }
        }
    }

