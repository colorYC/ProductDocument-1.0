iOS 推送
=============================

.. _iOS 推送:

简介
---------------------

由于苹果不支持后台 App 的长链接，所以当 App 处于后台或者被杀掉的情况下将无法收到音视频的呼叫，因此在这种情况下需借助 Push 机制来接收音视频呼叫。

iOS 推送分为 APNs 推送和 VoIP 推送，由于 VoIP 推送后必须跟随 calllkit 的调用，而 callkit 已经在大陆被禁，为保护用户的数据，苹果 iOS 13 限制具有 VoIP 功能的应用程序后台访问权限。因此 voip 推送也基本无法在大陆使用。根据苹果文档提示，只能用普通推送（ usernotifications ）来代替 voip 推送（ pushkit ）。这将影响所有用 xcode11 编译的应用。

因此，此处仅介绍 APNs 推送的集成方法。

APNs 推送集成
--------------------------

APNS 是 Apple Push Notification Service的缩写，也就是苹果的推送服务器。

APNS 推送可以分为三个阶段，如下图：

.. image:: images/apns_push_principle1.png

第一阶段：应用程序的服务器端把要发送的消息、目的iPhone的标识打包，发给APNS。

第二阶段：APNS在自身的已注册Push服务的iPhone列表中，查找有相应标识的iPhone，并把消息发送到iPhone。

第三阶段：iPhone把发来的消息传递给相应的应用程序，并且按照设定弹出Push通知。

更多关于 APNs 推送的信息请参考 `苹果开发指南 <https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/APNSOverview.html#//apple_ref/doc/uid/TP40008194-CH8-SW1>`_ 。

**APNs 注册流程**

.. image:: images/gettoken.png

1、应用程序注册 APNs 消息推送。

2、iOS从 APNs Server 获取 devicetoken，应用程序接收 device token。

3、应用程序将 device token 发送给程序的 PUSH 服务端程序。

集成流程
>>>>>>>>>>>>>>>>>>>>>>>>>

1. APNs 推送证书申请
>>>>>>>>>>>>>>>>>>>>>>>>>

证书申请分为以下几步：

- 准备CSR文件

- 生成带有Push Notifications功能的AppID

- 给该AppID的Push Notifications配置CSR

- 下载证书配置钥匙串

- 导出p12文件

具体如下：

1.1 准备 CSR 文件

首先通过证书助手生成一个 Certificate Signing Request(也就是 CSR)的请求文件。

.. image:: images/csr1.png

.. image:: images/csr2.png

继续之后选择保存位置，点击保存，此时该位置上会有一个 CertificateSigningRequest.certSigningRequest 的请求文件，也就是我们说的 CSR 文件。

1.2 生成带有Push Notifications功能的AppID 

.. image:: images/appid1.png

在此处点+添加，按需求填写信息即可。 

.. image:: images/appid2.png

.. image:: images/appid3.png

这里的bundleID是识别APP的唯一ID，一个APP对应一个APP ID 也就是一个bundleID，一般采用域名反写的方式命名，输入部分的英文建议如此。 

.. image:: images/appid4.png

这里要把 Push Notifications 勾选上，应用才能带有推送能力。 全部设定好之后继续，保存，生成了一个新的 APP ID 。

1.3 给该AppID的Push Notifications配置CSR

.. image:: images/setcsr1.png

这部分是为这个APP ID配置push notifications，一个SSL Certificate可以让你手机上的notification server连接到Apple Push Notification Service（APNS） 

可以看到配置部分有两个： 

Development SSL Certificate:开发推送证书配置，开发环境可以收到推送。 

Production SSL Certificate：生成推送证书配置，线上环境可以收到推送。

在配置页面选取之前生成的CSR文件：CertificateSigningRequest.certSigningRequest点击continue即可配置完成

.. image:: images/setcsr2.png

1.4 下载推送证书，加到自己电脑上的钥匙串里

.. image:: images/keychain1.png

点击Download，下载开发和生产的推送证书。 

双击两个证书，会自动添加在钥匙串里去。

.. image:: images/keychain2.png

1.5 导出p12文件

找到自己的APP（后面的名字为当时设定的bundleID），右键导出p12文件，填上密码，保存。

.. image:: images/keychain3.png

.. image:: images/keychain4.png

利用终端将 p12 转为 pem 格式

 具体命令如下：

 ::

    openssl pkcs12 -in apns_dev_cert.p12 -out apns_dev_cert.pem -nodes -clcerts -nokeys

    openssl pkcs12 -in apns_dev_cert.p12 -out apns_dev_key.pem -nodes -nocerts

    openssl rsa -in apns_dev_key.pem -out apns_dev_rsa_key.pem

    cat apns_dev_cert.pem  apns_dev_rsa_key.pem  > apns_dev.pem（要上传的文件）


