.. _设备控制(mac):

音频管理
============================

.. highlight:: objective-c


音频数据管理
---------------------

原始音频数据
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

在音频传输过程中，可以对采集的音频数据进行处理，以获取不同的播放效果。有两个时机可以进行处理。

1. 在音频采集之后，编码之前进行处理；

2. 在传输完成，解码后播放前进行处理。

具体如下：

- 音频采集之后，编码之前的处理

参考如下步骤，在你的项目中实现原始音频数据功能：

1.发起业务前通过 Zmf_AudioInputAddCallback 注册音频采集回调，并在该函数中实现一个 ZmfAudioInputCallback 类型的回调函数

2.成功注册后，JC SDK 会在捕捉到音频数据时通过回调函数回调采集到的原始音频数据相对应参数

3.用户拿到音频数据后，根据场景需要自行在回调函数中进行参数处理，处理后数据通过该回调函数返回给JC SDK。


首先注册音频输入回调
::

    /**
     * add mic data callback
     *
     * @param[in] pUser         the callback user data
     * @param[in] pfnCb         the callback
     *
     * @return                  0 on succeed, otherwise failed.
     */
    int Zmf_AudioInputAddCallback(void *pUser, ZmfAudioInputCallback pfnCb); 

回调类型说明::

    /** the callback to receive audio input data 
     *
     * @param[in] pUser  the user data registered by Zmf_AddAudioInputCallback
     * @param[in] inputId       unique name of the audio input
     * @param[in] iSampleRateHz  the sample rating of the pcm data
     * @param[in] iChannels     the channel number of the pcm data
     * @param[in] buf           the pcm data
     * @param[in] len           the pcm data length
     * @param[in,out] micLevel  
     * @param[in] playDelayMS   the play delay ms
     * @param[in] recDelayMS    the record dely ms
     * @param[in] clockDrift    the clock drift ms
     */
    typedef void (*ZmfAudioInputCallback)(void* pUser, const char* inputId, int iSampleRateHz, int iChannels,
                                       unsigned char *buf, int len, int *micLevel,
                                       int playDelayMS, int recDelayMS, int clockDrift);


回调注册后，当有音频数据采集进来时，可以对音频数据进行处理。 

示例代码
::

    id data; //采集的音频数据对象
    void* p = (__bridge void *)data;
    static void zmfAudioInputCallback(void* pUser, const char* inputId, int iSampleRateHz, int iChannels,
                                       unsigned char *buf, int len, int *micLevel,
                                       int playDelayMS, int recDelayMS, int clockDrift) {

        NSLog(@"音频数据处理");
    }
    - (void)call {
        //注册回调
        Zmf_AudioInputAddCallback(p, zmfAudioInputCallback);
        //发起呼叫
        [call call:@"对端号码" video:true extraParam:@"自定义透传字符串"];
    }


如果想移除回调，调用下面的接口
::

    /**
     * remove mic data callback
     *
     * @param[in]  pUser        the callback user data
     *
     * @return                  0 on succeed, otherwise failed.
     */
    int Zmf_AudioInputRemoveCallback(void *pUser);


示例代码
::

    id data; //采集的音频数据对象
    void* p = (__bridge void *)data;
    -(void)endCall {
        //移除回调
        Zmf_AudioInputRemoveCallback(p);
        //挂断通话
        [call term:item reason:JCCallReasonNone description:@"自己挂断"];
    }


- 解码后播放前的处理

参考如下步骤，在你的项目中实现原始音频数据处理功能：

1.发起业务前通过 Zmf_AudioOutputAddCallback 注册音频输出回调，并在该函数中实现一个 ZmfAudioOutputCallback 类型的回调函数。

2.成功注册后，JC SDK 会在捕捉到音频数据时通过回调函数回调采集到的原始音频数据。

3.用户拿到音频数据后，根据场景需要自行在回调函数中进行音频数据的处理，处理后的数据通过该回调函数返回给 JC SDK。


首先注册音频输出回调
::
    /**
     * add fill speak callback
     *
     * @param[in]  pUser        the callback user data
     *
     * @return                  0 on succeed, otherwise failed.
     */
    int  Zmf_AudioOutputAddCallback     (void *pUser, ZmfAudioOutputCallback pfnCb);

