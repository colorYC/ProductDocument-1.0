macOS 进阶
=========================

.. highlight:: objective-c

**集成进阶功能前，请确保您已经集成了基础的音视频通话功能**

.. _通话录音(macOS):

通话录音
-----------------------------

可以在通话中进行录音，开启或关闭录音需要根据当前的录音状态（audioRecord）来决定。如果正在录制或者通话被挂起或者挂起的情况下，不能进行音频录制。录音状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_ 对象获取。

开启或关闭录音接口如下
::

    /**
     * 语音通话录音，通过 JCCallItem 对象中的audioRecord状态来决定开启关闭录音
     *
     * @param item              JCCallItem 对象
     * @param enable            开启关闭录音
     * @param filePath          录音文件路径
     * @return                  返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)audioRecord:(JCCallItem* __nonnull)item enable:(bool)enable filePath:(NSString* __nullable)filePath;


示例代码::

    // 语音录制
    - (void)audioRecord:(JCCallItem* __nonnull)item {
        if (item.audioRecord) { // 正在录制中
           //可以做录音结束的处理
           [call audioRecord:item enable:false filePath:@"your filePath"];
            ...
        } else { // 未在录制中
            // 创建录音文件
            NSString *filePath; // 录音文件的绝对路径，SDK会自动创建录音文件
            if (filePath != nil) {
               // 开始录音
               [call audioRecord:item enable:true filePath:filePath];
                ...
            } else {
                // 录音失败的处理
            }
        } 
    }


开启或关闭录音时，录音状态会发生改变，并通过 onCallItemUpdate 回调上报 
::

    /**
     *  @brief 通话状态更新回调（当上层收到此回调时，可以根据 JCCallItem 对象获得该通话的所有信息及状态，从而更新该通话相关UI）
     *  @param item JCCallItem 对象
     *  @param changeParam 更新标识类
     */
    -(void)onCallItemUpdate:(JCCallItem* __nonnull)item changeParam:(JCCallChangeParam * __nullable)changeParam;


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

通话人数设置
-----------------------------

发起通话前可以通过 maxCallNum 属性设置通话的最大人数，默认为 1。
::

    call.maxCallNum = 2;

当通话超过最大数呼出会失败，收到来电会自动拒绝。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

发送消息
-----------------------------

调用下面的接口在通话中实现发消息的功能
::

    /**
     *  @brief 通过通话建立的通道发送数据
     *  @param item 需要发送数据的 JCCallItem 对象
     *  @param type 文本消息类型，用户可以自定义，例如text、xml等
     *  @param content 消息内容
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)sendMessage:(JCCallItem * __nonnull)item type:(NSString * __nonnull)type content:(NSString * __nonnull)content;


当通话中收到消息时，会收到 onMessageReceive 回调
::

    /**
     *  @brief 通话中收到消息的回调
     *  @param item JCCallItem 对象
     *  @param type 消息类型
     *  @param content 消息内容
     */
    -(void)onMessageReceive:(JCCallItem * __nonnull)item type:(NSString * __nonnull)type content:(NSString * __nonnull)content;

示例代码::

    [call sendMessage:item type:@"text" content:@"消息内容"];