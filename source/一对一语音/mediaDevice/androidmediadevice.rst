.. _设备控制(android):

音频设备管理
=========================

.. highlight:: java


音频数据管理
---------------------

原始音频数据
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

在音频传输过程中，可以对采集的音频数据进行处理，以获取不同的播放效果。有两个时机可以进行处理。

1. 在音频采集之后，编码之前进行处理；

2. 在传输完成，解码后播放前进行处理。

具体如下：

- 音频采集之后，编码之前的处理

首先注册音频输入回调
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
    ZmfAudio.inputAddCallback(ZmfAudio.InputCallback var0) 
    
回调注册后，当有音频数据采集进来时，可以进行对应的音频数据处理。 

如果想移除回调，调用下面的接口
::

    ZmfAudio.inputRemoveCallback(ZmfAudio.InputCallback var0)

- 解码后播放前的处理

首先注册音频输出回调
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
    
    ZmfAudio.outputAddCallback(ZmfAudio.OutputCallback var0)

回调注册后，当有解码后的音频数据进来时，可以进行对应的音频数据处理。 

如果想移除回调，调用下面的接口
::

    ZmfAudio.outputRemoveCallback(ZmfAudio.OutputCallback var0)


^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

自定义音频采集和渲染
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

在实时音频传输过程中，JC SDK 会启动默认的音频模块进行音频采集。但是对于不支持系统标准 API 的音频设备，或者想利用自己已经拥有的音频模块进行音频的采集和传输前处理时，可另起采集/播放线程，把自己采集/需要播放的音频数据放入菊风对应的接口中进行后续操作。

自定义音频采集接口如下（在收到登录成功的回调后调用）：

若需要使用自己的音视频设备并且 Zmf_AudioInitialize 初始化成功，在下面的回调函数中操作音频设备；

采集数据输入接口
::

    /**
     * The audio input data entry to ZMF, each callback will obtain the data.
     * Multiple data will mix in the callback of the jssmme Engine,
     * and the first input will be main channel.
     *
     * @param[in] inputId       unique name of the audio input           //输入设备id
     * @param[in] sampleRateHz  the sample rating of the pcm data        //采样率
     * @param[in] iChannels     the channel number of the pcm data       //通道数量
     * @param[in] buf           the pcm data                             //外部采集数据源
     * @param[in] len           the pcm data length                      //对应数据长度
     * @param[in,out] micLevel                                           //音量
     * @param[in] playDelayMS                                            //播放时延 通常取0
     * @param[in] recDelayMS                                             //采集时延 通常取0
     * @param[in] clockDrift                                             //时钟漂移 通常取0
     *
     */
    void Zmf_OnAudioInput (const char *inputId, int sampleRateHz, int iChannels, unsigned char *buf, int len, int *micLevel, int playDelayMS, int recDelayMS, int clockDrift); 

.. note::  此接口为将自己采集的音频数据输入到 JC SDK。


采集停止接口
::

    /**
     * tell ZMF the audio input has stopped
     *
     * @param[in] inputId       unique name of the device                  //输入设备id  
     */
    void Zmf_OnAudioInputDidStop(const char *inputId);


如果想在音频输出端使用自定义的播放数据，则调用下面的接口：

播放数据输入接口
::

    /**
     * The outlet which audio output can get data from.
     *
     * @param[in] outputId      unique name of the audio output           //输出设备id      
     * @param[in] sampleRateHz  the sample rating of the pcm data         //采样率 
     * @param[in] iChannels     the channel number of the pcm data        //通道数量

     * @param[in] buf           the pcm data to be filled                 //外部采集数据源 
     * @param[in] len           the pcm data length                       //对应数据buf长度
     */
    void Zmf_OnAudioOutput (const char *outputId, int sampleRateHz, int iChannels, unsigned char *buf, int len);

.. note::  此接口为将自定义音频输出数据输入到 JC SDK。

播放数据停止接口
::

    /**
     * tell ZMF the audio output has stopped
     *
     * @param[in] outputId      unique name of the device                  //输出设备id  
     */
    void Zmf_OnAudioOutputDidStop      (const char *outputId);


.. note:: 

     在自定义音频采集场景中，开发者需要自行管理音频数据的采集。在自定义音频渲染场景中，开发者需要自行管理音频数据的播放。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

音频设备管理
---------------------

音频设备管理主要用到 JCMediaDevice 类中的方法，具体如下：

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


