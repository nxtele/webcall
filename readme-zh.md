## 开始

### Web服务器设置
- 使用NXCLOUD SDK，必须拥有自己的Web服务器，且必须使用https访问。
- Web服务器应该部署audio目录，包含5个文件：
文件名|用途
--|:--
hangup.wav|挂机提示音
ringin.wav|呼入振铃提示音
ringout.wav|呼出的回铃音
connect.wav|呼叫接通提示音
online.wav|账号在线提示音

- 可以通过 profile 的参数audioSrcPath指定提示音的目录
> 设置参数 playTone 定义SDK播放通话过程中开启提示音<a href='#audiolist'>列表</a>
> 设置参数 audioSrcPath 可以指定audio文件所在路径

### SDK使用步骤
1. 导入 nxwebrtc.js。
2. 定义profile，设置 nxuser,nxpass ，创建NxwCall 类型的对象 nxwcall，使用 nxwcall.myEvents 设置回调方法。 
3. 发起连接，注册成功，进入 UA_READY 状态。
4. 发起呼叫、接通呼叫、挂断呼叫。
**在首次运行时，浏览器会弹出警告，必须允许麦克风的访问权限。**

### 获取WebCall账号
WebCall在登录时，需要使用Webcall账号，也就是下面示例中的nxuser/nxpass，您可以登录[NXCLOUD控制台](https://www.nxcloud.com/webCall/mobileList)获取和管理它们。

### 示例
#### 1. 引入nxwebrtc.js 
```html
<script src="your/path/nxwebrtc.js"></script>
```
```js
let NxwCall = NXW.default;  //对象的类型声明
let nxwcall = null;         //对象的全局实例，尚未初始化
```

#### 2. 定义profile
```js
let profile = {
    nxuser: “xxxxxxxx”, nxpass:”xxxxxxx”,
    logLevel: "error", playTone: 0xFF, nxtype: 6,
    audioElementId: "remoteAudio", playElementId: "playAudio"
  };
```
 - **nxuser和nxpass是NXCLOUD的分配的Webcall账户，不是NXCLOUD的用户账户。**
 - audioElementId 和playElementId 是页面的audio组件的id
 - 如果playTone请参考提示音<a href='#audiolist'>列表</a>。
 

#### 3. 编写回调函数
```js
function setupEvents(nxwcall) {
    let e = nxwcall.myEvents;
    console.log("setupEvents e=", e)

    e.on("onCallCreated", function (desnationNumber) {
        console.log("================", "onCallCreated", desnationNumber)        
    });
    e.on("onRegistered", function (sipId) {
        console.log("================", "onRegistered", sipId)        
    });
    e.on("onCallReceived", function (callerNumber) {
        console.log("================", "onCallReceived", callerNumber)        
    });
    e.on("onCallAnswered", function () {
        console.log("================", "onCallAnswered")
    });
}
```
nxwebrtc SDK库封装了多个<a href='#eventlist'>事件通知</a>,可以在相应的事件回调函数中，和业务逻辑互动。

#### 4. 初始化并启动对象
把定义的profile作为NxwCall的构造函数的参数，会自动创建nxwcall对象，并且尝试自动执行状态转换，先执行NXAPI 认证，然后连接到wss服务器，然后注册成功后进入UA_READY状态。
```js
function initApp() {  
  if (nxwcall == null) {
    nxwcall = new NxwCall(profile)
    setupEvents(nxwcall);
  }
}
```

#### 5. 开始测试
在 onRegistered 回调完成之后，才能执行呼出、和处理呼入请求。
本地麦克风测试：
```js
nxwcall.placeCall('9196')
``` 
远程服务器测试：
```js
nxwcall.placeCall('4444')
``` 
完成测试后，代表你的电话通道已经就绪。

## 术语
术语|含义|备注
--|:--|:--
WebRTC|Web Real-Time Communication|基于网页的语音实时通信
WSS|WebSocket Secure|Webrtc要求必须是wss访问语音服务器，通常为websocket over https

## Nxwebrtc使用说明

<h2 id='audiolist'></h2>
### NxwAppConfig 
属性|类型|必选|说明
--|:--|:--|:--
nxuser|string|M|
nxpass|string|M|
nxtype|number|M|NX语音通话生产环境设置为6
audioElementId|string|M|播放对方声音的HTML组件id
playElementId|string|M|播放振铃、回铃、挂掉提示音的audio组件id
logLevel|LogLevel|M|debug:调试，warn:告警，error:错误
playTone|number|O|ALL:0xFF,RINGIN:0x01,RINGOUT:0x02,CONNECTED:0x04,HANGUP:0x08,ONLINE:0x10,CUSTOM:0x80。无特殊需求，请设置为0xFF。
audioSrcPath|string|O|提示音wav文件路径，默认为audio
video|boolean|O|是否启用video，默认false
videoLocalElementId|string|O|本地视频video组件的id
videoRemoteElementId|string|O|远方视频video组件的id
autoAnswer|boolean|O|是否自动接通来电,默认false


### Nxwebrtc的状态
状态|说明|可能触发此状态的事件
--|:--|--
UA_INIT=0|初始状态|
UA_NXAPI|查询NXAPI，根据nxuser/nxpass验证|
UA_CONNECTING|尝试连接NX的wss服务器|
UA_CONNECTED|连接wss服务器成功|onServerConnect
UA_REGISTERING|尝试REGISTER注册中
UA_READY|注册成功，准备完成|onRegistered
UA_CALLING_OUT|外呼状态|onCallCreated
UA_INCOMING|呼入状态|onCallReceived
UA_TALKING|接通状态|onCallAnswered
UA_CALL_ENDING|呼叫结束中|
UA_CALL_END|呼叫完全结束|onCallHangup
UA_DISCONNECTED|从wss服务器断开|onServerDisconnect
UA_ERROR|SDK各种异常事件|error

<h2 id='eventlist'></h2>
### EventEmitter事件通知
Nxwebrtc封装了SIP底层协议栈的呼叫相关的事件，使用EventEmitter对象和业务交互，业务层可以注册回调函数。
Event|参数说明|说明
--|:--|:--
onCallCreated|RemoteNumber|发起呼叫创建成功，主叫模式
onCallAnswered|RemoteNumber|呼叫接通，主叫或被叫
onCallReceived|RemoteNumber|接收到呼叫请求，被叫模式
onCallHangup|RemoteNumber|挂断呼叫，主叫或被叫，对方先挂断
onServerConnect|-|连接到NX的wss服务器
onServerDisconnect|-|断开连接|到NX的wss服务器
onRegistered|-|注册成功
onUnregistered|-|注册失败，或注销成功，
onCallDTMFReceived|tone+":"+duration|DTMF按键信息和持续时间
onMessageReceived|message|MESSAGE消息的文本
error|Msg|异常事件的描述


### Nxwebrtc的属性
可以通过nxwebrtc对象的属性，和业务层的数据关联，在呼叫的CDR回调中有对应的数据。
参数|读写|说明
--|:--|:--
myState|R|只读当前NxwCall的状态机的状态
myOrderId|RW|发起呼叫之前设置此orderId信息
comingOrderId|RW|在呼入时的已经携带的orderId信息


## 主要方法
#### 发起呼叫
```js
placeCall(target: string, hdrs?: Array<string>)  // target：被叫号码
let hdrs = new Array("X-NXRTC-Key: value", "X-NXRTC-Abc: 123"); // hdrs： 可选附加的sip header 
nxwcall.placeCall(target,hdrs);
```

#### 接通来电
```js
answerCall(hdrs?: Array<string>) // 参数 hdrs 为接通SIP呼入，发送 200 OK 时携带的sip header
```

#### 拒接来电
```js
declineCall() // 对来电呼入，直接挂断。
```

#### 挂断呼叫
```js
hangupCall()  //对已经接通的呼出或呼入的SIP呼叫，本地主动挂断。
```

### 其它方法

#### 注册和注销账号

账号的手工注册和注销，一般使用中不需要此手工操作，本SDK会自动维护状态机，尽力保证已注册状态。

```js

register() // WebCall账号的注册。

unregister() // WebCall账号的注销。

```

#### 断开wss连接

在创建NxwCall的时候，会尝试自动连接wss服务器，此函数用于主动断开wss连接，如果账号已经注册，会先注销账号。可用用于切换账号或预关闭页面。

```js

disconnect() 

```

#### 播放提示音

- 可播放振铃、回铃、接通、挂断的系统预定义提示音，也可通过绑定的playElementId组件播放任意音乐.
- action支持 start和end，代表启动播放和停止播放。
- type支持预定义类型，包括
  > ringin：来电振铃
  > ringout：呼出回铃
  > connected：呼叫接通
  > hangup：呼叫挂断
  > online：账户在线。
  SDK在呼叫状态切换时会根据 profile.playTone 播放上述语音文件。
- type也支持其他的完整语音文件名，只要在audioSrcPath目录下面存在，通常为wav或mp3格式。
- 注意：当前页面在未曾鼠标键盘交互的情况下，页面后台放音会被浏览器自动静音。



```js

声明：play(action: string, type?: string) 

示例：nxwcall.play('start','ringout') //播放回铃音

     nxwcall.play('end','ringout') //停止回铃音

     nxwcall.play('start','my-music.wav') //播放自定义的提示音

```

#### 发送DTMF按键信息

在呼叫接通的情况下，发送DTMF信息，一般通过INFO消息，也可能通过RTP带内传输。

```js

声明：sendDTMF(tonestr: string)  

示例：nxwcall.sendDTMF('1'); //发生DTMF按键1

```

#### 关闭麦克风,本地静音

```js

muteCall(checked: boolean)

```

#### audio控件音，是否播放对方的声音

```js

silentCall(checked: boolean)

```

#### 设置本地放音音量

设置本地放音音量，参数volume表示最大音量的百分比，可选值为 [0,1]或者(1,100]，

```js

声明：setVolume(volume: number) 

示例：nxwcall.setVolume(0.8); //设置为最大音量的80%

     nxwcall.setVolume(80); //设置为最大音量的80%

```

## SDK 兼容性要求
### 浏览器要求
####  PC浏览器（win/linux/macOS）
- 手机浏览器（android/iOS）
- Chrome 56+
- Edge 79+
- Safari 16+
- Firefox 44+
- Opera 43+
- **不支持IE浏览器**

#### 手机浏览器（android/iOS）
- Chrome Android 105+
- Dafari iOS 11+
- UC Android 13.4+
- Android Browser 105+
- Firefox Android 104+
- QQ browser 13.1+
- Baidu Browser 13.18+
- KaiOS Browser 2.5+
