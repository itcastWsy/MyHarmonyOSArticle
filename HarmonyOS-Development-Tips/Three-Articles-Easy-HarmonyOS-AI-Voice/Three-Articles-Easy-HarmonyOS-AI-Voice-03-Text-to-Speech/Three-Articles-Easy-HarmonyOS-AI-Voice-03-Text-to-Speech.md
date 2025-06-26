# Mastering HarmonyOS AI Voice in Three Articles 03 - Text to Speech Synthesis

## Foreword

> Continued from the previous article **Mastering HarmonyOS AI Voice in Three Articles 02 - Audio File to Text Conversion**

The AI text-to-speech functionality provided by HarmonyOS NEXT can synthesize text of up to 10,000 characters into speech and play it back.

Example scenarios:

- When the phone is offline, system accessibility applications (screen readers) can integrate text-to-speech capability to provide playback functionality for visually impaired users.
- Similar to WeChat Reading, it can implement reading article content through voice, providing help when it's inconvenient to read articles, such as listening to books while delivering food.

## Implementation Effect

![image-20240829175251444](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B303-%E6%96%87%E6%9C%AC%E5%90%88%E6%88%90%E5%A3%B0%E9%9F%B3.assets/image-20240829175251444.png)

## Usage Process

1. Create text-to-speech engine
2. Set listener callbacks
3. Start synthesis

![image-20240829173019521](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B303-%E6%96%87%E6%9C%AC%E5%90%88%E6%88%90%E5%A3%B0%E9%9F%B3.assets/image-20240829173019521.png)

## Create Text-to-Speech Engine

> Complete wrapped code will be provided at the end of the article

Creating a text-to-speech engine requires first importing `textToSpeech`, then when calling its `createEngine` method, you need to prepare the initialization engine parameters.

![image-20240829173614612](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B303-%E6%96%87%E6%9C%AC%E5%90%88%E6%88%90%E5%A3%B0%E9%9F%B3.assets/image-20240829173614612.png)

## Set Listener Callbacks

After calling `createEngine`, a corresponding instance will be returned, at which point you can set listener callbacks.

1. onStart - Callback when playback starts
2. onStop - Callback when playback ends
3. onComplete - Callback after synthesis or playback completion, returns request ID and completion playback related information
4. onData - Callback during synthesis playback process, returns request ID, audio stream information, and audio additional information such as format, duration, etc. If you need to return audio stream information, please implement this interface.
5. onError - Callback when errors occur during synthesis playback process, returns request ID, error code, and error description.

![image-20240829174815936](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B303-%E6%96%87%E6%9C%AC%E5%90%88%E6%88%90%E5%A3%B0%E9%9F%B3.assets/image-20240829174815936.png)

## Start Synthesis

After completing the instance creation and setting listeners above, you can call the speak method to start synthesis. However, when calling speak, you also need to pass corresponding parameters.

![image-20240829175108921](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B303-%E6%96%87%E6%9C%AC%E5%90%88%E6%88%90%E5%A3%B0%E9%9F%B3.assets/image-20240829175108921.png)

## Wrapped Code

