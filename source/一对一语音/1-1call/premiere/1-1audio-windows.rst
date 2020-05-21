Windows
==============================

.. highlight:: csharp


前提条件
----------------------------------

- Microsoft Visual Studio 2017 或以上版本

- 支持 Windows 7 或以上版本的 Windows 设备

- 有效的菊风开发者账号（`免费注册 <http://developer.juphoon.com/signup>`_ ）


准备工作
------------------------------

开始之前，请您先做好如下准备工作：

SDK 下载
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

点击 `Windows SDK <http://developer.juphoon.com/document/cloud-communication-windows-sdk#2>`_ 进行下载。


AppKey 获取
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

AppKey 是应用在 菊风云平台 中的唯一标识。需要在 SDK 初始化的时候使用，AppKey 获取请参考 :ref:`创建应用 <创建应用>` 。


SDK 配置
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. _Windows SDK 配置:

准备工作：

1. 下载 Visual Studio 2013，请参考: `Visual Studio Downloads <https://www.visualstudio.com/en-us/downloads/download-visual-studio-vs.aspx>`_ 。

2. 安装 Directx End-User Runtime Web，请参考: `DirectX End-User Runtime Web <https://www.microsoft.com/zh-tw/download/details.aspx?id=35>`_ 。

.. note:: 菊风云平台 SDK Windows 版本（以下简称 SDK）支持 .net Framework 4.5及以上。

解压 SDK：

下载 Windows 版 SDK 并解压，解压后可以看到 bin 目录包含以下文件：

.. image:: images/windows_1.png

**导入Windows SDK**

1. 打开visual studio，新建WPF应用程序。

2. 点击“项目 > 添加引用”，将 bin 目录中的 JCSDK.dll，mtc.dll，mtcmanaged.dll，mtcwrap.dll，Newtonsoft.Json.dll，zmf.dll 和 zmfmanaged.dll 七个文件添加到您的工程目录中。

.. image:: images/windows_2.png

.. image:: images/windows_3_1.png

3. 在 Reference Manager 窗口中，可以看到添之后的文件，如下图：

.. image:: images/windows_4.png

4. 设置应用输出路径与库所在文件夹一致

.. image:: images/windows_5.png

5. 导入完成后编译运行，如果没有报错，恭喜您，您已经成功配置 SDK，可以进行 SDK 初始化了。


SDK 初始化
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. _Windows SDK 初始化:

在使用 SDK 之前，都应该首先进行 SDK 的初始化。

.. highlight:: csharp

初始化 SDK，具体接口如下：

::

    /// <summary>
    /// 创建 JCClient 实例
    /// </summary>
    /// <param name="appKey">用户从 Juphoon Cloud 平台申请的 AppKey 字符串</param>
    /// <param name="callback">回调接口，用于接收 JCClient 相关通知</param>
    /// <param name="extraParams">额外参数，没有则填 null </param>
    /// <returns></returns>
    public static JCClient create(Application app, string appKey, JCClientCallback callback, Dictionary<string, string> extraParams)

.. note::

       appKey 为准备工作中“获取 AppKey”步骤中取得的 AppKey。如果还未获取 AppKey，请参考 :ref:`创建应用 <创建应用>` 来获取。


示例代码::

    public bool initialize(Application app)
    {
        // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
        JCClient client = JCClient.create(app, "your appkey", this, null);
        return true;
    }


SDK 初始化之后，即可进行登录的集成。

登录
---------------------------

.. _Windows 登录:


登录涉及 JCClient 类，其主要作用是负责登录、登出管理及帐号信息存储。

登录之前，可以通过配置关键字进行登录的相关配置，如是否使用代理服务器登录以及服务器地址的设置，具体如下：

登录环境设置
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

服务器地址设置，包括国际环境服务器地址和国内环境服务器地址
::

    /// <summary>
    /// 设置登录相关属性
    /// </summary>
    /// <param name="key">设置的属性key值</param>
    /// <param name="value">设置的属性对应值</param>
    /// <returns>返回 ture 表示设置成功，false设置失败</returns>
    public bool setConfig(string key, string value)

其中，配置关键字有
::

    /// 服务器
    public const string JCClientConfigServer = "ConfigServer";
    /// 设备标识，用户可以自己传入设备标识
    public const string JCClientConfigDeviceId = "ConfigDeviceId";
    /// https代理, 例如 192.168.1.100:3128
    public const string JCClientConfigHttpsProxy = "ConfigHttpsProxy";

.. note::

    **国际环境** 服务器地址为 ``http:intl.router.justalkcloud.com:8080`` 。

    **国内环境** 服务器地址为 ``http:cn.router.justalkcloud.com:8080`` 。

