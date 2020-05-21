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

您可以在通话中进行录音，并设置录音文件的路径。开启或关闭录音需要根据当前的录音状态来决定，录音状态（audioRecord）可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_ 对象中的 getAudioRecord() 方法获得。接口如下：

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

.. _视频通话录制(android):

视频通话录制
----------------------------

菊风一对一通话视频录制为您提供 **本地视频源录制和远端视频源录制**。

开启或者关闭视频录制需要根据本端视频源录制状态（localVideoRecord）和远端视频源录制状态（remoteVideoRecord）来决定。视频录制状态可通过 `JCCallItem <http://developer.juphoon.com/portal/reference/android/com/juphoon/cloud/JCCallItem.html>`_ 对象中的 getLocalVideoRecord() 方法和 getRemoteVideoRecord() 方法获得。此外，您可以对录制视频的高度和宽度以及存储路径进行设置，接口如下：
::

    /**
     * 视频通话录制，通过 JCCallItem 对象中的localVideoRecord和remoteVideoRecord状态来决定开启关闭录制
     *
     * @param item      JCCallItem 对象
     * @param enable    开启关闭录制
     * @param remote    是否为远端视频源
     * @param width     录制视频宽像素
     * @param height    录制视频高像素
     * @param filePath  录制视频文件存储路径
     * @return          返回 true 表示正常执行调用流程，false 表示调用异常
     */
    public abstract boolean videoRecord(JCCallItem item, boolean enable, boolean remote, int width, int height, String filePath);

示例代码::

    public void onVideoRecord() {
        JCCallItem item = call.getCallItems().get(0);
        if (item.getLocalVideoRecord()) {
            if (call.videoRecord(item, false, false, 0, 0, "your filePath")) { // 本端录制结束
                ...
            } else {
                // 本端录制结束失败
            }
        } else if (item.getRemoteVideoRecord()) {
            if (call.videoRecord(item, false, true, 0, 0, "your filePath")) { // 远端录制结束
                ...
            } else {
               // 远端录制结束失败
            }
        } else {
            // 创建视频录制文件保存路径
            String filePath; // 视频录制文件的绝对路径，SDK会自动创建视频录制文件
            if (!TextUtils.isEmpty(filePath)) {
                if (!call.videoRecord(item, true, false, 640, 360, filePath)) {
                    // 视频录制失败
                    File file = new File(filePath);
                    file.delete();
                }
            } else {
                // 创建视频录制文件保存路径失败
            }
        }
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _截屏(android):

截屏
------------------------------

在视频通话中，如果想对当前的通话界面进行保存，可以使用截屏功能，截屏分为 **本端视频源截图和远端视频源截图**，接口如下：

::

    /**
     * 视频通话截图
     *
     * @param width     截屏宽度像素，-1为视频源像素
     * @param height    截屏高度像素，-1为视频源像素
     * @param filePath  文件路径
     */
    public boolean snapshot(int width, int height, String filePath)

示例代码::
  
        JCMediaDeviceVideoCanvas canvas = mediaDevice.startVideo(renderId, JCMediaDevice.RENDER_FULL_CONTENT);
        String filePath; // 截屏文件的绝对路径，SDK会自动创建截屏文件
        canvas.snapshot(-1, -1, filePath);


^^^^^^^^^^^^^^^^^^^^^^^^^^^

.. _推送(android):

推送
-----------------------------

通过集成推送，可以将通话信息即时告知用户，从而高通话的接通率。推送分为 Android 端的小米推送、华为推送以及苹果端的 VoIP 推送，详细集成步骤请参考 :ref:`推送<推送>` 模块。