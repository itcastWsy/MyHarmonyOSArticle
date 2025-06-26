# 32. Media Processing and Multimedia

## Introduction

Multimedia processing is a crucial aspect of modern mobile applications. HarmonyOS provides comprehensive APIs for handling audio, video, and camera functionalities. This article covers the essential techniques for implementing multimedia features in your HarmonyOS applications.

## Audio Processing and Playback

### Audio Player Implementation

```typescript
import media from "@ohos.multimedia.media";
import { Context } from "@ohos.ability.common";

export class AudioPlayer {
  private audioPlayer: media.AudioPlayer | null = null;
  private currentState: string = "idle";
  private onStateChangeCallback?: (state: string) => void;

  async initialize(): Promise<void> {
    try {
      this.audioPlayer = media.createAudioPlayer();
      this.setupEventListeners();
      console.log("Audio player initialized");
    } catch (error) {
      console.error("Failed to initialize audio player:", error);
      throw error;
    }
  }

  private setupEventListeners(): void {
    if (!this.audioPlayer) return;

    this.audioPlayer.on("play", () => {
      this.currentState = "playing";
      this.onStateChangeCallback?.("playing");
      console.log("Audio playback started");
    });

    this.audioPlayer.on("pause", () => {
      this.currentState = "paused";
      this.onStateChangeCallback?.("paused");
      console.log("Audio playback paused");
    });

    this.audioPlayer.on("stop", () => {
      this.currentState = "stopped";
      this.onStateChangeCallback?.("stopped");
      console.log("Audio playback stopped");
    });

    this.audioPlayer.on("finish", () => {
      this.currentState = "finished";
      this.onStateChangeCallback?.("finished");
      console.log("Audio playback finished");
    });

    this.audioPlayer.on("error", (error) => {
      console.error("Audio player error:", error);
      this.currentState = "error";
      this.onStateChangeCallback?.("error");
    });
  }

  async loadAudio(audioPath: string): Promise<void> {
    if (!this.audioPlayer) {
      throw new Error("Audio player not initialized");
    }

    try {
      this.audioPlayer.src = audioPath;
      await new Promise<void>((resolve, reject) => {
        this.audioPlayer!.on("dataLoad", () => {
          console.log("Audio data loaded");
          resolve();
        });
        this.audioPlayer!.on("error", reject);
      });
    } catch (error) {
      console.error("Failed to load audio:", error);
      throw error;
    }
  }

  async play(): Promise<void> {
    if (!this.audioPlayer) {
      throw new Error("Audio player not initialized");
    }

    try {
      await this.audioPlayer.play();
    } catch (error) {
      console.error("Failed to play audio:", error);
      throw error;
    }
  }

  async pause(): Promise<void> {
    if (!this.audioPlayer) {
      throw new Error("Audio player not initialized");
    }

    try {
      await this.audioPlayer.pause();
    } catch (error) {
      console.error("Failed to pause audio:", error);
      throw error;
    }
  }

  async stop(): Promise<void> {
    if (!this.audioPlayer) {
      throw new Error("Audio player not initialized");
    }

    try {
      await this.audioPlayer.stop();
    } catch (error) {
      console.error("Failed to stop audio:", error);
      throw error;
    }
  }

  async setVolume(volume: number): Promise<void> {
    if (!this.audioPlayer) {
      throw new Error("Audio player not initialized");
    }

    try {
      // Volume should be between 0 and 1
      const normalizedVolume = Math.max(0, Math.min(1, volume));
      await this.audioPlayer.setVolume(normalizedVolume);
    } catch (error) {
      console.error("Failed to set volume:", error);
      throw error;
    }
  }

  async seek(position: number): Promise<void> {
    if (!this.audioPlayer) {
      throw new Error("Audio player not initialized");
    }

    try {
      await this.audioPlayer.seek(position);
    } catch (error) {
      console.error("Failed to seek audio:", error);
      throw error;
    }
  }

  getCurrentTime(): number {
    return this.audioPlayer?.currentTime || 0;
  }

  getDuration(): number {
    return this.audioPlayer?.duration || 0;
  }

  getState(): string {
    return this.currentState;
  }

  setOnStateChange(callback: (state: string) => void): void {
    this.onStateChangeCallback = callback;
  }

  async release(): Promise<void> {
    if (this.audioPlayer) {
      try {
        await this.audioPlayer.release();
        this.audioPlayer = null;
        this.currentState = "idle";
        console.log("Audio player released");
      } catch (error) {
        console.error("Failed to release audio player:", error);
      }
    }
  }
}
```

