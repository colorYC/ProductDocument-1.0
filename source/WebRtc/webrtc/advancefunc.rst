进阶
========================

进阶功能主要是会议过程中的控制功能，包括 **设置会场成员的状态、设置成员昵称、设置对其他成员的订阅信息、提出成员、CDN 推流、录制以及分屏等**。

会议过程中的控制功能主要通过发送控制指令实现，接口如下
::
        
    jc.send(text, peer); 
    peer    可选的对端Id,http连接时被忽略
    text    具体的指令协议,参考文档中的说明


详细的控制指令说明如下：


设置对其他成员的订阅信息
-------------------------------

只有订阅了其他成员的视频信息，才能收到其他成员的音视频数据。

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 设置对其他成员的订阅信息
     - 
        - 明文格式

         vsubscribe <target> <value>

        - JSON格式

         {"cmd":"vsubscribe","target":"%s","state":%d}
         
         // 参数说明

         target 订阅对象

         state 0 - 不订阅，1 - 订阅视频


设置会场成员的属性
-------------------------------

会场中的每个成员都有4个属性: **固定ID，昵称，角色，状态 **。

每个属性的说明如下：

.. list-table::

   * - 固定 ID
     - 通常是账户名，不允许修改，保证唯一
   * - 昵称
     - 支持中文，允许动态修改
   * - 状态
     - 按位解释的整型

       只有开启音视频并转发，其他成员才有可能接收到对应的音视频数据

       - 0x1 转发视频
  
       - 0x2 转发音频

       - 0x4 视频开启

       - 0x8 音频开启
   * - 角色
     - 按位解释的整型
       
       注意只有媒体发送者 ( 0x2 )，才允许设置音视频开启状态 ( 0x4 | 0x8 )

       - 0x2 媒体发送者，允许开关本地音视频

       - 0x4 演示者，其状态改变事件将被广播

       - 0x8 会场拥有者，只存在唯一拥有者，由会场自动产生和迁移


成员属性设置的控制指令如下：


设置自身或其他成员角色
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 设置自身或其他成员角色
     - 
       - 明文格式

        setrole <mask> <role> [target]

       - JSON格式
        {"cmd":"setrole", "mask":%d, "role":%d, "target":%s}

        // 参数说明

        mask 掩码,表示操作的bit位

        role 角色，按位解释，设原状态为_ROLE_，则 _ROLE_ = (_ROLE_ & ~mask) | (mask & role);

        target 可选的设置对象，缺省时为自身


设置自身或其他成员状态
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 设置自身或其他成员状态
     - 
       - 明文格式

        setstate <mask> <state> [target]

        例如：

        - 关闭id0(即成员URI或屏幕URI)的音频的服务器转发

         setstate 0x2 0x0 id0

        - 关闭自身的音频发送
        
         setstate 0x8 0x0

        - 打开自身的音频发送

         setstate 0x8 0x8
        
       - JSON格式

        {"cmd":"setstate", "mask":%d, "state":%d, "target":%s}
         
        // 参数说明

        mask 掩码,表示操作的bit位

        role 角色，按位解释，设原状态为_ROLE_，则 _ROLE_ = (_ROLE_ & ~mask) | (mask & role);

        target 可选的设置对象，缺省时为自身


设置自身或其他成员昵称
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 设置自身或其他成员昵称
     - 
       - 明文格式

        setnick <str> [target]

       - JSON格式

        {"cmd":"setnick", "nick":"<str>", "target":%s}

        // 参数说明

        str utf8 编码的昵称

        target 可选的设置对象，缺省时为自身


设置会议属性
-------------------------------------

可以对会议的属性进行修改，如会场标题、设置是否接收视频、设置收到视频的布局等。具体如下：


修改会场标题
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
     - 参数说明
   * - 修改会场标题
     - 
        - 明文格式

         settitle <title>

        - JSON格式

         {"cmd":"settitle","title":%s}
     - 
         title utf8 编码的标题


