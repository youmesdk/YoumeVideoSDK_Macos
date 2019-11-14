# Video SDK for macOS 快速接入

## 概述
游密实时语音SDK（Video SDK）是游密科技公司旗下的一款专注于为开发者提供实时语音技术和服务的云产品。我们的研发团队来自腾讯，其中不少是拥有10年以上音视频经验的专家，专业专注；我们的服务端节点部署遍布全球，为用户提供高效稳定的实时云服务，且弹性可扩展，真正做到亿级支撑、毫秒级延迟、通话清晰流畅、按需取用，为用户带来优质的通话体验。通过Video SDK，能够让您轻松实现游戏内语音通话、主播、指挥、组队等多项功能。

## 四步集成

### Step1：注册账号
在[游密官网](https://console.youme.im/user/register)注册游密账号。

### Step2：添加游戏，获取`Appkey`
在控制台添加游戏，获得接入需要的**Appkey**、**Appsecret**。

### Step3：下载Video SDK包体
根据游戏使用的游戏引擎与开发语言，在[下载入口](https://www.youme.im/download.php?type=Talk)下载对应的SDK包体。

### Step4：调试集成
macOS 平台集成Video SDK：[集成方法](#macOS系统Xcode开发环境配置)

### macOS系统Xcode开发环境配置
添加头文件和依赖库:
1. 打开XCode工程，找到工程目录，新建一个文件夹（可命名为SDKInclude），然后将SDK下的include文件夹里的头文件夹都加入进来(右键点击选择“Add Files to ...”)

  ![](https://youme.im/doc/images/talk_macos_xcode_addFiles.png)

2. 添加库文件路径：Build Settings -> Search Paths -> Library Search Paths，直接将SDK的macos文件夹拖到xcode需要填入的位置，然后路径会自动生成;
3. 添加依赖库：在Build Phases  -> Link Binary With Libraries下添加：`libyoume_voice_engine.a`、`libYouMeCommon.a`、`libffmpeg3.3.a`、`libc++.tbd`、`libsqlite3.0.tbd`、`libz.dylib`、`libz.1.2.5.tbd`、`libresolv.9.tbd`、`SystemConfiguration.framework`、`CoreTelephony.framework`、`AVFoundation.framework`、`AudioToolBox.framework`、`CFNetwork.framework`、`VideoToolbox.framework`、`CoreVideo.framework`、`VideoDecodeAcceleration.framework`。
4. 为macOS10.14 以上版本添加录音权限配置
macOS 10.14 系统使用录音权限，需要在target的`info.plist`中新加`NSMicrophoneUsageDescription`键，值为字符串(授权弹窗出现时提示给用户)。首次录音时会向用户申请权限。配置方式如下：
![macOS10录音权限配置](https://youme.im/doc/images/im_macOS_record_config.jpg)
5. 为macOS 10.14 以上版本添加摄像头使用权限配置
macOS 10.14 系统使用摄像头权限，需要在target的`info.plist`中新加`NSCameraUsageDescription`键，值为字符串(授权弹窗出现时提示给用户)。首次开启摄像头时会向用户申请权限。


## API调用说明

API的调用可使用“[YMVoiceService getInstance]”来直接操作，接口使用的基本流程为`初始化`->`收到初始化成功回调通知`->`加入语音频道`->`收到加入频道成功回调通知`->`使用其它接口`->`离开语音频道`->`反初始化`，要确保严格按照上述的顺序使用接口。

## 视频关键接口流程
1. 在收到`YOUME_EVENT_JOIN_OK`事件后

    开关麦克风：`[[YMVoiceService  getInstance] setMicrophoneMute:false];`
    开关扬声器：`[[YMVoiceService  getInstance] setSpeakerMute:false];`
    控制自己的摄像头打开：`[[YMVoiceService  getInstance] startCapture];`
    控制自己的摄像头关闭：`[[YMVoiceService  getInstance] stopCapture];`
    
2. 以上步骤完成，可以创建自己的视频流渲染View：
`[[YMVoiceService getInstance ] createRender:local_userid parentView:parentView];`
开启摄像头之后会渲染到返回的UIView里。
远端有视频流过来时，会通知 `YOUME_EVENT_OTHERS_VIDEO_ON` 事件，在该事件里创建视频渲染组件
`[[YMVoiceService getInstance ] createRender:other_userid parentView:parentView];`
远端视频会渲染在返回的UIView里，同时会在以下事件里收到对应的视频数据回调，不过可以不做处理：

```objectivec
// 软解
- (void)onVideoFrameCallback: (NSString*)userId data:(void*) data len:(int)len width:(int)width height:(int)height fmt:(int)fmt timestamp:(uint64_t)timestamp;
// 硬解
- (void)onVideoFrameCallbackForGLES:(NSString*)userId  pixelBuffer:(CVPixelBufferRef)pixelBuffer timestamp:(uint64_t)timestamp;
```
    
3. 其它API
    是否屏蔽他人视频： `[[YMVoiceService getInstance] maskVideoByUserId:(NSString*) userId mask:(bool) mask];`
    切换摄像头：`[[YMVoiceService getInstance] switchCamera];`
    删除渲染绑定：`[[YMVoiceService getInstance] deleteRender:userid];`



### 初始化

* **语法**
``` objectivec
(YouMeErrorCode_t)initSDK:(id<VoiceEngineCallback>)delegate  appkey:(NSString*)appKey  appSecret:(NSString*)appSecret
        regionId:(YOUME_RTC_SERVER_REGION_t)regionId
           serverRegionName:(NSString*) serverRegionName;
```

* **功能**
初始化语音引擎，做APP验证和资源初始化。

* **参数说明**
`delegate`：实现了回调函数的委托对象
`appKey`：从游密申请到的 app key, 这个你们应用程序的唯一标识。
`appSecret`：对应 appKey 的私钥, 这个需要妥善保存，不要暴露给其他人。
`regionId`：设置首选连接服务器的区域码，如果在初始化时不能确定区域，可以填RTC_DEFAULT_SERVER，后面确定时通过 setServerRegion 设置。如果YOUME_RTC_SERVER_REGION定义的区域码不能满足要求，可以把这个参数设为 RTC_EXT_SERVER，然后通过后面的参数serverRegionName 设置一个自定的区域值（如中国用 "cn" 或者 “ch"表示），然后把这个自定义的区域值同步给游密，我们将通过后台配置映射到最佳区域的服务器。
`serverRegionName`：自定义的扩展的服务器区域名。不能为null，可为空字符串“”。只有前一个参数serverRegionId设为RTC_EXT_SERVER时，此参数才有效（否则都将当空字符串“”处理）。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouNeErrorCode类型定义](#YouMeErrorCode类型定义)。

* **异步回调**
``` objectivec
// 涉及到的主要回调事件有：
// YOUME_EVENT_INIT_OK  - 表明初始化成功
// YOUME_EVENT_INIT_FAILED - 表明初始化失败，最常见的失败原因是网络错误或者 AppKey-AppSecret 错误
(void)onYouMeEvent:(YouMeEvent_t)eventType errcode:(YouMeErrorCode_t)iErrorCode roomid:(NSString *)roomid param:(NSString *)param;
```

### 加入语音频道（单频道）

* **语法**
``` objectivec
(YouMeErrorCode_t) joinChannelSingleMode:(NSString *)strUserID channelID:(NSString *)strChannelID userRole:(YouMeUserRole_t)userRole checkRoomExist:(bool)checkRoomExist;
```

* **功能**
加入语音频道（单频道模式，每个时刻只能在一个语音频道里面）。

* **参数说明**
`strUserID`：全局唯一的用户标识，全局指在当前应用程序的范围内。
`strChannelID`：全局唯一的频道标识，全局指在当前应用程序的范围内。
`userRole`：用户在语音频道里面的角色，见YouMeUserRole定义。
`bCheckRoomExist`：是否检查频道存在时才加入，默认为false: true 当频道存在时加入、不存在时返回错误（多用于观众角色），false 无论频道是否存在都加入频道。

* **返回值**
返回YOUME_SUCCESS才会有异步回调通知。其它返回值请参考[YouNeErrorCode类型定义](#YouMeErrorCode类型定义)。

* **异步回调**
``` objectivec
// 涉及到的主要回调事件有：
// YOUME_EVENT_JOIN_OK - 成功进入语音频道
// YOUME_EVENT_JOIN_FAILED - 进入语音频道失败，可能原因是网络或服务器有问题，或是bCheckRoomExist为true时频道还未创建
(void)onYouMeEvent:(YouMeEvent_t)eventType errcode:(YouMeErrorCode_t)iErrorCode roomid:(NSString *)roomid param:(NSString *)param;
```


### 渲染视频数据

  具体方法参见demo

#### 创建渲染
* **语法**

```objectivec
-(UIView*) createRender:(NSString*) userId parentView:(UIView*)parentView;

```

* **参数说明**
    `userId`: userId 用户ID
    `parentView`: 渲染父视图
    
* **返回值**
    @return 返回渲染视图OpenGLESView对象
    
* **回调**
    参见`视频数据回调`

### 开始摄像头采集

* **语法**

```objectivec
- (YouMeErrorCode_t)startCapture;
```

* **返回值**

    错误码，YOUME_SUCCESS - 表示成功,其他 - 具体错误码



