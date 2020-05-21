Android 进阶
=========================

.. highlight:: java

**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(Context, "your appkey", this, null);
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
    JCMediaChannel mediaChannel = JCMediaChannel.create(client, mediaDevice, this);

.. note:: SDK 不支持模拟器运行，请使用真机。

.. _查询频道(android):

查询频道
---------------------------

如需查询频道相关信息，例如频道名称、是否存在、成员名、成员数，可以调用 query 接口进行查询操作
::

    /**
     * 查询频道相关信息，例如是否存在，人数等
     *
     * @param channelId 频道标识
     * @return          返回操作id，与 onQuery 回调中的 operationId 对应
     */
    public abstract int query(String channelId);

示例代码::

    mediaChannel.query("channelId");

查询操作发起后，UI 通过以下方法监听回调查询的结果：
::

    /**
     * 查询频道信息结果回调
     *
     * @param operationId 操作id，由 query 接口返回
     * @param result      查询结果，true 表示查询成功，false 表示查询失败
     * @param reason      查询失败原因，当 result 为 false 时该值有效
     * @param queryInfo   查询到的频道信息
     */
    public void onQuery(int operationId, boolean result, @JCMediaChannel.MediaChannelReason int reason, JCMediaChannelQueryInfo queryInfo);

示例代码::

    public void onQuery(int operationId, boolean result, @JCMediaChannel.MediaChannelReason int reason, JCMediaChannelQueryInfo queryInfo) {
       // 查询成功
       if (result) {
            // 频道标识
            String channelId = queryInfo.channelId;
            // 频道
            int number = queryInfo.number;
            // 频道成员数
            int clientCount = queryInfo.clientCount;
            // 频道成员列表
            List<String>  members = queryInfo.members;
       } else {
            // 查询失败
       }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _音频录制(android):

服务器音频录制
----------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前通过 JOIN_PARAM_RECORD 关键字设置录制参数，然后在加入频道的时候传入录制参数。

录制参数设置好后，需要根据目前的录制状态来判断是否启音频录制。其中录制状态可通过 recordState 属性获得。

recordState 有：
::

    /**
     * 无法进行录制
     */
    public static final int RECORD_STATE_NONE = 0;
    /**
     * 可以开启录制
     */
    public static final int RECORD_STATE_READY = 1;
    /**
     * 音频录制中
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

录制状态获取后，即可调用下面的接口开启或关闭音频录制
::

    /**
     * 开关视频录制
     * @param enable 是否开启视频录制
     *
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableRecord(boolean enable);


示例代码::

    // 设置录制参数
    Map<String, String> param = new HashMap<>();
    param.put(JCMediaChannel.JOIN_PARAM_RECORD, JCConfUtils.qiniuRecordParam(fase, bucketName, secretKey, accessKey, fileName));
    // 加入频道
    mediaChannel.join("channelId", param);
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

.. note:: 

       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       **如果想进行音频录制，需要将第一个参数设为 false**，即 param.put(JCMediaChannel.JOIN_PARAM_RECORD, JCConfUtils.qiniuRecordParam(false, bucketName, secretKey, accessKey, fileName));


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

