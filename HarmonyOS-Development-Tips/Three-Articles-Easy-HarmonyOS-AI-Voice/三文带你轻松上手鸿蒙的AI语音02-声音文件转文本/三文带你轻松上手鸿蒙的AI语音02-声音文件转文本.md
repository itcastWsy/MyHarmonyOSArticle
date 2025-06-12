# English Translation Required

This file is marked for translation from: 02-声音文件转文本.md

Original Chinese file path: 鸿蒙开发技巧\三文带你轻松上手鸿蒙的 AI 语音\三文带你轻松上手鸿蒙的 AI 语音 02-声音文件转文本\02-声音文件转文本.md

Please translate the content from the original Chinese file to English.
The translation should maintain:

- Technical accuracy
- Code examples (translate comments but keep code structure)
- Image references
- Link references
- Formatting (headers, lists, etc.)

---

# Mastering HarmonyOS AI Voice in Three Articles 02 - Audio File to Text Conversion

**Continued from the previous article**

## Foreword

This article primarily demonstrates how to use HarmonyOS AI voice functionality to recognize and convert audio files into text.

## Implementation Process

1. Use `AudioCapturer` to record audio and generate audio files
2. Use AI voice functionality for recognition

![image-20240829002516961](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B302-%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E8%BD%AC%E6%96%87%E6%9C%AC.assets/image-20240829002516961.png)

## Introduction to Two Audio Recording Libraries

In **HarmonyOS NEXT** application development, there are two core libraries for implementing audio recording:

1. AudioCapturer
2. AVRecorder

AVRecorder can only produce audio files in AAC format, which is not supported by our AI voice engine. The AI voice engine only supports PCM format, while AudioCapturer records audio in PCM format. Therefore, we choose AudioCapturer for audio recording.

# AudioCapturer

## AudioCapturer Introduction

AudioCapturer is an audio capturer used for recording PCM (Pulse Code Modulation) audio data, suitable for developers with audio development experience to implement more flexible recording functionality.

State Change Diagram

