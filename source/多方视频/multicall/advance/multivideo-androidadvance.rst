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

.. _屏幕共享(android):

屏幕共享
----------------------

屏幕共享可以让您和频道中的其他成员一起分享设备里的精彩内容，您可以在频道中利用屏幕共享的功能进行文档演示、在线教育演示、视频会议以及游戏过程分享等。

.. note:: 发起屏幕共享需要 Android 5.0 及以上。


屏幕共享采集属性设置
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

您可以调用 JCMediaDevice 类中的 setScreenCaptureProperty 方法设置屏幕共享采集属性，包括采集的高度、宽度和帧速率。
::

    /**
     * 设置屏幕共享采集属性
     * @param width     采集宽度，默认640
     * @param height    采集高度，默认360
     * @param frameRate 采集帧速率，默认10
     */
    public abstract void setScreenCaptureProperty(int width, int height, int frameRate);

.. note:: 该方法可以在开启屏幕共享前调用，也可以在屏幕共享中调用；如果在屏幕共享中调用，则设置的采集属性要在下次屏幕共享开启时生效。


开启或关闭屏幕共享
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

开启或关闭屏幕共享需要根据当前的屏幕共享状态进行判断，具体通过 screenUserId 进行判断。当 screenUserId 不为空时说明当前存在屏幕共享，不能再发起屏幕共享，只有当 screenUserId 为空时才可以发起屏幕共享。

屏幕共享状态是否变化通过 screenShare 判断。如果屏幕共享状态发生了改变会触发 onMediaChannelPropertyChange 回调
::

    /**
     * 属性变化回调，目前主要关注屏幕共享状态的更新
     *
     * @param propChangeParam 变化标识集合
     */
    void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);

如果当前不存在屏幕共享或者自己发起了屏幕共享，可以调用下面的方法开启或关闭屏幕共享
::

    /**
     * 开关屏幕分享
     * @param enable 是否开启屏幕分享
     *
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableScreenShare(boolean enable);

.. note::  

          屏幕共享发送方需要在 manifest 文件中做以下声明，否则无法发送本地视频桌面的视频流::

           <activity
                   android:name = "com.justalk.cloud.zmf.ZmfActivity"
                   android:theme = "@android:style/Theme.Dialog"/>


请求屏幕共享的视频流
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

如果频道中有成员开启了屏幕共享，其他成员将收到 onMediaChannelPropertyChange 的回调，并通过 getScreenUserId 属性获得发起屏幕共享的用户标识。

获得发起屏幕共享的用户标识后，可以调用 requestScreenVideo 方法请求屏幕共享的视频流
::

    /**
     * 请求屏幕共享的视频流
     * 当 pictureSize 为 JCMediaChannelPictureSizeNone 表示关闭请求
     *
     * @param screenUri     屏幕分享uri
     * @param pictureSize   视频请求尺寸类型
     * @return              返回 true 表示正常执行调用流程，false 表示调用异常
     * @see JCMediaChannel.PictureSize
     */
    public abstract boolean requestScreenVideo(String screenUri, @PictureSize int pictureSize);



