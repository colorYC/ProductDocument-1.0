**屏幕共享推流问题**

在开启屏幕共享之前，首先设置智能布局模式，参考 `此处 <http://developer.juphoon.com/portal/document/html/webrtc/webrtcreference.html#id2>`_ 。

::

        底层接口try {
                JSONObject object = new JSONObject();
                object.put( "VideoWidth", 480);
                object.put("VideoHeight", 480);
                object.put("MergeMode", 2);
                object.put("ScsMode",4);
                object.put("MergeFPS", 24);
                object.put("UserName",  "");

                param.param = object.toString();
                param.name = "Mtc_ConfConfigMediaMerge_i";
                ret = MtcConf.Mtc_ConfCommand(param.confId, param.name, param.param);
            } catch (JSONException e) {
                e.printStackTrace();
            }


.. note:: 具体参数值可以参考文档。

- 屏幕共享推流分辨率问题

如需修改推流录制或推流分辨率，可以参考 `此处 <http://developer.juphoon.com/portal/document/html/webrtc/webrtcreference.html#id5>`_ 。


        /// <summary>
        /// 设置屏幕桌面共享采集属性
        /// </summary>
        /// <param name="width">采集宽度</param>
        /// <param name="height">采集高度</param>
        /// <param name="framerate">帧速率</param>
        public void setScreenCaptureProperty(int width, int height, int framerate)
        {
            _screenCaptureWidth = width;
            _screenCaptureHeight = height;
            _screentCaptureFramerate = framerate;
        }

.. note:: 该方法在开启屏幕共享前调用。

要在 Cdn 状态变为 ready 时调用。因为推流实际上也算作一个成员，所以只有在这个推流成员加入以后才能发生开启推流的命令。



        // 屏幕共享和摄像头视频mix后一起推流， 推流器id具体值需要获取才可以，目前没有暴露这个接口，可以更改发临时包
        mediaChannel.sendMessage("text", "@推流器id  setmerge 0x400  3  24", null);直接这么设置就可以吗？