### Audio Recorder Implementation

```typescript
import media from "@ohos.multimedia.media";
import { Context } from "@ohos.ability.common";

export interface AudioRecorderConfig {
  audioEncodingBitRate: number;
  audioSampleRate: number;
  numberOfChannels: number;
  format: media.AudioOutputFormat;
  encoder: media.AudioEncoder;
  outputPath: string;
}

export class AudioRecorder {
  private audioRecorder: media.AudioRecorder | null = null;
  private isRecording: boolean = false;
  private onStateChangeCallback?: (state: string) => void;

  async initialize(config: AudioRecorderConfig): Promise<void> {
    try {
      this.audioRecorder = media.createAudioRecorder();
      await this.configureRecorder(config);
      this.setupEventListeners();
      console.log("Audio recorder initialized");
    } catch (error) {
      console.error("Failed to initialize audio recorder:", error);
      throw error;
    }
  }

  private async configureRecorder(config: AudioRecorderConfig): Promise<void> {
    if (!this.audioRecorder) return;

    const recorderConfig: media.AudioRecorderConfig = {
      audioEncodingBitRate: config.audioEncodingBitRate,
      audioSampleRate: config.audioSampleRate,
      numberOfChannels: config.numberOfChannels,
      format: config.format,
      encoder: config.encoder,
      uri: config.outputPath,
    };

    await this.audioRecorder.prepare(recorderConfig);
  }

  private setupEventListeners(): void {
    if (!this.audioRecorder) return;

    this.audioRecorder.on("prepare", () => {
      console.log("Audio recorder prepared");
      this.onStateChangeCallback?.("prepared");
    });

    this.audioRecorder.on("start", () => {
      console.log("Audio recording started");
      this.isRecording = true;
      this.onStateChangeCallback?.("recording");
    });

    this.audioRecorder.on("pause", () => {
      console.log("Audio recording paused");
      this.onStateChangeCallback?.("paused");
    });

    this.audioRecorder.on("resume", () => {
      console.log("Audio recording resumed");
      this.onStateChangeCallback?.("recording");
    });

    this.audioRecorder.on("stop", () => {
      console.log("Audio recording stopped");
      this.isRecording = false;
      this.onStateChangeCallback?.("stopped");
    });

    this.audioRecorder.on("error", (error) => {
      console.error("Audio recorder error:", error);
      this.isRecording = false;
      this.onStateChangeCallback?.("error");
    });
  }

  async startRecording(): Promise<void> {
    if (!this.audioRecorder) {
      throw new Error("Audio recorder not initialized");
    }

    try {
      await this.audioRecorder.start();
    } catch (error) {
      console.error("Failed to start recording:", error);
      throw error;
    }
  }

  async pauseRecording(): Promise<void> {
    if (!this.audioRecorder) {
      throw new Error("Audio recorder not initialized");
    }

    try {
      await this.audioRecorder.pause();
    } catch (error) {
      console.error("Failed to pause recording:", error);
      throw error;
    }
  }

  async resumeRecording(): Promise<void> {
    if (!this.audioRecorder) {
      throw new Error("Audio recorder not initialized");
    }

    try {
      await this.audioRecorder.resume();
    } catch (error) {
      console.error("Failed to resume recording:", error);
      throw error;
    }
  }

  async stopRecording(): Promise<void> {
    if (!this.audioRecorder) {
      throw new Error("Audio recorder not initialized");
    }

    try {
      await this.audioRecorder.stop();
    } catch (error) {
      console.error("Failed to stop recording:", error);
      throw error;
    }
  }

  isCurrentlyRecording(): boolean {
    return this.isRecording;
  }

  setOnStateChange(callback: (state: string) => void): void {
    this.onStateChangeCallback = callback;
  }

  async release(): Promise<void> {
    if (this.audioRecorder) {
      try {
        if (this.isRecording) {
          await this.stopRecording();
        }
        await this.audioRecorder.release();
        this.audioRecorder = null;
        this.isRecording = false;
        console.log("Audio recorder released");
      } catch (error) {
        console.error("Failed to release audio recorder:", error);
      }
    }
  }
}
```

## Video Processing

### Video Player Implementation

