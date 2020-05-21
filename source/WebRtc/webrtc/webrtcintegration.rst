WebRTC 通话集成
============================

通话集成流程
--------------------------------

1. 建立网页与 JusTalk Cloud 连接
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

首先在 HTML 中添加如下代码，细节请参考样例网页。
::

    <script src="webrtc.min.js"></script>

 
打开连接
::

    var jc = new WebRTC(config, trace);
   
      trace 为可选的跟踪打印函数
      config 支持如下格式:
            {
                    'url':"http://192.168.0.66:8881",
                    'confProps': {
                        "roomId":"10210311", 
                        "regionId":"10",
                        "mediaType":"1",
                        "appKey":"6c****************",
                        "accountName": "1111111",
                        "password": "1",
                        "mtcConfQualityGradeKey":1,
                        "mtcConfSecurityKey":0,
                        "mtcConfPasswordKey":"123123",
                        "mtcConfCompositeModeKey":2,
                        "mtcConfStateKey":3,
                        "mtcConfCapacityKey":4
                     }
                    'id':id,
                    'getsourceid':'https:/.....'
            }

其中：

.. list-table::
   :header-rows: 1

   * - 参数
     - 说明
   * - url
     - WS服务器或HTTP服务器地址
   * - id
     - 可选的网页标识,默认随机生成
   * - getsourceid
     - 可选的Chrome屏幕共享扩展网页
   * - confProps
     - 会议相关参数
        - 必要参数
            roomId    会议号

            accountName  账号

            password   账号密码

            appKey    APPKEY

        - 可选参数
            regionId      区域Id, 默认"0"

            mediaType     媒体类型, 0音频，默认1视频

            mtcConfPasswordKey       可选的会议密码,默认'123456'

            mtcConfQualityGradeKey   会议视频尺寸级别

            mtcConfSecurityKey       是否加密传输

            mtcConfCompositeModeKey  收到视频的混合模式

            mtcConfStateKey          加入时的初始状态

            mtcConfCapacityKey       会议允许最大人数
             
如果连接成功，则会收到 onopen 回调
::
        
    onopen(id, config) 

       id     本地Id
       config 配置信息
               .instanceId  为peer(对端Id)
               .iceServers  为媒体配置

如果发生错误，则会收到 onerror 回调
::

     onerror(error) 

       error   错误对象

               应该关闭连接并重置界面


2. 呼叫
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

连接成功后，即可进行呼叫，呼叫调用如下接口::
        
    jc.call(configuration, constraints, peer); 

       configuration  可选的标准 RTCConfiguration object,可以为null,
                      默认由服务器下发(由onopen事件返回)
       constraints    标准 MediaStreamConstraints object,
                      参考update()参数说明
       peer           可选的对端Id,http连接时被忽略   


呼叫成功后，对端即可收到 oncall 回调::

    oncall(peer)

      peer 对端Id


3. 通话连接
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

通话建立后，对端可通过 onaddstream 收到发送端的音视频流数据。
::

    onaddstream(stream, type)   

        stream  为标准 MediaStream 实例,
                使用HTML中的video/audio标签呈现
        type 可选为
            - 'localvideo', 本地视频
            - 'localshare', 本地共享
            - 'peervideo',  对端视频
            - 'peeraudio',  对端音频
            - 'peershare',  对端共享
            - 'peerdata',   对端非媒体数据

当发送端关闭发送音视频时，对端将收到 onremovestream 回调
::
    
    onremovestream(stream, type)

       stream  为标准 MediaStream 实例
       type    - 'localvideo', 本地视频
               - 'localshare', 本地共享
               - 'peervideo',  对端视频
               - 'peeraudio',  对端音频
               - 'peershare',  对端共享
               - 'peerdata',   对端非媒体数据



4. 挂断
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

如果结束通话，则调用挂断接口::

    jc.hangup(); 


此时对端将收到 onhangup 回调::

    onhangup()


会议结束后，关闭连接调用下面的接口
::

    jc.close();
       jc = null;


连接关闭后会收到 onclose 回调
::

    onclose()

      会话已关,应该关闭连接并重置界面






     



