macOS
============================

.. highlight:: objective-c

.. _通话状态更新(mac1-1):

本节内容提供通话中的状态更新以及通话过程控制的接口说明。

通话状态更新
-----------------------------

通话过程中，如果通话状态发生了改变，如开启关闭静音、开启关闭通话保持、活跃状态切换、网络变化等，将会收到通话状态更新的回调
::

    /**
     *  @brief 通话状态更新回调（当上层收到此回调时，可以根据 JCCallItem 对象获得该通话的所有信息及状态，从而更新该通话相关UI）
     *  @param item JCCallItem 对象
     *  @param changeParam 更新标识类
     */
    -(void)onCallItemUpdate:(JCCallItem* __nonnull)item changeParam:(JCCallChangeParam * __nullable)changeParam;


关于 JCCallChangeParam 的说明请参考 JCCallItem.h 文件。

.. note::
     
       静音状态、通话保持状态、活跃状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html>`_ 对象获得。

示例代码::

    -(void)onCallItemUpdate:(JCCallItem* __nonnull)item changeParam:(JCCallChangeParam * __nullable)changeParam {
        JCCallItem* callItem = item;
        if (changeParam.mute) { // 开启静音
            ...
        } else if (changeParam.sate) { // 通话状态变化
            ...
        } else if (changeParam.held) { // 被挂起变化
            ...
        } else if (changeParam.active) { // 活跃状态变化
            ...
        } else if (changeParam.netStatus) { // 网络状态变化
            ...
        }
        ...
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _通话过程控制(mac1-1):

通话过程控制
-----------------------------

通话静音
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

通过下面的方法开启或关闭静音，开启关闭静音需要根据 JCCallItem 中的静音状态（`mute <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/mute>`_）来决定，静音开启后，对方将听不到您的声音
::

    /**
     *  @brief 静音，通过 JCCallItem 对象中的静音状态来决定开启关闭静音
     *  @param item JCCallItem 对象
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)mute:(JCCallItem* __nonnull)item;


开启关闭呼叫保持
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

调用下面的方法对通话对象进行呼叫保持或解除呼叫保持（当通话对象处于被保持状态（即状态为held）时不可以进行此操作），开启或关闭呼叫保持需要根据 JCCallItem 对象中（`hold <http://developer.juphoon.com/portal/reference/ios/Classes/JCCallItem.html#//api/name/hold>`_）的呼叫保持状态来决定
::

    /**
     *  @brief                  呼叫保持，通过 JCCallItem 对象中的呼叫保持状态来决定开启关闭呼叫保持
     *  @param item             JCCallItem 对象
     *  @return                 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)hold:(JCCallItem* __nonnull)item;


切换活跃通话
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

调用下面的方法对通话中被保持的对象和活跃的通话对象进行切换

::

    /**
     *  @brief 切换活跃通话
     *  @param item 需要变为活跃状态的 JCCallItem 对象
     *  @return 返回 true 表示正常执行调用流程，false 表示调用异常
     */
    -(bool)becomeActive:(JCCallItem* __nonnull)item;

示例代码
::

    //获取活跃通话对象
    JCCallItem *item = [call getActiveCallItem];;
    //开启或关闭静音
    [call mute:item];
    //开启关闭呼叫保持
    [call hold:item];
    //切换活跃通话
    [call becomeActive:item];