```typescript
import media from "@ohos.multimedia.media";

export class VideoPlayer {
  private videoPlayer: media.VideoPlayer | null = null;
  private surfaceId: string = "";
  private currentState: string = "idle";
  private onStateChangeCallback?: (state: string) => void;

  async initialize(surfaceId: string): Promise<void> {
    try {
      this.videoPlayer = media.createVideoPlayer();
      this.surfaceId = surfaceId;
      this.setupEventListeners();
      console.log("Video player initialized");
    } catch (error) {
      console.error("Failed to initialize video player:", error);
      throw error;
    }
  }

  private setupEventListeners(): void {
    if (!this.videoPlayer) return;

    this.videoPlayer.on("play", () => {
      this.currentState = "playing";
      this.onStateChangeCallback?.("playing");
      console.log("Video playback started");
    });

    this.videoPlayer.on("pause", () => {
      this.currentState = "paused";
      this.onStateChangeCallback?.("paused");
      console.log("Video playback paused");
    });

    this.videoPlayer.on("stop", () => {
      this.currentState = "stopped";
      this.onStateChangeCallback?.("stopped");
      console.log("Video playback stopped");
    });

    this.videoPlayer.on("finish", () => {
      this.currentState = "finished";
      this.onStateChangeCallback?.("finished");
      console.log("Video playback finished");
    });

    this.videoPlayer.on("error", (error) => {
      console.error("Video player error:", error);
      this.currentState = "error";
      this.onStateChangeCallback?.("error");
    });

    this.videoPlayer.on("videoSizeChanged", (width, height) => {
      console.log(`Video size changed: ${width}x${height}`);
    });
  }

  async loadVideo(videoPath: string): Promise<void> {
    if (!this.videoPlayer) {
      throw new Error("Video player not initialized");
    }

    try {
      this.videoPlayer.url = videoPath;
      await this.videoPlayer.setDisplaySurface(this.surfaceId);

      await new Promise<void>((resolve, reject) => {
        this.videoPlayer!.on("dataLoad", () => {
          console.log("Video data loaded");
          resolve();
        });
        this.videoPlayer!.on("error", reject);
      });
    } catch (error) {
      console.error("Failed to load video:", error);
      throw error;
    }
  }

  async play(): Promise<void> {
    if (!this.videoPlayer) {
      throw new Error("Video player not initialized");
    }

    try {
      await this.videoPlayer.play();
    } catch (error) {
      console.error("Failed to play video:", error);
      throw error;
    }
  }

  async pause(): Promise<void> {
    if (!this.videoPlayer) {
      throw new Error("Video player not initialized");
    }

    try {
      await this.videoPlayer.pause();
    } catch (error) {
      console.error("Failed to pause video:", error);
      throw error;
    }
  }

  async stop(): Promise<void> {
    if (!this.videoPlayer) {
      throw new Error("Video player not initialized");
    }

    try {
      await this.videoPlayer.stop();
    } catch (error) {
      console.error("Failed to stop video:", error);
      throw error;
    }
  }

  async seek(position: number): Promise<void> {
    if (!this.videoPlayer) {
      throw new Error("Video player not initialized");
    }

    try {
      await this.videoPlayer.seek(position);
    } catch (error) {
      console.error("Failed to seek video:", error);
      throw error;
    }
  }

  async setPlaybackSpeed(speed: number): Promise<void> {
    if (!this.videoPlayer) {
      throw new Error("Video player not initialized");
    }

    try {
      await this.videoPlayer.setSpeed(media.PlaybackSpeed.SPEED_FORWARD_2_00_X);
    } catch (error) {
      console.error("Failed to set playback speed:", error);
      throw error;
    }
  }

  getCurrentTime(): number {
    return this.videoPlayer?.currentTime || 0;
  }

  getDuration(): number {
    return this.videoPlayer?.duration || 0;
  }

  getVideoWidth(): number {
    return this.videoPlayer?.width || 0;
  }

  getVideoHeight(): number {
    return this.videoPlayer?.height || 0;
  }

  getState(): string {
    return this.currentState;
  }

  setOnStateChange(callback: (state: string) => void): void {
    this.onStateChangeCallback = callback;
  }

  async release(): Promise<void> {
    if (this.videoPlayer) {
      try {
        await this.videoPlayer.release();
        this.videoPlayer = null;
        this.currentState = "idle";
        console.log("Video player released");
      } catch (error) {
        console.error("Failed to release video player:", error);
      }
    }
  }
}
```

### Video Recorder Implementation

