## 简介
欢迎使用腾讯云游戏多媒体引擎 SDK 。为方便 Android 开发者调试和接入腾讯云游戏多媒体引擎产品 API，这里向您介绍适用于 Android 开发的接入技术文档。


## 使用流程图
![](https://main.qcloudimg.com/raw/bf2993148e4783caf331e6ffd5cec661.png)

### 使用 GME 重要事项

|重要接口     | 接口含义|
| ------------- |:-------------:|
|Init    		|初始化 GME 	|
|Poll    		|触发事件回调	|
|EnterRoom	 	|进房  		|
|EnableMic	 	|开麦克风 	|
|EnableSpeaker		|开扬声器 	|

**说明**
**GME 的接口调用成功后返回值为 QAVError.OK，数值为0。**
**GME 的接口调用要在同一个线程下。**
**GME 加入房间需要鉴权，请参考文档关于鉴权部分内容。**

**GME 需要调用 Poll 接口触发事件回调。**


## 初始化相关接口
未初始化前，SDK 处于未初始化阶段，需要初始化鉴权后，通过初始化 SDK，才可以进房。

|接口     | 接口含义   |
| ------------- |:-------------:|
|Init    	|初始化 GME	| 
|Poll    	|触发事件回调	|
|Pause   	|系统暂停	|
|Resume 	|系统恢复	|
|Uninit    	|反初始化 GME 	|


### 获取单例
在使用语音功能时，需要首先获取 ITMGContext 对象。
####  函数原型 

```
public static ITMGContext GetInstance(Context context)
```

|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| context    |Context |应用程序上下文对象|


####  示例代码  

```
import com.tencent.TMG.ITMGContext; 
TMGContext.getInstance(this);
```

### 注册回调
接口类采用 Delegate 方法用于向应用程序发送回调通知。将回调函数注册给 SDK，用于接受回调的信息。

####  示例代码 
```
private ITMGContext.ITMGDelegate itmgDelegate = null;
```
在构造函数中重写这个回调函数，对回调的参数进行处理。

|参数     | 类型         |意义|
| ------------- |:-------------:| ------------- |
| type    	|ITMGContext.ITMG_MAIN_EVENT_TYPE 	|回调的事件类型				|
| data    	|Intent 消息类型  						|回调的相关信息，事件数据	|



####  示例代码  
```
itmgDelegate= new ITMGContext.ITMGDelegate() {
            @Override
 			public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
                }
        };
```


将回调函数注册给 SDK，要在进房之前设置。
####  函数原型 
```
ITMGContext public int SetTMGDelegate(ITMGDelegate delegate)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| delegate    |ITMGDelegate |SDK 回调函数|
####  示例代码  
```
TMGContext.GetInstance(this).SetTMGDelegate(itmgDelegate);
```



### 初始化 SDK
参数获取见文档：[游戏多媒体引擎接入指引](https://cloud.tencent.com/document/product/607/10782)。
此接口需要来自腾讯云控制台的 SdkAppId 号码作为参数，再加上 openId，这个 openId 是唯一标识一个用户，规则由 App 开发者自行制定，App 内不重复即可（目前只支持 INT64）。
初始化 SDK 之后才可以进房。
####  函数原型

```
ITMGContext public int Init(String sdkAppId, String openID)
```

|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| sdkAppId    	|String  |来自腾讯云控制台的 SdkAppId 号码				|
| openID    		|String  |OpenID 只支持 Int64 类型（转为string传入），必须大于 10000，用于标识用户|

####  示例代码 


```
ITMGContext.GetInstance(this).Init(sdkAppId, openID);
```
### 触发事件回调
通过在 update 里面周期的调用 Poll 可以触发事件回调。
####  函数原型

```
ITMGContext int Poll()
```
####  示例代码
```
ITMGContext.GetInstance(this).Poll();
```

### 系统暂停
当系统发生 Pause 事件时，需要同时通知引擎进行 Pause。
####  函数原型

```
ITMGContext int Pause()
```

### 系统恢复
当系统发生 Resume 事件时，需要同时通知引擎进行 Resume。
####  函数原型

```
ITMGContext int Resume()
```



### 反初始化 SDK
反初始化 SDK，进入未初始化状态。
####  函数原型

```
ITMGContext int Uninit()
```
####  示例代码
```
ITMGContext.GetInstance(this).Uninit();
```

## 实时语音房间相关接口
初始化之后，SDK 调用进房后进去了房间，才可以进行实时语音通话。

|接口     | 接口含义   |
| ------------- |:-------------:|
|GenAuthBuffer    	|初始化鉴权|
|EnterRoom   		|加入房间|
|IsRoomEntered   	|是否已经进入房间|
|ExitRoom 		|退出房间|
|ChangeRoomType 	|修改用户房间音频类型|
|GetRoomType 		|获取用户房间音频类型|


### 实时语音鉴权信息
生成 AuthBuffer，用于相关功能的加密和鉴权，相关参数获取及详情见 [GME密钥文档](https://cloud.tencent.com/document/product/607/12218)。    
该接口返回值为 Byte[] 类型。离线语音获取鉴权时，房间号参数必须填0。

该接口返回值为 Byte[] 类型。
####  函数原型
```
AuthBuffer public native byte[] genAuthBuffer(int sdkAppId, int roomId, String identifier, String key)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| appId    		|int   		|来自腾讯云控制台的 SdkAppId 号码		|
| roomId    		|int   		|房间号，只支持32位				|
| openID    	|String 	|用户标识					|
| key    		|string 	|来自腾讯云控制台的密钥				|


####  示例代码  
```
import com.tencent.av.sig.AuthBuffer;//头文件
byte[] authBuffer=AuthBuffer.getInstance().genAuthBuffer(Integer.parseInt(sdkAppId), Integer.parseInt(strRoomID),identifier, key);
```



### 加入房间
用生成的鉴权信息进房，会收到消息为 ITMG_MAIN_EVENT_TYPE_ENTER_ROOM 的回调。加入房间默认不打开麦克风及扬声器。


####  函数原型
```
ITMGContext public abstract void  EnterRoom(int roomId, int roomType, byte[] authBuffer)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| roomId 	|int		|房间号，只支持32位	|
| roomType 	|int		|房间音频类型		|
| authBuffer	|byte[]	|鉴权码				|

|音频类型     	|含义|参数|音量类型|控制台推荐采样率设置|适用场景|
| ------------- |------------ | ---- |---- |---- |---- |
| ITMG_ROOM_TYPE_FLUENCY			|流畅音质	|1|扬声器：通话音量；耳机：媒体音量	|如对音质无特殊需求，16K采样率即可；					|流畅优先、超低延迟实时语音，应用在游戏内开黑场景，适用于 FPS、MOBA 等类型的游戏；	|							
| ITMG_ROOM_TYPE_STANDARD			|标准音质	|2|扬声器：通话音量；耳机：媒体音量	|根据对音质的需求，可以选择 16k/48k 采样率				|音质较好，延时适中，适用于狼人杀、棋牌等休闲游戏的实时通话场景；	|												
| ITMG_ROOM_TYPE_HIGHQUALITY		|高清音质	|3|扬声器：媒体音量；耳机：媒体音量	|为了保证最佳效果，建议控制台设置 48k 采样率的高音质配置	|超高音质，延时相对大一些，适用于音乐舞蹈类游戏以及语音社交类 APP；适用于播放音乐、线上K歌等有高音质要求的场景；	|

- 如对音量类型或场景有特殊需求，请联系一线客服反馈；
- 控制台采样率设置会直接影响游戏语音效果，请在 [控制台](https://console.cloud.tencent.com/gamegme) 上再次确认采样率设置是否符合项目使用场景。
####  示例代码  
```
ITMGContext.GetInstance(this).EnterRoom(Integer.parseInt(relationId),roomType, authBuffer);    
```

### 加入房间事件的回调
加入房间完成后会发送信息 ITMG_MAIN_EVENT_TYPE_ENTER_ROOM，在 OnEvent 函数中进行判断。

####  示例代码  
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if (ITMGContext.ITMG_MAIN_EVENT_TYPE.ITMG_MAIN_EVENT_TYPE_ENTER_ROOM == type)
        {
           	 //收到进房成功事件
        }
	}

```


### 判断是否已经进入房间
通过调用此接口可以判断是否已经进入房间，返回值为 bool 类型。
####  函数原型  
```
ITMGContext public boolean IsRoomEntered()
```
####  示例代码  
```
ITMGContext.GetInstance(this).IsRoomEntered();
```

### 退出房间
通过调用此接口可以退出所在房间。这是一个同步接口，调用返回时会释放所占用的设备资源。

#### 函数原型  
```
ITMGContext public void ExitRoom()
```
####  示例代码  
```
ITMGContext.GetInstance(this).ExitRoom();
```

### 退出房间回调
退出房间完成后会有回调，消息为 ITMG_MAIN_EVENT_TYPE_EXIT_ROOM。
####  示例代码  
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if (ITMGContext.ITMG_MAIN_EVENT_TYPE.ITMG_MAIN_EVENT_TYPE_EXIT_ROOM == type)
        {
            //收到退房成功事件
        }
}
```

### 修改用户房间音频类型
此接口用于修改用户房间音频类型，结果参见回调事件，事件类型为 ITMG_MAIN_EVENT_TYPE_CHANGE_ROOM_TYPE。
####  函数原型  
```
IITMGContext TMGRoom public void ChangeRoomType(int nRoomType)
```


|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| nRoomType    |int    |希望房间切换成的类型，房间音频类型参考 EnterRoom 接口|

####  示例代码  
```
ITMGContext.GetInstance(this).GetRoom().ChangeRoomType(nRoomType);
```


### 获取用户房间音频类型
此接口用于获取用户房间音频类型，返回值为房间音频类型，返回值为0时代表获取用户房间音频类型发生错误，房间音频类型参考 EnterRoom 接口。

####  函数原型  
```
IITMGContext TMGRoom public  int GetRoomType()
```

####  示例代码  
```
ITMGContext.GetInstance(this).GetRoom().GetRoomType();
```


### 房间类型完成回调
房间类型设置完成后，回调的事件消息为 ITMG_MAIN_EVENT_TYPE_CHANGE_ROOM_TYPE，返回的参数为 result、error_info 及 new_room_type，new_room_type 代表的信息如下，在 OnEvent 函数中对事件消息进行判断。

|事件子类型     | 代表参数   |含义|
| ------------- |:-------------:|-------------|
| ITMG_ROOM_CHANGE_EVENT_ENTERROOM		|1 	|表示在进房的过程中，自带的音频类型与房间不符合，被修改为所进入房间的音频类型	|
| ITMG_ROOM_CHANGE_EVENT_START			|2	|表示已经在房间内，音频类型开始切换（例如调用 ChangeRoomType 接口后切换音频类型 ）|
| ITMG_ROOM_CHANGE_EVENT_COMPLETE		|3	|表示已经在房间，音频类型切换完成|
| ITMG_ROOM_CHANGE_EVENT_REQUEST			|4	|表示房间成员调用 ChangeRoomType 接口，请求切换房间音频类型|	


####  示例代码  
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if (ITMGContext.ITMG_MAIN_EVENT_TYPE.ITMG_MAIN_EVENT_TYPE_CHANGE_ROOM_TYPE == type)
        {
		//房间类型改变后的处理
	 }
}
```

