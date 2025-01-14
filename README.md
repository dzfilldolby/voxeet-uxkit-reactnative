# Voxeet UXKit React Native

## SDK License agreement

Before using the react-native plugin, please review and accept the [Dolby Software License Agreement](SDK_LICENSE.md).

## Getting started

Use the following commands to install and configure the UXKit on React Native:

```bash
npm install @voxeet/react-native-voxeet-conferencekit --save
npx react-native link @voxeet/react-native-voxeet-conferencekit
```

> Note: for iOS & Android, you must perform some mandatory modifications to your project.

## Mandatory modifications

### iOS (react-native >= 0.60)

1. Open your Xcode workspace from YOUR_PROJECT/ios/YOUR_PROJECT.xcworkspace

2. Go to your target settings -> 'Signing & Capabilities' -> '+ Capability' -> 'Background Modes'
    - Turn on 'Audio, AirPlay and Picture in Picture'  
    - Turn on 'Voice over IP'

    If you want to support CallKit (receiving incoming call when application is killed) with VoIP push notification, enable 'Push Notifications' (you will need to upload your [VoIP push certificate](https://developer.apple.com/account/ios/certificate/) to the [Dolby.io Dashboard](https://dolby.io/dashboard/)).

3. Privacy **permissions**, add two new keys in the Info.plist:
    - Privacy - Microphone Usage Description
    - Privacy - Camera Usage Description

4. Open a terminal and go to YOUR_PROJECT/ios
    ```bash
    pod install
    ```
    If you are using react-native 0.64, there is a known bug from the library FBReactNativeSpec. You have to go into your Pods project in Xcode workspace, select FBReactNativeSpec target, "Build Phases" section, drag and drop "[CP-User] Generate Specs" step just under "Dependencies" step (2nd position). You have to do this step after every pod install or pod update.

5. Open your .xcworkspace project, select Product > Scheme > Edit scheme > Build > Uncheck "Parallelize Build".

### Android

1. In `android/app/build.gradle`, add the maven repository and set the `minSdkVersion` to at least **21**.

2. Insert the following lines inside the dependencies block in `android/app/build.gradle`:
    **Warning: the SDK is only compatible with the Hermes engine**

    ```gradle
    project.ext.react = [
        enableHermes: true,  // clean and rebuild if changing
    ]
    ```

    A pickFirst option must be used for the libc++ shared object:

    ```gradle
    android {
        packagingOptions {
            pickFirst '**/armeabi-v7a/libc++_shared.so'
            pickFirst '**/x86/libc++_shared.so'
            pickFirst '**/arm64-v8a/libc++_shared.so'
            pickFirst '**/x86_64/libc++_shared.so'
        }
    }
    ```

3. Open up `android/app/src/main/java/[...]/MainActivity.java`
    
    If you are using `Expo` you can skip this step.
    
    If your `MainActivity` extends `ReactActivity`, change from `MainActivity extends ReactActivity` to `MainActivity extends RNVoxeetActivity`. With the following import: `import com.voxeet.specifics.RNVoxeetActivity`


## Usage

```javascript
import { VoxeetSDK } from "@voxeet/react-native-voxeet-conferencekit";
```

Note: the VoxeetEvents is now deprecated and will disappear from the library itself.

### initialization

```
VoxeetSDK.initialize(appKey, appSecret);
```

or 

```
VoxeetSDK.initializeToken(accessToken, () => {
    return new Promise((resolve, reject) => {
        ... //get the new accessToken
        resolve(theNewAccessToken);
    });
});
```

### Open a session (+ check for conference to join)


Once the SDK is initialized, try to connect your current user as soon as possible.

```
await VoxeetSDK.connect(new UserInfo("externalId", "name", "optAvatarUrl"));
```

Once the session has been started, if an incoming call has been accepted by the user, it will be initiated right away.

### Join and leave conferences

Use the corresponding method to perform the action :

```
const conference = await VoxeetSDK.create({ alias: "yourConferenceAlias" });
await VoxeetSDK.join(conference.conferenceId);
```

To leave, use the following

```
await VoxeetSDK.leave(conferenceId);
```

### Invite participants


## Events

You can subscribe to events via the `addListener` (and unsubscribe via the corresponding `removeListener`) method in `VoxeetSDK.events`

### Example

```
import { VoxeetSDK } from "@voxeet/react-native-voxeet-conferencekit";
import { ConferenceStatusUpdatedEvent } from "@voxeet/react-native-voxeet-conferencekit";

const onConferenceStatus = (event: ConferenceStatusUpdatedEvent) => {
  console.warn("event received", event);
}

VoxeetSDK.events.addListener("ConferenceStatusUpdatedEvent", onConferenceStatus);
```

### ConferenceStatusUpdatedEvent

- conferenceId: `string`
- conferenceAlias: `string|undefined`
- status: `ConferenceStatus`

## Configuration

Depending on your environment, you must configure your project according to the public documentation, [Voxeet UXKit iOS](https://github.com/voxeet/voxeet-uxkit-ios) and [Voxeet UXKit Android](https://github.com/voxeet/voxeet-uxkit-android).

## Local build

To build locally the TypeScript definition, run the following command:

```bash
npm run build-library
```

The typescript command line needs local dev resolutions (available in the `package.json`)

```bash
npm i -D @types/react ...
```