示例代码::

    JJCClient client = JCClient.create(app, "your appkey", this, null);
    // 设置登录地址（国内环境）
    client.setConfig(JCClientConfigServer, "http:cn.router.justalkcloud.com:8080");
    // 设置登录地址（国际环境）
    client.setConfig(JCClientConfigServer, "http:intl.router.justalkcloud.com:8080");


设置登录相关参数后，可以调用下面的方法获取相关的配置
::

    /// <summary>
    /// 获取设置属性的值
    /// </summary>
    /// <param name="key">需要获取的属性key值</param>
    /// <returns>需要获取的属性值</returns>
    public string getConfig(string key)

示例代码::

    // 获取登录配置
    client.getConfig(JCClientConfigServer);


发起登录
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

登录参数设置之后，即可调用 login 接口发起登录操作::

    /// <summary>
    /// 登录 Juphoon Cloud 平台，只有登录成功后才能进行平台上的各种业务
    /// 登录结果通过 JCCallCallback 通知
    /// <param name="username">用户标识</param>
    /// <param name="password">密码，如果设置为免鉴权则可以填任意字符串</param>
    /// <returns>true 表示正常执行调用流程，false 表示调用异常</returns>
    /// <remarks>注意:用户名为英文数字和'+' '-' '_' '.'，长度不要超过64字符，'-' '_' '.'不能作为第一个字符</remarks>
    /// <remarks>当用户不存在时会自动创建该用户</remarks>
    public bool login(string username, string password)

结果返回 true 表示正常执行调用流程，false 表示调用异常。

.. note:: 用户名大小写不敏感，用户名为英文、数字和'+' '-' '_' '.'，长度不要超过64字符，'-' '_' '.'不能作为第一个字符。

示例代码::

    client.login(userId, password);

登录的结果通过 onLogin 回调接口上报::

    /// <summary>
    /// 登录结果回调
    /// </summary>
    /// <param name="result">true 表示登录成功，false 表示登录失败</param>
    /// <param name="reason">当 result 为 false 时该值有效，了解具体原因</param>
    void onLogin(bool result, JCClientReason reason);

其中，JCClientReason 有
::

        /// <summary>
        /// 正常
        /// </summary>
        None,
        /// <summary>
        /// sdk 未初始化
        /// </summary>
        SDKNotInit,
        /// <summary>
        /// 无效参数
        /// </summary>
        InvalidParam,
        /// <summary>
        /// 函数调用失败
        /// </summary>
        CallFucntionError,
        /// <summary>
        /// 当前状态无法再次登录
        /// </summary>
        StateCannotLogin,
        /// <summary>
        /// 超时
        /// </summary>
        TimeOut,
        /// <summary>
        /// 网络异常
        /// </summary>
        NetWork,
        /// <summary>
        /// appkey 错误
        /// </summary> 
        AppKey,
        /// <summary>
        /// 账号密码错误
        /// </summary>
        Auth,
        /// <summary>
        /// 无该用户
        /// </summary>
        NoUser,
        /// <summary>
        /// 强制登出
        /// </summary>
        ServerLogout,
        /// <summary>
        /// 其他错误
        /// </summary>
        Other = 100,

登录成功之后，SDK 会自动保持与服务器的连接状态，直到用户主动调用登出接口，或者因为帐号在其他设备登录导致该设备登出。


登出
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

登出调用下面的方法，登出后不能进行平台上的各种业务操作
::

    /// <summary>
    /// 登出 Juphoon Cloud 平台
    /// </summary>
    /// <returns>返回 true 表示正常执行调用流程，false 表示调用异常，异常错误通过 JCClientCallback 通知</returns>
    public bool logout();

登出结果通过 onLogout 回调接口上报::
    
    /// <summary>
    /// 登出回调
    /// </summary>
    /// <param name="reason">登出原因</param>
    void onLogout(JCClientReason reason);


当登录状态发生改变时，会收到 onClientStateChange 回调：
::

    /// <summary>
    /// 登录状态变化通知
    /// </summary>
    /// <param name="state">当前状态值</param>
    /// <param name="oldState">之前状态值</param>
    void onClientStateChange(JCClientState state, JCClientState oldState);

JCClientState 有::

    // 未初始化
    NotInit,
    // 未登录
    Idle,
    // 登录中
    Logining,
    // 登录成功
    Logined,
    // 登出中
    Logouting,

示例代码::

    private void onClientStateChange(JCClientState state, JCClientState oldState)
        {
            if (state == JCClientState.Idle) { // 未登录
               ...
            }
            else if (state == JCClientState.Logining) { // 登录中
               ...
            }
            else if (state == JCClientState.Logined) { // 登录成功
                ...
            }
            else if (state == JCClientState.Logouting) { // 登出中
                ...
            }
        }


