Android 进阶
=========================

.. highlight:: java


**集成进阶功能前，请确保您已经进行了模块的初始化**
::

    // 初始化
    JCClient client = JCClient.create(Context, "your appkey", this, null);
    JCMediaDevice mediaDevice = JCMediaDevice.create(client, this);
    JCCall call = JCCall.create(client, mediaDevice, this);

.. note:: SDK 不支持模拟器运行，请使用真机。

.. _通话录音(android):

通话录音
-----------------------------

您可以在通话中进行录音，并设置录音文件的路径。开启或关闭录音需要根据当前的录音状态来决定，如果正在录制或者通话被挂起或者挂起的情况下，不能进行音频录制。录音状态（audioRecord）可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_ 对象中的 getAudioRecord() 方法获得。

录音状态的改变通过 onCallItemUpdate 回调上报
::

    /**
     * 通话状态更新回调（当上层收到此回调时，可以根据 JCCallItem 对象获得该通话的所有信息及状态，从而更新该通话相关UI）
     *
     * @param item           JCCallItem 对象，当 item 为 null 时表示全部更新
     * @param changeParam    更新标识类
     */
    void onCallItemUpdate(JCCallItem item, JCCallItem.ChangeParam changeParam);

开启或关闭录音接口如下
::

    /**
     * 语音通话录音，通过 JCCallItem 对象中的audioRecord状态来决定开启关闭录音
     *
     * @param item      JCCallItem 对象
     * @param enable    开启关闭录音
     * @param filePath  录音文件路径
     * @return          返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean audioRecord(JCCallItem item, boolean enable, String filePath);


示例代码::

        JCCallItem item = call.getCallItems().get(0);
        if (item.getAudioRecord()) {
            // 录音结束
            call.audioRecord(item, false, "your filePath");
        } else {
            // 创建录音保存文件路径
            String filePath; // 录音文件的绝对路径，SDK会自动创建录音文件
            if (!TextUtils.isEmpty(filePath)) {
                // 开始录音
                call.audioRecord(item, true, filePath);
            }
        }


^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _推送(android):

推送
-----------------------------

通过集成推送，可以将通话信息即时告知用户，从而高通话的接通率。推送分为 Android 端的小米推送、华为推送以及苹果端的 APNS 推送 和 VoIP 推送，详细集成步骤请参考 :ref:`推送<推送>` 模块。