```typescript
import media from "@ohos.multimedia.media";

export interface VideoRecorderConfig {
  audioEncodingBitRate: number;
  audioSampleRate: number;
  numberOfChannels: number;
  videoEncodingBitRate: number;
  videoFrameRate: number;
  videoFrameWidth: number;
  videoFrameHeight: number;
  outputFormat: media.ContainerFormatType;
  outputPath: string;
}

export class VideoRecorder {
  private videoRecorder: media.VideoRecorder | null = null;
  private surfaceId: string = "";
  private isRecording: boolean = false;
  private onStateChangeCallback?: (state: string) => void;

  async initialize(
    config: VideoRecorderConfig,
    surfaceId: string
  ): Promise<void> {
    try {
      this.videoRecorder = media.createVideoRecorder();
      this.surfaceId = surfaceId;
      await this.configureRecorder(config);
      this.setupEventListeners();
      console.log("Video recorder initialized");
    } catch (error) {
      console.error("Failed to initialize video recorder:", error);
      throw error;
    }
  }

  private async configureRecorder(config: VideoRecorderConfig): Promise<void> {
    if (!this.videoRecorder) return;

    const recorderConfig: media.VideoRecorderConfig = {
      audioSourceType: media.AudioSourceType.AUDIO_SOURCE_TYPE_MIC,
      videoSourceType: media.VideoSourceType.VIDEO_SOURCE_TYPE_SURFACE_YUV,
      profile: {
        audioBitrate: config.audioEncodingBitRate,
        audioChannels: config.numberOfChannels,
        audioCodec: media.CodecMimeType.AUDIO_AAC,
        audioSampleRate: config.audioSampleRate,
        fileFormat: config.outputFormat,
        videoBitrate: config.videoEncodingBitRate,
        videoCodec: media.CodecMimeType.VIDEO_AVC,
        videoFrameWidth: config.videoFrameWidth,
        videoFrameHeight: config.videoFrameHeight,
        videoFrameRate: config.videoFrameRate,
      },
      url: config.outputPath,
      rotation: 0,
    };

    await this.videoRecorder.prepare(recorderConfig);
  }

  private setupEventListeners(): void {
    if (!this.videoRecorder) return;

    this.videoRecorder.on("prepare", () => {
      console.log("Video recorder prepared");
      this.onStateChangeCallback?.("prepared");
    });

    this.videoRecorder.on("start", () => {
      console.log("Video recording started");
      this.isRecording = true;
      this.onStateChangeCallback?.("recording");
    });

    this.videoRecorder.on("pause", () => {
      console.log("Video recording paused");
      this.onStateChangeCallback?.("paused");
    });

    this.videoRecorder.on("resume", () => {
      console.log("Video recording resumed");
      this.onStateChangeCallback?.("recording");
    });

    this.videoRecorder.on("stop", () => {
      console.log("Video recording stopped");
      this.isRecording = false;
      this.onStateChangeCallback?.("stopped");
    });

    this.videoRecorder.on("error", (error) => {
      console.error("Video recorder error:", error);
      this.isRecording = false;
      this.onStateChangeCallback?.("error");
    });
  }

  async getInputSurface(): Promise<string> {
    if (!this.videoRecorder) {
      throw new Error("Video recorder not initialized");
    }

    try {
      const surface = await this.videoRecorder.getInputSurface();
      return surface;
    } catch (error) {
      console.error("Failed to get input surface:", error);
      throw error;
    }
  }

  async startRecording(): Promise<void> {
    if (!this.videoRecorder) {
      throw new Error("Video recorder not initialized");
    }

    try {
      await this.videoRecorder.start();
    } catch (error) {
      console.error("Failed to start recording:", error);
      throw error;
    }
  }

  async pauseRecording(): Promise<void> {
    if (!this.videoRecorder) {
      throw new Error("Video recorder not initialized");
    }

    try {
      await this.videoRecorder.pause();
    } catch (error) {
      console.error("Failed to pause recording:", error);
      throw error;
    }
  }

  async resumeRecording(): Promise<void> {
    if (!this.videoRecorder) {
      throw new Error("Video recorder not initialized");
    }

    try {
      await this.videoRecorder.resume();
    } catch (error) {
      console.error("Failed to resume recording:", error);
      throw error;
    }
  }

  async stopRecording(): Promise<void> {
    if (!this.videoRecorder) {
      throw new Error("Video recorder not initialized");
    }

    try {
      await this.videoRecorder.stop();
    } catch (error) {
      console.error("Failed to stop recording:", error);
      throw error;
    }
  }

  isCurrentlyRecording(): boolean {
    return this.isRecording;
  }

  setOnStateChange(callback: (state: string) => void): void {
    this.onStateChangeCallback = callback;
  }

  async release(): Promise<void> {
    if (this.videoRecorder) {
      try {
        if (this.isRecording) {
          await this.stopRecording();
        }
        await this.videoRecorder.release();
        this.videoRecorder = null;
        this.isRecording = false;
        console.log("Video recorder released");
      } catch (error) {
        console.error("Failed to release video recorder:", error);
      }
    }
  }
}
```

