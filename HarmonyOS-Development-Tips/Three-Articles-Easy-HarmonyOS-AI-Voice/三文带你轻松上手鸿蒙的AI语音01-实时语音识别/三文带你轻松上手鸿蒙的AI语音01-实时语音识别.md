# Three Articles to Get You Started with HarmonyOS AI Voice 01 - Real-time Speech Recognition

## Preface

**HarmonyOSNext** integrates powerful AI functionality. Core Speech Kit (Basic Speech Service) is one of the many AI features it provides.

Core Speech Kit (Basic Speech Service) integrates basic AI capabilities for speech, including Text-to-Speech (TextToSpeech) and Speech Recognition (SpeechRecognizer) capabilities, facilitating user interaction with devices and enabling mutual conversion between real-time input speech and text.

Simply put, Core Speech Kit mainly provides two major speech AI functions:

1. **Speech Recognition**
2. **Text-to-Speech**

## Speech Recognition Introduction

**Speech recognition functionality** can convert audio information (short speech mode up to 60s, long speech mode up to 8h) into text.

Speech recognition can achieve:

1. **Real-time speech-to-text**
2. **Audio file to text conversion**

## **Real-time Speech-to-Text**

### Implementation Process

Let me first introduce the speech recognition process, the text-to-speech process later will be similar:

1. Request permissions
2. Create AI speech engine
3. Set listening callbacks
4. Start listening

> tips: Complete code is at the end of each function, can be read in conjunction with the encapsulated code

## Request Permissions

![image-20240828222453973](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/image-20240828222453973.png)

During the development of functionality, it's necessary to call the phone's microphone function. Therefore, permissions need to be actively requested.

![image-20240828222047223](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/image-20240828222047223.png)

Requesting permissions involves 3 steps:

1. Declare permissions
2. Check if permissions are granted
3. Request permissions

### Declare Permissions

1. Add the following configuration code _requestPermissions_ in `\entry\src\main\module.json5`

   ```json
   {
     "module": {
       ...
       "requestPermissions": [
         {
           "name": "ohos.permission.MICROPHONE",
           "reason": "$string:voice_reason",
           "usedScene": {
             "abilities": [
               "FormAbility"
             ],
             "when": "always"
           }
         }
       ],
     }
   }
   ```

2. Add the permission reason `voice_reason` in `\entry\src\main\resources\base\element\string.json`

   ```json
   {
     "string": [
       {
         "name": "module_desc",
         "value": "module description"
       },
       {
         "name": "EntryAbility_desc",
         "value": "description"
       },
       {
         "name": "EntryAbility_label",
         "value": "label"
       },
       {
         "name": "voice_reason",
         "value": "Used to access user's recording"
       }
     ]
   }
   ```

### Check Permissions

In actual development, before requesting permissions, we can first call the interface `checkAccessTokenSync` to check if permissions are already granted. If not, actively request permissions.

### Request Permissions

When we need to request permissions for certain functionality, we can achieve this by calling `requestPermissionsFromUser`

### Encapsulated Permission Code

Since permission code in **HarmonyOSNext** is not encapsulated and difficult to use, here's an encapsulated version.

Unencapsulated example code:

![image-20240828224125035](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/image-20240828224125035.png)

---

Encapsulated code

`entry\src\main\ets\utils\permissionMananger.ets`

```typescript
// Import necessary modules, including permission management related functionality
import {
  abilityAccessCtrl,
  bundleManager,
  common,
  Permissions,
} from "@kit.AbilityKit";

export class PermissionManager {
  // Static method to check if given permissions are already granted
  static checkPermission(permissions: Permissions[]): boolean {
    // Create an access token manager instance
    let atManager: abilityAccessCtrl.AtManager =
      abilityAccessCtrl.createAtManager();
    // Initialize tokenID to 0, will get real tokenID later
    let tokenID: number = 0;

    // Get bundle information of this application
    const bundleInfo = bundleManager.getBundleInfoForSelfSync(
      bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
    );

    // Set tokenID to application's access token ID
    tokenID = bundleInfo.appInfo.accessTokenId;

    // If no permissions are passed, return false indicating no permissions
    if (permissions.length === 0) {
      return false;
    } else {
      // Check if all requested permissions are granted
      return permissions.every(
        (permission) =>
          abilityAccessCtrl.GrantStatus.PERMISSION_GRANTED ===
          atManager.checkAccessTokenSync(tokenID, permission)
      );
    }
  }

  // Async static method to request user authorization for specified permissions
  static async requestPermission(permissions: Permissions[]): Promise<boolean> {
    // Create an access token manager instance
    let atManager: abilityAccessCtrl.AtManager =
      abilityAccessCtrl.createAtManager();

    // Get context (assuming getContext is a method that can get UI ability context)
    let context: Context = getContext() as common.UIAbilityContext;

    // Request user authorization for specified permissions
    const result = await atManager.requestPermissionsFromUser(
      context,
      permissions
    );

    // Check if request result is successful (each element in authResults array should be 0, indicating success)
    return (
      !!result.authResults.length &&
      result.authResults.every((authResults) => authResults === 0)
    );
  }
}
```

