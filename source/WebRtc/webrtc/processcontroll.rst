通话过程管理
===========================

会议事件订阅
---------------------------

会议过程中，会议状态的改变则通过发送订阅事件指令 subscribe 进行订阅。

::

    明文格式
    subscribe <hex>

    JSON字符串格式
    {"cmd":"subscribe", "mask":<hex>}

      hex 事件位集合
        0x1 自身状态改变,   对应 onselfstate
        0x2 其他成员状态改变,对应 onactorstate/onactorleave
        0x4 会场属性改变,   对应 onconfstate
        0x8 聊天信息,       对应 onconfchat


订阅前可以设置订阅的对象和属性。

设置订阅
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    vsubscribe <target> <state>
		//target 订阅对象
		//state  布尔型, 0 - 表示不订阅, 1 - 订阅视频

例如：

- 订阅会场成员 actor

假设 actor 的 uri 为 [username:actor@sample.cloud.justalk.com]，则指令为：
::

    vsubscribe [username:actor@sample.cloud.justalk.com] 1

- 取消订阅会场成员 actor 

假设 actor 的 uri 为 [username:actor@sample.cloud.justalk.com]，则指令为：
::

    vsubscribe [username:actor@sample.cloud.justalk.com] 0


订阅成功之后即可收到订阅成员的状态改变回调。


其他成员状态改变回调
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    {"cmd":"onactorstate",
     "val":{"id0":{"nick":%s,"role":%d, "state":%d, "subscribe":%d},...}}

        id0  是成员URI,全局唯一, 
        nick 是UTF8编码的昵称,支持中文
        role 是角色位整型:
              0x2 - 发送者
        state 是状态位整型:
                0x1 - 转发视频
                0x2 - 转发音频
                0x4 - 打开视频
                0x8 - 打开音频
        subscribe 是订阅信息
                1 - 订阅视频


自身状态改变回调
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    {"cmd":"onselfstate", "val":{"nick":%s,"role":%d, "state":%d}}


成员离开回调
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    {"cmd":"onactorleave", "val":["id0","id1",...]}


会场属性改变回调
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    {"cmd":"onconfstate", "val":{"screen":%s, "title":%s, "sharer":%s}}

        screen 是屏幕URI
        title  是UTF8编码的标题,支持中文
        sharer 是屏幕共享者URI


协商状态改变回调
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

只有协商状态为‘stable’，才允许更新本地媒体(调用update/share)
::

    onsignalingstatechange(state)

      state 可选为
          - 'stable'  没有进行协商的稳定状态
          - 'have-local-offer'     协商进行中
          - 'have-remote-offer'    协商进行中
          - 'have-remote-pranswer' 协商进行中
          - 'have-local-pranswer'  协商进行中


连接状态改变回调
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

只要连接状态为‘failed’，则将不可恢复的通信中断
::

    oniceconnectionstatechange(state)

      state 可选为
          - 'new' 初始状态
          - 'checking' 探测进行状态
          - 'connected' 成功连接
          - 'completed' 探测结束
          - 'disconnected' 断开连接
          - 'failed' 连接过程已失败,不可恢复
          - 'closed' 关闭连接
