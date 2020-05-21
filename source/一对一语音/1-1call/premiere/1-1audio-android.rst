Android
============================

.. _一对一信令通话-Android:


前提条件
----------------------------------

- Android SDK API 等级 16 或以上

- 支持 Android 4.1 或以上版本的移动设备

- 有效的菊风开发者账号（`免费注册 <http://developer.juphoon.com/signup>`_ ）


准备工作
----------------------------

开始之前，请您先做好如下准备工作：

SDK 下载
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

点击 `Android SDK <http://developer.juphoon.com/document/cloud-communication-android-sdk#2>`_ 进行下载。

AppKey 获取
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

AppKey 是应用在 菊风云平台 中的唯一标识。需要在 SDK 初始化的时候使用，AppKey 获取请参考 :ref:`创建应用 <创建应用>` 。


SDK 配置
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

1. 下载 SDK，拷贝 libs 文件夹内的 armeabi-v7a、X86、mtc.jar 、JCSDK.jar 和 zmf.jar 到您工程目录中的 libs 目录下，并打开工程，如下图所示:

.. image:: images/android_sdklist.png

.. image:: images/quickstart_android1.png

2. 为能连接到我们的 so 库，在您工程 build.gradle 文件中确保增加以下配置，如图:

.. image:: images/set_sdk_android2.png

3. 修改您工程中 Application 配置文件 AndroidManifest.xml，**请确保已经加入以下特性和权限信息**。具体信息可以参考 :ref:`Android 权限说明<Android 权限说明>` 。
::

    <uses-feature android:name="android.hardware.camera" />
    <uses-feature android:name="android.hardware.camera.autofocus" />
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.CAMERA" />
    <uses-permission android:name="android.permission.RECORD_AUDIO" />
    <uses-permission android:name="android.permission.MODIFY_AUDIO_SETTINGS" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.VIBRATE"/>
    <uses-permission android:name="android.permission.BLUETOOTH" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />

.. note::

    您在 AndroidManifest 中进行权限配置时，请确保您能够获得打开摄像头、音视频录制等相关权限。

4. 配置完成后编译运行，如果没有报错，恭喜您，您已经成功配置 SDK，可以进行 SDK 初始化了。

.. note:: SDK 不支持模拟器运行，请使用真机。

SDK 初始化
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. _Android SDK 初始化:

在使用 SDK 之前，都应该首先进行 SDK 的初始化。

.. highlight:: java

初始化 SDK，具体接口如下：

::

    /**
     * 创建 JCClient 实例
     *
     * @param appKey      用户从 Juphoon Cloud 平台上申请的 AppKey 字符串
     * @param callback    回调接口，用于接收 JCClient 相关通知
     * @param extraParams 额外参数，没有则填null
     * @return JCClient 对象
     */
    public static JCClient create(Context context, String appKey, JCClientCallback callback, Map<String, String> extraParams)

.. note::

       appKey 为准备工作中“获取 AppKey”步骤中取得的 AppKey。如果还未获取 AppKey，请参考 :ref:`创建应用 <创建应用>` 来获取。


示例代码::

    public boolean initialize(Context context) {
        // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
        JCClient client = JCClient.create(Context, "your appkey", this, null);
        return true;
    }


SDK 初始化之后，即可进行登录的集成。

登录
-------------------------

.. _Android 登录:

登录涉及 JCClient 类，其主要作用是负责登录、登出管理及帐号信息存储。

.. highlight:: java

登录之前，可以通过配置关键字进行登录的相关配置，如是否使用代理服务器登录以及服务器地址的设置，具体如下：

登录环境设置
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

服务器地址设置，包括国际环境服务器地址和国内环境服务器地址
::

    /**
     * 设置配置相关参数<br>
     * CONFIG_KEY_SERVER_ADDRESS, CONFIG_KEY_HTTPS_PROXY 均需要在 login 之前调用<br>
     *
     * @param key    参数关键字
     * @param value  参数值
     * @return 返回 true 表示设置成功，false 表示设置失败
     * @see JCClient.ConfigKey
     */
    public abstract boolean setConfig(@ConfigKey String key, String value);


其中，配置关键字有
::

    /**
     * 服务器地址
     */
    public static final String CONFIG_KEY_SERVER_ADDRESS = "config_key_server_address";
    /**
     * 设备id
     */
    public static final String CONFIG_KEY_DEVICE_ID = "config_key_device_id";
    /**
     * https代理, 例如 192.168.1.100:3128
     */
    public static final String CONFIG_KEY_HTTPS_PROXY = "config_key_https_proxy";


.. note::

    **国际环境** 服务器地址为 ``http:intl.router.justalkcloud.com:8080`` 。

    **国内环境** 服务器地址为 ``http:cn.router.justalkcloud.com:8080`` 。


