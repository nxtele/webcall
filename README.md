## start

### Web server settings
- To use NXCLOUD SDK, you must have your own web server and must use https to access.
- The web server should deploy the audio directory, containing 5 files:
filename|purpose
--|:--
hangup.wav|Hang-up prompt tone
ringin.wav|Incoming ringing tone
ringout.wav|Ringback tone for outgoing calls
connect.wav|Call connected tone
online.wav|Account online prompt tone

- You can specify the directory of the prompt through the parameter audioSrcPath of the profile
> Set the parameter playTone to define the prompt tone to be enabled during the SDK playback call <a href='#audiolist'>list</a>
> Set the parameter audioSrcPath to specify the path where the audio file is located

### SDK usage steps
1. Import nxwebrtc.js.
2. Define the profile, set nxuser, nxpass, create an object of type NxwCall nxwcall, and use nxwcall.myEvents to set the callback method.
3. Initiate a connection, register successfully, and enter the UA_READY state.
4. Make a call, connect a call, and hang up a call.
** On first run, the browser will pop up a warning that access to the microphone must be allowed. **

### Get WebCall account
When logging in to WebCall, you need to use the Webcall account, which is nxuser/nxpass in the example below. You can log in to the [NXCLOUD console](https://www.nxcloud.com/webCall/mobileList) to obtain and manage them.

### Example
#### 1. Introduce nxwebrtc.js
```html
<script src="your/path/nxwebrtc.js"></script>
````
````js
let NxwCall = NXW.default; //Type declaration of the object
let nxwcall = null; // the global instance of the object, not yet initialized
````

#### 2. Define the profile
````js
let profile = {
    nxuser: "xxxxxxx", nxpass: "xxxxxxx",
    logLevel: "error", playTone: 0xFF, nxtype: 6,
    audioElementId: "remoteAudio", playElementId: "playAudio"
  };
````
 - **nxuser and nxpass are NXCLOUD's assigned webcall accounts, not NXCLOUD's user accounts. **
 - audioElementId and playElementId are the ids of the audio component of the page
 - If you playTone, please refer to the tone <a href='#audiolist'>list</a>.
 

#### 3. Write the callback function
````js
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
````
The nxwebrtc SDK library encapsulates multiple <a href='#eventlist'>event notifications</a>, which can interact with business logic in the corresponding event callback functions.

#### 4. Initialize and start the object
Taking the defined profile as the parameter of the NxwCall constructor, the nxwcall object will be automatically created, and it will try to perform state transition automatically. First perform NXAPI authentication, then connect to the wss server, and then enter the UA_READY state after successful registration.
````js
function initApp() {
  if (nxwcall == null) {
    nxwcall = new NxwCall(profile)
    setupEvents(nxwcall);
  }
}
````

#### 5. Start testing
After the onRegistered callback is completed, the outgoing call can be executed and the incoming call can be processed.
Local microphone test:
````js
nxwcall.placeCall('9196')
````
Remote server test:
````js
nxwcall.placeCall('4444')
````
After completing the test, your phone channel is ready.

## the term
term | meaning | remarks
--|:--|:--
WebRTC|Web Real-Time Communication|Web-based voice real-time communication
WSS|WebSocket Secure|Webrtc requires wss to access the voice server, usually websocket over https

## Nxwebrtc Instructions for Use

<h2 id='audiolist'></h2>
### NxwAppConfig
Attribute|Type|Required|Description
--|:--|:--|:--
nxuser|string|M|
nxpass|string|M|
nxtype|number|M|NX voice call production environment is set to 6
audioElementId|string|M|The id of the HTML component that plays the audio of the other party
playElementId|string|M|The id of the audio component that plays the ringing, ringback, and hangup tone
logLevel|LogLevel|M|debug, warn, error
playTone|number|O|ALL:0xFF,RINGIN:0x01,RINGOUT:0x02,CONNECTED:0x04,HANGUP:0x08,ONLINE:0x10,CUSTOM:0x80. No special requirements, please set it to 0xFF.
audioSrcPath|string|O|Prompt sound wav file path, the default is audio
video|boolean|O| Whether to enable video, default false
videoLocalElementId|string|O|id of the video component of the local video
videoRemoteElementId|string|O|id of remote video video component
autoAnswer|boolean|O|Whether the incoming call is automatically connected, the default is false


### Status of Nxwebrtc
state|description|events that may trigger this state
--|:--|--
UA_INIT=0|Initial State|
UA_NXAPI|Query NXAPI, verify according to nxuser/nxpass|
UA_CONNECTING|Attempting to connect to NX's wss server|
UA_CONNECTED|connected to wss server successfully|onServerConnect
UA_REGISTERING|Attempting REGISTER registration
UA_READY|Registered successfully, ready to complete|onRegistered
UA_CALLING_OUT|Outgoing Call Status|onCallCreated
UA_INCOMING|Incoming Status|onCallReceived
UA_TALKING|On Status|onCallAnswered
UA_CALL_ENDING|Call ending|
UA_CALL_END|Call completely ended|onCallHangup
UA_DISCONNECTED|Disconnected from wss server|onServerDisconnect
UA_ERROR|SDK various abnormal events|error

<h2 id='eventlist'></h2>
### EventEmitter event notification
Nxwebrtc encapsulates the call-related events of the SIP bottom protocol stack, and uses the EventEmitter object to interact with the business, and the business layer can register callback functions.
Event|Parameter Description|Description
--|:--|:--
onCallCreated|RemoteNumber|Initial call created successfully, calling mode
onCallAnswered|RemoteNumber|Call connected, calling or called
onCallReceived|RemoteNumber| received a call request, called mode
onCallHangup|RemoteNumber| hang up the call, the calling or the called, the other party hangs up first
onServerConnect|-|Connect to NX's wss server
onServerDisconnect|-|disconnect|to NX's wss server
onRegistered|-|Registered successfully
onUnregistered|-| failed to register, or successfully unregistered,
onCallDTMFReceived|tone+":"+duration|DTMF button information and duration
onMessageReceived|message|MESSAGE The text of the message
error|Msg|A description of the abnormal event


### Properties of Nxwebrtc
It can be associated with the data of the business layer through the properties of the nxwebrtc object, and there is corresponding data in the CDR callback of the call.
parameter|read-write|description
--|:--|:--
myState|R| Read only the state of the state machine of the current NxwCall
myOrderId|RW|Set this orderId information before making a call
comingOrderId|RW|The already carried orderId information when calling in


## Main method
#### Make a call
````js
placeCall(target: string, hdrs?: Array<string>) // target: called number
let hdrs = new Array("X-NXRTC-Key: value", "X-NXRTC-Abc: 123"); // hdrs: optional additional sip header
nxwcall.placeCall(target,hdrs);
````

#### Connect the call
````js
answerCall(hdrs?: Array<string>) // The parameter hdrs is the sip header carried when 200 OK is sent to connect to the SIP call
````

#### Reject call
````js
declineCall() // For incoming calls, hang up directly.
````

#### hang up call
````js
hangupCall() //For an outgoing or incoming SIP call that has been connected, the local party will actively hang up.
````

### Other methods

#### Register and cancel account

The manual registration and cancellation of the account is generally not required for this manual operation. The SDK will automatically maintain the state machine and try its best to ensure the registered state.

````js

register() // Registration of WebCall account.

unregister() // Logout of the WebCall account.

````

#### disconnect wss connection

When creating NxwCall, it will try to automatically connect to the wss server. This function is used to actively disconnect the wss connection. If the account has been registered, the account will be cancelled first. Can be used to switch accounts or pre-close pages.

````js

disconnect()

````

#### Play beep

- It can play the system pre-defined prompt tones of ringing, ringback, connecting and hanging up, and can also play any music through the bound playElementId component.
- Action supports start and end, which means start and stop playback.
- type supports predefined types, including
  > ringin: ringing for incoming calls
  > ringout: outgoing ringback
  > connected: the call is connected
  > hangup: the call hangs up
  > online: the account is online.
  The SDK will play the above voice file according to profile.playTone when the call state is switched.
- type also supports other complete voice file names, as long as they exist under the audioSrcPath directory, usually in wav or mp3 format.
- Note: If the current page has no mouse and keyboard interaction, the background playback of the page will be automatically muted by the browser.



````js

Declaration: play(action: string, type?: string)

Example: nxwcall.play('start','ringout') //Play ringback tone

     nxwcall.play('end','ringout') //stop ringback tone

     nxwcall.play('start','my-music.wav') //Play custom sound

````

#### Send DTMF key message

When the call is connected, the DTMF information is sent, usually through the INFO message, and possibly through RTP in-band transmission.

````js

Declaration: sendDTMF(tonestr: string)

Example: nxwcall.sendDTMF('1'); // DTMF key 1 occurs

````

#### Turn off the microphone, mute locally

````js

muteCall(checked: boolean)

````

#### audio control sound, whether to play the other party's sound

````js

silentCall(checked: boolean)

````

#### Set the local playback volume

Set the local playback volume, the parameter volume represents the percentage of the maximum volume, the optional value is [0,1] or (1,100],

````js

Statement: setVolume(volume: number)

Example: nxwcall.setVolume(0.8); //set to 80% of the maximum volume

nxwcall.setVolume(80); //Set to 80% of the maximum volume

````

## SDK Compatibility Requirements
### Browser Requirements
#### PC browser (win/linux/macOS)
- Mobile browser (android/iOS)
- Chrome 56+
-Edge 79+
- Safari 16+
- Firefox 44+
- Opera 43+
- **Does not support IE browser**

#### Mobile browser (android/iOS)
- Chrome Android 105+
- Dafari iOS 11+
- UC Android 13.4+
- Android Browser 105+
- Firefox Android 104+
- QQ browser 13.1+
- Baidu Browser 13.18+
- KaiOS Browser 2.5+