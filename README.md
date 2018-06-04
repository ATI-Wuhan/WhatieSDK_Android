## WahtieSDK for Android updated at 2018-06-04

```
What's new:
All APIs for electrical outlets. SDKs for bulbs will be provided about June 6 or 7, 2018. 
Note: it's better to review all details on June 7. 
```

WhatieSDK is an SDK provided by ATI TECHNOLOGY (WUHAN) CO.,LTD. for the 3rd party accessing to our IOT cloud platform easily and quickly. Using this SDK, developers can do almost all funcation points on electrical outlets and RGBW bulbs, such as user registration/login, smart configration, add/share/unbind/delete devices, device control, timing countdown, timer, etc.


## Requirements
* IDE：Android Studio

## Installation

(1)Import WhatieSDK-x.x.x.aar:
```java
Import “.aar” file into “lib” of your project. 
```
(2)Add following code to “build.gralde” file in your project:
```java
compile(name:'WhatieSDK-x.x.x', ext:'aar')

repositories {
flatDir {
dirs 'libs'
}
}
```
(3)Import the required third-party plug-in:
```java
compile 'com.mylhyl:zxingscanner:2.0.0'   //encode and decode QRcode
compile 'com.lzy.net:okgo:3.0.4'          // HTTP connection
compile 'com.alibaba:fastjson:1.2.20'      //json2object
compile 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.1.0'     //about mqtt
compile 'org.eclipse.paho:org.eclipse.paho.android.service:1.1.1'   //about mqtt
compile 'org.greenrobot:eventbus:3.0.0'       // communations among threads
```

## Usage
### 1. Initialise
(1)Add the following code to the “onCreate()” method of your Application.class. Example:
```java
EHomeInterface.getINSTANCE().init(this)
```
(2)Add the following code to AndroidManifest.xml:
```java
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<uses-permission android:name="android.permission.CHANGE_WIFI_MULTICAST_STATE" /> 
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE" />
<uses-permission android:name="android.permission.WAKE_LOCK"/>
<uses-permission android:name="android.permission.CAMERA"/>
```
(3) Declare the following service of AndroidManifest.xml:
```java
<service android:name="org.eclipse.paho.android.service.MqttService"/>
<service android:name="com.d9lab.ati.whatiesdk.mqtt.MyMqttService"/>
<service android:name="com.d9lab.ati.whatiesdk.tcp.TcpService"/>
<service android:name="com.d9lab.ati.whatiesdk.udp.UdpService"/>
```
### 2. User Management

The SDK provides user management functions, such as user login, user logout, login password update. The only information you should give to SDK is: (1) the login email and (2) the encrypted password (Note: the SDK does not need the original password characters, it only needs ciphertext, like MD5 text). Here, the login email is the email used for logining into your APP, like Vivitar APP. The encrypted password is the ciphertext of the original password, generated in your APP.

Note: all other information on user management procedure is not needed for SDK.

#### 2.1 Login
Login with user email、password、accessId and accessKey.
```java
/**
* ensure single sign-on, or the background service will report a mistake
* @param mContext  The activity that uses this method
* @param email      user’s email
* @param password  user’s password
* @param accessId   company’s accessId
* @param accessKey  company’s accessKey
* @param callback    callback of network cummunicaiton
*/
EHomeInterface.getINSTANCE().login(mContext, email, password, accessId, accessKey,         new UserCallback() {             @Override             public void onSuccess(Response<BaseModelResponse<User>> response) {                 if (response.body().isSuccess()) {
/**                    * The following methods must be called when login success.                    */                     EHome.getInstance().setLogin(true);                     EHome.getInstance().setmUser(response.body().getValue());                     EHome.getInstance().setToken(response.body().getToken());                 } else {                     Toast.makeText(mContext, "Login fail.", Toast.LENGTH_SHORT).show();                 }             }             @Override             public void onError(Response<BaseModelResponse<User>> response) {                 super.onError(response);                  Toast.makeText(mContext, "Login fail.", Toast.LENGTH_SHORT).show();             }         });
```
#### 2.2 updateLoginPassword
Update login password with email、newPwd、oldPwd、accessId and accessKey.
```java
/**  *  * @param mContext    * @param email  * @param oldPwd  * @param newPwd  * @param accessId  * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().changePwd(mContext, email, oldPwd, newPwd, accessId, accessKey,         new BaseCallback() {             @Override             public void onSuccess(Response<BaseResponse> response) {             }             @Override             public void onError(Response<BaseResponse> response) {                 super.onError(response);              }         });

```
#### 2.3 Logout
```java
/**  *  * @param mContext    * @param accessId     * @param accessKey   * @param baseCallback   */
EHomeInterface.getINSTANCE().logOut(mContext, Constant.ACCESS_ID, Constant.ACCESS_KEY,
new BaseCallback() {     @Override     public void onSuccess(Response<BaseResponse> response) {
EHome.getInstance().logOut(); // This method must be called when logout success.      }
@Override    public void onError(Response<BaseResponse> response) {          super.onError(response);      } });
```