## Camera Integration

### Camera Manager Implementation

```typescript
import camera from "@ohos.multimedia.camera";
import image from "@ohos.multimedia.image";

export class CameraManager {
  private cameraManager: camera.CameraManager | null = null;
  private cameraInput: camera.CameraInput | null = null;
  private previewOutput: camera.PreviewOutput | null = null;
  private photoOutput: camera.PhotoOutput | null = null;
  private videoOutput: camera.VideoOutput | null = null;
  private captureSession: camera.CaptureSession | null = null;
  private surfaceId: string = "";

  async initialize(context: Context, surfaceId: string): Promise<void> {
    try {
      this.surfaceId = surfaceId;
      this.cameraManager = camera.getCameraManager(context);
      await this.setupCamera();
      console.log("Camera manager initialized");
    } catch (error) {
      console.error("Failed to initialize camera:", error);
      throw error;
    }
  }

  private async setupCamera(): Promise<void> {
    if (!this.cameraManager) return;

    try {
      // Get available cameras
      const cameras = this.cameraManager.getSupportedCameras();
      if (cameras.length === 0) {
        throw new Error("No cameras available");
      }

      // Use the first available camera (usually back camera)
      const cameraDevice = cameras[0];

      // Create camera input
      this.cameraInput = this.cameraManager.createCameraInput(cameraDevice);

      // Setup event listeners
      this.setupCameraEventListeners();

      // Open camera
      await this.cameraInput.open();

      // Create capture session
      this.captureSession = this.cameraManager.createCaptureSession();

      // Setup preview output
      await this.setupPreviewOutput();

      // Setup photo output
      await this.setupPhotoOutput();

      // Configure and start session
      await this.configureSession();
    } catch (error) {
      console.error("Failed to setup camera:", error);
      throw error;
    }
  }

  private setupCameraEventListeners(): void {
    if (!this.cameraInput) return;

    this.cameraInput.on("error", (error) => {
      console.error("Camera input error:", error);
    });

    this.cameraInput.on("focusStateChange", (state) => {
      console.log("Focus state changed:", state);
    });

    this.cameraInput.on("exposureStateChange", (state) => {
      console.log("Exposure state changed:", state);
    });
  }

  private async setupPreviewOutput(): Promise<void> {
    if (!this.cameraManager || !this.captureSession) return;

    try {
      const previewProfiles = this.cameraManager.getSupportedOutputCapability(
        this.cameraInput!.cameraDevice
      ).previewProfiles;

      if (previewProfiles.length > 0) {
        this.previewOutput = this.cameraManager.createPreviewOutput(
          previewProfiles[0],
          this.surfaceId
        );

        await this.captureSession.addOutput(this.previewOutput);
      }
    } catch (error) {
      console.error("Failed to setup preview output:", error);
      throw error;
    }
  }

  private async setupPhotoOutput(): Promise<void> {
    if (!this.cameraManager || !this.captureSession) return;

    try {
      const photoProfiles = this.cameraManager.getSupportedOutputCapability(
        this.cameraInput!.cameraDevice
      ).photoProfiles;

      if (photoProfiles.length > 0) {
        this.photoOutput = this.cameraManager.createPhotoOutput(
          photoProfiles[0]
        );

        this.photoOutput.on("photoAvailable", (photo) => {
          console.log("Photo captured");
          this.handlePhotoCaptured(photo);
        });

        this.photoOutput.on("error", (error) => {
          console.error("Photo output error:", error);
        });

        await this.captureSession.addOutput(this.photoOutput);
      }
    } catch (error) {
      console.error("Failed to setup photo output:", error);
      throw error;
    }
  }

  private async configureSession(): Promise<void> {
    if (!this.captureSession || !this.cameraInput) return;

    try {
      await this.captureSession.addInput(this.cameraInput);
      await this.captureSession.commitConfig();
      await this.captureSession.start();
      console.log("Camera session started");
    } catch (error) {
      console.error("Failed to configure camera session:", error);
      throw error;
    }
  }

  private handlePhotoCaptured(photo: image.Image): void {
    // Handle the captured photo
    console.log("Processing captured photo");

    // You can process the image here
    // For example, save to file, apply filters, etc.

    // Remember to release the image when done
    photo.release();
  }

  async capturePhoto(): Promise<void> {
    if (!this.photoOutput) {
      throw new Error("Photo output not initialized");
    }

    try {
      const photoSettings: camera.PhotoCaptureSetting = {
        quality: camera.QualityLevel.QUALITY_LEVEL_HIGH,
        rotation: camera.ImageRotation.ROTATION_0,
      };

      await this.photoOutput.capture(photoSettings);
      console.log("Photo capture initiated");
    } catch (error) {
      console.error("Failed to capture photo:", error);
      throw error;
    }
  }

  async switchCamera(): Promise<void> {
    if (!this.cameraManager || !this.captureSession) return;

    try {
      const cameras = this.cameraManager.getSupportedCameras();
      if (cameras.length < 2) {
        console.log("Only one camera available");
        return;
      }

      // Stop current session
      await this.captureSession.stop();

      // Switch to next camera
      const currentCamera = this.cameraInput?.cameraDevice;
      const currentIndex = cameras.findIndex(
        (cam) => cam.cameraId === currentCamera?.cameraId
      );
      const nextIndex = (currentIndex + 1) % cameras.length;
      const nextCamera = cameras[nextIndex];

      // Close current camera
      await this.cameraInput?.close();

      // Open new camera
      this.cameraInput = this.cameraManager.createCameraInput(nextCamera);
      await this.cameraInput.open();

      // Reconfigure session
      await this.captureSession.removeInput(this.cameraInput);
      await this.captureSession.addInput(this.cameraInput);
      await this.captureSession.commitConfig();
      await this.captureSession.start();

      console.log("Camera switched successfully");
    } catch (error) {
      console.error("Failed to switch camera:", error);
      throw error;
    }
  }

  async setFlashMode(mode: camera.FlashMode): Promise<void> {
    if (!this.cameraInput) {
      throw new Error("Camera input not initialized");
    }

    try {
      if (this.cameraInput.hasFlash()) {
        await this.cameraInput.setFlashMode(mode);
        console.log("Flash mode set:", mode);
      } else {
        console.log("Flash not available on this camera");
      }
    } catch (error) {
      console.error("Failed to set flash mode:", error);
      throw error;
    }
  }

  async setZoom(ratio: number): Promise<void> {
    if (!this.cameraInput) {
      throw new Error("Camera input not initialized");
    }

    try {
      const zoomRange = this.cameraInput.getZoomRatioRange();
      const clampedRatio = Math.max(
        zoomRange.min,
        Math.min(zoomRange.max, ratio)
      );

      await this.cameraInput.setZoomRatio(clampedRatio);
      console.log("Zoom ratio set:", clampedRatio);
    } catch (error) {
      console.error("Failed to set zoom ratio:", error);
      throw error;
    }
  }

  async release(): Promise<void> {
    try {
      if (this.captureSession) {
        await this.captureSession.stop();
        await this.captureSession.release();
      }

      if (this.cameraInput) {
        await this.cameraInput.close();
      }

      if (this.previewOutput) {
        await this.previewOutput.release();
      }

      if (this.photoOutput) {
        await this.photoOutput.release();
      }

      if (this.videoOutput) {
        await this.videoOutput.release();
      }

      console.log("Camera manager released");
    } catch (error) {
      console.error("Failed to release camera manager:", error);
    }
  }
}
```

