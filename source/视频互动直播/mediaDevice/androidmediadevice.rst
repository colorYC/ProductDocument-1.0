.. _设备控制(android):

音频管理
=========================

.. highlight:: java

音频数据管理
------------------------

原始音频数据
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

在音频传输过程中，可以对采集的音频数据进行处理，以获取不同的播放效果。有两个时机可以进行处理。

1. 在音频采集之后，编码之前进行处理；

2. 在传输完成，解码后播放前进行处理。

具体如下：

- 音频采集之后，编码之前的处理

参考如下步骤，在你的项目中实现原始音频数据功能：

1.发起业务前通过 ZmfAudio.inputAddCallback 注册音频采集回调，并在该函数中实现一个 ZmfAudio.InputCallback 类型的回调函数

2.成功注册后，JC SDK 会在捕捉到音频数据时通过回调函数回调采集到的原始音频数据相对应参数

3.用户拿到音频数据后，根据场景需要自行在回调函数中进行参数处理，处理后数据通过该回调函数返回给JC SDK。


首先注册音频输入回调
::

    /**
     * add audio input data callback
     *
     * @param[in] callback      the callback user data
     *
     * @return                  void.
     */
    ZmfAudio.inputAddCallback(ZmfAudio.InputCallback var0);

回调类型说明
::

    /** the callback to receive audio input data
     * @param[in] inputId         unique name of the audio input
     * @param[in] iSampleRateHz   the sample rate of the pcm data
     * @param[in] iChannels       the channel number of the pcm data
     * @param[in] data             the pcm data
     * @param[in] playDelayMS      the play delay ms
     * @param[in] recDelayMS       the record dely ms
     * @param[in] clockDrift       the clock drift ms
     * @return                     void
     */
    public interface InputCallback
    {
        void onAudioInputFrame(String inputId, int sampleRateHz, int iChannels, ByteBuffer data, int playDelayMS, int recDelayMS, int clockDrift);
    }