### 3. SmartConfig and Device Init

#### 3.1 Define inner class “WhatieAsyncTask” in activity that config network：
```java
private class WhatieAsyncTask extends AsyncTask<String, Void, List<IEsptouchResult>> {      @Override     protected void onPreExecute() {     }      @Override     protected List<IEsptouchResult> doInBackground(String... params) {         int taskResultCount = -1;         synchronized (mLock) {             String apSsid = mWifiAdmin.getWifiConnectedSsidAscii(params[0]);             String apBssid = params[1];             String apPassword = params[2];             String taskResultCountStr = params[3];             taskResultCount = Integer.parseInt(taskResultCountStr);             mEsptouchTask = new EsptouchTask(apSsid, apBssid, apPassword, mContext);         }         List<IEsptouchResult> resultList = mEsptouchTask.executeForResults(taskResultCount);         return resultList;     }      @Override     protected void onPostExecute(List<IEsptouchResult> result) {         IEsptouchResult firstResult = result.get(0);         if (!firstResult.isCancelled()) {             if (firstResult.isSuc()) {                  //TODO             } else {                  //TODO             }         }     } }
```
#### 3.2 After get smartconfig token：
```java
private EspWifiAdminSimple mWifiAdmin = new EspWifiAdminSimple(this);
String apSsid = mWifiAdmin.getWifiConnectedSsid();
String apBssid = mWifiAdmin.getWifiConnectedBssid();

/**  * Must be connected to 2.4 G Wi-Fi, 
* and router, mobile phone and device are close enough  * @param apSsid            * @param apBssid  * @param tokenAndPwd   token + router’s password
*/
 new WhatieAsyncTask().execute(apSsid, apBssid, tokenAndPwd, "1");
```
#### 3.3 Result of config device:
Bind Success:
Receiving the MqttBindSuccessEvent event.
Bind failed:
Receiving the MqttAlreadyBindEvent event means the device has been bound by others.

#### 3.4 Get smartconfig token
```java
/**  *  * @param mContext    * @param accessId  * @param accessKey  * @param baseStringCallback  */
EHomeInterface.getINSTANCE().getNetToken(mContext, accessId, accessKey, new BaseStringCallback() {     @Override     public void onSuccess(Response<BaseModelResponse<String>> response) {         if (response.body().isSuccess()){                      }     }     @Override     public void onError(Response<BaseModelResponse<String>> response) {         super.onError(response);      } });
```
#### 3.5 Initialize device
```java
/**  * After receiving the MqttBindSuccessEvent, this method must be called.  * @param mContext  * @param devId             device’s devId  * @param name             device’s name  * @param accessId  * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().getStarted(mContext,devId,name, accessId, accessKey,,         new BaseCallback() {             @Override             public void onSuccess(Response<BaseResponse> response) {                 if(response.body().isSuccess()){                 }             }              @Override             public void onError(Response<BaseResponse> response) {                 super.onError(response);             }         });
```

### 4. Device Controlling
#### 4.1 Get My Devices List

