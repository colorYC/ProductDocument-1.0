媒体管理
=======================

动态开关本地媒体
::

    jc.update(constraints) 

       constraints  标准 MediaStreamConstraints object, 
                    按如下格式
                        {
                            "audio":true,
                            "video": {
                                "width": {"max": 640},
                                "height":{"max": 480}
                            }
                        }
                        max 可以限制视频最大尺寸, min 限制最小尺寸

