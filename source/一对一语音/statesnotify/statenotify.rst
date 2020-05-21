通话状态通知
=======================

.. _通话状态通知:

简介
-----------------

获取通话状态通知是 菊风云平台 通话业务提供的一项服务。通过该服务，您的服务器可以获得通话的实时状态通知，从而使您能够在通话业务的基础上增加额外的业务控制。

文档描述基于您已经完成以下工作：

- 创建您的应用

- 使用 SDK 接口完成集成通话功能

基本架构

.. image:: images/statenotifyframework.png


控制台配置通知服务器
------------------------

为了获取通话状态通知，您需要在控制台设置接收通话状态通知的服务器地址、鉴权用户名和密码。进入控制台-应用-设置-高级，打开“状态通知服务器”配置页面，如下图：

.. image:: images/kongzhitaiset.png

其中：

- 服务器 URL 是您的服务器访问的 URL 信息

- 图示中服务器访问协议为 HTTP，地址为 112.124.116.65，端口为 7123

- 如果您的服务器通过 HTTPS 访问，请对应选择 HTTPS

- 如果您的服务器通过域名访问，这里需要填入对应域名

- 如果您的服务器有特定的访问路径，请一并填入。例如 112.124.116.65:7123/call_status/

- HTTP 鉴权域用户名和用户密码，是用于访问您的服务器的用户名和密码

了解了获取通话状态通知的基本内容后，下面将介绍一对一通话状态通知的内容。

通话状态通知流程
------------------------

.. highlight:: JavaScript

一对一通话中，不同情况下的通话状态通知流程如下图所示：

Case1：正常呼叫

.. image:: images/callstatenormal.png

Case2：被叫拒接

.. image:: images/callstatereject.png

Case3：被叫超时未响应

.. image:: images/callstatemissed.png

**重发机制**

为保证状态通知的可靠性，菊风云平台 服务器按照以下规则重发通知请求：

- 事件发生时发送第一次通知请求

- 没有收到正确响应的情况下，间隔 2 分钟重发一次，一共重发 5 次

- 正确响应要求响应中的 tid 和请求保持一致，并且 ret 的值为 true


消息格式
--------------------

statusNotify 消息用于通知通话状态，由菊风服务器发起 RESTful 请求，由客户服务器响应。

**获取通话状态通知的操作如下：**

请求
----------------------

HTTP报文头：

.. list-table::
   :header-rows: 1

   * - 报文头域	
     - 是否必选	
     - 值	
     - 说明
   * - POST	
     - 是
     - 	
     - 
   * - Authorization	
     - 否
     - Base64编码	
     - ”Basic Base64('[域用户名]:[域密码]')"
   * - Domain
     - 否	
     - 发起请求者的域名	
     - 
   * - Content-Type	
     - 是
     - application/json	
     - 
   * - Content-Length
     - 是
     - HTTP Body的实际长度		
     - 

**HTTP报文样例：**

::

    POST / HTTP/1.1
    Authorization: Basic dXNlcjpwYXNzd29yZA==
    Content-Length: 535
    Content-Type: application/json
    Domain: john.lu_juphoon.com
    Host: 182.92.66.126:71234

    {"tid":4174673023,
    "oid":"OpenCallUser",
    "cmd":"statusNotify.OpenCall.CallEx",
    "in":{"callId":94532840325125,
        "status":"call released by ([username:t123@test.com]) "
        "params":{"CallId":"94532840325125",
                    "CallAppId":"2",
                    "CallDomainId":"100299",
                    "CallMediaType":"voice",
                    "ReleaseCallBy":"[username:t123@test.com]",
                    "CallAnswerTime":"1442456676",
                    "CalleeAccountId":"[username:t123@test.com]",
                    "CallerAccountId":"[username:t456@test.com]",
                    "CallOrignateTime":"1442456670",
                    "ReleaseCallReason":"User Terminate",
                    "CallTalkingDuration":"47"
                    },
        },
    "params":{}
    }

其中：

- Authorization头：值为“Basic [Base64(菊风分配的域用户名:密码)]”。此处假定客户指定的HTTP鉴权用户名为user，密码为password，"dXNlcjpwYXNzd29yZA=="为(“user:password”)字符串经过Base64编码后的结果。

参数说明：

- id：事务ID。一次状态通知交互过程。客户服务响应消息中应携带对应值。

- callId：通话ID，一次通话的唯一标识。

- status：通话状态，包括：

 - "user(xxx) trying"，表示正在尝试呼叫xxx用户，但尚未接通"user(xxx) joined"，表示xxx用户已接听，被接入到呼叫中

 - "user(xxx) left”，表示xxx用户从呼叫中断开

 - "call released by (xxx)”，表示呼叫已被释放。xxx可能是用户，也可能是服务器

- 详细信息

 - CallerAccountId：主叫用户的帐号

 - CalleeAccountId：被叫用户的帐号

 - CallMediaType：媒体类型，语音为 voice，视频为 video

 - CallOriginateTime：呼叫发起的 UNIX 时间戳，单位秒

 - CallAnswerTime：呼叫接听的 UNIX 时间戳，单位秒。如果值为 0 表示未进入通话状态

 - CallTalkingDuration：通话时长，单位为秒。若值为0表示未进入通话状态

 - ReleaseCallBy：呼叫释放方信息：

  - 如果值为用户帐号则代表由该用户释放呼叫

  - 如果值为 OpenCallServer 则代表由客户侧服务器释放

  - 如果值为CallServer则代表由菊风侧服务器释放。

 - ReleaseCallReason: 释放的原因，包括：

  - "Not Exists:xxx"，代表xxx用户的账户ID不合法

  - "No Sessions:xxx", 代表xxx用户的帐号ID未正常登录（Login）

  - "Inactive Call Clean", 代表主叫客户端未能在规定的时间内（120s）发起呼叫，原先分配的CallId被清理掉。

  - "User Terminate:xxxx", 代表主叫或被叫用户主动挂机而导致的呼叫释放，其中 xxxx 代表终端释放的原因值

  - "OpenCallServer Terminate", 代表客户服务器调用releaseCall操作而导致的呼叫释放

  - "Timeout", 代表被叫侧超时（60s内）未接听呼叫，而导致服务器释放掉呼叫。


响应
--------------------

HTTP报文头：

.. list-table::
   :header-rows: 1

   * - 报文头域	
     - 是否必选	
     - 值	
   * - 202 Accepted
     - 是
     - 	
   * - Authorization	
     - 否
     - Base64编码	
   * - Domain
     - 否	
     - 发起请求者的域名	
   * - Content-Length
     - 是
     - HTTP Body的实际长度	

**HTTP报文样例：**
::

    HTTP/1.1 202 Accepted
    Cntent-Length: 29

    {"tid":4174673023,"ret":true}	

其中：

- tid：事务ID。您的HTTP服务器回复的响应中，该字段的值应与请求报文保持一致。

- ret：true表示收到通知。
