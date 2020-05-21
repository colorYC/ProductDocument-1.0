Windows 进阶
==========================

.. highlight:: csharp

**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化各模块，因为这些模块实例将被频繁使用，建议声明在单例中
    JCClient client = JCClient.create(app, "your appkey", this, null);           
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);               
    JCCall call = JCCall.create(client, mediaDevice, this);

.. _通话录音(windows):

通话录音
-----------------------------

您可以在通话中进行录音，并设置录音文件的路径。开启或关闭录音需要根据当前的录音状态（audioRecord）来决定。如果正在录制或者通话被挂起或者挂起的情况下，不能进行音频录制。录音状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/windows/html/0267696e-79ee-8d46-c086-3c071a2b2b3a.htm>`_ 对象获得。

录音状态的改变通过 onCallItemUpdate 回调上报
::

    /// <summary>
    /// 通话状态更新回调
    /// 当上层收到此回调时，可以根据JCCallItem对象获得该通话所有信息及状态，从而更新通话相关UI
    /// </summary>
    /// <param name="item">JCCallItem对象</param>
    // <param name="changeParam">更新标识类</param>
    void onCallItemUpdate(JCCallItem item, JCCallItem.ChangeParam changeParam);


开启或关闭录音接口如下
::

    /// <summary>
    /// 通话录音，通过JCCallItem对象中的呼叫保持状态来决定开启关闭呼叫保持
    /// </summary>
    /// <param name="item">JCCallItem对象</param>
    /// <param name="enable">开启关闭录音</param>
    /// <param name="filePath">录音文件路径</param>
    /// <returns>返回true表示正常执行调用流程，false表示调用异常</returns>
    public bool audioRecord(JCCallItem item, bool enable, string filePath)
   

示例代码::

        JCCallItem item = call.callItems[0];
        if (item.audioRecord)
        {
            // 录音结束
            call.audioRecord(item, false, "your filePath");
        }
        else
        {
            // 创建录音文件保存路径
            String filePath; // 录音文件的绝对路径，SDK会自动创建录音文件
            if (filtPath.Length > 0)
            {
                // 开始录音
                call.audioRecord(item, true, filePath);
            }
        }
    }