```typescript
import { textToSpeech } from "@kit.CoreSpeechKit";

class TextToSpeechManager {
  /** Text-to-speech engine */
  private ttsEngine: textToSpeech.TextToSpeechEngine | null = null;
  /** Configuration parameters for creating engine */
  private static extraParam: Record<string, Object> = {
    // Style interaction-broadcast: broadcast style
    style: "interaction-broadcast",
    // Region information. Optional, defaults to "CN" when not set, currently only supports "CN".
    locate: "CN",
    // Engine name. Optional, engine name, defaults to empty when not set, currently only supports single application, single instance
    name: "EngineName",
  };
  /** Configuration parameters for creating engine */
  private static initParamsInfo: textToSpeech.CreateEngineParams = {
    // Language, currently only supports "zh-CN" Chinese.
    language: "zh-CN",
    // Voice. 0 is Ling Xiaoshan female voice, currently only supports Ling Xiaoshan female voice.
    person: 0,
    // Mode. 0 is online, currently not supported; 1 is offline, currently only supports offline mode.
    online: 1,
    extraParams: TextToSpeechManager.extraParam,
  };
  /** Session ID, one instance can only be used once */
  private requestId: string;

  constructor() {
    this.requestId = `tts` + Date.now();
  }

  /** Create engine */
  async createEngine() {
    return (this.ttsEngine = await textToSpeech.createEngine(
      TextToSpeechManager.initParamsInfo
    ));
  }

  /** Set callback listener */
  async setListener(callback?: (res: textToSpeech.CompleteResponse) => void) {
    // Set speak callback information
    let speakListener: textToSpeech.SpeakListener = {
      // Start playback callback
      onStart(requestId: string, response: textToSpeech.StartResponse) {
        console.info(
          `onStart, requestId: ${requestId} response: ${JSON.stringify(
            response
          )}`
        );
      },
      // Synthesis completion and playback completion callback
      onComplete(requestId: string, response: textToSpeech.CompleteResponse) {
        console.info(
          `onComplete, requestId: ${requestId} response: ${JSON.stringify(
            response
          )}`
        );
        callback && callback(response);
      },
      // Stop playback callback
      onStop(requestId: string, response: textToSpeech.StopResponse) {
        console.info(
          `onStop, requestId: ${requestId} response: ${JSON.stringify(
            response
          )}`
        );
      },
      // Return audio stream
      onData(
        requestId: string,
        audio: ArrayBuffer,
        response: textToSpeech.SynthesisResponse
      ) {
        console.info(
          `onData, requestId: ${requestId} sequence: ${JSON.stringify(
            response
          )} audio: ${JSON.stringify(audio)}`
        );
      },
      // Error callback
      onError(requestId: string, errorCode: number, errorMessage: string) {
        console.error(
          `onError, requestId: ${requestId} errorCode: ${errorCode} errorMessage: ${errorMessage}`
        );
      },
    };
    // Set callback
    this.ttsEngine?.setListener(speakListener);
  }

  /** Start conversion */
  async speak(originalText: string) {
    // Set playback related parameters
    let extraParam: Record<string, Object> = {
      queueMode: 0,
      // Speech rate. Optional, supports range [0.5-2], defaults to 1 when not passed.
      speed: 1,
      // Volume. Optional, supports range [0-2], defaults to 1 when not passed
      volume: 2,
      // Pitch.
      // Optional, supports range [0.5-2], defaults to 1 when not passed
      pitch: 1,
      // Context, language used for playing Arabic numerals. Optional, currently only supports "zh-CN" Chinese, defaults to "zh-CN" when not passed.
      languageContext: "zh-CN",
      // Audio type, currently only supports "pcm"
      audioType: "pcm",
      // Channel. Optional, parameter range 0-16, integer type, can refer to audio stream usage to choose suitable audio scene. Defaults to 3 when not passed, voice assistant channel
      soundChannel: 3,
      // Synthesis type. Optional, defaults to 1 when not passed. 0: Only synthesis without playback, returns audio stream. 1: Synthesis and playback without returning audio stream.
      playType: 1,
    };
    let speakParams: textToSpeech.SpeakParams = {
      requestId: this.requestId, // requestId can only be used once within the same instance, please do not set repeatedly
      extraParams: extraParam,
    };
    // Call playback method
    this.ttsEngine?.speak(originalText, speakParams);
  }

  /** Stop conversion */
  async stop() {
    this.ttsEngine?.stop();
  }
}

export default TextToSpeechManager;
```

## Usage in Page

`Index.ets`

![image-20240829175251444](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B303-%E6%96%87%E6%9C%AC%E5%90%88%E6%88%90%E5%A3%B0%E9%9F%B3.assets/image-20240829175251444.png)

```typescript
import { PermissionManager } from '../utils/permissionMananger'
import { Permissions } from '@kit.AbilityKit'
import SpeechRecognizerManager from '../utils/SpeechRecognizerManager'
import { AudioCapturerManager } from '../utils/AudioCapturerManager'
import TextToSpeechManager from '../utils/TextToSpeechManager'

@Entry
@Component
struct Index {
  @State
  text: string = ""
  fileName: string = ""
  // 1. Request permissions
  fn1 = async () => {
    // Prepare the permissions to request - microphone permission
    const permissions: Permissions[] = ["ohos.permission.MICROPHONE"]
    // Check if we have permission
    const isPermission = await PermissionManager.checkPermission(permissions)
    if (!isPermission) {
      // If no permission, actively request it
      PermissionManager.requestPermission(permissions)
    }
  }
  // 2. Real-time speech recognition
  fn2 = () => {
    SpeechRecognizerManager.init(res => {
      console.log("Real-time speech recognition", JSON.stringify(res))
      this.text = res.result
    })
  }
  // 3. Start recording
  fn3 = () => {
    this.fileName = Date.now().toString()
    AudioCapturerManager.startRecord(this.fileName)
  }
  // 4. Stop recording
  fn4 = () => {
    AudioCapturerManager.stopRecord()
  }
  // 5. Audio file to text conversion
  fn5 = () => {
    SpeechRecognizerManager.init2(res => {
      this.text = res.result
      console.log("Audio file to text conversion", JSON.stringify(res))
    }, this.fileName)
  }
  // 6. Text to speech synthesis
  fn6 = async () => {
    const tts = new TextToSpeechManager()
    await tts.createEngine()
    tts.setListener((res) => {
      console.log("res", JSON.stringify(res))
    })
    tts.speak("I'll send you away, thousands of miles away")
  }

  build() {
    Column({ space: 10 }) {
      Text(this.text)

      Button("Request Permission")
        .onClick(this.fn1)
      Button("Real-time Speech Recognition")
        .onClick(this.fn2)

      Button("Start Recording")
        .onClick(this.fn3)
      Button("Stop Recording")
        .onClick(this.fn4)

      Button("Audio File to Text")
        .onClick(this.fn5)
      Button("Text to Speech")
        .onClick(this.fn6)

    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

## Summary

The AI text-to-speech functionality provided by HarmonyOS NEXT can synthesize text of up to 10,000 characters into speech and play it back.

The usage involves 3 steps:

1. Create text-to-speech engine
2. Set listener callbacks
3. Start synthesis
