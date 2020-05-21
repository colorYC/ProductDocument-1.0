参考文件
=====================

.. _创建应用:

创建应用
--------------------------------

.. _注册成为开发者:

**1.注册开发者账号**

开始之前请先 `注册成为开发者 <http://developer.juphoon.com/signup>`_ 。

如果您已经拥有开发者帐号，请直接 `登录 <http://developer.juphoon.com/signin>`_ ，并开始创建您的应用。

登录之后，您可以通过 菊风云平台 开发者帐号使用开发者工具和创建应用。

**2.创建应用**

登录开发者帐号后，点击 “添加应用”，输入应用名称以创建应用。

.. image:: images/appcreate1.png

.. note:: 应用名称创建后不可修改。

创建完成后，系统会自动为您的应用生成 **AppKey**。

点击 “显示" 即可以看到 AppKey。

.. image:: images/appkey1.png

**AppKey 是应用在 菊风云平台 中的唯一标识。** 

在创建应用时，``同一个控制台帐号中创建的多个应用属于同一个域。同域中的应用是可以互通的。`` 所以同一帐号下的应用即使 AppKey 不同，也可以进行业务往来。例如，从一个应用登录的用户可以呼叫另一个应用登录的用户。

应用创建完成后，即可对应用进行设置，应用设置包括基本设置和高级设置。 其中，免签权模式表示帐号由用户自行生成，用户名、密码等信息保存在 菊风云平台 服务器中，服务器不检查密码。

.. image:: images/signature1.png

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _AppKey 说明:

AppKey 说明
--------------------------------

AppKey 是应用在 菊风云平台 中的唯一标识。

在创建应用时，同一个控制台帐号中创建的多个应用属于同一个域。同域中的应用是可以互通的，所以同一帐号下的应用即时 AppKey 不同，也可以进行业务往来。例如，从一个应用登录的用户可以呼叫另一个应用登录的用户。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _iOS 导入动态库:

iOS 导入动态库
---------------------

在 Mac 环境下解压下载的压缩包，解压后的文件夹内有 JCSDK 文件夹。JCSDK 文件夹里包含了 JCSDKOC.framework。

**拷贝文件**

将 JCSDK 文件夹拷贝到您工程所在的目录下，如下图（仅供参考）：

.. image:: images/sdkshare1.png

**工程设置**

1. 导入文件和库

点击 ‘General’，在 “Embedded Binaries” 一栏，点击 ‘+’ 符号，然后导入 JCSDK 文件夹下的 JCSDKOC.framework

.. image:: images/frameworkshare.png

点击 ‘General’，点击 ‘+’ 符号，在 “Linked Frameworks and Libraries” 一栏，导入 SDK 依赖的其他库，如下图：

.. image:: images/sharelibinput.png

2. 设置 Framework Search Paths 路径

点击 ‘Build Settings’，找到 Framework Search Paths 和 Header Search Paths，在右侧输入路径。如下图：

.. image:: images/iOS_integration_AddDynamicFrameworkPath.png

.. image:: images/headerfile.png

.. note:: 在设置 Framework Search Paths 时，一般在完成第1步导入 JCSDKOC.framework 后，Xcode 会自动生成该路径
       如果 Xcode 没有自动生成路径，用户要根据 JCSDKOC.framework 和 lib 库文件所在目录，手动设置路径。

3. 设置 Enable Bitcode 为 NO

点击 ‘Build Settings’，找到 Enable Bitcode 设置为 NO，如下图：

.. image:: images/iOS_integration_DynamicBitcode.png

4. 设置预处理宏定义

点击 ‘Build Settings’，找到 Preprocessor Macros，在右侧输入 ZPLATFORM=ZPLATFORM_IOS，如下图：

.. image:: images/static_import_ios.png

5. 设置 Documentation Comments 为 NO

点击 ‘Build Settings’，找到 Documentation Comments 设置为 NO，如下图：

.. image:: images/static_import_ios1.png

6. 设置后台运行模式

点击 ‘Capabilities’，找到 Background Modes，勾选红框内的 Audio, AirPlay, and Picture in Picture，如下图：

.. image:: images/iOS_integration_DynamicBackgroundModes.png

**权限设置**

7. 设置麦克风和摄像头的权限

