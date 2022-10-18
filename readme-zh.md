# 开始

## 快速开始
### 初始化你的web服务器，必须使用https访问。
假设你的web服务器地址是： https://your.website.com/ 。 你的web服务器静态资源是一个空目录。

### 下载静态资源
进入你的web服务器根目录：
```shell
git clone https://github.com/nxtele/webcall
```

### 运行demo页面
在浏览器中打开： https://your.website.com/example/demo.html

**如果浏览器弹出'是否允许使用麦克风' 提示，选择允许**

### 获取WebCall账号
WebCall在登录时，需要使用Webcall账号，也就是下面示例中的nxuser/nxpass，您可以登录[NXCLOUD控制台](https://www.nxcloud.com/webCall/mobileList)获取和管理它们。

## SDK使用说明

### SDK使用步骤
1. 导入 nxwebrtc.js。
2. 定义profile，设置 nxuser,nxpass(WebCall账号),logLevel,playTone等属性 ，
3. new NxwCall(profile) 创建对象 nxwcall，并基于 nxwcall.myEvents 设置回调方法。 
4. nxwcall会自动启动状态机，在注册成功后，进入 UA_READY 状态，可呼入呼出。
5. 通常需要在回调方法中，针对收到的事件，执行相关的处理。可以调用api执行对应的功能：发起呼叫、接通呼叫、挂断呼叫。

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
 - **nxuser和nxpass是NXCLOUD的分配的Webcall账户，不是NXCLOUD的用户账户**
 - audioElementId与playElementId 是页面的audio组件的id
 - 如果需要自定义playTone请参考<a href='#audiolist'>列表</a>。
 

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

### 自定义playTone
在发起通话开始，直到通话结束，NXCLOUD一共定义了5种默认提示音，如果你完全不修改提示音的播放机制，设置playTone=0xFF。如果你不想播放任何提示音，设置playTone=0x00。 如果你想完全播放自定义提示音，设置playTone=0x80。
以下是控制播放提示音的几个值：它们是'按位与'的关系：

值|含义
--|:--
0xFF | ALL: 播放全部
0x01 | RINGIN: 允许SDK自动播放振铃提示音
0x02 | RINGOUT: 允许SDK自动播放回铃提示音
0x04 | CONNECTED: 允许SDK自动播放接通提示音
0x08 | HANGUP: 允许SDK自动播放挂断提示音
0x10 | ONLINE: 允许SDK自动播放上线提示音
0x80 | CUSTOM: 允许SDK播放你的自定义语音文件，格式为wav或者mp3，单声道，8kHZ或16kHZ

#### NXCLOUD SDK默认的5个提示音：
通过 profile 的参数audioSrcPath指定提示音的目录

<h2 id='audiolist'></h2>

文件名|用途
--|:--
ringin.wav|振铃提示音
ringout.wav|回铃提示音
connected.wav|接通提示音
hangup.wav|挂断提示音
online.wav|上线提示音


## Nxwebrtc使用说明

### NxwAppConfig 
属性|类型|必选|说明
--|:--|:--|:--
nxuser|string|M|
nxpass|string|M|
nxtype|number|M|NX语音通话生产环境设置为6
audioElementId|string|M|播放对方声音的HTML组件id
playElementId|string|M|播放振铃、回铃、挂掉提示音的audio组件id
logLevel|LogLevel|M|debug:调试，warn:告警，error:错误
playTone|number|O|提示音播放控制
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
```js

声明：play(action: string, filename?: string) 

示例：nxwcall.play('start','my-music.wav') //播放自定义的提示音

     nxwcall.play('end') //停止播放所有提示音
```
- action: start 或 end，启动播放和停止播放。
- filename 音频文件名。

**注意：当前页面在未曾鼠标键盘交互的情况下，页面后台放音会被浏览器自动静音**

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