### 成员状态变化
该事件在状态变化才通知，状态不变化的情况下不通知。如需实时获取成员状态，请在上层收到通知时缓存，事件消息为 ITMG_MAIN_EVNET_TYPE_USER_UPDATE，参数 intent 包含两个信息，event_id 及 user_list，在 OnEvent 函数中对事件消息进行判断。
音频事件的通知有一个阈值，超过这个阈值才会发送通知。超过两秒没有收到音频包才通知“有成员停止发送音频包”消息。

|event_id     | 含义         |应用侧维护内容|
| ------------- |:-------------:|-------------|
|ITMG_EVENT_ID_USER_ENTER    				|有成员进入房间			|应用侧维护成员列表		|
|ITMG_EVENT_ID_USER_EXIT    				|有成员退出房间			|应用侧维护成员列表		|
|ITMG_EVENT_ID_USER_HAS_AUDIO    		|有成员发送音频包		|应用侧维护通话成员列表	|
|ITMG_EVENT_ID_USER_NO_AUDIO    			|有成员停止发送音频包	|应用侧维护通话成员列表	|

####  示例代码  
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if (ITMGContext.ITMG_MAIN_EVENT_TYPE.ITMG_MAIN_EVNET_TYPE_USER_UPDATE == type)
        {
		//更新成员状态
		int nEventID = data.getIntExtra("event_id", 0);
		String[] identifierList =data.getStringArrayExtra("user_list");
		 switch (nEventID)
 		    {
 		    case ITMG_EVENT_ID_USER_ENTER:
  			    //有成员进入房间
  			    break;
 		    case ITMG_EVENT_ID_USER_EXIT:
  			    //有成员退出房间
			    break;
		    case ITMG_EVENT_ID_USER_HAS_AUDIO:
			    //有成员发送音频包
			    break;
		    case ITMG_EVENT_ID_USER_NO_AUDIO:
			    //有成员停止发送音频包
			    break;
		    default:
			    break;
 		    }
	}
}
```

### 消息详情

|消息     | 消息代表的意义   
| ------------- |:-------------:|
|ITMG_MAIN_EVENT_TYPE_ENTER_ROOM    				       |进入音视频房间消息|
|ITMG_MAIN_EVENT_TYPE_EXIT_ROOM    				         	|退出音视频房间消息|
|ITMG_MAIN_EVENT_TYPE_ROOM_DISCONNECT    		       |房间因为网络等原因断开消息|
|ITMG_MAIN_EVENT_TYPE_CHANGE_ROOM_TYPE				|房间类型变化事件|

### 消息对应的Data详情
|消息     | Data         |例子|
| ------------- |:-------------:|------------- |
| ITMG_MAIN_EVENT_TYPE_ENTER_ROOM    				|result; error_info					|{"error_info":"","result":0}|
| ITMG_MAIN_EVENT_TYPE_EXIT_ROOM    				|result; error_info  					|{"error_info":"","result":0}|
| ITMG_MAIN_EVENT_TYPE_ROOM_DISCONNECT    		|result; error_info  					|{"error_info":"waiting timeout, please check your network","result":0}|
| ITMG_MAIN_EVENT_TYPE_CHANGE_ROOM_TYPE    		|result; error_info; new_room_type	|{"error_info":"","new_room_type":0,"result":0}|



## 实时语音音频接口
初始化 SDK 之后进房，在房间中，才可以调用实时音频语音相关接口。
调用场景举例：

当用户界面点击打开/关闭麦克风/扬声器按钮时，建议如下方式：
- 对于大部分的游戏类 App，推荐调用 EnableMic 及 EnbaleSpeaker 接口，相当于总是应该同时调用 EnableAudioCaptureDevice/EnableAudioSend 和 EnableAudioPlayDevice/EnableAudioRecv 接口；

- 其他类型的移动端 App 例如社交类型 App，打开或者关闭采集设备，会伴随整个设备（采集及播放）重启，如果此时 App 正在播放背景音乐，那么背景音乐的播放也会被中断。利用控制上下行的方式来实现开关麦克风效果，不会中断播放设备。具体调用方式为：在进房的时候调用 EnableAudioCaptureDevice(true) && EnabledAudioPlayDevice(true) 一次，点击开关麦克风时只调用 EnableAudioSend/Recv 来控制音频流是否发送/接收。

如目的是互斥（释放录音权限给其他模块使用），建议使用 PauseAudio/ResumeAudio。

|接口     | 接口含义   |
| ------------- |:-------------:|
|PauseAudio    				       	   |暂停音频引擎		|
|ResumeAudio    				      	 |恢复音频引擎		|
|EnableMic    						|开关麦克风|
|GetMicState    						|获取麦克风状态|
|EnableAudioCaptureDevice    		|开关采集设备		|
|IsAudioCaptureDeviceEnabled    	|获取采集设备状态	|
|EnableAudioSend    				|打开关闭音频上行	|
|IsAudioSendEnabled    				|获取音频上行状态	|
|GetMicLevel    						|获取实时麦克风音量	|
|SetMicVolume    					|设置麦克风音量		|
|GetMicVolume    					|获取麦克风音量		|
|EnableSpeaker    					|开关扬声器|
|GetSpeakerState    					|获取扬声器状态|
|EnableAudioPlayDevice    			|开关播放设备		|
|IsAudioPlayDeviceEnabled    		|获取播放设备状态	|
|EnableAudioRecv    					|打开关闭音频下行	|
|IsAudioRecvEnabled    				|获取音频下行状态	|
|GetSpeakerLevel    					|获取实时扬声器音量	|
|SetSpeakerVolume    				|设置扬声器音量		|
|GetSpeakerVolume    				|获取扬声器音量		|
|EnableLoopBack    					|开关耳返			|

### 暂停音频引擎的采集和播放
调用此接口暂停音频引擎的采集和播放，此接口为同步接口，且只在进房后有效。
如果想单独释放采集或者播放设备，请参考接口 EnableAudioCaptureDevice 及 EnableAudioPlayDevice。

####  函数原型  
```
ITMGContext ITMGAudioCtrl public int PauseAudio()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().PauseAudio();
```

### 恢复音频引擎的采集和播放
调用此接口恢复音频引擎的采集和播放，此接口为同步接口，且只在进房后有效。
#### 函数原型  
```
ITMGContext ITMGAudioCtrl public int ResumeAudio()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().ResumeAudio();
```

### 开启关闭麦克风
此接口用来开启关闭麦克风。加入房间默认不打开麦克风及扬声器。

####  函数原型  
```
ITMGContext public void EnableMic(boolean isEnabled)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| isEnabled    |boolean     |如果需要打开麦克风，则传入的参数为 true，如果关闭麦克风，则参数为 false|
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().EnableMic(true);
```

### 麦克风状态获取
此接口用于获取麦克风状态，返回值 0 为关闭麦克风状态，返回值 1 为打开麦克风状态，返回值 2 为麦克风设备正在操作中，返回值 3 为麦克风设备不存在，返回值 4 为设备没初始化好。
####  函数原型  
```
ITMGContext TMGAudioCtrl public int GetMicState() 
```
####  示例代码  
```
int micState = ITMGContext.GetInstance(this).GetAudioCtrl().GetMicState();
```

### 开启关闭采集设备
此接口用来开启/关闭采集设备。加入房间默认不打开设备。
- 只能在进房后调用此接口，退房会自动关闭设备。
- 在移动端，打开采集设备通常会伴随权限申请，音量类型调整等操作。

####  函数原型  
```
ITMGContext public int EnableAudioCaptureDevice(boolean isEnabled)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| isEnabled    |boolean     |如果需要打开采集设备，则传入的参数为 true，如果关闭采集设备，则参数为 false|