## Multimedia File Handling

### Media File Manager

```typescript
import fs from "@ohos.file.fs";
import { Context } from "@ohos.ability.common";

export interface MediaMetadata {
  fileName: string;
  filePath: string;
  fileSize: number;
  mimeType: string;
  duration?: number;
  width?: number;
  height?: number;
  createdAt: Date;
  modifiedAt: Date;
}

export class MediaFileManager {
  private static context: Context;
  private static mediaDir: string = "";

  static initialize(context: Context): void {
    this.context = context;
    this.mediaDir = context.filesDir + "/media";
    this.ensureMediaDirectory();
  }

  private static ensureMediaDirectory(): void {
    try {
      if (!fs.accessSync(this.mediaDir)) {
        fs.mkdirSync(this.mediaDir);
      }
    } catch (error) {
      console.error("Failed to create media directory:", error);
    }
  }

  static async saveAudioFile(
    audioData: ArrayBuffer,
    fileName: string
  ): Promise<string> {
    const filePath = `${this.mediaDir}/audio_${fileName}`;

    try {
      const file = fs.openSync(
        filePath,
        fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE
      );
      await fs.write(file.fd, audioData);
      fs.closeSync(file);

      console.log("Audio file saved:", filePath);
      return filePath;
    } catch (error) {
      console.error("Failed to save audio file:", error);
      throw error;
    }
  }

  static async saveVideoFile(
    videoData: ArrayBuffer,
    fileName: string
  ): Promise<string> {
    const filePath = `${this.mediaDir}/video_${fileName}`;

    try {
      const file = fs.openSync(
        filePath,
        fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE
      );
      await fs.write(file.fd, videoData);
      fs.closeSync(file);

      console.log("Video file saved:", filePath);
      return filePath;
    } catch (error) {
      console.error("Failed to save video file:", error);
      throw error;
    }
  }

  static async saveImageFile(
    imageData: ArrayBuffer,
    fileName: string
  ): Promise<string> {
    const filePath = `${this.mediaDir}/image_${fileName}`;

    try {
      const file = fs.openSync(
        filePath,
        fs.OpenMode.READ_WRITE | fs.OpenMode.CREATE
      );
      await fs.write(file.fd, imageData);
      fs.closeSync(file);

      console.log("Image file saved:", filePath);
      return filePath;
    } catch (error) {
      console.error("Failed to save image file:", error);
      throw error;
    }
  }

  static async getMediaMetadata(filePath: string): Promise<MediaMetadata> {
    try {
      const stat = fs.statSync(filePath);
      const fileName = filePath.split("/").pop() || "";

      const metadata: MediaMetadata = {
        fileName: fileName,
        filePath: filePath,
        fileSize: stat.size,
        mimeType: this.getMimeType(fileName),
        createdAt: new Date(stat.ctime),
        modifiedAt: new Date(stat.mtime),
      };

      return metadata;
    } catch (error) {
      console.error("Failed to get media metadata:", error);
      throw error;
    }
  }

  private static getMimeType(fileName: string): string {
    const extension = fileName.split(".").pop()?.toLowerCase();

    switch (extension) {
      case "mp3":
        return "audio/mpeg";
      case "wav":
        return "audio/wav";
      case "aac":
        return "audio/aac";
      case "mp4":
        return "video/mp4";
      case "avi":
        return "video/avi";
      case "mov":
        return "video/quicktime";
      case "jpg":
      case "jpeg":
        return "image/jpeg";
      case "png":
        return "image/png";
      case "gif":
        return "image/gif";
      default:
        return "application/octet-stream";
    }
  }

  static async listMediaFiles(
    type?: "audio" | "video" | "image"
  ): Promise<MediaMetadata[]> {
    try {
      const files = fs.listFileSync(this.mediaDir);
      const mediaFiles: MediaMetadata[] = [];

      for (const file of files) {
        const filePath = `${this.mediaDir}/${file}`;

        if (type) {
          const prefix = `${type}_`;
          if (!file.startsWith(prefix)) {
            continue;
          }
        }

        const metadata = await this.getMediaMetadata(filePath);
        mediaFiles.push(metadata);
      }

      return mediaFiles.sort(
        (a, b) => b.modifiedAt.getTime() - a.modifiedAt.getTime()
      );
    } catch (error) {
      console.error("Failed to list media files:", error);
      throw error;
    }
  }

  static async deleteMediaFile(filePath: string): Promise<void> {
    try {
      fs.unlinkSync(filePath);
      console.log("Media file deleted:", filePath);
    } catch (error) {
      console.error("Failed to delete media file:", error);
      throw error;
    }
  }

  static async copyMediaFile(
    sourcePath: string,
    destFileName: string
  ): Promise<string> {
    const destPath = `${this.mediaDir}/${destFileName}`;

    try {
      fs.copyFileSync(sourcePath, destPath);
      console.log("Media file copied:", destPath);
      return destPath;
    } catch (error) {
      console.error("Failed to copy media file:", error);
      throw error;
    }
  }

  static async clearMediaCache(): Promise<void> {
    try {
      const files = fs.listFileSync(this.mediaDir);

      for (const file of files) {
        const filePath = `${this.mediaDir}/${file}`;
        fs.unlinkSync(filePath);
      }

      console.log("Media cache cleared");
    } catch (error) {
      console.error("Failed to clear media cache:", error);
      throw error;
    }
  }
}
```