示例代码::

    JJCClient client = JCClient.create(Context, "your appkey", this, null);
    // 设置登录地址（国内环境）
    client.setConfig(JCClientConfigServer, "http:cn.router.justalkcloud.com:8080");
     // 设置登录地址（国际环境）
    client.setConfig(JCClientConfigServer, "http:intl.router.justalkcloud.com:8080");


设置登录相关参数后，可以调用下面的方法获取相关的配置
::

    /**
     * 获取配置相关参数
     *
     * @param key 参数关键字
     * @return 成功返回字符串类型具体值, 失败返回 NULL
     * @see JCClient.ConfigKey
     */
    public abstract String getConfig(@ConfigKey String key);

示例代码::

    JJCClient client = JCClient.create(Context, "your appkey", this, null);
    // 获取登录配置
    client.getConfig(JCClientConfigServer);


发起登录
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

登录参数设置之后，即可调用 login 接口发起登录操作::

    /**
     * 登录 Juphoon Cloud 平台，只有登陆成功后才能进行平台上的各种业务
     * 登录结果通过 JCCallCallback 通知<br>
     * 注意:用户名为英文数字和'+' '-' '_' '.'，长度不要超过64字符，'-' '_' '.'不能作为第一个字符
     *
     * @param userId    用户名
     * @param password  密码，但不能为空
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常，异常错误通过 JCClientCallback 通知
     */
    public abstract boolean login(String userId, String password);

.. note:: 用户名大小写不敏感，用户名为英文、数字和'+' '-' '_' '.'，长度不要超过64字符，'-' '_' '.'不能作为第一个字符。

示例代码::

    client.login(userI, password);

登录操作执行之后，登录的结果通过 onLogin 回调接口上报
::

    /**
     * 登录结果回调
     *
     * @param result  true 表示登陆成功，false 表示登陆失败
     * @param reason  当 result 为 false 时该值有效
     */
    void onLogin(boolean result, @JCClient.ClientReason int reason);

其中，ClientReason 有
::

    /**
     * 正常
     */
    public static final int REASON_NONE = 0;
    /**
     * sdk 未初始化
     */
    public static final int REASON_SDK_NOT_INIT = 1;
    /**
     * 无效参数
     */
    public static final int REASON_INVALID_PARAM = 2;
    /**
     * 函数调用失败
     */
    public static final int REASON_CALL_FUNCTION_ERROR = 3;
    /**
     * 当前状态无法再次登录
     */
    public static final int REASON_STATE_CANNOT_LOGIN = 4;
    /**
     * 超时
     */
    public static final int REASON_TIMEOUT = 5;
    /**
     * 网络异常
     */
    public static final int REASON_NETWORK = 6;
    /**
     * appkey 错误
     */
    public static final int REASON_APPKEY = 7;
    /**
     * 账号密码错误
     */
    public static final int REASON_AUTH = 8;
    /**
     * 无该用户
     */
    public static final int REASON_NOUSER = 9;
    /**
     * 强制登出
     */
    public static final int REASON_SERVER_LOGOUT = 10;
    /**
     * 其他错误
     */
    public static final int REASON_OTHER = 100;

登录成功之后，SDK 会自动保持与服务器的连接状态，直到用户主动调用登出接口，或者因为帐号在其他设备登录导致该设备登出。


登出
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

登出调用下面的方法，登出后不能进行平台上的各种业务操作
::

    /**
     * 登出 Juphoon Cloud 平台，登出后不能进行平台上的各种业务
     *
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常，异常错误通过 JCClientCallback 通知
     */
    public abstract boolean logout();


登出结果通过 onLogout 回调接口上报::

    /**
     * 登出回调
     *
     * @param reason 登出原因
     */
    void onLogout(@JCClient.ClientReason int reason);


当登录状态发生改变时，会收到 onClientStateChange 回调
::

    /**
     * 登录状态变化通知
     *
     * @param state    当前状态值
     * @param oldState 之前状态值
     */
    void onClientStateChange(@JCClient.ClientState int state, @JCClient.ClientState int oldState);


ClientState 有::

    // 未初始化
    public static final int STATE_NOT_INIT = 0;
    // 未登录
    public static final int STATE_IDLE = 1;
    // 登录中
    public static final int STATE_LOGINING = 2;
    // 登录成功
    public static final int STATE_LOGINED = 3;
    // 登出中
    public static final int STATE_LOGOUTING = 4;


示例代码::

    public void onClientStateChange(@JCClient.ClientState int state, @JCClient.ClientState int oldState) {
         if (state == JCClient.STATE_IDLE) { // 未登录
           ...
        } else if (state == JCClient.STATE_LOGINING) { // 正在登录
           ...
        } else if (state == JCClient.STATE_LOGINED) { // 登录成功
           ... 
        } else if (state == JCClient.STATE_LOGOUTING) { // 登出中
           ...
        }
    }


