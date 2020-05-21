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

.. _屏幕共享(windows):

桌面或窗口共享
----------------------

桌面或窗口可以让您和频道中的其他成员一起分享设备里的精彩内容，您可以在频道中利用桌面或窗口共享的功能进行文档演示、在线教育演示、视频会议以及游戏过程分享等。

开启桌面或窗口共享前可以调用 :ref:`获取桌面列表<获取桌面列表(windows)>` 接口或者 :ref:`获取窗口列表<获取窗口列表(windows)>` 接口获取桌面列表和窗口列表。

获取桌面/窗口列表
>>>>>>>>>>>>>>>>>>>>>>>>>>>

开启桌面或窗口共享前，需要调用下面的接口获取桌面列表或者窗口列表：

- 获取窗口列表
::

    /// <summary>
    /// 窗口列表
    /// </summary>
    public List<JCMediaDeviceWindow> windowsDevices

其中，JCMediaDeviceWindow 有以下几个变量
::

    /// <summary>
    /// 窗口
    /// </summary>
    public class JCMediaDeviceWindow {
        /// <summary>
        /// 窗口名称
        /// </summary>
        public string windowName;
        /// <summary>
        /// 窗口id
        /// </summary>
        public string windowId;
    }

- 获取桌面列表
::

    /// <summary>
    /// 桌面列表
    /// </summary>
    public List<JCMediaDeviceDesktop> desktopDevices

其中，JCMediaDeviceDesktop 有以下几个变量
::

    /// <summary>
    /// 桌面
    /// </summary>
    public class JCMediaDeviceDesktop {
        /// <summary>
        /// 桌面名称
        /// </summary>
        public string desktopName;
        /// <summary>
        /// 桌面id
        /// </summary>
        public string desktopId;
    }


示例代码
::

    // 获取桌面列表
    List<JCMediaDeviceDesktop> desktopDevices = mediaDevice.desktopDevices;
    // 开启或关闭桌面或窗口共享
    mediaChannel.enableScreenOrWindowShare(mediaDevice.desktopDevices[0]);


屏幕共享采集属性设置
>>>>>>>>>>>>>>>>>>>>>>>>>>>

您可以调用 JCMediaDevice 类中的 setScreenCaptureProperty 方法设置屏幕共享采集属性，包括采集的高度、宽度和帧速率。
::

    /// <summary>
    /// 设置屏幕桌面共享采集属性
    /// </summary>
    /// <param name="width">采集宽度</param>
    /// <param name="height">采集高度</param>
    /// <param name="framerate">帧速率</param>
    public void setScreenCaptureProperty(int width, int height, int framerate)

.. note:: 该方法可以在开启屏幕共享前调用，也可以在屏幕共享中调用；如果在屏幕共享中调用，则设置的采集属性要在下次屏幕共享开启时生效。


开启或关闭屏幕共享
>>>>>>>>>>>>>>>>>>>>>>>>>>>

开启或关闭屏幕共享需要根据当前的屏幕共享状态进行判断，具体通过 screenUserId 进行判断。当 screenUserId 不为空时说明当前存在屏幕共享，不能再发起屏幕共享，只有当 screenUserId 为空时才可以发起屏幕共享。

屏幕共享状态是否变化通过 screenShare 判断。如果屏幕共享状态发生了改变会触发 onMediaChannelPropertyChange 回调
::

    /// <summary>
    /// 属性变化回调，目前主要关注屏幕共享和窗口共享状态的更新
    /// </summary>
    void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);

如果当前不存在屏幕共享或者自己发起了屏幕共享，可以调用下面的方法开启或关闭屏幕共享
::

    /// <summary>
    /// 开启关闭桌面屏幕共享，内部根据当前状态决定是否开启
    /// </summary>
    /// <param name="enable">是否开启屏幕共享</param>
    /// <param name="videoSource">桌面或窗口id</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool enableScreenOrWindowShare(bool enable, string videoSource);


请求屏幕共享的视频流
>>>>>>>>>>>>>>>>>>>>>>>>>>>

如果频道中有成员开启了屏幕共享，其他成员将收到 onMediaChannelPropertyChange 的回调，并通过 screenUserId 属性获得发起屏幕共享的用户标识。

获得发起屏幕共享的用户标识后，可以调用 requestScreenVideo 方法请求屏幕共享的视频流
::
    
    /// <summary>
    /// 请求屏幕共享的视频流
    /// 当pictureSize未None表示关闭请求
    /// </summary>
    /// <param name="screenUri">屏幕分享uri</param>
    /// <param name="pictureSize">视频请求尺寸类型</param>
    /// <returns>返回true表示调用成功，false表示调用失败</returns>
    public bool requestScreenVideo(string screenUri, JCMediaChannelPictureSize pictureSize)

示例代码

::

    public void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam) {
        if (propChangeParam.screenShare) {
            if (mediaChannel.screenUserId = nil) {
                // 开启屏幕共享
                mediaChannel.enableScreenOrWindowShare(true, videoSource);
                // 请求屏幕共享的视频流
                JCMediaDeviceVideoCanvas screenShare = mediaDevice.startVideo(mediaChannel.getScreenRenderId(), JCMediaDevice.RENDER_FULL_CONTENT);
                mediaChannel.requestScreenVideo(mediaChannel.getScreenRenderId(),JCMediaChannel.PICTURESIZE_LARGE);
            } else if (mediaChannel.screenUserId != nil && "自己开启了屏幕共享") {
                // 关闭屏幕共享
                mediaChannel.enableScreenOrWindowShare(false, videoSource);
            }
        }
    }



^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CDN 推流(windows):


CDN 推流
----------------------

CDN 推流服务适用于各类音视频直播场景，如企业级音视频会议、赛事、游戏直播、在线教育、娱乐直播等。

CDN 推流集成简单高效，开发者只需调用相关 API 即可将 CDN 推流无缝对接到自己的业务应用中。

推流地址设置
>>>>>>>>>>>>>>>>>>>>>>>>>>>

如要开启 CDN 推流，需在 **加入频道前** 进行 CDN 推流地址的设置。具体为通过 JOIN_PARAM_CDN 关键字进行配置

示例代码

::

    // 设置 CDN 推流地址
    Dictionary<string, string> joinparams = new Dictionary<string, string>();
    joinparams.Add(JCMediaChannelConstants.JOIN_PARAM_CDN, "your cdnurl");
    // 加入频道
    mediaChannel.join("channelId", joinparams);


CDN 状态获取
>>>>>>>>>>>>>>>>>>>>>>>>>>>

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
>>>>>>>>>>>>>>>>>>>>>>>>>>>

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


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _音视频录制(windows):

服务器音视频录制
------------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前设置录制参数，然后在加入频道的时候传入录制参数。

设置录制参数
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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