示例代码::

    public void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam) {
        if (propChangeParam.screenShare) {
            if (mediaChannel.screenUserId = nil) {
                // 开启屏幕共享
                mediaChannel.enableScreenShare(true);
                // 请求屏幕共享的视频流
                JCMediaDeviceVideoCanvas screenShare = mediaDevice.startVideo(mediaChannel.getScreenRenderId(), JCMediaDevice.RENDER_FULL_CONTENT);
                mediaChannel.requestScreenVideo(mediaChannel.getScreenRenderId(),JCMediaChannel.PICTURESIZE_LARGE);
            } else if (mediaChannel.screenUserId != nil && "自己开启了屏幕共享") {
                // 关闭屏幕共享
                mediaChannel.enableScreenShare(false);
            }
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _CDN 推流(android):

CDN 推流
----------------------

CDN 推流服务适用于各类音视频直播场景，如企业级音视频会议、赛事、游戏直播、在线教育、娱乐直播等。

CDN 推流集成简单高效，开发者只需调用相关 API 即可将 CDN 推流无缝对接到自己的业务应用中。

推流地址设置
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

如要开启 CDN 推流，需在 **加入频道前** 进行 CDN 推流地址的设置。具体为通过 JOIN_PARAM_CDN 关键字进行配置

示例代码
::

    // 设置 CDN 推流地址
    Map<String, String> param = new HashMap<>();
    param.put(JCMediaChannel.JOIN_PARAM_CDN, cdnAddress);
    // 加入频道
    mediaChannel.join("channelId", param);


CDN 状态获取
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

开启 CDN 推流前需要判断 CDN 的状态，通过 cdnState 属性获取推流器状态。只有 CDN 当前状态不为 JCMediaChannelCdnStateNone 时才可以进行 CDN 推流。其中，CDN 推流状态有以下几种：
::

    // 无法进行CDN推流
    public static final int CDN_STATE_NONE = 0;
    // 可以开启CDN推流
    public static final int CDN_STATE_READY = 1;
    // CDN推流中
    public static final int CDN_STATE_RUNNING = 2;

CDN 状态的变化通过 onMediaChannelPropertyChange 回调上报
::

    /**
     * 属性变化回调，目前主要关注屏幕共享状态的更新
     *
     * @param propChangeParam 变化标识集合
     */
    void onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam);


开启或关闭 CDN 推流
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

CDN 状态获取后，即可根据 CDN 的状态开启或关闭 CDN 推流，接口如下
::

    /**
     * 开关Cdn推流
     * 在收到 onMediaChannelPropertyChange 回调后检查是否开启
     *
     * @param enable       是否开启Cdn推流
     * @param keyInterval  推流关键帧间隔(毫秒)，当 enable 为 true 时有效，-1表示使用默认值(5000毫秒)，有效值需要>=1000
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean enableCdn(boolean enable, int keyInterval);


示例代码
::

    public onMediaChannelPropertyChange(JCMediaChannel.PropChangeParam propChangeParam) {
        if (propChangeParam.cdnState) { // CDN 状态变化
            // 根据CDN推流状态判断是否开启推流
            if (mediaChannel.getCdnState() = JCMediaChannel.CDN_STATE_NONE) {
                // 无法使用 CDN 推流
            } else if (mediaChannel.getCdnState() == JCMediaChannel.CDN_STATE_READY) {
                // 可以开启 CDN 推流
                mediaChannel.enableCdn(true, 0);
            } else if (mediaChannel.getCdnState() == JCMediaChannel. CDN_STATE_RUNNING) {
                // CDN 推流中，可以关关闭 CDN 推
                mediaChannel.enableCdn(false, 0);
            }
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _音视频录制(android):

服务器音视频录制
----------------------

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前设置录制参数，然后在加入频道的时候传入录制参数。

设置录制参数
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

服务器音频视频录制将录制的文件保存在七牛云上，因此，如果需要进行服务器音视频录制，需要在加入频道之前通过 JOIN_PARAM_RECORD 关键字设置录制参数，然后在加入频道的时候传入录制参数。

示例代码::

    // 设置录制参数
    Map<String, String> param = new HashMap<>();
    param.put(JCMediaChannel.JOIN_PARAM_RECORD, JCConfUtils.qiniuRecordParam(true, bucketName, secretKey, accessKey, fileName));

.. note:: 

       AccessKey、SecretKey、BucketName、fileKey 需要在七牛云注册账号之后获得。
       如果想进行语音录制，需要将第一个参数设为 false，即 param.put(JCMediaChannel.JOIN_PARAM_RECORD, JCConfUtils.qiniuRecordParam(false, bucketName, secretKey, accessKey, fileName));


获取录制状态
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

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