集成登录后，即可进行相关业务的集成。

``SDK 支持前后台模式，可以在应用进入前台或者后台时调用 JCClient 类中的 setForeground 方法进行设置``

::

    /**
     * 设置是否为前台, 在有控制后台网络的手机上当进入前台时主动触发
     *
     * @param foreground 是否为前台
     */
    public abstract void setForeground(boolean foreground);

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

完成以上步骤，就做好了基础工作，您可以开始集成业务了。


业务集成
----------------------------

一对一语音通话涉及以下类：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - `JCCall <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCall.html>`_
     - 一对一通话类，包含一对一语音和视频通话功能
   * - `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_ 
     - 通话对象类，此类主要记录通话的一些状态，UI 可以根据其中的状态进行显示逻辑
   * - `JCCallCallback <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallCallback.html>`_
     - 通话模块回调代理
   * - `JCMediaDevice <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCMediaDevice.html>`_
     - 设备模块，主要用于视频、音频设备的管理
   * - `JCMediaDeviceCallback <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCMediaDeviceCallback.html>`_
     - 设备模块回调代理

更多接口的详细信息请参考 `API 说明文档 <http://developer.juphoon.com/portal/reference/android/>`_ 。

*接口调用逻辑和相关状态*

.. image:: 1-1workflowandroid.png

*说明：黑色字体表示接口，棕色字体表示通话状态*

.. note::

    通话方向（direction）及通话状态（state）可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_  对象中的 `getDirection() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getDirection-->`_ 方法和 `getState() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCall.html#STATE_INIT>`_ 方法获得。

.. highlight:: java

**开始集成通话功能前，请先进行** ``模块的初始化``

创建 JCCall 实例
::

    /**
     * 创建JCCall实例
     *
     * @param client        JCClient实例
     * @param mediaDevice   JCMediaDevice实例
     * @param callback      回调接口，用于接收 JCCall 相关回调事件
     * @return JCCall       JCCall实例
     */
    public static JCCall create(JCClient client, JCMediaDevice mediaDevice, JCCallCallback callback);

创建 JCMediaDevice 实例
::

    /**
     * 创建 JCMediaDevice 对象
     *
     * @param client   JCClient 对象
     * @param callback JCMediaDeviceCallback 回调接口，用于接收 JCMediaDevice 相关通知
     * @return 返回 JCMediaDevice 对象
     */
    public static JCMediaDevice create(JCClient client, JCMediaDeviceCallback callback)


示例代码
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
    JCCall call = JCCall.create(client, mediaDevice, this);


**开始集成**

1. 拨打通话
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

主叫调用下面的接口发起语音通话，此时 video 传入值为 false
::

    /**
     * 一对一呼叫
     *
     * @param userId        用户标识
     * @param video         是否视频呼叫
     * @param extraParam    透传参数，设置后被叫方可获取该参数
     * @return              返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean call(String userId, boolean video, String extraParam);

.. note:: 

       调用此接口会自动打开音频设备。

       extraParam 为自定义透传字符串，被叫可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_  对象中的 `getExtraParam() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getExtraParam-->`_ 方法获取 extraParam 属性。

示例代码
::

    // 发起语音呼叫
    call.call("peer number", false, "自定义透传字符串");

通话发起后，主叫和被叫均会收到新增通话的回调，此时通话状态变为 STATE_PENDING
::

    /**
     * 新增通话回调
     *
     * @param item JCCallItem 对象
     */
    void onCallItemAdd(JCCallItem item);

示例代码::

    public void onCallItemAdd(JCCallItem item) {
        // 新增通话回调
    }


.. note::

        如果主叫想取消通话，可以直接转到第4步，调用第4步中的挂断通话的接口。这种情况下调用挂断后，通话状态变为 STATE_CANCEL.


2. 应答通话
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

