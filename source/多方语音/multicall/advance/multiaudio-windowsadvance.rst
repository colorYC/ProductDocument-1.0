Windows 进阶
==========================

.. highlight:: csharp

**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(app, "your appkey", this, null);           
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);             
    JCMediaChannel mediaChannel = JCMediaChannel.create(client, mediaDevice, this);

.. _查询频道(windows):

查询频道
---------------------------

如需查询频道相关信息，例如频道名称、是否存在、成员名、成员数，可以调用 query 接口进行查询操作
::

    /// <summary>
    /// 查询频道相关信息，例如是否存在、人数等
    /// </summary>
    /// <param name="channelId">频道标识</param>
    /// <returns>操作Id，与onQuery回调中的operationI对应</returns>
    public int query(string channelId)

示例代码

::

    mediaChannel.query("channelId");


查询操作发起后，UI 通过以下方法监听回调查询的结果：
::

    /// <summary>
    /// 查询频道信息结果回调
    /// </summary>
    /// <param name="operationId">操作id，由query接口返回</param>
    /// <param name="result">ture表示查询成功，false表示查询失败</param>
    /// <param name="reason">查询失败原因，当result为false时该值有效</param>
    /// <param name="queryInfo">查询到的频道信息</param>
    public void onQuery(int operationId, bool result, JCMediaChannelReason reason, JCMediaChannelQueryInfo queryInfo);

示例代码::

    public void onQuery(int operationId, bool result, JCMediaChannelReason reason, JCMediaChannelQueryInfo queryInfo) {
       // 查询成功
       if (result) {
            // 查询频道标识
            String channelId = queryInfo.channelId;
            // 查询频道号
            int number = queryInfo.number;
            // 查询频道成员数
            int clientCount = queryInfo.clientCount;
            // 查询频道成员列表
            List<string> members = queryInfo.members;
       } else {
            // 查询失败
       }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _音频录制(windows):

服务器音频录制
------------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前通过 JOIN_PARAM_RECORD 关键字设置录制参数，然后在加入频道的时候传入录制参数。

录制参数设置好后，需要根据目前的录制状态来判断是否启音频录制。其中录制状态可通过 recordState 属性获得。

recordState 有::

    /// 无法进行录制
    None,
    /// 可以开启录制
    Ready,
    /// 音频录制中
    Running


录制状态的变化通过 onMediaChannelPropertyChange 回调上报
::

    /// <summary>
    /// 属性变化回调，目前主要关注屏幕共享和窗口共享状态的更新
    /// </summary>
    void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);

录制状态获取后，即可调用下面的接口开启或关闭音频录制
::

    /// <summary>
    /// 开关视频录制，内部根据当前状态决定是否开启
    /// <param name="enable">是否开启屏幕录制</param>
    /// </summary>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableRecord(bool enable)
   

示例代码

::

    // 设置录制参数
    Dictionary<string, string> joinparams = new Dictionary<string, string>();
    joinparams.Add(JCMediaChannelConstants.JOIN_PARAM_RECORD, "{\"MtcConfIsVideoKey\":\"false\",
                 \"Storage\":{
                 \"Protocol\":\"qiniu\",
                 \"BucketName\": \"用户填入\",
                 \"SecretKey\": \"用户填入\",
                 \"AccessKey\": \"用户填入\", 
                 \"FileKey\": \" * *.mp4\"
                 }
             }"
        );
    string recordParam = JCConfUtils.qiniuRecordParam(Properties.Settings.Default.RecordVideo, Properties.Settings.Default.RecordBucketName, Properties.Settings.Default.RecordSecretKey, Properties.Settings.Default.RecordAccessKey, Properties.Settings.Default.RecordFileName);
    joinparams.Add(JCMediaChannelConstants.JOIN_PARAM_REGION, JCMediaChannelRegion.REGION_CHINA.ToString());
    // 加入频道
    mediaChannel.join("channelId", joinparams);
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

.. note:: 
    
       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       **如果进行音频录制，需要将 MtcConfIsVideoKey 值设为 false**。


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _发送消息(windows):


发送消息
-------------------------

如果想在频道中给其他成员发送消息，可以调用下面的接口
::

    /// <summary>
    /// 频道中发送消息，当 toUserId 不为 null 时，content 不能大于 4k
    /// </summary>
    /// <param name="type">消息类型</param>
    /// <param name="content">消息内容</param>
    /// <param name="toUserId">接收方成员的userid，值为null发送给所有人</param>
    /// <returns>是否发送成功</returns>
    public bool sendMessage(string type,string content,string toUserId)

其中，消息类型（type）为自定义类型。

示例代码

::

    public void onJoin(bool result, JCMediaChannelReason reason, string channelId) {
        // 发送给所有成员
        mediaChannel.sendMessage("text", "content", null);
        // 发送给某个成员
        mediaChannel.sendMessage("text", "content", "userId");
    }


当频道中的其他成员收到消息时，会收到 onMessageReceive 回调
::

    /// <summary>
    /// 接收频道消息的回调
    /// </summary>
    /// <param name="type">消息类型</param>
    /// <param name="content">消息内容</param>
    /// <param name="fromUserId">消息发送成员userId</param>
    public void onMessageReceive(string type, string content, string fromUserId);