1.6 验证证书是否工作。首先执行下面的命令
::
 
    telnet gateway.sandbox.push.apple.com 2195

它将尝试发送一个规则的，不加密的连接到 APNs 服务。如果看到下面的反馈，那说明您的 Mac 能够到达 APNs。按下 Ctrl+C 关闭连接。如果得到一个错误信息，那么需要确保您的防火墙允许2195端口。


.. image:: images/provecer.png

下面要使用之前生成的 SSL 证书和私钥来设置一个安全的链接去链接苹果服务器：
::

    openssl s_client -connect gateway.sandbox.push.apple.com:2195 -cert apns_dev_cert.pem -key apns_dev_rsa_key.pem

执行完这一句命令后需要输入密码：
::

    Enter pass phrase for apns_dev_rsa_key.pem:


由于密码为空，直接按回车。 如果链接是成功的，您可以随便输入一个字符，按下回车，服务器就会断开链接，如果建立连接时有问题，OpenSSL 会返回一个错误信息。

如果您看到以下内容，说明链接是成功的：

::

    CONNECTED(00000003)
    depth=1 /C=US/O=Entrust, Inc./OU=www.entrust.net/rpa is incorporated by reference/OU=(c) 2009 Entrust, Inc./CN=Entrust Certification Authority - L1C
    verify error:num=20:unable to get local issuer certificate
    verify return:0
    ---
    Certificate chain
    0 s:/C=US/ST=California/L=Cupertino/O=Apple Inc./CN=gateway.sandbox.push.apple.com
    i:/C=US/O=Entrust, Inc./OU=www.entrust.net/rpa is incorporated by reference/OU=(c) 2009 Entrust, Inc./CN=Entrust Certification Authority - L1C
    1 s:/C=US/O=Entrust, Inc./OU=www.entrust.net/rpa is incorporated by reference/OU=(c) 2009 Entrust, Inc./CN=Entrust Certification Authority - L1C        i:/O=Entrust.net/OU=www.entrust.net/CPS_2048 incorp. by ref. (limits liab.)/OU=(c) 1999 Entrust.net Limited/CN=Entrust.net Certification Authority (2048)
    ---
    Server certificate
    -----BEGIN CERTIFICATE-----
    MIIFMzCCBBugAwIBAgIETCMmsDANBgkqhkiG9w0BAQUFADCBsTELMAkGA1UEBhMC
    ...
    -----END CERTIFICATE-----
    subject=/C=US/ST=California/L=Cupertino/O=Apple Inc./CN=gateway.sandbox.push.apple.com
    issuer=/C=US/O=Entrust, Inc./OU=www.entrust.net/rpa is incorporated by reference/OU=(c) 2009 Entrust, Inc./CN=Entrust Certification Authority - L1C
    ---
    Acceptable client certificate CA names
    /C=US/O=Apple Inc./OU=Apple Certification Authority/CN=Apple Root CA
    /C=US/O=Apple Inc./OU=Apple Worldwide Developer Relations/CN=Apple Worldwide         Developer Relations Certification Authority
    /C=US/O=Apple Inc./OU=Apple Certification Authority/CN=Apple Application Integration Certification Authority
    ---
    SSL handshake has read 3160 bytes and written 2179 bytes
    ---
    New, TLSv1/SSLv3, Cipher is AES256-SHA
    Server public key is 2048 bit
    Secure Renegotiation IS supported
    Compression: NONE
    Expansion: NONE
    SSL-Session:
        Protocol  : TLSv1
        Cipher    : AES256-SHA
        Session-ID: 
        Session-ID-ctx: 
        Master-Key:1D3F740E6FFF3AE1C56E09CC3876E701FC18D211652EF0C9B11D1C13F9357C71F44CDB11421AA47087E18ED86FFAD373
        Key-Arg   : None
        Start Time: 1444985977
        Timeout   : 300 (sec)
        Verify return code: 0 (ok)
    ---


2. 上传 APNs 推送证书
>>>>>>>>>>>>>>>>>>>>>>>>>

APNs 证书生成之后，登录菊风云控制台进行对应的设置，具体如下：

- 登录菊风云控制台，进入要进行推送设置的应用详情，找到 Push 设置里的 APNS 配置

.. image:: images/platform_push_ios_juphoonVoipPush.png
   :width: 900  
   :height: 400

- 推送设置分为 Release 和 Debug，其中 Release 用于苹果正式环境；Debug 用于开发环境。

下面以 AppId 为 com.juphoon.cloud.JCSample 为例，分别添加上述两种模式证书，证书为上述生成的 apns_dev.pem 文件。

``Release 的 Bundle ID 为 com.juphoon.cloud.JCSample``

``Debug 的 Bundle  ID 为 com.juphoon.cloud.JCSample.DEBUG``