被叫收到 onCallItemAdd 回调事件，此时可通过 JCCallItem 中的 `getVideo() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getVideo-->`_ 方法以及 `getDirection() <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html#getDirection-->`_ 方法获取 video 和 direction 属性，并根据 video 属性的值以及 direction 属性的值 DIRECTION_IN 判断是视频呼入还是语音呼入，然后可以调用下面的接口进行应答，**语音通话只能进行语音应答**
::

    /**
     * 接听
     *
     * @param item  JCCallItem 对象
     * @param video 针对视频呼入可以选择以视频接听还是音频接听
     * @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean answer(JCCallItem item, boolean video);

示例代码::

    public void onCallItemAdd(JCCallItem item) {
        // 如果是语音呼入且在振铃中
        if (item.getState() == JCCall.STATE_PENDING) {
            if (item.getDirection() == JCCall.DIRECTION_IN && !item.getVideo()) {
                // 应答通话
                call.answer(item, false);
            }
        }
    }

通话接听后，通话状态变为 STATE_CONNECTING。

.. note::

        如果要拒绝通话，可以直接转到第4步，调用第4步中的挂断通话的接口。这种情况下调用挂断后，通话状态变为 STATE_CANCELED。


3. 通话建立
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

被叫接听通话后，双方将建立连接，此时，主叫和被叫都将会收到通话更新的回调，连接成功之后，通话将建立。通话状态变为 STATE_TALKING。

现在您可以进行一对一语音通话了。

如果已经在语音通话中，但又有新通话进来，可以选择接听或挂断，如果选择接听，则原来的一路通话将被保持。


4. 挂断通话
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

主叫或者被叫均可以调用下面的方法挂断通话
::

    /**
     * 挂断
     *
     * @param item          JCCallItem 对象
     * @param reason        挂断原因
     * @param description   挂断描述
     * @return              返回 true 表示正常执行调用流程，false 表示调用异常
     * @see CallReason
     */
    public abstract boolean term(JCCallItem item, @CallReason int reason, String description);


示例代码::
    
    JCCallItem item = call.getCallItems().get(0);
    call.term(item, JCCall.REASON_NONE, null);


通话挂断后，UI 会收到移除通话的回调，通话状态变为 STATE_OK。
::

    /**
     * 移除通话回调
     *
     * @param item          JCCallItem 对象
     * @param reason        通话结束原因
     * @param description   通话结束原因的描述，只有被动挂断的时候，才会收到这个值，其他情况下则返回空字符串
     */
    void onCallItemRemove(JCCallItem item, @JCCall.CallReason int reason, String description);

示例代码::

    public void onCallItemRemove(JCCallItem item, @JCCall.CallReason int reason, String description) {  // 移除通话回调
       //界面处理
    }


其中，reason 有以下几种

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - REASON_NONE = 0
     - 无异常
   * - REASON_NOT_LOGIN = 1
     - 未登录
   * - REASON_CALL_FUNCTION_ERROR = 2
     - 函数调用错误
   * - REASON_TIMEOUT = 3
     - 超时
   * - REASON_NETWORK = 4
     - 网络错误
   * - REASON_OVER_LIMIT = 5
     - 超出通话上限
   * - REASON_TERM_BY_SELF = 6
     - 自己挂断
   * - REASON_ANSWER_FAIL = 7
     - 应答失败
   * - REASON_BUSY = 8
     - 忙
   * - REASON_DECLINE = 9
     - 拒接
   * - REASON_USER_OFFLINE = 10
     - 用户不在线
   * - REASON_NOT_FOUND = 11
     - 无此用户
   * - REASON_REJECT_VIDEO_WHEN_HAS_CALL
     - 已有通话拒绝视频来电
   * - REASON_REJECT_WHEN_HAS_VIDEO_CALL
     - 已有视频通话拒绝来电
   * - REASON_OTHER = 100
     - 其他错误


**通话挂断的其他情况：**

如果拨打通话时，**对方未在线，或者主叫呼叫后立即挂断**，则对方再次上线时会收到未接来电的回调

::

    /**
     * 上报服务器拉取的未接来电
     *
     * @param item    JCCallItem 对象
     */
    void onMissedCallItem(JCCallItem item);

此时通话状态变为 STATE_MISSED。


Sample 代码
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

**关键代码实现：**

1.初始化 JC SDK 以及通话和媒体设备模块

::

    public void initialize() 
    {
        // AppKey为创建应用获取的AppKey
        JCClient client = JCClient.create(context, "AppKey", this, null);
        JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
        JCCall call = JCCall.create(client, mediaDevice, this);
    }


2.登录

::

    public void login()
    {
        client.login("用户名", "密码");
    }

3.拨打语音通话

::

    public void voiceCall() 
    {
        // 调用接口发起语音呼叫
        call.call("peer number", false, "自定义透传字符串");
    }

4.应答通话

::

    public void onCallItemAdd(JCCallItem item) {
        // 应答通话
        call.answer(item, false);
    }

5.挂断通话

::

    public void endCall {
        JCCallItem item = call.getCallItems().get(0);
        call.term(item, JCCall.REASON_NONE, null);
    }


**更多功能**

- :ref:`通话状态更新<通话状态更新(android1-1)>`

- :ref:`通话过程控制<通话过程控制(android1-1)>`

- :ref:`获取网络状态<获取网络状态(android1-1)>`

- :ref:`设备控制<设备控制(android)>`


**进阶**

在实现语音通话的过程中，您可能还需要添加以下功能来增强您的应用：

- :ref:`通话录音<通话录音(android)>`

- :ref:`推送<推送(android)>`