回调类型说明
::

    /** the callback to get audio output buffer, when Zmf_OnAudioOutput() invoked.
     *
     * @param[in] pUser          the callback user data
     * @param[in] outputId       unique name of the audio output
     * @param[in] iSampleRateHz  the sample rating of the pcm data
     * @param[in] iChannels      the channel number of the pcm data
     * @param[in] buf            the pcm data
     * @param[in] len            the pcm data length
     */
    
    /** the callback to fill audio output buffer
     * @param[in] pUser  the user data registered by Zmf_AudioOutputAddCallback
     */
    typedef int  (*ZmfAudioOutputCallback)(void* pUser, const char* outputId, int iSampleRateHz, int iChannels,
                                         unsigned char *buf, int len);

回调注册后，当有解码后的音频数据进来时，可以进行对应的音频数据处理。 

示例代码
::

    static void zmfAudioOutputCallback(void* pUser, const char* outputId, int iSampleRateHz, int iChannels,
                                         unsigned char *buf, int len) {

        NSLog(@"音频数据处理");
    }
    id data; //解码后的音频数据
    void* p = (__bridge void *)data;
    - (void)call {
        //注册回调
        Zmf_AudioOutputAddCallback(p, zmfAudioOutputCallback);
        //发起业务
        [call call:@"对端号码" video:true extraParam:@"自定义透传字符串"];
    }


如果想移除回调，调用下面的接口
::

    /**
     * remove fill speak callback
     *
     * @param[in]  pUser        the callback user data
     *
     * @return                  0 on succeed, otherwise failed.
     */
    int  Zmf_AudioOutputRemoveCallback  (void *pUser);