点击页面底部的“保存修改”按钮保存设置。 设置中 Release 对应 生产（Production） 证书，Debug 对应 开发（Sandbox）证书。 APNs 证书均有有效期的限制，开发证书的有效期是3个月，生产证书的有效期是1年，证书过期后无法推送消息。因此，请务必在证书到期前重新上传新证书，以保证推送服务持续正常。


3. 代码集成
>>>>>>>>>>>>>>>>>>>>>>>>>

.. highlight:: objective-c

完成以上步骤即可进行代码集成，具体如下：

1. 创建 JCPush 对象

::

    /**
     *  @brief 创建 JCPush 对象
     *  @param client JCClient 对象
     *  @return 返回 JCPush 对象
     */
    +(JCPush*)create:(JCClient*)client;

示例代码::

    JCClient *client = [JCClient create:@"your appkey" callback:self extraParams:nil];
    JCPush *push = [JCPush create:client];

2. 注册 APNs 推送

- 注册 APNs 通知

注册推送。在APP完成初始化时，可通过系统函数 registerForRemoteNotifications 或 registerForRemoteNotificationTypes 告知系统需要APNs离线推送：
::

    UIUserNotificationSettings *userNotifySetting = [UIUserNotificationSettings 
                settingsForTypes:UIUserNotificationTypeBadge | UIUserNotificationTypeSound | 
                UIUserNotificationTypeAlert categories:nil];
    [[UIApplication sharedApplication] registerUserNotificationSettings:settings];
    [[UIApplication sharedApplication] registerForRemoteNotifications];


- 调用接口进行推送注册

 注册推送后，您可以在相关的回调中调用下面的接口设置苹果服务器获取的 token、设置通话推送信息、设置消息推送信息、添加推送模板

- 设置苹果服务器获取的 token

::

    /**
     *  @brief 设置苹果服务器获取的token
     *  @param deviceToken token 值
     *  @param voip 是否是 voip token
     *  @param debug 是否是 debug 模式
    */
    -(id)initWithToken:(NSData*)deviceToken voip:(bool)voip debug:(bool)debug;

.. note:: debug 参数值需要依据开发环境而定，发布版设置为 false，开发版设置为 true。


- 设置通话推送信息
::

    /**
     *  @brief 设置通话推送信息
     *  @param sound 声音资源，例如 ring.m4r，为 nil 时则用默认声音
     *  @param seconds 消息过期时间
     */
    -(id)initWithCall:(NSString*)sound expiration:(int)seconds;


- 设置消息推送信息
::

    /**
     *  @brief 设置消息推送信息
     *  @param infoType 消息类型
     *  @param tip 提示内容，不包含发送者，例如 “xx:发送了条消息”，其中"发送了条消息"为tip值，如果要提示发送内容，则填 nil
     *  @param sound 声音资源，例如 ring.m4r
     *  @param seconds 消息过期时间
     */
    -(id)initWithText:(NSString*)infoType tip:(NSString*)tip sound:(NSString*)sound expiration:(int)seconds;


.. note::

    tip 为提示内容：
     - 如果 tip 值为空，则会在提示中显示消息详情；
     - 如果 tip 值不为空，则只显示消息的标题。
    例如“xx:发送了条消息”，其中"发送了条消息"为 tip 值。


- 添加推送模板
::

    /**
     *  @brief 添加推送模板，用于服务器将不同类型的推送以不同的内容格式推给客户端
     *  @param info JCPushTemplate 对象
     *  @return true 表示成功 false 表示失败
     */
    -(bool)addPushInfo:(JCPushTemplate*)info;

示例代码
::

    - (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
        // 设置苹果服务器获取的token
        [_push addPushInfo:[[JCPushTemplate alloc] initWithToken:deviceToken.token voip:false debug:PushEnv]];
        // 设置通话推送信息
        [_push addPushInfo:[[JCPushTemplate alloc] initWithCall:nil expiration:2419200]];
        // 设置消息推送信息
        [_push addPushInfo:[[JCPushTemplate alloc] initWithText:@"text" tip:nil sound:nil expiration:2419200]];
    }


注册完成后，当 APNS 服务器推送消息到对应 token 的设备时将会触发下面的回调
::
    
    -(void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
        NSLog(@"receiveRemoteNotification,userInfo is %@",userInfo);
    }


验证 APNs 推送
>>>>>>>>>>>>>>>>>>>>>>>>>

APNs 推送集成后，即可进行验证，具体如下：

1. 使用用户名登录您的 App，登录后将 App 从后台杀掉。

2. 进入 `Juphoon for developer <http://developer.juphoon.com>`_ ->控制台 ->我的应用 ->设置 ->基本 ->验证 Push。

.. image:: images/push_prove0.png

3. 输入用户名和推送内容，点击验证，此时页面应提示“push 信息发送到服务器成功”。

.. image:: images/push_prove.png