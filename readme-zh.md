## 开始
### SDK使用流程
1. 导入 nxwebrtc.js。
2. 定义profile，设置 nxuser,nxpass ，创建NxwCall 类型的对象 nxwcall，获取e = nxwcall.myEvents 之后注册事件回调。 
3. 发起连接，注册成功，进入 UA_READY 状态。
4. 发起呼叫、接通呼叫、挂断呼叫。
**在首次运行时，浏览器会弹出警告，必须允许麦克风的访问权限。**

### Web服务器设置
- 运行该SDK的网站必须使用https访问
- Web服务器应该部署audio目录，包含hangup.wav，ringin.wav，ringout.wav三个文件，分别表示挂机提示音，呼入振铃提示音，呼出的回铃音
> 设置参数 playTone=false 可关闭提示音
> 设置参数 audioSrcPath 可以指定audio文件所在路径


### 示例
#### 1. 引入nxwebrtc.js 
```js
<script src="your/path/nxwebrtc.js"></script>

let NxwCall = NXW.default;  //对象的类型声明
let nxwcall = null;         //对象的全局实例，尚未初始化
```

#### 2. 定义profile
```js
let profile = {
    nxuser: “xxxxxxxx”, nxpass:”xxxxxxx”,
    logLevel: "debug", playTone: true, nxtype: 6,
    audioElementId: "remoteAudio", playElementId: "playAudio"
  };
```
 - nxuser和nxpass是NXCLOUD的分配的话机账户，不是NXCLOUD的用户账户。
 - audioElementId 和playElementId 是页面的audio组件的id
 - 如果playTone设置为true，需要保证在你的web服务器的audio目录下存在angup.wav，ringin.wav，ringout.wav三个文件。
 

#### 3. 编写回调函数
```js
function setupEvents(nxwcall) {
let e = nxwcall.myEvents;
    console.log("setupEvents e=", e)

    e.on("onCallCreated", function (param1) {
    console.log("================", "onCallCreated", param1)        
    });
    e.on("onCallAnswered", function (param1) {
    console.log("================", "onCallAnswered", param1)
    list.push({ ts: new Date(), key: "onCallAnswered", value: param1 })
    });
}
```
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

## 术语
术语|含义|备注
--|:--|:--
WebRTC|Web Real-Time Communication|基于网页的语音实时通信
WSS|WebSocket Secure|Webrtc要求必须是wss访问语音服务器，通常为websocket over https

## Nxwebrtc使用说明
### NxwAppConfig 
属性|类型|必选|说明
--|:--|:--|:--
nxuser|string|M
nxpass|string|M
nxtype|number|M|NX语音通话生产环境设置为6
audioElementId|string|M|播放对方声音的HTML组件id
playElementId|string|M|播放振铃、回铃、挂掉提示音的audio组件id
logLevel|LogLevel|M|log:调试||warn:告警|error:错误
playTone|boolean|O|是否播放tone，默认为true
audioSrcPath|string|O|Tone的wav文件路径，默认为./audio
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
OrderId|RW|发起呼叫之前设置此orderId信息
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
```js
register() // 话机账号的注册。
unregister() // 话机账号的注销。
```

#### 断开wss连接
```js
disconnect() // 在创建NxwCall的时候，会尝试自动连接wss服务器，此函数用于主动断开wss连接，如果账号已经注册，会先注销账号。
```
#### 发送DTMF按键信息
```js
sendDTMF(tonestr: string)  // 在呼叫接通的情况下，发送DTMF信息，一般通过INFO消息，也可能通过RTP带内传输。
```
#### 关闭麦克风,本地静音
```js
muteCall(checked: boolean)
```
#### audio控件静音，对方静音
```js
silentCall(checked: boolean)
```
#### 设置本地放音音量
```js
setVolume(volume: number) // volume可选值为 [0,1]或者(1,100]
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
