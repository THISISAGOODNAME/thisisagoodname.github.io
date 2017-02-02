---
layout: post
title: "相机研究"
description: "相机研究"
category: Qt
tags: [Qt,多媒体]
---

&#160; &#160; &#160; &#160;最近接手了扫一扫的开发工作，
感觉和算法工程师越来越远了。反正以后要经常和相机打交道，
就一口气把相机的一些配置研究了一下。

<!-- more -->

* Table of Contents
{:toc}

# Qt Quick Camera

Camera对象用来访问和控制系统的物理设备，完成拍照功能。它能控制闪光、曝光、聚焦等拍照相关的各种配置

**digitalZoom** 配置数字变焦；maximumDigitalZoom保存相机支持的最大数字变焦倍数，如果是1.0，则说明不支持数字变焦。

**opticalZoom** 配置光学变焦(支持的手机很少)；maximumOpticalZoom保存最大变焦倍数，如果是1.0表示不支持光学变焦。

**captureMode** 属性是枚举值，支持`Camera.CaptureViewfinder`(仅预览)、`Camera.CaptureStillImage`(捕捉静态图片)、`Camera.CaptureVideo`(录制视频)三种模式。

**focus** focus是Camera的一个属性，类型是CameraFocus，用来控制聚焦和焦点模式。

{% highlight javascript %}
Camera {
    id: camera
    focus {
        focusMode: Camera.FocusMacro
        focusPointMode: Camera.FocusPointCustom
        customFocusPoint: Qt.point(0.2, 0.2)
    }
}
{% endhighlight %} 

1. *focusMode* 属性是个枚举值，对应的枚举类型对应6种聚焦方式，`Camera.ManualFocus`代表手动聚焦，`Camera.AutoFocus`代表自动聚焦，`Camera.MacroFocus`代表微距聚焦，用于拍摄离相机很近的物体。手机上不是每种方式都支持，`CameraFocus`类的`isFocusModeSupported(mode)`用来查询物理设备是否支持指定的聚焦方式。
2. *focusPointMode* 属性定义焦点模式，也是枚举值，对应4种焦点模式：`Camera.FocusPointAuto`(自动焦点)、`Camera.FocusPointCenter`(中心焦点)、`Camera.FocusPointFaceDetection`(面部识别)、`Camera.FocusPointCustom`(自定义焦点，使用`customFocusPoint`属性指定的点作为焦点)。设置焦点模式前最好也判断下设备是否支持，`isFocusPointMode(mode)`方法可以判断。

**exposure** CameraExpose用来控制相机的曝光选项，你可以通过Camera的exposure属性完成这项复杂有晦涩的工作。CameraExposure允许控制曝光相关的各种选项，如果想制作专业的拍照应用，一定要弄明白各种选项的含义，还要知道如何组合不同的选项以获得最佳的拍照效果。

{% highlight javascript %}
Camera {
    id: camera
    
    exposure.manualAperture: 3.5
    exposure.manualShutterSpeed: 0.1
    exposure.exposureCompensation: -1.0
    exposure.exposureMode: Camera.ExposureAuto
    Component.onCompleted: {
        exposure.setAutoIsoSensitivity();
    }
}
{% endhighlight %} 

1. *exposureMode* 属性是枚举类型，可以取`Camera.ExposureAuto`(自动)、`Camera.ExposureManual`(手动)、`Camera.ExposureSports`(运动)、`Camera.ExposureLargeAperture`(大光圈、近景)、`Camera.ExposureSmallAperture`(小光圈、远景)、`Camera.ExposureBacklight`(背光)、`Camera.ExposureNight`(夜间)、`Camera.ExposurePortrait`(人物)、`Camera.ExposureBeach`(海岸)等
2. *shutterSpeed* 属性表示当前快门速度，`manualShutterSpeed`代表手动设置快门速度，单位是秒。`setAutoShutterSpeed()`打开自动快门模式。
3. *iso* 属性表示当前的感光系数，`manualIso`保存手动设置的感光度，`setAutoIsoSensitivity()`可以打开自动感光模式。iso值越大，感光越快，成像越模糊；iso值越小，感光越慢，得到的图像越细腻。
4. *aperture* 属性表示当前光圈大小，`manualAperture`表示手动设置的光圈大小，`setAutoAperture()`可以打开自动模式。光圈一般以F加数字表示，如F3.5。
5. *exposureCompensation* 属性表示曝光补偿。照片的好坏与曝光量有关，就是说，通过多少的光线可以使镜头得到清晰的图像。曝光量又与通光时间(快门速度决定)、通光面积(光圈大小决定)有关，快门速度与光圈大小、感光速度有关···总之，相机的各种设置互相牵扯非常复杂。
 
**flash** flash属性(CameraFlash类型)是控制闪光的，主要的选项就一个：mode。mode是枚举类型，可以取这些值：`Camera.FlashOff`(闪光灯关闭)、`Camera.FlashOn`(闪光灯打开)、`Camera.FlashAuto`(自动)、`Camera.FlashRedEyeReduction`(去红眼)···

{% highlight javascript %}
Camera {
    id: camera
    
    flash.mode: Camera.FlashRedEyeReduction
}
{% endhighlight %} 

**imageProcessing** 很多相机固件支持对镜头捕捉的图片做一些处理，如降噪、白平衡、锐化、对比度、饱和度等。Camera的imageProcessing属性(CameraImageProcessing类型)可以设置这些选项(前提是相机的固件支持)。

1. *constract*属性用来设置对比度，在-1.0~1.0之间，默认是0.
2. *denoisingLevel*属性设置降噪级别，为-1.0表示禁止降噪处理，为0表示应用默认的降噪处理，为1.0表示尽最大可能降噪。
3. *sharpeningLevel*属性设置锐化级别，为-1.0表示禁止锐化，为0表示应用默认的锐化级别，为1.0表示尽最大可能锐化。
4. *whiteBalanceMode*是个枚举值，枚举类属于类CameraImageProcessing，用来设置白平衡模式，有`WhiteBalanceManual`(手动，在此模式下manualWhiteBalance属性生效)、`WhiteBalanceAuto`(自动)、`WhiteBalanceSunlight`(日光)、`WhiteBalanceCloudy`(阴天)、`WhiteBalanceShade`(阴影)、`WhiteBalanceTungsten`(钨光、白炽灯、室内光)、`WhiteBalanceFluorescent`(荧光灯)、`WhiteBalanceFlash`(闪光)、`WhiteBalanceSunset`(日落)等白平衡模式。所谓白平衡，就是数码相机对于白色物体的还原。当我们用肉眼看世界的时候，在不同的光线条件下，对颜色的感觉是基本相同的，比如在清晨太阳初升的时候看到白色物体，感觉他是百的；在夜晚昏暗的灯光下看到白色的物体，依旧感觉他是白色的。但是镜头只能客观地记录环境颜色，并且CCD/CMOS输出经常不平衡，就会导致成像后色彩失真，比如在日光灯下拍出的样片容易发黄···因此需要调整感光器件对于各种颜色的感应强度，使色彩平衡。白色物体在不同光线下，人眼很容易确认其为白色，所以就将白色作为确认其他颜色是否平衡的标准，当白色能反应为白色时，其他颜色也应该正常了，这就是白平衡的含义。
5. *saturation*属性设置饱和度，取值在-1.0~1.0之间，默认值为0。

# 总结

&#160; &#160; &#160; &#160;相机还真是挺复杂的，以前就知道安卓的mediarecorder简单的使用，现在才明白那些玩设备的调光多麻烦啊。