#### 示例代码

```
打开采集设备
ITMGContext.GetInstance(this).GetAudioCtrl().EnableAudioCaptureDevice(true);
```

### 采集设备状态获取
此接口用于采集设备状态获取。
#### 函数原型

```
ITMGContext public boolean IsAudioCaptureDeviceEnabled()
```
#### 示例代码

```
bool IsAudioCaptureDevice =ITMGContext.GetInstance(this).GetAudioCtrl().IsAudioCaptureDeviceEnabled();
```

### 打开关闭音频上行
此接口用于打开/关闭音频上行。如果采集设备已经打开，那么会发送采集到的音频数据。如果采集设备没有打开，那么仍旧无声。采集设备的打开关闭参见接口 EnableAudioCaptureDevice。

#### 函数原型

```
ITMGContext public int EnableAudioSend(boolean isEnabled)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| isEnabled    |boolean     |如果需要打开音频上行，则传入的参数为 true，如果关闭音频上行，则参数为 false|

#### 示例代码  

```
ITMGContext.GetInstance(this).GetAudioCtrl().EnableAudioSend(true);
```

### 音频上行状态获取
此接口用于音频上行状态获取。
#### 函数原型  
```
ITMGContext TMGAudioCtrl boolean IsAudioSendEnabled()
```
####  示例代码  
```
bool IsAudioSend =  =ITMGContext.GetInstance(this).GetAudioCtrl().IsAudioSendEnabled();
```

### 获取麦克风实时音量
此接口用于获取麦克风实时音量，返回值为 int 类型。
####  函数原型  
```
ITMGContext TMGAudioCtrl int GetMicLevel() 
```
####  示例代码  
```
int micLevel = ITMGContext.GetInstance(this).GetAudioCtrl().GetMicLevel();
```

### 设置麦克风的软件音量
此接口用于设置麦克风的软件音量。参数 volume 用于设置麦克风的软件音量，当数值为 0 的时候表示静音，当数值为 100 的时候表示音量不增不减，默认数值为 100。
####  函数原型  
```
ITMGContext TMGAudioCtrl int SetMicVolume(int volume) 
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| volume    |int      |设置音量，范围 0 到 200|
#### 示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().SetMicVolume(volume);
```
###  获取麦克风的软件音量
此接口用于获取麦克风的软件音量。返回值为一个 int 类型数值，返回值为101代表没调用过接口 SetMicVolume。
####  函数原型  
```
ITMGContext TMGAudioCtrl public int GetMicVolume()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().GetMicVolume();
```

### 开启关闭扬声器
此接口用于开启关闭扬声器。
####  函数原型  
```
ITMGContext public void EnableSpeaker(boolean isEnabled)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| isEnabled    |boolean       |如果需要关闭扬声器，则传入的参数为 false，如果打开扬声器，则参数为 true|
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().EnableSpeaker(true);
```

### 扬声器状态获取
此接口用于扬声器状态获取。返回值 0 为关闭扬声器状态，返回值 1 为打开扬声器状态，返回值 2 为扬声器设备正在操作中，返回值 3 为扬声器设备不存在，返回值 4 为设备没初始化好。
####  函数原型  
```
ITMGContext TMGAudioCtrl public int GetSpeakerState() 
```

####  示例代码  
```
int micState = ITMGContext.GetInstance(this).GetAudioCtrl().GetSpeakerState();
```



### 开启关闭播放设备
此接口用于开启关闭播放设备。

#### 函数原型  
```
ITMGContext public int EnableAudioPlayDevice(boolean isEnabled)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| isEnabled    |boolean        |如果需要关闭播放设备，则传入的参数为 false，如果打开播放设备，则参数为 true|
#### 示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().EnableAudioPlayDevice(true);
```

### 播放设备状态获取
此接口用于播放设备状态获取。
#### 函数原型

```
ITMGContext public int IsAudioPlayDeviceEnabled()
```
#### 示例代码  

```
bool IsAudioPlayDevice = ITMGContext.GetInstance(this).GetAudioCtrl().IsAudioPlayDeviceEnabled();
```

### 打开关闭音频下行
此接口用于打开/关闭音频下行。如果播放设备已经打开，那么会播放房间里其他人的音频数据。如果播放设备没有打开，那么仍旧无声。播放设备的打开关闭参见接口 参见EnableAudioPlayDevice。

#### 函数原型  

```
ITMGContext public int EnableAudioRecv(boolean isEnabled)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| isEnabled    |boolean     |如果需要打开音频下行，则传入的参数为 true，如果关闭音频下行，则参数为 false|