```java
/**  * After get device list, “saveDevices” method must be called.  * @param mContext    * @param accessId  * @param accessKey  * @param callback  */
EHomeInterface.getINSTANCE().getMyDevices(mContext, accessId, accessKey, new DevicesCallback() {     @Override     public void onSuccess(Response<BaseListResponse<DeviceVo>> response) {         if (response.body().isSuccess()){             EHome.getInstance().saveDevices(response.body().getList());         }     }      @Override     public void onError(Response<BaseListResponse<DeviceVo>> response) {         super.onError(response);      } });

```
#### 4.2 Operate Device


The device can only be controlled in the case of a MQTT or TCP connection.
(1)    Determine whether the MQTT is connected or not：
```java
EHome.getInstance().isMqttOn()
```
(2)    Determine whether there is an TCP connection with the device：
```java
EHome.getLinkedTcp().containsKey(devId);
```
Turn on the device
```java
/**  *  * @param devId  device’s devId  */
EHomeInterface.getINSTANCE().turnOn(devId);
```
Turn off the device
```java
/**  *  * @param devId  device’s devId  */
EHomeInterface.getINSTANCE().turnOff(devId);
```
Unbind the device
```java
/**  *  * @param devId  device’s devId  */
EHomeInterface.getINSTANCE().unbind(devId);
```
#### 4.3 Edit the device name

```java
/**  *  * @param mContext    * @param deviceId            device’s devId  * @param newName           device’s new name  * @param accessId  * @param accessKey  * @param devicesCallback  */
EHomeInterface.getINSTANCE().changeDeviceName(mContext, deviceId, newName, accessId, accessKey,         new BaseCallback() {             @Override             public void onSuccess(Response< BaseResponse > response) {                 if (response.body().isSuccess()) {                 }             }             @Override             public void onError(Response<BaseResponse> response) {                 super.onError(response);              }         });
```
#### 4.4 Unbind offline device

```java
/**  *  * @param mContext    * @param id                device’s id  * @param accessId  * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().deleteDevice(mContext, id, accessId, accessKey,     new BaseCallback() {     @Override     public void onSuccess(Response<BaseResponse> response) {         if (response.body().isSuccess()) {
EHome.getInstance().removeDevice(devId);  //this method must be called after delete success.         }     }     @Override     public void onError(Response<BaseResponse> response) {         super.onError(response);      } });
```

#### 4.5 Delete sharing device
```java
/**  *  * @param mContext    * @param devId                  device’s devId  * @param accessId  * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().deleteSharedDevice(mContext, devId, accessId, accessKey,
new BaseCallback() {     @Override     public void onSuccess(Response<BaseResponse> response) {         if (response.body().isSuccess()) {
EHome.getInstance().removeDevice(devId);  //this method must be called after delete success.         }     }     @Override     public void onError(Response<BaseResponse> response) {         super.onError(response);      } });
```

#### 4.6 Share device by QR code
The sharing device is coordinated by the device owner and the shared person. This scheme is to achieve the purpose of sharing devices by scanning the QR Code. First of all, the owner of the device spliced into a string by including information such as the owner id "adminId", device id "deviceId", timestamp "timestamp", etc. The string generates a QR Code for the shared user to scan the code to obtain the information and then pass it to the following method. After success, the sharedUser can further control the device.
```java
/**  *  * @param mContext    * @param adminId                     device owner’s id  * @param deviceId                     device’s id（not devId）  * @param timestamp                   time when generate the QRcode  * @param accessId      * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().addSharedDevice(mContext,adminId,deviceId,timestamp,accessId, accessKey     new BaseCallback() {     @Override     public void onSuccess(Response<BaseResponse> response) {         if(response.body().isSuccess()){         }     }      @Override     public void onError(Response<BaseResponse> response) {         super.onError(response);      } });

```

Note: QR code is a bit more complex for users, and it may be kicked out. Share device by email will be provided on June 6, 2018. By then, you can share device only by email. It is very simple.