完成以上步骤，就做好了基础工作，您可以开始集成业务了。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

业务集成
-----------------------------------

一对一语音通话涉及以下类：

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - `JCCall <http://developer.juphoon.com/portal/reference/windows/html/e1a40c0e-ec58-49c2-3063-295fb883e86f.htm>`_
     - 一对一通话类，包含一对一语音和视频通话功能
   * - `JCCallItem <http://developer.juphoon.com/portal/reference/windows/html/0267696e-79ee-8d46-c086-3c071a2b2b3a.htm>`_
     - 通话对象类，此类主要记录通话的一些状态，UI 可以根据其中的状态进行显示逻辑
   * - `JCCallCallback <http://developer.juphoon.com/portal/reference/windows/html/25bca4ea-ad43-2cbb-42a8-b4e626739711.htm>`_
     - 通话模块回调代理
   * - `JCMediaDevice <http://developer.juphoon.com/portal/reference/windows/html/034d5af6-ec04-5148-7ec5-04e27e93e8c2.htm>`_
     - 设备模块，主要用于视频、音频设备的管理
   * - `JCMediaDeviceCallback <http://developer.juphoon.com/portal/reference/windows/html/3a00aa12-4e18-cf90-4610-b2c9c63b7a7b.htm>`_
     - 设备模块回调代理

更多接口的详细信息请参考 `API 说明文档 <http://developer.juphoon.com/portal/reference/windows/html/e36ffb00-647f-0198-a895-56556009f19d.htm>`_ 。


*接口调用逻辑和相关状态*

.. image:: 1-1workflowwindows.png

*说明：黑色字体表示接口，棕色字体表示通话状态*

.. note::

    通话方向（呼入或呼出）及通话状态（振铃、连接中、通话中等）可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/windows/html/0267696e-79ee-8d46-c086-3c071a2b2b3a.htm>`_  对象中的 `direction <http://developer.juphoon.com/portal/reference/windows/html/024186ef-6f0d-a89a-ac66-56f82874fc43.htm>`_ 和 `state <http://developer.juphoon.com/portal/reference/windows/html/bf54b32e-b6cc-7192-4119-6d0e603d6967.htm>`_ 获得。


**开始集成通话功能前，请先进行** ``模块的初始化``

创建 JCCall 实例
::

    /// <summary>
    /// 创建JCCall实例
    /// </summary>
    /// <param name="client">JCClient实例</param>
    /// <param name="mediaDevice">JCMediaDevice实例</param>
    /// <param name="callback">回调接口，用于接收JCCall相关回调事件</param>
    /// <returns>JCCall实例</returns>
    public static JCCall create(JCClient.JCClient client, JCMediaDevice.JCMediaDevice mediaDevice, JCCallCallback callback)

创建 JCMediaDevice 实例
::

        /// <summary>
        /// 创建JCMediaDevice实例
        /// </summary>
        /// <param name="client">JCClient实例</param>
        /// <param name="callback">JCMediaDeviceCallback回调函数，用于接收JCMediaDevice相关事件</param>
        /// <returns>JCMediaDevice实例</returns>
        public static JCMediaDevice create(JCClient.JCClient client, JCMediaDeviceCallback callback)


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

    /// <summary>
    /// 一对一呼叫
    /// </summary>
    /// <param name="userId">用户标识</param>
    /// <param name="video">是否为视频呼叫</param>
    /// <param name="extraParam">透传参数，设置后被叫方可获取该参数</param>>
    /// <returns>返回true表示正常执行调用流程，false表示调用异常</returns>
    public bool call(string userId, bool video, string extraParam)

.. note:: 

       调用此接口会自动打开音频设备。

       extraParam 为自定义透传字符串，被叫可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/windows/html/0267696e-79ee-8d46-c086-3c071a2b2b3a.htm>`_  对象中的 `extraParam <http://developer.juphoon.com/portal/reference/windows/html/e0226cbc-1ca1-ef9c-5e8e-d3dc853d618d.htm>`_ 属性获得。


示例代码
::

    call.call("peer number", false, "自定义透传字符串");

通话发起后，主叫和被叫均会收到新增通话的回调，通话状态变为 Pending
::

    /// <summary>
    /// 新增通话回调
    /// </summary>
    /// <param name="item">JCCallItem对象</param>
    void onCallItemAdd(JCCallItem item);

示例代码::

    public void onCallItemAdd(JCCallItem item)
        {
          // 收到新增通话回调
        }


.. note::

        如果主叫想取消通话，可以直接转到第4步，调用第4步中的挂断通话的接口。这种情况下调用挂断后，通话状态变为 Cancel.


2. 应答通话
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