#### 示例代码  

```
ITMGContext.GetInstance(this).GetAudioCtrl().EnableAudioRecv(true);
```



### 音频下行状态获取
此接口用于音频下行状态获取。
#### 函数原型  
```
ITMGContext TMGAudioCtrl public boolean IsAudioRecvEnabled()
```

####  示例代码  
```
bool IsAudioRecv = ITMGContext.GetInstance(this).GetAudioCtrl().IsAudioRecvEnabled();
```

### 获取扬声器实时音量
此接口用于获取扬声器实时音量。返回值为 int 类型数值，表示扬声器实时音量。
####  函数原型  
```
ITMGContext TMGAudioCtrl public int GetSpeakerLevel() 
```

####  示例代码  
```
int SpeakLevel = ITMGContext.GetInstance(this).GetAudioCtrl().GetSpeakerLevel();
```

### 设置扬声器的软件音量
此接口用于设置扬声器的软件音量。
参数 volume 用于设置扬声器的软件音量，当数值为 0 的时候表示静音，当数值为 100 的时候表示音量不增不减，默认数值为 100。

####  函数原型  
```
ITMGContext TMGAudioCtrl public int SetSpeakerVolume(int volume) 
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| volume    |int        |设置音量，范围 0 到 200|
#### 示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().SetSpeakerVolume(volume);
```