取消注册回调
::

    id data; //解码后的音频数据
    void* p = (__bridge void *)data;
    - (void)endCall {
        //取消注册回调
        Zmf_AudioOutputRemoveCallback(p);
        //挂断通话
        [call term:item reason:JCCallReasonNone description:@"自己挂断"];
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

音频设备初始化接口
::

    /**
     * @brief Initialize Audio module of ZMF(Zero Media Framework).
     * @param  applicationContext For Windows, it can be the handle of the window,
     *                            The notification event will be sent to that window.
     *                            Or it can be callback function of type ZmfEventListenCallback.
     *                            For Android, it must be the Context.
     *                            For iOS, it should be NULL and is ignored.
     * @return                    0 on succeed, otherwise failed.
     */
    int Zmf_AudioInitialize(void *applicationContext);

采集数据输入接口
::

       /**
        * The audio input data entry to ZMF, each callback will obtain the data.
        * Multiple data will mix in the callback of the jssmme Engine,
        * and the first input will be main channel.
        *
        * @param[in] inputId       unique name of the audio input       //输入设备id
        * @param[in] sampleRateHz  the sample rating of the pcm data    //外部采样率 取值范围：8000，16000，32000，44100，48000取决于外部
        * @param[in] iChannels     the channel number of the pcm data   //通道数量 取值范围：1或2
        * @param[in] buf           the pcm data                         //外部采集数据源
        * @param[in] len           the pcm data length                  //对应数据长度
        * @param[in,out] micLevel                                       //音量，取值范围：0-100，会根据实际输入音频返回音量值
        * @param[in] playDelayMS                                        //播放时延 通常取0
        * @param[in] recDelayMS                                         //采集时延 通常取0
        * @param[in] clockDrift                                         //时钟漂移 通常取0
        *
        */
        void Zmf_OnAudioInput (const char *inputId, int sampleRateHz, int iChannels, unsigned char *buf, int len, int *micLevel, int playDelayMS, int recDelayMS, int clockDrift); 

示例代码
::

    //初始化音频设备
    Zmf_AudioInitialize(NULL);
    -(void)voiceCall {
        // 输入长度为length，采样频率16000，通道数为1的pcm数据片段
        Zmf_OnAudioInput("Test",16000,1,pcmdata,length,0,0,0,0);
        //发起呼叫
        [call call:@"对端号码" video:true extraParam:@"自定义透传字符串"]; 
    }

.. note::  此接口为将自己采集的音频数据输入到 JC SDK。


采集停止接口
::

    /**
     * tell ZMF the audio input has stopped
     *
     * @param[in] inputId       unique name of the device  //输入设备id  
     */
    void Zmf_OnAudioInputDidStop(const char *inputId);


示例代码
::

    -(void)endCall {
        //停止采集
        Zmf_OnAudioInputDidStop("Test");
        //挂断通话
        [call term:item reason:JCCallReasonNone description:@"自己挂断"];
    }


如果想在音频输出端使用自定义的播放数据，则调用下面的接口：

播放数据输入接口
::

       /**
        * The outlet which audio output can get data from.
        *
        * @param[in] outputId      unique name of the audio output       //输出设备id      
        * @param[in] sampleRateHz  the sample rating of the pcm data     //采样率  取值范围：8000，16000，32000，44100，48000取决于外部
        * @param[in] iChannels     the channel number of the pcm data    //通道数量 取值范围：1或2
        * @param[in] buf           the pcm data to be filled             //外部采集数据源 
        * @param[in] len           the pcm data length                   //对应数据buf长度
        */
        void Zmf_OnAudioOutput (const char *outputId, int sampleRateHz, int iChannels, unsigned char *buf, int len);


示例代码
::

    //初始化音频设备
    Zmf_AudioInitialize(NULL);
    -(void)call {
        // 输入长度为length，采样频率16000，通道数为1的pcm数据片段
        Zmf_OnAudioOutput("Test",16000,1,buf,length);
        //发起呼叫
        [call call:@"对端号码" video:true extraParam:@"自定义透传字符串"];
    }


.. note::  此接口为将自定义音频输出数据输入到 JC SDK。

播放数据停止接口
::

    /**
     * tell ZMF the audio output has stopped
     *
     * @param[in] outputId      unique name of the device    //输出设备id  
     */
    void Zmf_OnAudioOutputDidStop(const char *outputId);


示例代码
::

    -(void)endCall {
        //停止播放数据
        Zmf_OnAudioOutputDidStop("Test");
        //挂断通话
        [call term:item reason:JCCallReasonNone description:@"自己挂断"];
    }
  

.. note:: 

     在自定义音频采集场景中，开发者需要自行管理音频数据的采集。在自定义音频渲染场景中，开发者需要自行管理音频数据的播放。

^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

音频设备管理
---------------------

音频设备管理主要用到 JCMediaDevice 类中的方法，具体如下：

获取音频输入设备列表
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

通过 JCMediaDevice 类的 audioInputs 属性和 audioInput 属性获取音频输入设备列表及当前音频输入设备，将会返回 JCMediaDeviceAudio 对象。

::

    /// 音频输入设备列表
    @property (nonatomic, readonly) NSArray<JCMediaDeviceAudio*>* __nonnull audioInputs;

    /// 当前音频输入设备
    @property (nonatomic, strong) JCMediaDeviceAudio* __nullable audioInput;

JCMediaDeviceAudio 对象的属性如下：

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 描述
   * - 音频设备id
     - NSString
     - audioId
   * - 音频设备名字
     - NSString
     - audioName
   * - 输入还是输出设备
     - bool
     - input


获取音频输出设备列表
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

通过 JCMediaDevice 类的 audioOutputs 属性和 audioOutput 属性获取音频输出设备列表及当前音频输出设备
::

    /// 音频输出设备列表
    @property (nonatomic, readonly) NSArray<JCMediaDeviceAudio*>* __nonnull audioOutputs;

    /// 当前音频输出设备
    @property (nonatomic, strong) JCMediaDeviceAudio* __nullable audioOutput;

JCMediaDeviceAudio 对象的属性如下：

.. list-table::
   :header-rows: 1

   * - 名称
     - 类型
     - 描述
   * - 音频设备id
     - NSString
     - audioId
   * - 音频设备名字
     - NSString
     - audioName
   * - 输入还是输出设备
     - bool
     - input


示例代码::


    // 获取音频输入设备列表
    NSArray<JCMediaDeviceAudio*>* audioInputDevices = mediaDevice.audioInputs;

    // 获取音频输出设备列表
    NSArray<JCMediaDeviceAudio*>* audioOutputDevices = mediaDevice.audioOutputs;


开启/关闭音频设备
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    /**
     *  @brief 启动音频，一般正式开启通话前需要调用此接口
     *  @return 成功返回 true，失败返回 false
     */
    -(bool)startAudio;

    /**
     *  @brief 停止音频，一般在通话结束时调用
     *  @return 成功返回 true，失败返回 false
     */
    -(bool)stopAudio;


示例代码::

    // 打开音频设备
    [mediaDevice startAudio];
    // 关闭音频设备
    [mediaDevice stopAudio];