示例代码
::

    public void onAudioInputFrame(String inputId, int sampleRateHz, int iChannels, ByteBuffer data, int playDelayMS, int recDelayMS, int clockDrift) {
        System.out.println("音频采集数据处理");
    }
    public void joinChannel() {
        //注册回调
        ZmfAudio.inputAddCallback(onAudioInputFrame);
        // 加入直播
        mediaChannel.join("频道id", null);


回调注册后，当有音频数据采集进来时，可以对音频数据进行处理。  

如果想移除回调，调用下面的接口
::

    /**
     * remove input audio data callback
     *
     * @param[in]  callback      the callback user data
     *
     * @return                  void.
     */
    ZmfAudio.inputRemoveCallback(ZmfAudio.InputCallback var0);


示例代码
::

    public void leaveChannel() {
        //移除回调
        ZmfAudio.inputRemoveCallback(onAudioInputFrame);
        JCCallItem item = call.getCallItems().get(0);
        // 离开直播
        mediaChannel.leave();
    }


- 解码后播放前的处理

参考如下步骤，在你的项目中实现原始音频数据功能：

1.发起业务前通过 ZmfAudio.outputAddCallback 注册音频输出回调，并在该函数中实现一个 ZmfAudio.OutputCallback 类型的回调函数

2.成功注册后，JC SDK 会在捕捉到音频数据时通过回调函数回调采集到的原始音频数据相对应参数

3.用户拿到音频数据后，根据场景需要自行在回调函数中进行参数处理，处理后数据通过该回调函数返回给JC SDK。

首先注册音频输出回调
::

    /**
     * add audio output data callback
     *
     * @param[in] callback      the callback user data
     *
     * @return                  >= 0 on succeed is handle, otherwise failed.
     */
    ZmfAudio.outputAddCallback(ZmfAudio.OutputCallback var0);

回调类型说明
::

    /**
     * The callback to receive audio output data
     *
     * @param[in] captureId       audio output unique name
     * @param[in] iSampleRateHz   the sample rate of the pcm data
     * @param[in] iChannels       the channel number of the pcm data
     * @param[in] data            the pcm data
     *
     * @return                    void
     */
    public interface OutputCallback
    {
        void onAudioOutputFrame(String outputId, int sampleRateHz, int iChannels, ByteBuffer data);
    }


回调注册后，当有解码后的音频数据进来时，可以进行对应的音频数据处理。 

示例代码
::

    public void void onAudioOutputFrame(String outputId, int sampleRateHz, int iChannels, ByteBuffer data) {
        System.out.println("音频解码后的数据处理");
    }
    public void joinChannel() {
        //注册回调
        ZmfAudio.outputAddCallback(onAudioOutputFrame);
        // 加入直播
        mediaChannel.join("频道id", null);
    }


如果想移除回调，调用下面的接口
::

    /**
     * remove output audio data callback
     *
     * @param[in] callback      the callback user data
     *
     * @return                  void.
     */
    ZmfAudio.outputRemoveCallback(ZmfAudio.OutputCallback var0)


示例代码
::

    public void leaveChannel() {
        //移除回调
        ZmfAudio.outputRemoveCallback(onAudioOutputFrame);
        JCCallItem item = call.getCallItems().get(0);
        // 离开直播
        mediaChannel.leave();
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

自定义音频采集和渲染
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

在实时音频传输过程中，JC SDK 会启动默认的音频模块进行音频采集。但是对于不支持系统标准 API 的音频设备，或者想利用自己已经拥有的音频模块进行音频的采集和传输前处理时，可另起采集/播放线程，把自己采集/需要播放的音频数据放入菊风对应的接口中进行后续操作。

参考如下步骤，在你的项目中实现自定义音频源功能：

1.通过 JC SDK 提供的接口将外部设备采集/自定义的播放数据输入到 JC SDK 进行后续操作。

2. 如果想停止外部设备采集/自定义的播放数据的输入，则调用 JC SDK 提供的接口停止数据输入即可。


自定义音频采集接口如下（在收到登录成功的回调后调用）：

若需要使用自己的音视频设备并且 Zmf_AudioInitialize 初始化成功，在下面的回调函数中操作音频设备；

采集数据输入接口
::

    /**
     * The audio input data entry to ZMF
     *
     * @param[in] inputId       unique name of the audio input                       //输入设备id
     * @param[in] sampleRateHz  the sample rating of the pcm data                    //采样率 取值范围：8000，16000，32000，44100，48000取决于外部
     * @param[in] iChannels     the channel number of the pcm data                   //通道数量 取值范围：1或2
     * @param[in] data          the pcm data                                         //外部采集数据源
     * @param[in] playDelayMS   playout delay in ms                                  //播放时延 通常取0
     * @param[in] recDelayMS    record delay in ms                                   //采集时延 通常取0
     * @param[in] clockDrift    clock drift between playout and record timestamp //时钟漂移 通常取0
     *
     */
    static public void onInput (String inputId, int sampleRateHz, int iChannels, ByteBuffer data,
            						 int playDelayMS, int recDelayMS, int clockDrift)


.. note::  此接口为将自己采集的音频数据输入到 JC SDK。

示例代码
::

    //初始化音频设备
    android.content.Context activity;
    ZmfAudio.initialize(activity);
    public void joinChannel() {
        // 输入长度为length，采样频率16000，通道数为1的pcm数据片段
        onInput("Test",16000,1,pcmdata,length,0,0,0,0);
        // 加入直播
        mediaChannel.join("频道id", null);
    }


采集停止接口
::

    /**
     * tell ZMF the audio input has stopped
     *
     * @param[in] inputId       unique name of the device              //输出设备id 
     */
    static public void onInputDidStop(String inputId)


示例代码
::

    public void leaveChannel() {
        //停止采集
        onInputDidStop("Test");
        // 离开直播
        mediaChannel.leave();
    }

如果想在音频输出端使用自定义的播放数据，则调用下面的接口：

播放数据输入接口
::

    **
     * The outlet which audio output can get data from.
     *
     * @param[in] outputId      unique name of the audio output       //输出设备id
     * @param[in] sampleRateHz  the sample rating of the pcm data     //采样率 取值范围：8000，16000，32000，44100，48000取决于外部
     * @param[in] iChannels     the channel number of the pcm data    //通道数量 取值范围：1或2
     * @param[in] data          the pcm data to be filled             //外部采集数据源
     */
    static public void onOutput (String outputId, int sampleRateHz, int iChannels, ByteBuffer data);


.. note::  此接口为将自定义音频输出数据输入到 JC SDK。

示例代码
::

    //初始化音频设备
    android.content.Context activity;
    ZmfAudio.initialize(activity);
    public void joinChannel() {
        // 输入长度为length，采样频率16000，通道数为1的pcm数据片段
        onOutput("Test",16000,1,buf,length);
        // 加入直播
        mediaChannel.join("频道id", null);
    }

播放数据停止接口
::

    /**
     * tell ZMF the audio output has stopped
     *
     * @param[in] inputId       unique name of the device         //输入设备id
     */
    static public void onOutputDidStop(String outputId)

.. note:: 

     在自定义音频采集场景中，开发者需要自行管理音频数据的采集。在自定义音频渲染场景中，开发者需要自行管理音频数据的播放。

示例代码
::

    public void leaveChannel() {
        //停止播放数据，业务中或者业务结束时均可以调用
        onOutputDidStop("Test");
        // 离开直播
        mediaChannel.leave();
    }


^^^^^^^^^^^^^^^^^^^^^^^^^^^

音频设备管理
-----------------------

开启/关闭扬声器
>>>>>>>>>>>>>>>>>>>>>>>>

::

    /**
     * 开启关闭扬声器
     *
     * @param enable 是否开启
     */
    public abstract void enableSpeaker(boolean enable);


开启/关闭音频设备
>>>>>>>>>>>>>>>>>>>>>>>>

::

    /**
     * 启动音频，一般正式开启通话前需要调用此接口
     *
     * @return 成功返回 true，失败返回 false
     */
    public abstract boolean startAudio();

    /**
     * 停止音频，一般在通话结束时调用
     *
     * @return 成功返回 true，失败返回 false
     */
    public abstract boolean stopAudio();


**示例代码**

::

    // 开启扬声器
    mediaDevice.enableSpeaker(true);
    // 开启音频设备
    mediaDevice.startAudio();
    // 关闭音频设备
    mediaDevice.stopAudio();