### 获取扬声器的软件音量
此接口用于获取扬声器的软件音量。返回值为 int 类型数值，代表扬声器的软件音量，返回值为101代表没调用过接口 SetSpeakerVolume。
Level 是实时音量，Volume 是扬声器的软件音量，最终声音音量相当于 Level*Volume%。举个例子：实时音量是数值是 100 的话，此时Volume的数值是 60，那么最终发出来的声音数值也是 60。

####  函数原型  
```
ITMGContext TMGAudioCtrl public int GetSpeakerVolume()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().GetSpeakerVolume();
```


### 启动耳返
此接口用于启动耳返。
####  函数原型  
```
ITMGContext TMGAudioCtrl public int EnableLoopBack(boolean enable)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| enable    |boolean         |设置是否启动|
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioCtrl().EnableLoopBack(true);
```


## 实时语音伴奏相关接口
|接口     | 接口含义   |
| ------------- |:-------------:|
|StartAccompany    				       |开始播放伴奏|
|StopAccompany    				   	|停止播放伴奏|
|IsAccompanyPlayEnd				|伴奏是否播放完毕|
|PauseAccompany    					|暂停播放伴奏|
|ResumeAccompany					|重新播放伴奏|
|SetAccompanyVolume 				|设置伴奏音量|
|GetAccompanyVolume				|获取播放伴奏的音量|
|SetAccompanyFileCurrentPlayedTimeByMs 				|设置播放进度|

### 开始播放伴奏
调用此接口开始播放伴奏。支持 m4a、AAC、wav、mp3 一共四种格式。调用此 API，音量会重置。

####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int StartAccompany(String filePath, boolean loopBack, int loopCount) 
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| filePath    	|String    	|播放伴奏的路径					|
| loopBack  	|boolean    	|是否混音发送，一般都设置为 true，即其他人也能听到伴奏	|
| loopCount	|int    		|循环次数，数值为 -1 表示无限循环	|
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().StartAccompany(filePath,true,loopCount,duckerTimeMs);
```

### 播放伴奏的回调
开始播放伴奏完成后，回调函数调用 OnEvent，事件消息为 ITMG_MAIN_EVENT_TYPE_ACCOMPANY_FINISH，在 OnEvent 函数中对事件消息进行判断。
传递的参数 intent 包含两个信息，一个是 result，另一个是 file_path。
####  示例代码  
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if (ITMGContext.ITMG_MAIN_EVENT_TYPE.ITMG_MAIN_EVENT_TYPE_ACCOMPANY_FINISH == type)
        {
		//播放伴奏的事件回调
	}
}
```

### 停止播放伴奏
调用此接口停止播放伴奏。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int StopAccompany(int duckerTimeMs)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| duckerTimeMs    |int             |淡出时间|

####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().StopAccompany(duckerTimeMs);
```

### 伴奏是否播放完毕
如果播放完毕，返回值为 true，如果没播放完，返回值为 false。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public boolean IsAccompanyPlayEnd() 
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().IsAccompanyPlayEnd();
```


### 暂停播放伴奏
调用此接口暂停播放伴奏。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int PauseAccompany()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().PauseAccompany();
```


### 重新播放伴奏
此接口用于重新播放伴奏。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int ResumeAccompany()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().ResumeAccompany();
```



### 设置伴奏音量
设置 DB 音量，默认值为 100，数值大于 100 音量增益，数值小于 100 音量减益，值域为 0 到 200。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int SetAccompanyVolume(int vol)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| vol    |int             |音量数值|

####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().SetAccompanyVolume(Volume);
```

### 获取播放伴奏的音量
此接口用于获取 DB 音量。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int GetAccompanyVolume()
```
####  示例代码  
```
string currentVol = "VOL: " + ITMGContext.GetInstance(this).GetAudioEffectCtrl().GetAccompanyVolume();
```

### 获得伴奏播放进度
以下两个接口用于获得伴奏播放进度。需要注意：Current / Total = 当前循环次数，Current % Total = 当前循环播放位置。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public long GetAccompanyFileTotalTimeByMs()
ITMGContext TMGAudioEffectCtrl public long GetAccompanyFileCurrentPlayedTimeByMs()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().GetAccompanyFileTotalTimeByMs();
ITMGContext.GetInstance(this).GetAudioEffectCtrl().GetAccompanyFileCurrentPlayedTimeByMs();
```


### 设置播放进度
此接口用于设置播放进度。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int SetAccompanyFileCurrentPlayedTimeByMs(long time)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| time    |long                |播放进度，以毫秒为单位|

####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().SetAccompanyFileCurrentPlayedTimeByMs(time);
```


## 实时语音音效相关接口
|接口     | 接口含义   |
| ------------- |:-------------:|
|PlayEffect    		|播放音效|
|PauseEffect    	|暂停播放音效|
|PauseAllEffects	|暂停所有音效|
|ResumeEffect    	|重新播放音效|
|ResumeAllEffects	|重新播放所有音效|
|StopEffect 		|停止播放音效|
|StopAllEffects		|停止播放所有音效|
|SetVoiceType 		|变声特效|
|GetEffectsVolume	|获取播放音效的音量|
|SetEffectsVolume 	|设置播放音效的音量|