### Using Permission Code in Pages

`Index.ets`

```typescript
import { PermissionManager } from '../utils/permissionMananger'
import { Permissions } from '@kit.AbilityKit'

@Entry
@Component
struct Index {
  // 1 Request permissions
  fn1 = async () => {
    // Prepare needed permissions - microphone permission
    const permissions: Permissions[] = ["ohos.permission.MICROPHONE"]
    // Check if permissions are granted
    const isPermission = await PermissionManager.checkPermission(permissions)
    if (!isPermission) {
      // If no permissions, actively request them
      PermissionManager.requestPermission(permissions)
    }
  }

  build() {
    Column() {
      Button("Request Permissions")
        .onClick(this.fn1)
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

![image-20240828224918956](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/image-20240828224918956.png)

## Real-time Speech Recognition Steps

> The following mainly implements real-time speech recognition

### Create AI Speech Engine

Creating an AI speech engine mainly involves the following steps:

1. Declare AI speech engine configuration parameters, mainly including language, region information, etc.

   ![image-20240828225611828](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/image-20240828225611828.png)

2. Call the `createEngine` method to start creation and return an AI speech instance engine

### Set AI Speech Listening Callbacks

Before starting speech recognition, you need to first set speech recognition callbacks `setListener`. It mainly has the following categories:

1. Start recognition callback
2. Event callback
3. Recognition result callback
4. Recognition completion callback
5. Recognition error callback

![image-20240828225832286](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/image-20240828225832286.png)

### Start Listening to Real-time Speech

You need to first configure listening parameters, then you can call `startListening` to implement speech recognition.

Parameter configuration: The main configuration difference between real-time speech recognition and audio file recognition is in the `recognitionMode` field, **0 indicates real-time speech recognition**

![image-20240828230419298](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/image-20240828230419298.png)

### Encapsulated Speech Recognition Code

`\entry\src\main\ets\utils\SpeechRecognizerManager.ets`

```typescript
import { speechRecognizer } from "@kit.CoreSpeechKit";

class SpeechRecognizerManager {
  /**
   * Language information
   * Speech mode: long
   */
  private static extraParam: Record<string, Object> = {
    locate: "CN",
    recognizerMode: "long",
  };
  private static initParamsInfo: speechRecognizer.CreateEngineParams = {
    /**
     * Region information
     * */
    language: "zh-CN",
    /**
     * Offline mode: 1
     */
    online: 1,
    extraParams: this.extraParam,
  };
  /**
   * Engine
   */
  private static asrEngine: speechRecognizer.SpeechRecognitionEngine | null =
    null;
  /**
   * Recording result
   */
  static speechResult: speechRecognizer.SpeechRecognitionResult | null = null;
  /**
   * Session ID
   */
  private static sessionId: string = "asr" + Date.now();

  /**
   * Create engine
   */
  private static async createEngine() {
    // Set engine creation parameters
    SpeechRecognizerManager.asrEngine = await speechRecognizer.createEngine(
      SpeechRecognizerManager.initParamsInfo
    );
  }

  /**
   * Set callbacks
   */
  private static setListener(
    callback: (srr: speechRecognizer.SpeechRecognitionResult) => void = () => {}
  ) {
    // Create callback object
    let setListener: speechRecognizer.RecognitionListener = {
      // Start recognition success callback
      onStart(sessionId: string, eventMessage: string) {},
      // Event callback
      onEvent(sessionId: string, eventCode: number, eventMessage: string) {},
      // Recognition result callback, including intermediate and final results
      onResult(
        sessionId: string,
        result: speechRecognizer.SpeechRecognitionResult
      ) {
        SpeechRecognizerManager.speechResult = result;
        callback && callback(result);
      },
      // Recognition completion callback
      onComplete(sessionId: string, eventMessage: string) {},
      // Error callback, error codes are returned through this method
      // For example: returning error code 1002200006, recognition engine is busy, engine is recognizing
      // For more error codes, please refer to error code reference
      onError(sessionId: string, errorCode: number, errorMessage: string) {},
    };
    // Set callback
    SpeechRecognizerManager.asrEngine?.setListener(setListener);
  }