设置是否接收视频
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
     - 参数说明
   * - 设置是否接收视频
     - 
       - 明文格式

        publish <bool>

       - JSON格式

        {”cmd":"publish", "video":<bool>}
     - 
        bool 布尔值，可选为true,false, 0, 1



设置收到视频的布局
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
     - 参数说明
   * - 设置收到视频的布局
     - 
       - 明文格式

        setmergemode <mergeMode>

       - JSON格式

        {”cmd":"setmergemode", "mergeMode":%d}
     - 
        mergeMode 整型，布局模式

         1 平铺模式,所有视频均分平铺

         2 讲台模式,共享为大图,其他视频为小图

         3 演讲模式,共享为大图,共享者视频为小图,其他不显示

         4 自定义模式,由setlayout指令设置所有视频布局

         5 智能模式, 可用setmergemode_i,setmerge_i 调整相应的参数


 
设置分屏模式
-------------------------------

1. 实现自定义模式
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

实现自定义模式之前，需要确保收到视频的布局为自定义模式(@setmergemode 4)才能看到效果。

.. list-table::
   :header-rows: 1

   * - 指令格式
     - 参数说明
   * - JSON格式：

       {"cmd":"setlayout","layout":[[<id0>,<ps> x, y, w, h],

       ["id1",0x200, 0.1, 0.1, 1.0, 1.0], ...]}
     - 
       id0：成员URI或屏幕URI

       ps：视频尺寸级别 0x100, 0x200, 0x300, 0x400

       x,y,w,h：单位[0.0,1.0]的坐标和尺寸

2. 调整智能模式的布局
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

调整智能模式的布局之前，需要确保收到视频的布局为智能模式(@setmergemode 5)才能看到效果。

.. list-table::
   :header-rows: 1

   * - 指令格式
     - 参数说明
   * - 明文格式：

       setmergemode_i <mode> <scsMode> [uri]

       例如：设置智能模式为“自由模式”，带屏幕共享为“主持人模式”，
       
       大小屏放大用户默认为qqqq，则指令为：

       @setmergemode_i 1 3 [username:qqqq@sample.cloud.justalk.com]
     - 
       mode:智能分屏模式

        1 freeLayout

        2 rectLayout

        3 bigSmallX2

        4 bigSmallX3

        ....,

        11 bigSmallX10

        12 bigSmallTop

        13 bigSmallBotto
        
       scsMode：智能分屏带屏幕共享模式

        1 screen

        2 platform

        3 platformCover

        4 speaker

       uri: 大小屏默认放大的用户uri


3. 调整智能模式的详细参数
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令格式
     - 参数说明
   * - 明文格式：

       setmerge_i <width> <height> <mode> <scsMode> <fps> [uri]

       例如：设置宽为1600，高为900，智能模式为“自由模式”， 
           
       带屏幕共享为“主持人模式”，帧数为24帧，大小屏放大用户默认为qqqq

       则指令为：

       @setmerge_i 1600 900 1 3 24 [username:qqqq@sample.cloud.justalk.com]
     - 
       width：宽

       height：高

       mode：智能分屏模式，
       
       参见上方setmergemode_i

       scsMode：智能分屏带屏幕共享模式，

       参见上方setmergemode_iv

       fps:帧数 1-30

       uri: 大小屏默认放大的用户uri

关于智能分屏模式和调整智能分屏模式的详细参数的具体指令信息，请参考 :ref:`智能分屏<智能分屏>`。


录制
---------------------------------

1. 开始录制
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令格式
     - 参数说明
   * - JSON格式：

       {"cmd":"StartRecord","Video":true,"Storage":
          {
            "Protocol":"qiniu",

            "AccessKey":"Th2jW*********",

            "SecretKey":"Nh***********",

            "BucketName":"*******",

            "FileKey":"1234.mp4",

            "SplitFileSize":1024
           }
         }
     - 
       Video：是否视频会议，true为视频会议

       Protocol：云存储协议，目前支持七牛

       AccessKey、SecretKey：七牛云个人中心->密钥管理查看

       BucketName：七牛云的存储名称

       FileKey：上传到七牛的文件名

       SplitFileSize：录制文件超过多少KB自动分割，不填，默认是500M


2. 停止录制
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令格式
     - 相关说明
   * - JSON格式：

       {"cmd":"StopRecord"}

       假设 base64编码后为 eyJjbWQiOiJTdG9wUmVjb3JkIn0=，
       
       会议号为12345678，则停止录制的命令如下：

       sendtext eyJjbWQiOiJTdG9wUmVjb3JkIn0= 
       
       [username:delivery_12345678@delivery.cloud.justalk.com]
     - 
        录制器或推流器也是一个confDelivery，
        
        可以通过sendtext命令通知录制器/推流器来操作

        录制器或推流器uri为：
        
        [username:delivery_会议号@delivery.cloud.justalk.com]

.. note:: 录制默认分辨率为 360P，如需修改录制分辨率则要调用 SetVideoLevel 接口进行修改。

关于设置录制分辨率和录指文件大小的详细指令信息请参阅 :ref:`修改录制或推流分辨率<修改录制或推流分辨率>` 和 :ref:`修改录制文件大小<修改录制文件大小>`。


CDN 推流
-------------------------------

1. 设置推流地址
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 设置推流地址
     - 
       - 明文格式

        setpushuri <uri>

        设置推流地址，若以'/'结尾，则自动添加当前会议号。
        
        必须在开始“视频混合”前设置

        录制器或推流器也是一个confDelivery，可以通过sendtext命令通知录制器/推流器来操作

        录制器或推流器uri为：[username:delivery_会议号@delivery.cloud.justalk.com]
   
        // 举例

        假设会议号为12345678，则指令为：

        @sendtext @delivery_12345678 setpushuri rtmp://video-center.alivecdn.com/juphoon11/juphoon?
          
        vhost=broadcast.justalkcloud.com [username:delivery_12345678@delivery.cloud.justalk.com]


2. 设置推流参数
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 设置推流参数
     - 
       - 明文格式

        setpushuriparm <parm>

        最终完全的推流的 URL 等于：推流地址 + 空格 + 推流参数  

        必须在开始“视频混合”前设置
        
        // 举例

        假设会议号为12345678，则指令为：
 
        @sendtext @delivery_12345678 setpushuriparm socks=192.12.1.2 
          
        [username:delivery_12345678@delivery.cloud.justalk.com]


3. 开始推流
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 开始推流
     - 
       - 明文格式

        startpush

        开始向 CDN 推流，相应的网页观看地址由 CDN 服务提供商决定

        // 举例

        假设会议号为12345678，则指令为：
 
        @sendtext @delivery_12345678 startpush 
          
        [username:delivery_12345678@delivery.cloud.justalk.com]


4. 停止推流
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 停止推流
     - 
       - 明文格式

        stoppush

        // 举例

        假设会议号为12345678，则指令为：
 
        @sendtext @delivery_12345678 stoppush 
          
        [username:delivery_12345678@delivery.cloud.justalk.com]


关于设置推流分辨率的详细指令信息请参阅 :ref:`修改录制或推流分辨率<修改录制或推流分辨率>`。


播放
----------------------------------------

开始播放
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 开始播放
     - 
       - 明文格式
          play [playFile]

       - JSON 格式
          {"cmd":"play", "playFile":%s}
  
		      将自动添加发送角色,和相应的音频发送状态.

		      playFile 必须为 ConfDelivery 所在机器上的文件路径.默认 playFile.wav


停止播放
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>> 

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 停止播放
     - 
       - 明文格式
          stopplay

       - JSON 格式
          {"cmd":"stopPlay"}


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

发送会议聊天文本
--------------------------

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
     - 参数说明
   * - 发送会议聊天文本

     - 
       - 明文格式

        sendtext <str> [target]
        
       - JSON格式

        {"cmd":"sendtext", "text":"<str>", "target":%s}
     - 
        str utf8 编码的文本

        target 可选的目标对象，缺省时将广播


消息发送后，其他成员将会收到 onconfchat 回调
::

    {"cmd":"onconfchat", "val":{"from":%s, "text":%s}}
        from  是发言者URI
        text  是UTF8编码的文本,支持中文


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

踢出成员
-------------------------------

会议中还可以踢出成员，具体如下：

.. list-table::
   :header-rows: 1

   * - 指令内容
     - 指令格式
   * - 踢出成员
     - 
        - 明文格式

         kickuser [uri]  

        - JSON格式 

         {cmd:"kickuser", "target":"%s"}

         // 参数说明

         target 目标对象


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

屏幕共享
------------------------------------

实现屏幕共享功能需要以下操作：

- Google Chrome 必须安装 [额外扩展][ScreenCapturing]，请参考 `Chrome 屏幕采集扩展[ScreenCapturing] <https://chrome.google.com/webstore/detail/screen-capturing/ajhifddimkapgcifgcodmmfdlknahffk>`_ 。
- getsourceid 参数必须指定有效的https网址，缺省时需要公网访问能力。


- 动态切换屏幕共享

切换屏幕共享，将自动进行重新协商
::

    jc.share(toggle)

       toggle  是否共享窗口或屏幕


发送DTMF
------------------------------------
::

    jc.dtmf(tones,duration,inter_tone_gap)

       tones     DMTF的音符,0-9,A-D或a-d,#,*. 忽略无法识别的字符

       duration  可选的每个音符的持续毫秒,不能超过6000或小于70. 默认100ms 

       inter_tone_gap   可选的音符间隔,至少为50ms,但应尽可能短. 默认70ms