### 播放音效
此接口用于播放音效。参数中音效 id 需要 App 侧进行管理，唯一标识一个独立文件。文件支持 m4a、AAC、wav、mp3 一共四种格式。
#### 函数原型  
```
ITMGContext TMGAudioEffectCtrl public int PlayEffect(int soundId, String filePath, boolean loop) 
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| soundId    	|int    		|音效 id|
| filePath    	|String		|音效路径|
| loop    		|boolean	|是否重复播放|
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().PlayEffect(soundId,filePath,loop);
```


### 暂停播放音效
此接口用于暂停播放音效。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int PauseEffect(int soundId)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| soundId    |int                    |音效 id|

####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().PauseEffect(soundId);
```

### 暂停所有音效
调用此接口暂停所有音效。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int PauseAllEffects()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().PauseAllEffects();
```

### 重新播放音效
此接口用于重新播放音效。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int ResumeEffect(int soundId)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| soundId    |int                    |音效 id|
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().ResumeEffect(soundId);
```



### 重新播放所有音效
调用此接口重新播放所有音效。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int ResumeAllEffects()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().ResumeAllEffects();
```

### 停止播放音效
此接口用于停止播放音效。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int StopEffect(int soundId)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| soundId    |int                    |音效 id|
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().StopEffect(soundId);
```

### 停止播放所有音效
调用此接口停止播放所有音效。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int StopAllEffects()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().StopAllEffects();
```

### 变声特效
调用此接口设置变声特效。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl  public int setVoiceType(int type);
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| type    |int                    |表示本端音频变声类型|



|类型参数     |参数代表|意义|
| ------------- |-------------|------------- |
|ITMG_VOICE_TYPE_ORIGINAL_SOUND  		|0	|原声			|
|ITMG_VOICE_TYPE_LOLITA    				|1	|萝莉			|
|ITMG_VOICE_TYPE_UNCLE  				|2	|大叔			|
|ITMG_VOICE_TYPE_INTANGIBLE    			|3	|空灵			|
|ITMG_VOICE_TYPE_DEAD_FATBOY  			|4	|死肥仔			|
|ITMG_VOICE_TYPE_HEAVY_MENTA			|5	|重金属			|
|ITMG_VOICE_TYPE_DIALECT 				|6	|歪果仁			|
|ITMG_VOICE_TYPE_INFLUENZA 				|7	|感冒			|
|ITMG_VOICE_TYPE_CAGED_ANIMAL 			|8	|困兽			|
|ITMG_VOICE_TYPE_HEAVY_MACHINE			|9	|重机器			|
|ITMG_VOICE_TYPE_STRONG_CURRENT			|10	|强电流			|
|ITMG_VOICE_TYPE_KINDER_GARTEN			|11	|幼稚园			|
|ITMG_VOICE_TYPE_HUANG 					|12	|小黄人			|


####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().setVoiceType(0);
```



### 获取播放音效的音量
获取播放音效的音量，为线性音量，默认值为 100，数值大于 100 为增益效果，数值小于 100 为减益效果。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int GetEffectsVolume()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().GetEffectsVolume();
```


### 设置播放音效的音量
调用此接口设置播放音效的音量。
####  函数原型  
```
ITMGContext TMGAudioEffectCtrl public int SetEffectsVolume(int volume)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| volume    |int                    |音量数值|

####  示例代码  
```
ITMGContext.GetInstance(this).GetAudioEffectCtrl().SetEffectsVolume(Volume);
```






## 离线语音
使用离线语音及转文字功能需要先初始化 SDK。

|接口     | 接口含义   |
| ------------- |:-------------:|
|ApplyPTTAuthbuffer    |鉴权初始化	|
|SetMaxMessageLength    |限制最大语音信息时长	|
|StartRecording		|启动录音		|
|StopRecording    	|停止录音		|
|CancelRecording	|取消录音		|
|UploadRecordedFile 	|上传语音文件		|
|DownloadRecordedFile	|下载语音文件		|
|PlayRecordedFile 	|播放语音		|
|StopPlayFile		|停止播放语音		|
|GetFileSize 		|语音文件的大小		|
|GetVoiceFileDuration	|语音文件的时长		|
|SpeechToText 		|语音识别成文字		|

### 鉴权初始化
在初始化 SDK 之后调用鉴权初始化，authBuffer 的获取参见上文实时语音鉴权信息接口。
#### 函数原型  
```
ITMGContext TMGPTT public void ApplyPTTAuthbuffer(String authBuffer)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| authBuffer    |String                    |鉴权|

#### 示例代码  
```
ITMGContext.GetInstance(this).GetPTT().ApplyPTTAuthbuffer(authBuffer);
```

### 限制最大语音信息时长
限制最大语音消息的长度，最大支持 60 秒。
####  函数原型  
```
ITMGContext TMGPTT public void SetMaxMessageLength(int msTime)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| msTime     |  int           |语音时长|
####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().SetMaxMessageLength(msTime);
```


### 启动录音
此接口用于启动录音。
####  函数原型  
```
ITMGContext TMGPTT public void StartRecording(String fileDir)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| fileDir    |String                     |存放的语音路径|
####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().StartRecording(fileDir);
```

### 启动录音的回调
启动录音完成后的回调调用函数 OnEvent，事件消息为 ITMG_MAIN_EVNET_TYPE_PTT_RECORD_COMPLETE， 在 OnEvent 函数中对事件消息进行判断。
传递的参数包含两个信息，一个是 result，另一个是 file_path。

####  示例代码  
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if (ITMGContext.ITMG_MAIN_EVENT_TYPE.ITMG_MAIN_EVNET_TYPE_PTT_RECORD_COMPLETE == type)
        	{
            		//启动录音的回调
        	}
}
```

### 停止录音
此接口用于停止录音。
####  函数原型  
```
ITMGContext TMGPTT public int StopRecording()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().StopRecording();
```



### 取消录音
调用此接口取消录音。
####  函数原型  
```
ITMGContext TMGPTT public int CancelRecording()
```
####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().CancelRecording();
```