被叫收到 onCallItemAdd 回调事件，并通过 JCCallItem 中的 `video <http://developer.juphoon.com/portal/reference/windows/html/19f4b28d-7c00-3f7e-ace5-0615f5b609b3.htm>`_ 属性以及 `direction <http://developer.juphoon.com/portal/reference/windows/html/024186ef-6f0d-a89a-ac66-56f82874fc43.htm>`_ 属性值 In 判断是视频呼入还是语音呼入，此时可以调用下面的接口进行应答，**语音通话只能进行语音应答**
::

    /// <summary>
    /// 接听
    /// </summary>
    /// <param name="item">JCCallItem对象</param>
    /// <param name="video">针对视频呼入可选择视频接听和音频接听</param>
    /// <returns>成功失败</returns> 
    public bool answer(JCCallItem item, bool video)


示例代码::

    public void onCallItemAdd(JCCallItem item) {
        // 如果是语音呼入且在振铃中
        if (item.direction == JCCallDirection.In && !item.video) {
        // 应答通话
        call.answer(item, false);
        }
    }

通话接听后，通话状态变为 Connecting。

.. note::

        如果要拒绝通话，可以直接转到第4步，调用第4步中的挂断通话的接口。这种情况下调用挂断后，通话状态变为 Canceled。


3. 通话建立
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

被叫接听通话后，双方将建立连接，此时，主叫和被叫都将会收到通话更新的回调，通话状态变为 Talking。连接成功之后，通话将建立。

现在您可以进行一对一语音通话了。

如果已经在语音通话中，但又有新通话进来，可以选择接听或挂断，如果选择接听，则原来的一路通话将被保持。

4. 挂断通话
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

主叫或者被叫均可以调用下面的方法挂断通话
::

    /// <summary>
    /// 挂断
    /// </summary>
    /// <param name="item">JCCallItem对象</param>
    /// <param name="reason">挂断原因，参考</param>
    /// <param name="description">挂断描述</param>
    /// <returns>返回true表示正常执行调用流程，false表示调用异常</returns>
    public bool term(JCCallItem item, JCCallReason reason, string description)

示例代码
::

    // 挂断通话
    JCCallItem item = call.callItems[0];
    call.term(item, JCCallReason.TermBySelf, "挂断");


通话挂断后，UI 会收到移除通话的回调，通话状态变为 Ok。
::

    /// <summary>
    /// 移除通话回调
    /// </summary>
    /// <param name="item">JCCallItem对象</param>
    /// <param name="reason">通话结束原因</param>
    void onCallItemRemove(JCCallItem item, JCCallReason reason);

示例代码
::

    public void onCallItemRemove(JCCallItem item, JCCallReason reason)
    {
       //界面处理
    }


其中，reason 有以下几种

.. list-table::
   :header-rows: 1

   * - 名称
     - 描述
   * - None
     - 无异常
   * - NotLogin
     - 未登录
   * - CallFunctionError
     - 函数调用错误
   * - TimeOut
     - 超时
   * - NetWorkError
     - 网络错误
   * - CallOverLimit
     - 超出通话上限
   * - TermBySelf
     - 自己挂断
   * - AnswerFail
     - 应答失败
   * - Busy
     - 忙
   * - Decline
     - 拒接
   * - UserOffline 
     - 用户不在线
   * - NotFound
     - 无此用户
   * - RejectVideoWhenHasCall
     - 已有通话拒绝视频来电
   * - RejectWhenHasVideoCall
     - 已有视频通话拒绝来电
   * - Other = 100
     - 其他错误


如果拨打通话时，**对方未在线，或者主叫呼叫后立即挂断**，则对方再次上线时会收到未接来电的回调

::

        /// <summary>
        /// 上报服务器拉取的未接来电
        /// </summary>
        /// <param name="item">JCCallItem对象</param>
        void onMissedCallItem(JCCallItem item);

此时通话状态变为 Missed。


Sample 代码
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

**关键代码实现：**

1.初始化 JC SDK 以及通话和媒体设备模块

::

    public void initialize() 
    {
        // AppKey为创建应用获取的AppKey
        JCClient client = JCClient.create(app ,"AppKey", this, null);
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
        // 调用接口发起呼叫
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
        // 挂断通话
        JCCallItem item = call.callItems[0];
        call.term(item, JCCallReason.TermBySelf, "挂断");
    }

**更多功能**

- :ref:`通话状态更新<通话状态更新(windows1-1)>`

- :ref:`通话过程控制<通话过程控制(windows1-1)>`

- :ref:`获取网络状态<获取网络状态(windows1-1)>`

- :ref:`设备控制<设备控制(windows)>`


**进阶**


在实现语音通话的过程中，您可能还需要添加以下功能来增强您的应用：

- :ref:`通话录音<通话录音(windows)>`