  /**
   * Start listening
   * */
  static startListening() {
    // Set parameters for starting recognition
    let recognizerParams: speechRecognizer.StartParams = {
      // Session id
      sessionId: SpeechRecognizerManager.sessionId,
      // Audio configuration information
      audioInfo: {
        // Audio type. Currently only supports "pcm"
        audioType: "pcm",
        // Audio sample rate. Currently only supports 16000 sample rate
        sampleRate: 16000,
        // Audio channel count information. Currently only supports channel 1
        soundChannel: 1,
        // Audio sample bit depth. Currently only supports 16 bits
        sampleBit: 16,
      },
      // Recording recognition
      extraParams: {
        // 0: Real-time recording recognition - automatically opens microphone to record real-time speech
        recognitionMode: 0,
        // Maximum supported audio duration
        maxAudioDuration: 60000,
      },
    };
    // Call start recognition method
    SpeechRecognizerManager.asrEngine?.startListening(recognizerParams);
  }

  /**
   * Cancel recognition
   */
  static cancel() {
    SpeechRecognizerManager.asrEngine?.cancel(
      SpeechRecognizerManager.sessionId
    );
  }

  /**
   * Release AI speech-to-text engine
   */
  static shutDown() {
    SpeechRecognizerManager.asrEngine?.shutdown();
  }

  /**
   * Stop and release resources
   */
  static async release() {
    SpeechRecognizerManager.cancel();
    SpeechRecognizerManager.shutDown();
  }

  /**
   * Initialize AI speech-to-text engine
   */
  static async init(
    callback: (srr: speechRecognizer.SpeechRecognitionResult) => void = () => {}
  ) {
    await SpeechRecognizerManager.createEngine();
    SpeechRecognizerManager.setListener(callback);
    SpeechRecognizerManager.startListening();
  }
}

export default SpeechRecognizerManager;
```

### Calling Speech Recognition Code in Pages

```typescript
import { PermissionManager } from '../utils/permissionMananger'
import { Permissions } from '@kit.AbilityKit'
import SpeechRecognizerManager from '../utils/SpeechRecognizerManager'

@Entry
@Component
struct Index {
  @State
  text: string = ""
  // 1 Request permissions
  fn1 = async () => {
    // Prepare needed permissions - microphone permission
    const permissions: Permissions[] = ["ohos.permission.MICROPHONE"]
    // Check if permissions are granted
    const isPermission = await PermissionManager.checkPermission(permissions)
    if (!isPermission) {
      // If no permissions, actively request them
      PermissionManager.requestPermission(permissions)
    }
  }
  // 2 Real-time speech recognition
  fn2 = () => {
    SpeechRecognizerManager.init(res => {
      console.log("Real-time speech recognition", JSON.stringify(res))
      this.text = res.result
    })
  }

  build() {
    Column({ space: 10 }) {
      Text(this.text)

      Button("Request Permissions")
        .onClick(this.fn1)
      Button("Real-time Speech Recognition")
        .onClick(this.fn2)
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

---

![PixPin_2024-08-28_23-19-49](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/PixPin_2024-08-28_23-19-49.gif)

---

### Speech Recognition Result Analysis

**Data format after successful speech recognition is as follows**

```
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":"是否给你承诺的太多"}
Real-time speech recognition {"isFinal":true,"isLast":false,"result":"是否给你承诺的太多？"}
Real-time speech recognition {"isFinal":false,"isLast":false,"result":""}
```

What needs attention:

1. Recognition function is continuously triggered, continuously triggered when sound is collected

2. isFinal indicates whether a sentence is finished

   ![image-20240828232352345](HarmonyNext%E4%B8%AD%E7%9A%84AI%E8%AF%AD%E9%9F%B3%E5%8A%9F%E8%83%BD%E4%BD%93%E9%AA%8C.assets/image-20240828232352345.png)

3. isLast indicates whether this speech recognition session is finished

## Summary

**HarmonyOSNext** integrates powerful AI functionality. Core Speech Kit (Basic Speech Service) is one of the many AI features it provides.

Core Speech Kit (Basic Speech Service) integrates basic AI capabilities for speech, including Text-to-Speech (TextToSpeech) and Speech Recognition (SpeechRecognizer) capabilities, facilitating user interaction with devices and enabling mutual conversion between real-time input speech and text.

Simply put, Core Speech Kit mainly provides two major speech AI functions:

1. **Speech Recognition**
2. **Text-to-Speech**

Speech recognition can achieve:

1. **Real-time speech-to-text**
2. **Audio file to text conversion**

This article mainly implements **real-time speech-to-text**. **Audio file to text conversion** will be explained in the next article.