### 上传语音文件
此接口用于上传语音文件。
#### 函数原型  
```
ITMGContext TMGPTT public void UploadRecordedFile(String filePath)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| filePath    |String                      |上传的语音路径|
#### 示例代码  
```
ITMGContext.GetInstance(this).GetPTT().UploadRecordedFile(filePath);
```


### 上传语音完成的回调
上传语音完成后，事件消息为 ITMG_MAIN_EVNET_TYPE_PTT_UPLOAD_COMPLETE， 在 OnEvent 函数中对事件消息进行判断。
传递的参数包含三个信息，result，file_path 和 file_id。
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if(ITMGContext.ITMG_MAIN_EVENT_TYPE.ITMG_MAIN_EVNET_TYPE_PTT_UPLOAD_COMPLETE== type)
       	 {
           	//上传语音完成的回调
       	 }
}
```


### 下载语音文件
此接口用于下载语音文件。
#### 函数原型  
```
ITMGContext TMGPTT public void DownloadRecordedFile(String fileID, String downloadFilePath)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| fileID    			|String                      |文件的url路径	|
| downloadFilePath 	|String                      |文件的本地保存路径	|
####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().DownloadRecordedFile(url,path);
```


### 下载语音文件完成回调
下载语音完成后，事件消息为 ITMG_MAIN_EVNET_TYPE_PTT_DOWNLOAD_COMPLETE， 在 OnEvent 函数中对事件消息进行判断。
传递的参数包含三个信息，result、file_path 和 file_id。

```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if(ITMGContext.ITMG_MAIN_EVNET_TYPE_PTT_DOWNLOAD_COMPLETE== type)
        {
            //下载成功        
	}
}
```



### 播放语音
此接口用于播放语音。
####  函数原型  
```
ITMGContext TMGPTT public int PlayRecordedFile(String downloadFilePath) 
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| downloadFilePath    |String   |文件的路径|
####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().PlayRecordedFile(downloadFilePath);
```


### 播放语音的回调
播放语音的回调，事件消息为 ITMG_MAIN_EVNET_TYPE_PTT_PLAY_COMPLETE， 在 OnEvent 函数中对事件消息进行判断。
传递的参数包含两个信息，一个是 result，另一个是 file_path。
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if(ITMGContext.ITMG_MAIN_EVNET_TYPE_PTT_PLAY_COMPLETE== type)
       	 	{
			//播放语音的回调 
		}
}
```




### 停止播放语音
此接口用于停止播放语音。
####  函数原型  
```
ITMGContext TMGPTT public int StopPlayFile()
```

####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().StopPlayFile();
```



### 获取语音文件的大小
通过此接口，获取语音文件的大小。
####  函数原型  
```
ITMGContext TMGPTT public int GetFileSize(String filePath)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| filePath    |String                     |语音文件的路径|
####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().GetFileSize(path);
```

### 获取语音文件的时长
此接口用于获取语音文件的时长，单位毫秒。
####  函数原型  
```
ITMGContext TMGPTT public int GetVoiceFileDuration(String filePath)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| filePath    |String                     |语音文件的路径|
####  示例代码  
```
ITMGContext.GetInstance(this).GetPTT().GetVoiceFileDuration(path);
```


### 将指定的语音文件识别成文字
此接口用于将指定的语音文件识别成文字。

#### 函数原型  
```
ITMGContext TMGPTT public int SpeechToText(String fileID)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| fileID    |String                     |语音文件 url|
#### 示例代码  
```
ITMGContext.GetInstance(this).GetPTT().SpeechToText(fileID);
```

### 识别回调
将指定的语音文件识别成文字的回调，事件消息为 ITMG_MAIN_EVNET_TYPE_PTT_SPEECH2TEXT_COMPLETE， 在 OnEvent 函数中对事件消息进行判断。
传递的参数包含三个信息，result、file_path 和 text，其中 text 为识别的文本。
```
public void OnEvent(ITMGContext.ITMG_MAIN_EVENT_TYPE type, Intent data) {
	if(ITMGContext.ITMG_MAIN_EVNET_TYPE_PTT_SPEECH2TEXT_COMPLETE == type)
       	 {
            //成功识别语音文件       
	 }
}
```
## 高级 API

### 获取版本号
获取 SDK 版本号，用于分析。
#### 函数原型
```
ITMGContext public void GetSDKVersion()
```
#### 示例代码  
```
ITMGContext.GetInstance(this).GetSDKVersion();
```

### 设置打印日志等级
用于设置打印日志等级。
#### 函数原型
```
ITMGContext int SetLogLevel(int logLevel, bool enableWrite, bool enablePrint)
```



|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| logLevel    		|int   		|打印日志级别		|
| enableWrite    	|bool   				|是否写文件，默认为是	|
| enablePrint    	|bool   				|是否写控制台，默认为是	|




|ITMG_LOG_LEVEL|意义|
| -------------------------------|----------------------|
|TMG_LOG_LEVEL_NONE=0		|不打印日志			|
|TMG_LOG_LEVEL_ERROR=1		|打印错误日志（默认）	|
|TMG_LOG_LEVEL_INFO=2			|打印提示日志		|
|TMG_LOG_LEVEL_DEBUG=3		|打印开发调试日志	|
|TMG_LOG_LEVEL_VERBOSE=4		|打印高频日志		|
#### 示例代码  
```
ITMGContext.GetInstance(this).SetLogLevel(1,true,true);
```



### 设置打印日志路径
用于设置打印日志路径。默认路径为： /sdcard/Android/data/xxx.xxx.xxx/files。
#### 函数原型
```
ITMGContext int SetLogPath(String logDir)
```

|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| logDir    		|String   		|路径|

#### 示例代码  
```
ITMGContext.GetInstance(this).SetLogPath(path);
```


