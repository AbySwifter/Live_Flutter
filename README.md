# live_flutter_plugin
English | [简体中文](./README_zh-CN.md)

The solution allows anchors to compete with each other and co-anchor with viewers in real time, with a global end-to-end latency of below 300 ms on average, and supports 1080p resolution.

## Features
- Supports RTMP, HTTP-FLV, TRTC and WebRTC.
- View capturing, which allows you to capture the video images of the current livestream.
- Delay adjustment, which allows you to set the minimum time and maximum time for auto adjustment of the player cache.
- Customized video collection, which allows you to customize your audio and video data sources based on the project requirements.
- Beauty filters, filters, and stickers. The pusher contains multiple sets of beauty retouch algorithms (natural & smooth) and multiple color space filters (supporting custom filters).
- QoS traffic throttling technology and uplink network adaptation capability. The pusher can adjust the audio and video data volumes in real time based on the actual network conditions on the host side.

## plugin class files

```
.
├── manager
│   ├── tx_audio_effect_manager.dart  audio effect manager
│   ├── tx_beauty_manager.dart  beauty manager
│   ├── tx_device_manager.dart  device manager
│   └── tx_live_manager.dart  pusher/player create/destroy
├── v2_tx_live_code.dart  Definitions of error codes and warning codes of Tencent Cloud LVB.
├── v2_tx_live_def.dart  Key type definitions for Tencent Cloud LVB.
├── v2_tx_live_player.dart  Tencent Cloud live player-This player pulls audio and video data from the specified livestreaming URL and plays the data after decoding and local rendering.
├── v2_tx_live_player_observer.dart Tencent Cloud live player callback notification.
├── v2_tx_live_premier.dart  V2TXLive High-level interface
├── v2_tx_live_pusher.dart  Tencent Cloud live pusher-The live pusher encodes the local audio and video data and pushes the encoded data to a specified push URL.
├── v2_tx_live_pusher_observer.dart Live pusher callback notification.
└── widget
    └── v2_tx_live_video_widget.dart V2TXLiveVideoWidget

```

## Integrating the SDK
The SDK for Flutter has been published on Pub. You can have the SDK downloaded and updated automatically by configuring pubspec.yaml.

### 1. Add the following dependency to pubspec.yaml of your project.

```dart
dependencies:
  live_flutter_plugin: latest version number

```
### 2. Grant camera and mic permissions to enable the audio and video call features.

### iOS
#### 1. Add requests for camera and mic permissions in Info.plist:

```dart
<key>NSCameraUsageDescription</key>
<string>Video calls are possible only with camera permission.</string>
<key>NSMicrophoneUsageDescription</key>
<string>Audio calls are possible only with mic permission.</string>
```
#### 2. Add the field io.flutter.embedded_views_preview and set the value to Yes.

### Android

#### 1. Open /android/app/src/main/AndroidManifest.xml.
#### 2. Add xmlns:tools="http://schemas.android.com/tools" to “manifest”.
#### 3. Add tools:replace="android:label" to “application”.
>Note: Without the above steps, the “Android Manifest merge failed” error will occur and the compilation will fail.

![](https://main.qcloudimg.com/raw/7a37917112831488423c1744f370c883.png)

## Quick Start

### 1. Setup [License](./example/README.md)

```dart
import 'package:live_flutter_plugin/v2_tx_live_premier.dart';

 /// License Management View (https://console.cloud.tencent.com/live/license)
setupLicense() {
  // License URL of your application
  var LICENSEURL = "";
  // License key of your application
  var LICENSEURLKEY = "";
  V2TXLivePremier.setLicence(LICENSEURL, LICENSEURLKEY);
}

```

### 2. Setup V2TXLivePusher
>note: Please use a real device, Simulator are not supported.

#### 2.1 init V2TXLivePusher
```dart
import 'package:live_flutter_plugin/v2_tx_live_pusher.dart';

/// V2TXLivePusher Initialization
initPusher() {
  _livePusher = V2TXLivePusher(V2TXLiveMode.v2TXLiveModeRTC);
  /// setup Pusher listener
  _livePusher.addListener(onPusherObserver);
}

/// pusher observer
onPusherObserver(V2TXLivePusherListenerType type, param) {
  debugPrint("==pusher listener type= ${type.toString()}");
  debugPrint("==pusher listener param= $param");
}

```

#### 2.2 Setup Video RenderView

```dart
import 'package:live_flutter_plugin/widget/v2_tx_live_video_widget.dart';

// viewId: view ID generated by `V2TXLiveVideoWidget`
Widget renderView() {
  return V2TXLiveVideoWidget(
    onViewCreated: (viewId) async {
      _livePusher.setRenderViewID(_renderViewId);
    },
  );
}

```

#### 2.3 Start Push

```dart
startPush() async {
  // open camera
  await _livePusher.startCamera(true);
  // open microphone
  await _livePusher.startMicrophone();
  // generate RTMP/TRTC url
  var url = "";
  // start push
  await _livePusher.startPush(url);
}

```

#### 2.4 Stop Push
```dart

stopPush() async {
  // close camera
  await _livePusher.stopCamera();
  // close microphone
  await _livePusher.stopMicrophone();
  // stop push
  await _livePusher.stopPush();
}

```

### 3. Setup V2TXLivePlayer
>note: Please use other devices to test。

#### 3.1 Init V2TXLivePlayer

```dart
import 'package:live_flutter_plugin/v2_tx_live_player.dart';

initPlayer() {
   /// Create V2TXLivePlayer
  _livePlayer = V2TXLivePlayer();
  _livePlayer.addListener(onPlayerObserver);
}

/// Player observer
onPlayerObserver(V2TXLivePlayerListenerType type, param) {
  debugPrint("==player listener type= ${type.toString()}");
  debugPrint("==player listener param= $param");
}
```

#### 3.2 Setup Video RenderView

```dart
import 'package:live_flutter_plugin/widget/v2_tx_live_video_widget.dart';

// viewId: view ID generated by `V2TXLiveVideoWidget`
Widget renderView() {
  return V2TXLiveVideoWidget(
    onViewCreated: (viewId) async {
      // set video renderView
      _livePlayer.setRenderViewID(_renderViewId);
    },
  );
}

```

#### 3.3 Start Play

```dart
startPlay() async {
  // generate RTMP/TRTC/Led url
  var url = ""
  // start play
  await _livePlayer?.startPlay(url);
}
```

#### 3.4 Stop Play
```dart
/// stop play
stopPlay() {
  _livePlayer.stopPlay();
}
```


## Common problem

### How do I view logs?
live_flutter_plugin logs are compressed and encrypted by default with the XLOG extension. You can set setLogCompressEnabled to specify whether to encrypt logs. If a log filename contains C (compressed), the log is compressed and encrypted; if it contains R (raw), the log is in plaintext.
* iOS：Documents/log of the application sandbox
* Android
  * 6.7 or below: /sdcard/log/tencent/liteav
  * 6.8 or above: /sdcard/Android/data/package name/files/log/tencent/liteav/
 
### IOS cannot display video (Android is good)

Please confirm io.flutter.embedded_views_preview is `YES` in your info.plist

### Android Manifest merge failed

Please Open '/example/android/app/src/main/AndroidManifest.xml' file。

1.Add xmlns:tools="http://schemas.android.com/tools" to manifest

2.Add tools:replace="android:label" to application

![](https://main.qcloudimg.com/raw/7a37917112831488423c1744f370c883.png)