#### 4.7 Countdown and timer
##### 4.7.1 Set countdown
```java
/**  *  * @param devId    device’s devId  * @param state     execution state of device  * @param duration   countdown time = hour*60*60+min*60  */
EHomeInterface.getINSTANCE().setCountdown(devId, state, duration);
```
##### 4.7.2 Cancel countdown
```java
/**  *  * @param devId   device’s devId  */
EHomeInterface.getINSTANCE().cancelCountdown(devId);
```
##### 4.7.3 Get device’s countdown
```java
/**  *                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           * @param mContext  * @param deviceId  //device’s id  * @param accessId  * @param accessKey  * @param clockCallback  */
EHomeInterface.getINSTANCE().getCountdown(mContext,deviceId, accessId, accessKey,         new ClockCallback() {             @Override             public void onSuccess(Response<BaseListResponse<ClockVo>> response) {             }             @Override             public void onError(Response<BaseListResponse<ClockVo>> response) {                 super.onError(response);              }         });
```

##### 4.7.4 Add timer
```java
/**  *   * @param mContext  * @param deviceId  * @param timerType //A sevent-bit binary string. From the lowest bit to highest bit, indicating Monday, Tuesday, Wednesday, Thursday, Friday, Saturday, Sunday respectively. “1” means this timer will execute, and “0” means not. For example, “1100000” means timer will execute on Sunday and Saturday, “0000000” means timer will execute only once without repeat. 
* @param hour  //from “00” to “23”  * @param min   //from “00” to “59”  * @param dps   //execution state of device  * @param accessId  * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().addTimer(mContext, deviceId, timetype, finishHour, finishMin, deviceState, accessId, accessKey,         new BaseCallback() {             @Override             public void onSuccess(Response<BaseResponse> response) {                             }              @Override             public void onError(Response<BaseResponse> response) {                 super.onError(response);             }         });
```
##### 4.7.5 Delete timer
```java
/**  *  * @param mContext  * @param clockId  * @param accessId  * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().deleteTimer(mContext, clockId, accessId, accessKey,         new BaseCallback() {             @Override             public void onSuccess(Response<BaseResponse> response) {             }             @Override             public void onError(Response<BaseResponse> response) {                 super.onError(response);             }         });
```
##### 4.7.6 Change timer’s information
```java
/**  *  * @param mContext  * @param clockId  //timer’s id  * @param timerType  * @param hour  * @param min  * @param dps  * @param accessId  * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().editTimer(mContext, clockId, timetype, finishHour, finishMin, deviceState, accessId, accessKey,         new BaseCallback() {             @Override             public void onSuccess(Response<BaseResponse> response) {             }             @Override             public void onError(Response<BaseResponse> response) {                 super.onError(response);             }         });
```

##### 4.7.7 Open/close timer
```java
/**  *  * @param mContext  * @param clockId  * @param state  //state of timer  * @param accessId  * @param accessKey  * @param baseCallback  */
EHomeInterface.getINSTANCE().editTimerStatus(mContext, clockId, state, accessId, accessKey,         new BaseCallback() {             @Override             public void onSuccess(Response<BaseResponse> response) {             }             @Override             public void onError(Response<BaseResponse> response) {                 super.onError(response);             }         });
```
### 5 Subscription event
```java
All event means can be found in the corresponding class file. Please check the use of Eventbus at first.
Turn on event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttReceiveOnEvent event) {}
Turn off event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttReceiveOffEvent event) {}
Unbind event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttReceiveUnbindEvent event) {}
Offline event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttReceiveStatusEvent event) {}
Bind success event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttBindSuccessEvent event) {}
Already bind event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttAlreadyBindEvent event) {}
Shared device turn on event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttReceiveSharedOnEvent event) {}
Shared device turn off event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttReceiveSharedOffEvent event) {}
Shared device offline event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttReceiveSharedStatusEvent event) {}
Set countdown success event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttSetCdSuccessEvent event) {}
Cancel countdown success event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttCancelCdSuccessEvent event) {}
Add/open timer success event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttSetTimerSuccessEvent event) {}
Delete/close timer success event
@Subscribe(threadMode = ThreadMode.MAIN, priority = 1, sticky = true) public void onEventMainThread(MqttCancelTimerSuccessEvent event) {}
```

### Attention
1、When activity enters onPause state，please call the following method:
EHome.getInstance().cancelTag(this);
2、When turn on/ off a device, please determine whether this device has countdown. If so, cancel countdown at same time.
