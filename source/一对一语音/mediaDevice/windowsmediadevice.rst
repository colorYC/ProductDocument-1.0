.. _设备控制(windows):

音频管理
=======================

.. highlight:: csharp

音频设备管理
---------------------

音频设备管理主要用到 JCMediaDevice 类中的方法，具体如下：

获取音频输入设备列表
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    /// <summary>
    /// 音频输入设备列表
    /// </summary>
    public List<JCMediaDeviceInput> audioInputDevices

其中，JCMediaDeviceInput 有以下几个变量
::
    
    /// <summary>
    /// 话筒
    /// </summary>
    public class JCMediaDeviceInput {
        /// <summary>
        /// 话筒名称
        /// </summary>
        public string inputName;
        /// <summary>
        /// 话筒id
        /// </summary>
        public string inputId;
    }


获取音频输出设备列表
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    /// <summary>
    /// 音频输出设备列表
    /// </summary>
    public List<JCMediaDeviceOutput> audioOutputDevices

其中，JCMediaDeviceOutput 有以下几个变量::

    /// <summary>
    /// 扬声器
    /// </summary>
    public class JCMediaDeviceOutput {
        /// <summary>
        /// 扬声器名称
        /// </summary>
        public string outputName;
        /// <summary>
        /// 扬声器id
        /// </summary>
        public string outputId;
    }



示例代码::


    // 获取音频输入设备列表
    List<JCMediaDeviceInput> audioInputDevices = mediaDevice.audioInputDevices;

    // 获取音频输出设备列表
    List<JCMediaDeviceOutput> audioOutputDevices = mediaDevice.audioOutputDevices;


开启/关闭音频设备
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    /// <summary>
    /// 启动音频，一般正式开启通话前需要调用此接口
    ///</summary>
    ///<returns>启动成功失败</returns>
    public bool startAudio()

    /// <summary>
    /// 停止音频，一般在通话结束时调用
    /// </summary>
    /// <returns>停止音频成功失败</returns>
    public bool stopAudio()


打开输入设备
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    /// <summary>
    /// 打开输入设备
    /// </summary>
    /// <param name="input">输入设备</param>
    /// <returns>打开输入设备成功失败</returns>
    public bool startInput(JCMediaDeviceInput input)


关闭输入设备
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    /// <summary>
    /// 关闭输入设备
    /// </summary>
    /// <returns>关闭输入设备成功失败</returns>
    public bool stopInput(JCMediaDeviceInput input)


打开输出设备
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

::

    /// <summary>
    /// 打开输出设备
    /// </summary>
    /// <param name="output">输出设备</param>
    /// <returns> 打开输出设备成功失败</returns>
    public bool startOutput(JCMediaDeviceOutput output)


关闭输出设备
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
::

    /// <summary>
    /// 关闭输出设备
    /// </summary>
    /// <returns>关闭输出设备成功失败</returns>
    public bool stopOutput()

示例代码::

    // 打开音频
    mediaDevice.startAudio();

    // 关闭音频
    mediaDevice.stopAudio();

    // 打开输入设备
    mediaDevice.startInput(mediaDevice.audioInputDevices[0]);

    // 打开输出设备
    mediaDevice.startOutput(mediaDevice.audioOutputDevices[0]); 