### 获取诊断信息
获取音视频通话的实时通话质量的相关信息。该接口主要用来查看实时通话质量、排查问题等，业务侧可以忽略。
#### 函数原型  
```
IITMGContext TMGRoom public String GetQualityTips() 
```
#### 示例代码  
```
ITMGContext.GetInstance(this).GetRoom().GetQualityTips();
```

### 加入音频数据黑名单
将某个 id 加入音频数据黑名单。返回值为 0 表示调用失败。
#### 函数原型  

```
ITMGContext ITMGAudioCtrl AddAudioBlackList(String openId)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| openId    |String      |需添加黑名单的 id|
#### 示例代码  

```
ITMGContext.GetInstance(this).GetAudioCtrl().AddAudioBlackList(openId);
```

### 移除音频数据黑名单
将某个 id 移除音频数据黑名单。返回值为 0 表示调用失败。
#### 函数原型  

```
ITMGContext ITMGAudioCtrl RemoveAudioBlackList(String openId)
```
|参数     | 类型         |意义|
| ------------- |:-------------:|-------------|
| openId    |String      |需移除黑名单的id|
#### 示例代码  

```
ITMGContext.GetInstance(this).GetAudioCtrl().RemoveAudioBlackList(openId);
```


## 回调消息

#### 消息列表：

|消息     | 消息代表的意义   
| ------------- |:-------------:|
|ITMG_MAIN_EVENT_TYPE_ENTER_ROOM    		|进入音频房间消息		|
|ITMG_MAIN_EVENT_TYPE_EXIT_ROOM    		|退出音频房间消息		|
|ITMG_MAIN_EVENT_TYPE_ROOM_DISCONNECT		|房间因为网络等原因断开消息	|
|ITMG_MAIN_EVENT_TYPE_CHANGE_ROOM_TYPE		|房间类型变化事件		|
|ITMG_MAIN_EVENT_TYPE_ACCOMPANY_FINISH		|伴奏结束消息			|
|ITMG_MAIN_EVNET_TYPE_USER_UPDATE		|房间成员更新消息		|
|ITMG_MAIN_EVNET_TYPE_PTT_RECORD_COMPLETE	|PTT 录音完成			|
|ITMG_MAIN_EVNET_TYPE_PTT_UPLOAD_COMPLETE	|上传 PTT 完成			|
|ITMG_MAIN_EVNET_TYPE_PTT_DOWNLOAD_COMPLETE	|下载 PTT 完成			|
|ITMG_MAIN_EVNET_TYPE_PTT_PLAY_COMPLETE		|播放 PTT 完成			|
|ITMG_MAIN_EVNET_TYPE_PTT_SPEECH2TEXT_COMPLETE	|语音转文字完成			|

#### Data 列表：

|消息     | Data         |例子|
| ------------- |:-------------:|------------- |
| ITMG_MAIN_EVENT_TYPE_ENTER_ROOM    		|result; error_info			|{"error_info":"","result":0}|
| ITMG_MAIN_EVENT_TYPE_EXIT_ROOM    		|result; error_info  			|{"error_info":"","result":0}|
| ITMG_MAIN_EVENT_TYPE_ROOM_DISCONNECT    	|result; error_info  			|{"error_info":"waiting timeout, please check your network","result":0}|
| ITMG_MAIN_EVENT_TYPE_CHANGE_ROOM_TYPE    	|result; error_info; new_room_type	|{"error_info":"","new_room_type":0,"result":0}|
| ITMG_MAIN_EVENT_TYPE_SPEAKER_NEW_DEVICE	|result; error_info  			|{"deviceID":"{0.0.0.00000000}.{a4f1e8be-49fa-43e2-b8cf-dd00542b47ae}","deviceName":"扬声器 (Realtek High Definition Audio)","error_info":"","isNewDevice":true,"isUsedDevice":false,"result":0}|
| ITMG_MAIN_EVENT_TYPE_SPEAKER_LOST_DEVICE    	|result; error_info  			|{"deviceID":"{0.0.0.00000000}.{a4f1e8be-49fa-43e2-b8cf-dd00542b47ae}","deviceName":"扬声器 (Realtek High Definition Audio)","error_info":"","isNewDevice":false,"isUsedDevice":false,"result":0}|
| ITMG_MAIN_EVENT_TYPE_MIC_NEW_DEVICE    	|result; error_info  			|{"deviceID":"{0.0.1.00000000}.{5fdf1a5b-f42d-4ab2-890a-7e454093f229}","deviceName":"麦克风 (Realtek High Definition Audio)","error_info":"","isNewDevice":true,"isUsedDevice":true,"result":0}|
| ITMG_MAIN_EVENT_TYPE_MIC_LOST_DEVICE    	|result; error_info 			|{"deviceID":"{0.0.1.00000000}.{5fdf1a5b-f42d-4ab2-890a-7e454093f229}","deviceName":"麦克风 (Realtek High Definition Audio)","error_info":"","isNewDevice":false,"isUsedDevice":true,"result":0}|
| ITMG_MAIN_EVNET_TYPE_USER_UPDATE    		|user_list;  event_id			|{"event_id":1,"user_list":["0"]}|
| ITMG_MAIN_EVNET_TYPE_PTT_RECORD_COMPLETE 	|result; file_path  			|{"filepath":"","result":0}|
| ITMG_MAIN_EVNET_TYPE_PTT_UPLOAD_COMPLETE 	|result; file_path;file_id  		|{"file_id":"","filepath":"","result":0}|
| ITMG_MAIN_EVNET_TYPE_PTT_DOWNLOAD_COMPLETE	|result; file_path;file_id  		|{"file_id":"","filepath":"","result":0}|
| ITMG_MAIN_EVNET_TYPE_PTT_PLAY_COMPLETE 	|result; file_path  			|{"filepath":"","result":0}|
| ITMG_MAIN_EVNET_TYPE_PTT_SPEECH2TEXT_COMPLETE	|result; file_path;file_id		|{"file_id":"","filepath":"","result":0}|