![img](https://alliance-communityfile-drcn.dbankcdn.com/FileServer/getFile/cmtyPub/011/111/111/0000000000011111111.20240823151419.32894604214630336127965931722903:50001231000000:2800:44354A28E476D96FF43C8924431552C1A70AF66AEA065977DE5567CE12956ECC.png?needInitFileName=true?needInitFileName=true)

The main workflow for using AudioCapturer is:

1. Create AudioCapturer instance
2. Call start method to begin recording
3. Call stop method to stop recording
4. Call release method to release the instance

## Creating AudioCapturer Instance

> Complete, ready-to-use wrapped code will be provided at the end of the article. The following code examples are based on this wrapped code.

We create an AudioCapturer instance by calling the createAudioCapturer method, which requires passing relevant parameters.

![image-20240829003846034](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B302-%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E8%BD%AC%E6%96%87%E6%9C%AC.assets/image-20240829003846034.png)

## Calling start Method to Begin Recording

When calling the start method, you need to prepare relevant data such as:

1. Provide a recording filename (customizable)
2. Callback function for writing recording data (continuously triggered during recording)
3. Call the start method

![image-20240829004425443](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B302-%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E8%BD%AC%E6%96%87%E6%9C%AC.assets/image-20240829004425443.png)

## Calling stop Method to Stop Recording

Calling the stop method is relatively simple - just call it directly.

![image-20240829004829409](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B302-%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E8%BD%AC%E6%96%87%E6%9C%AC.assets/image-20240829004829409.png)

## Calling release Method to Release Instance

Similarly straightforward

![image-20240829004910026](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B302-%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E8%BD%AC%E6%96%87%E6%9C%AC.assets/image-20240829004910026.png)

## Wrapped Audio Recording Code

`\entry\src\main\ets\utils\AudioCapturerManager.ets` Here's a summary of this class's properties and methods:

### Properties

- **static audioCapturer**:
  - Type is `audio.AudioCapturer | null`, a static property used to store the current audio capturer instance.
- **private static recordFilePath**:
  - Type is `string`, a static private property used to store the recording file path.

### Methods

- **static async createAudioCapturer()**:
  - If `audioCapturer` already exists, returns that instance directly; otherwise creates a new audio capturer instance, sets its audio stream info and audio capture info, then creates and returns the new instance.
- **static async startRecord(fileName: string)**:
  - Async static method to start the recording process. First calls `createAudioCapturer()` method to ensure there's an audio capturer instance. Then initializes buffer size and opens or creates a `.wav` recording file with the specified name. Defines a read data callback function to write captured data to the file. Finally starts recording and records the recording file path.
- **static async stopRecord()**:
  - Async static method to stop the recording process. Stops the audio capturer's work, releases its resources, and clears the `audioCapturer` instance.

```typescript
// Import audio processing module
import { audio } from "@kit.AudioKit";
// Import file system module
import fs from "@ohos.file.fs";

// Define a class for managing audio recording
export class AudioCapturerManager {
  // Static property to store the current audio capturer instance
  static audioCapturer: audio.AudioCapturer | null = null;
  // Static private property to store the recording file path
  private static recordFilePath: string = "";

  // Static async method to create audio capturer instance
  static async createAudioCapturer() {
    if (AudioCapturerManager.audioCapturer) {
      return AudioCapturerManager.audioCapturer;
    }
    // Set audio stream info configuration
    let audioStreamInfo: audio.AudioStreamInfo = {
      samplingRate: audio.AudioSamplingRate.SAMPLE_RATE_16000, // Set sampling rate to 16kHz
      channels: audio.AudioChannel.CHANNEL_1, // Set to mono channel
      sampleFormat: audio.AudioSampleFormat.SAMPLE_FORMAT_S16LE, // Set sample format to 16-bit little endian
      encodingType: audio.AudioEncodingType.ENCODING_TYPE_RAW, // Set encoding type to raw data
    };

    // Set audio capturer info configuration
    let audioCapturerInfo: audio.AudioCapturerInfo = {
      source: audio.SourceType.SOURCE_TYPE_MIC, // Set microphone as audio source
      capturerFlags: 0, // Capturer flags, default value here
    };

    // Create audio capturer options object
    let audioCapturerOptions: audio.AudioCapturerOptions = {
      streamInfo: audioStreamInfo, // Use the audio stream info defined above
      capturerInfo: audioCapturerInfo, // Use the audio capturer info defined above
    };

    // Create audio capturer instance
    AudioCapturerManager.audioCapturer = await audio.createAudioCapturer(
      audioCapturerOptions
    );

    // Return the created audio capturer instance
    return AudioCapturerManager.audioCapturer;
  }

  // Static async method to start recording process
  static async startRecord(fileName: string) {
    await AudioCapturerManager.createAudioCapturer();
    // Initialize buffer size
    let bufferSize: number = 0;

    // Define an internal class to set options for writing to file
    class Options {
      offset?: number; // File write position offset
      length?: number; // Length of data to write
    }

    // Get the application's file directory path
    let path = getContext().filesDir;

    // Set the complete path for the recording file
    let filePath = `${path}/${fileName}.wav`;

    // Open or create recording file
    let file = fs.openSync(
      filePath,
      fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE
    );

    // Define a callback function for reading data
    let readDataCallback = (buffer: ArrayBuffer) => {
      // Create an options object for writing to file
      let options: Options = {
        offset: bufferSize, // Current position offset in file
        length: buffer.byteLength, // Data length
      };
      // Write data to file
      fs.writeSync(file.fd, buffer, options);
      // Update buffer size
      bufferSize += buffer.byteLength;
    };

    // Register read data event listener for the audio capturer instance
    AudioCapturerManager.audioCapturer?.on("readData", readDataCallback);

    // Start recording
    AudioCapturerManager.audioCapturer?.start();

    AudioCapturerManager.recordFilePath = filePath;
    // Return the recording file path
    return filePath;
  }

  // Static async method to stop recording process
  static async stopRecord() {
    // Stop the audio capturer's work
    await AudioCapturerManager.audioCapturer?.stop();
    // Release the audio capturer's resources
    await AudioCapturerManager.audioCapturer?.release();
    // Clear the audio capturer instance
    AudioCapturerManager.audioCapturer = null;
  }
}
```

# Starting Audio Recording on the Page

![image-20240829005514157](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B302-%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E8%BD%AC%E6%96%87%E6%9C%AC.assets/image-20240829005514157.png)

You can check if the recording file is actually generated through the following path:

`/data/app/el2/100/base/your_project_bundle_name/haps/entry/files`

![image-20240829005634585](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B302-%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E8%BD%AC%E6%96%87%E6%9C%AC.assets/image-20240829005634585.png)

### Page Code

`Index.ets`

```typescript
import { PermissionManager } from '../utils/permissionMananger'
import { Permissions } from '@kit.AbilityKit'
import SpeechRecognizerManager from '../utils/SpeechRecognizerManager'
import { AudioCapturerManager } from '../utils/AudioCapturerManager'

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
    }
    .width("100%")
    .height("100%")
    .justifyContent(FlexAlign.Center)
  }
}
```

# Using AI Voice Functionality to Convert Audio Files to Text

This process is actually similar to the real-time speech recognition functionality from the previous chapter, but with one additional step:

1. Create AI voice engine
2. Register voice event listeners
3. **Start listening**
4. **Read the audio file**

## Creating AI Voice Engine

```typescript
/**
 * Create engine
 */
private static async createEngine() {
  // Set engine creation parameters
  SpeechRecognizerManager.asrEngine = await speechRecognizer.createEngine(SpeechRecognizerManager.initParamsInfo)
}
```

## Register Voice Event Listeners

```typescript
/**
 * Set callback
 */
private static setListener(callback: (srr: speechRecognizer.SpeechRecognitionResult) => void = () => {
}) {
  // Create callback object
  let setListener: speechRecognizer.RecognitionListener = {
    // Recognition start success callback
    onStart(sessionId: string, eventMessage: string) {
    },
    // Event callback
    onEvent(sessionId: string, eventCode: number, eventMessage: string) {
    },
    // Recognition result callback, including intermediate and final results
    onResult(sessionId: string, result: speechRecognizer.SpeechRecognitionResult) {
      SpeechRecognizerManager.speechResult = result
      callback && callback(result)
    },
    // Recognition complete callback
    onComplete(sessionId: string, eventMessage: string) {
    },
    // Error callback, error codes returned through this method
    // For example: error code 1002200006 returned means recognition engine is busy, engine is recognizing
    // For more error codes, please refer to error code reference
    onError(sessionId: string, errorCode: number, errorMessage: string) {
      console.log("errorMessage", errorMessage)
    },
  }
  // Set callback
  SpeechRecognizerManager.asrEngine?.setListener(setListener);
}
```

## Start Listening

> Need to set recognitionMode to 1, indicating audio file recognition

```typescript
/**
 * Start listening
 * */
static startListening2() {
  try {
    // Set parameters for starting recognition
    let recognizerParams: speechRecognizer.StartParams = {
      // Session ID
      sessionId: SpeechRecognizerManager.sessionId,
      // Audio configuration info
      audioInfo: {
        // Audio type. Currently only supports "pcm"
        audioType: 'pcm',
        // Audio sampling rate. Currently only supports 16000 sampling rate
        sampleRate: 16000,
        // Audio return channel info. Currently only supports channel 1
        soundChannel: 1,
        // Audio return sample bit depth. Currently only supports 16 bits
        sampleBit: 16
      },
      // Recording recognition
      extraParams: {
        // 0: Real-time recording recognition - automatically opens microphone for real-time voice recording
        // 1: Audio file recognition
        "recognitionMode": 1,
        // Maximum supported audio duration
        maxAudioDuration: 60000
      }
    }
    // Call start recognition method
    SpeechRecognizerManager.asrEngine?.startListening(recognizerParams);
  } catch (e) {
    console.log("e", e.code, e.message)
  }
};
```

## Reading Audio File

> Need to call SpeechRecognizerManager.asrEngine?.writeAudio to process the audio file

```typescript
/**
 *
 * @param fileName {string} Audio file name
 */
private static async writeAudio(fileName: string) {
  let ctx = getContext();
  let filePath: string = `${ctx.filesDir}/${fileName}.wav`;
  let file = fileIo.openSync(filePath, fileIo.OpenMode.READ_WRITE);
  let buf: ArrayBuffer = new ArrayBuffer(1280);
  let offset: number = 0;
  while (1280 == fileIo.readSync(file.fd, buf, {
    offset: offset
  })) {
    let uint8Array: Uint8Array = new Uint8Array(buf);
    // Call AI voice engine for recognition
    SpeechRecognizerManager.asrEngine?.writeAudio(SpeechRecognizerManager.sessionId, uint8Array);
    offset = offset + 1280;
  }
  fileIo.closeSync(file);
}
```

## One-Step Call

```typescript
/**
 * Initialize AI speech-to-text engine
 */
static async init2(callback: (srr: speechRecognizer.SpeechRecognitionResult) => void = () => {
}, fileName: string) {
  try {
    await SpeechRecognizerManager.createEngine()
    SpeechRecognizerManager.setListener(callback)
    SpeechRecognizerManager.startListening2()
    SpeechRecognizerManager.writeAudio(fileName)
  } catch (e) {
    console.log("e", e.message)
  }
}
```

## Complete Code

```typescript
import { speechRecognizer } from "@kit.CoreSpeechKit";
import { fileIo } from "@kit.CoreFileKit";

class SpeechRecognizerManager {
  /**
   * Language information
   * Speech mode: short
   */
  private static extraParam: Record<string, Object> = {
    locate: "CN",
    recognizerMode: "short",
  };
  private static initParamsInfo: speechRecognizer.CreateEngineParams = {
    /**
     * Region information
     * */
    language: "zh-CN",
    /**
     * Online mode: 1
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
   * Set callback
   */
  private static setListener(
    callback: (srr: speechRecognizer.SpeechRecognitionResult) => void = () => {}
  ) {
    // Create callback object
    let setListener: speechRecognizer.RecognitionListener = {
      // Recognition start success callback
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
      // Recognition complete callback
      onComplete(sessionId: string, eventMessage: string) {},
      // Error callback, error codes returned through this method
      // For example: error code 1002200006 returned means recognition engine is busy
      // For more error codes, please refer to error code reference
      onError(sessionId: string, errorCode: number, errorMessage: string) {
        console.log("errorMessage", errorMessage);
      },
    };
    // Set callback
    SpeechRecognizerManager.asrEngine?.setListener(setListener);
  }

  /**
   * Start listening
   * */
  static startListening() {
    try {
      // Set parameters for starting recognition
      let recognizerParams: speechRecognizer.StartParams = {
        // Session ID
        sessionId: SpeechRecognizerManager.sessionId,
        // Audio configuration info
        audioInfo: {
          // Audio type. Currently only supports "pcm"
          audioType: "pcm",
          // Audio sampling rate. Currently only supports 16000 sampling rate
          sampleRate: 16000,
          // Audio return channel info. Currently only supports channel 1
          soundChannel: 1,
          // Audio return sample bit depth. Currently only supports 16 bits
          sampleBit: 16,
        },
        // Recording recognition
        extraParams: {
          // 0: Real-time recording recognition - automatically opens microphone for real-time voice
          recognitionMode: 0,
          // Maximum supported audio duration
          maxAudioDuration: 60000,
        },
      };
      // Call start recognition method
      SpeechRecognizerManager.asrEngine?.startListening(recognizerParams);
    } catch (e) {
      console.log("e", e.code, e.message);
    }
  }

  /**
   *
   * @param fileName {string} Audio file name
   */
  private static async writeAudio(fileName: string) {
    let ctx = getContext();
    let filePath: string = `${ctx.filesDir}/${fileName}.wav`;
    let file = fileIo.openSync(filePath, fileIo.OpenMode.READ_WRITE);
    let buf: ArrayBuffer = new ArrayBuffer(1280);
    let offset: number = 0;
    while (
      1280 ==
      fileIo.readSync(file.fd, buf, {
        offset: offset,
      })
    ) {
      let uint8Array: Uint8Array = new Uint8Array(buf);
      // Call AI voice engine for recognition
      SpeechRecognizerManager.asrEngine?.writeAudio(
        SpeechRecognizerManager.sessionId,
        uint8Array
      );
      // Delay 40ms
      SpeechRecognizerManager.sleep(40);
      offset = offset + 1280;
    }
    fileIo.closeSync(file);
  }

  static sleep(time: number) {
    return new Promise<null>((resolve: Function) => {
      setTimeout(() => {
        resolve();
      }, 40);
    });
  }

  /**
   * Start listening
   * */
  static startListening2() {
    try {
      // Set parameters for starting recognition
      let recognizerParams: speechRecognizer.StartParams = {
        // Session ID
        sessionId: SpeechRecognizerManager.sessionId,
        // Audio configuration info
        audioInfo: {
          // Audio type. Currently only supports "pcm"
          audioType: "pcm",
          // Audio sampling rate. Currently only supports 16000 sampling rate
          sampleRate: 16000,
          // Audio return channel info. Currently only supports channel 1
          soundChannel: 1,
          // Audio return sample bit depth. Currently only supports 16 bits
          sampleBit: 16,
        },
        // Recording recognition
        extraParams: {
          // 0: Real-time recording recognition - automatically opens microphone
          // 1: Audio file recognition
          recognitionMode: 1,
          // Maximum supported audio duration
          maxAudioDuration: 60000,
        },
      };
      // Call start recognition method
      SpeechRecognizerManager.asrEngine?.startListening(recognizerParams);
    } catch (e) {
      console.log("e", e.code, e.message);
    }
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

  /**
   * Initialize AI speech-to-text engine
   */
  static async init2(
    callback: (
      srr: speechRecognizer.SpeechRecognitionResult
    ) => void = () => {},
    fileName: string
  ) {
    try {
      await SpeechRecognizerManager.createEngine();
      SpeechRecognizerManager.setListener(callback);
      SpeechRecognizerManager.startListening2();
      SpeechRecognizerManager.writeAudio(fileName);
    } catch (e) {
      console.log("e", e.message);
    }
  }
}

export default SpeechRecognizerManager;
```

# Page Code

![image-20240920104405824](%E4%B8%89%E6%96%87%E5%B8%A6%E4%BD%A0%E8%BD%BB%E6%9D%BE%E4%B8%8A%E6%89%8B%E9%B8%BF%E8%92%99%E7%9A%84AI%E8%AF%AD%E9%9F%B302-%E5%A3%B0%E9%9F%B3%E6%96%87%E4%BB%B6%E8%BD%AC%E6%96%87%E6%9C%AC.assets/image-20240920104405824.png)

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