点击 ‘Info’，然后添加麦克风和摄像头权限，如下图：

.. image:: images/iOS_integration_DynamicPermissions.png

8. 编译运行

以上步骤进行完后，编译工程，如果没有报错，恭喜您，您已经成功配置 SDK，可以进行下一步了。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _macOS 导入静态库:

macOS 导入静态库
---------------------

**导入静态库**

在 Mac 环境下打开下载的 iOS SDK，在 sdk 文件夹内包含了 lib、JCSDKOC.framework 和 include 三个文件。

.. image:: images/ios_sdk_list1.png

``拷贝文件``

将 sdk 文件夹拷贝到您工程所在的目录下。

``工程设置``

**1. 导入 SDK**

打开 Xcode（以 Xcode 11.0 为例），进入 TARGETS > Project Name > Build Phases > Link Binary with Libraries 菜单，点击 + 添加如下依赖的库。

.. image:: images/macsdk0.png

继续点击 + ，再点击 Add Other…，找到 JCSDKOC.framework、libmtc.a 和 libzmf.a 文件并添加。

.. image:: images/macstaticlib1.png

进入 TARGETS > Project Name > General > Frameworks, Libraries, and Embedded Content 菜单，添加的 JCSDKOC.framework 状态应该为 Do Not Embed

.. image:: images/macstaticlib2.png

**2. 设置路径**

点击 ‘Build Settings’，找到 Framework Search Paths、Header Search Paths 和 Library Search Paths，在右侧输入路径。如下图：

.. image:: images/macsdk2.png

.. note:: 在设置 Framework Search Paths 时，一般在完成第1步导入 JCSDKOC.framework 后，Xcode 会自动生成该路径
       如果 Xcode 没有自动生成路径，用户要根据 JCSDKOC.framework、lib 和 include 文件所在目录，手动设置路径。

**3. 设置预处理宏定义**

点击 ‘Build Settings’，找到 Preprocessor Macros，在右侧输入 ZPLATFORM=ZPLATFORM_OSX，如下图：

.. image:: images/macsdk4.png

**4. 设置 Other Linker Flags 的参数为 -ObjC**

点击 ‘Build Settings’，找到 Other Linker Flags 并添加参数 -ObjC，如下图：

.. image:: images/macsdk3.png

**5. Capabilities 设置**

若你的项目已启用 App Sandbox 或 Hardened Runtime 设置，点击 ‘Capabilities’，在 App Sandbox 中 勾选以下内容：

.. image:: images/macsdk5.png

**6. 权限设置**

设置麦克风和摄像头的权限

点击 ‘Info’，然后添加麦克风和摄像头权限，如下图：

.. image:: images/macsdk6.png

.. list-table::
   :header-rows: 1

   * - Key
     - Type
     - Value
   * - Privacy - Microphone Usage Description
     - String
     - 使用麦克风的目的，如语音通话。
   * - Privacy - Camera Usage Description
     - String
     - 使用摄像头的目的，如视频通话。

**7. 编译运行**

以上步骤进行完后，编译工程，如果没有报错，恭喜您，您已经成功配置 SDK，可以进行下一步了。


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _Android 权限说明:

Android 权限说明
--------------------------

.. list-table::
   :header-rows: 1

   * - 权限
     - 介绍
   * - INTERNET
     - 网络权限，登录与通话必须
   * - ACCESS_NETWORK_STATE
     - 访问网络状态权限，登录与通话必须
   * - ACCESS_WIFI_STATE
     - 访问WIFI权限，登录与通话必须
   * - READ_PHONE_STATE
     - 访问设备信息权限，登录必须
   * - WAKE_LOCK
     - 保持后台在线权限，保持后台在线必须
   * - CAMERA
     - 摄像头权限，视频通话必须
   * - RECORD_AUDIO
     - 录音权限，通话必须
   * - MODIFY_AUDIO_SETTINGS
     - 修改音频设置权限，通话必须
   * - WRITE_EXTERNAL_STORAGE
     - 读写外部存储权限，存储必须
   * - VIBRATE
     - 手机振动
   * - BLUETOOTH
     - 蓝牙功能
   * - SYSTEM_ALERT_WINDOW
     - 悬浮窗功能