## UI Components for Media

### Media Player Component

```typescript
@Component
export struct MediaPlayerComponent {
  @State private isPlaying: boolean = false;
  @State private currentTime: number = 0;
  @State private duration: number = 0;
  @State private volume: number = 0.7;
  @State private mediaPath: string = '';

  private audioPlayer: AudioPlayer = new AudioPlayer();
  private timer: number = -1;

  async aboutToAppear() {
    await this.audioPlayer.initialize();

    this.audioPlayer.setOnStateChange((state: string) => {
      switch (state) {
        case 'playing':
          this.isPlaying = true;
          this.startProgressTimer();
          break;
        case 'paused':
        case 'stopped':
          this.isPlaying = false;
          this.stopProgressTimer();
          break;
        case 'finished':
          this.isPlaying = false;
          this.currentTime = 0;
          this.stopProgressTimer();
          break;
      }
    });
  }

  aboutToDisappear() {
    this.stopProgressTimer();
    this.audioPlayer.release();
  }

  private startProgressTimer(): void {
    this.timer = setInterval(() => {
      this.currentTime = this.audioPlayer.getCurrentTime();
      this.duration = this.audioPlayer.getDuration();
    }, 1000);
  }

  private stopProgressTimer(): void {
    if (this.timer !== -1) {
      clearInterval(this.timer);
      this.timer = -1;
    }
  }

  private async playPause(): Promise<void> {
    if (this.isPlaying) {
      await this.audioPlayer.pause();
    } else {
      if (this.mediaPath) {
        await this.audioPlayer.loadAudio(this.mediaPath);
      }
      await this.audioPlayer.play();
    }
  }

  private async stop(): Promise<void> {
    await this.audioPlayer.stop();
    this.currentTime = 0;
  }

  private async onSeek(value: number): Promise<void> {
    const seekTime = (value / 100) * this.duration;
    await this.audioPlayer.seek(seekTime);
    this.currentTime = seekTime;
  }

  private async onVolumeChange(value: number): Promise<void> {
    this.volume = value / 100;
    await this.audioPlayer.setVolume(this.volume);
  }

  private formatTime(seconds: number): string {
    const mins = Math.floor(seconds / 60);
    const secs = Math.floor(seconds % 60);
    return `${mins.toString().padStart(2, '0')}:${secs.toString().padStart(2, '0')}`;
  }

  build() {
    Column({ space: 20 }) {
      // Media information
      Text('Media Player')
        .fontSize(24)
        .fontWeight(FontWeight.Bold)

      // Progress bar
      Column() {
        Slider({
          value: this.duration > 0 ? (this.currentTime / this.duration) * 100 : 0,
          min: 0,
          max: 100,
          step: 1
        })
          .width('100%')
          .onChange((value: number) => {
            this.onSeek(value);
          })

        Row() {
          Text(this.formatTime(this.currentTime))
            .fontSize(12)

          Spacer()

          Text(this.formatTime(this.duration))
            .fontSize(12)
        }
        .width('100%')
      }

      // Control buttons
      Row({ space: 20 }) {
        Button('Stop')
          .onClick(() => this.stop())

        Button(this.isPlaying ? 'Pause' : 'Play')
          .onClick(() => this.playPause())
      }

      // Volume control
      Row() {
        Text('Volume')
          .fontSize(16)

        Slider({
          value: this.volume * 100,
          min: 0,
          max: 100,
          step: 1
        })
          .width('70%')
          .onChange((value: number) => {
            this.onVolumeChange(value);
          })
      }
      .width('100%')
    }
    .width('100%')
    .padding(20)
  }
}
```

## Best Practices

### 1. Resource Management

- Always release media resources when done
- Use try-catch blocks for error handling
- Implement proper cleanup in component lifecycle methods

### 2. Performance Optimization

- Use appropriate codec settings for quality vs. file size balance
- Implement efficient streaming for large media files
- Consider memory usage when processing multiple media files

### 3. User Experience

- Provide progress indicators for long operations
- Handle permission requests gracefully
- Implement proper error messaging

### 4. Security Considerations

- Validate media file formats before processing
- Implement access controls for sensitive media
- Consider encryption for private media content

## Conclusion

HarmonyOS provides comprehensive multimedia capabilities through its media APIs. By implementing proper audio and video handling, camera integration, and file management, you can create rich multimedia experiences in your applications. Remember to handle resources properly, implement error handling, and consider performance implications when working with multimedia content.

The examples provided serve as a foundation for building more complex multimedia features. Consider extending these implementations based on your specific application requirements, such as adding filters, effects, or streaming capabilities.
