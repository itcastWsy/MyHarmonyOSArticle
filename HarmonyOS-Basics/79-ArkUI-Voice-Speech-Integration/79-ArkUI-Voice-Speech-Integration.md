# ArkUI Voice and Speech Integration

## Introduction

Voice and speech integration enables natural language interaction in ArkUI applications. This guide covers speech recognition, text-to-speech synthesis, voice commands, and audio processing for conversational interfaces.

## Speech Recognition System

### Speech Recognition Manager

```typescript
interface SpeechConfig {
  language: string;
  continuous: boolean;
  maxAlternatives: number;
  confidenceThreshold: number;
  noiseReduction: boolean;
}

interface SpeechResult {
  transcript: string;
  confidence: number;
  alternatives: string[];
  isFinal: boolean;
  timestamp: number;
}

class SpeechRecognitionManager {
  private isListening = false;
  private config: SpeechConfig;
  private resultListeners = new Set<(result: SpeechResult) => void>();
  private errorListeners = new Set<(error: string) => void>();
  private recognition: any; // WebSpeech API or native recognition

  constructor(config: SpeechConfig) {
    this.config = config;
    this.initializeRecognition();
  }

  async startListening(): Promise<boolean> {
    if (this.isListening) return false;

    try {
      await this.requestMicrophonePermission();
      this.recognition.start();
      this.isListening = true;
      return true;
    } catch (error) {
      this.notifyError(`Failed to start listening: ${error.message}`);
      return false;
    }
  }

  stopListening(): void {
    if (this.isListening) {
      this.recognition.stop();
      this.isListening = false;
    }
  }

  setLanguage(language: string): void {
    this.config.language = language;
    this.recognition.lang = language;
  }

  onResult(listener: (result: SpeechResult) => void): void {
    this.resultListeners.add(listener);
  }

  onError(listener: (error: string) => void): void {
    this.errorListeners.add(listener);
  }

  private initializeRecognition(): void {
    if ("webkitSpeechRecognition" in window) {
      this.recognition = new webkitSpeechRecognition();
    } else if ("SpeechRecognition" in window) {
      this.recognition = new SpeechRecognition();
    } else {
      this.notifyError("Speech recognition not supported");
      return;
    }

    this.recognition.continuous = this.config.continuous;
    this.recognition.maxAlternatives = this.config.maxAlternatives;
    this.recognition.lang = this.config.language;

    this.recognition.onresult = (event) => {
      const result = event.results[event.results.length - 1];
      const transcript = result[0].transcript;
      const confidence = result[0].confidence;

      if (confidence >= this.config.confidenceThreshold) {
        const speechResult: SpeechResult = {
          transcript,
          confidence,
          alternatives: Array.from(result).map((r) => r.transcript),
          isFinal: result.isFinal,
          timestamp: Date.now(),
        };
        this.notifyResult(speechResult);
      }
    };

    this.recognition.onerror = (event) => {
      this.notifyError(event.error);
    };

    this.recognition.onend = () => {
      this.isListening = false;
      if (this.config.continuous) {
        this.recognition.start();
        this.isListening = true;
      }
    };
  }

  private async requestMicrophonePermission(): Promise<void> {
    const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
    stream.getTracks().forEach((track) => track.stop());
  }

  private notifyResult(result: SpeechResult): void {
    this.resultListeners.forEach((listener) => listener(result));
  }

  private notifyError(error: string): void {
    this.errorListeners.forEach((listener) => listener(error));
  }
}
```

### Voice Command System

```typescript
interface VoiceCommand {
  pattern: string | RegExp;
  action: string;
  parameters?: string[];
  description: string;
  examples: string[];
}

interface CommandMatch {
  command: VoiceCommand;
  confidence: number;
  parameters: Record<string, string>;
}

class VoiceCommandProcessor {
  private commands = new Map<string, VoiceCommand>();
  private commandHandlers = new Map<
    string,
    (params: Record<string, string>) => void
  >();

  registerCommand(id: string, command: VoiceCommand): void {
    this.commands.set(id, command);
  }

  registerHandler(
    action: string,
    handler: (params: Record<string, string>) => void
  ): void {
    this.commandHandlers.set(action, handler);
  }

  processVoiceInput(transcript: string): CommandMatch | null {
    const normalizedInput = transcript.toLowerCase().trim();

    for (const [id, command] of this.commands) {
      const match = this.matchCommand(normalizedInput, command);
      if (match) {
        this.executeCommand(command.action, match.parameters);
        return {
          command,
          confidence: match.confidence,
          parameters: match.parameters,
        };
      }
    }

    return null;
  }

  private matchCommand(
    input: string,
    command: VoiceCommand
  ): { confidence: number; parameters: Record<string, string> } | null {
    if (typeof command.pattern === "string") {
      const regex = new RegExp(
        command.pattern.replace(/\{(\w+)\}/g, "(?<$1>\\w+)"),
        "i"
      );
      const match = input.match(regex);

      if (match) {
        return {
          confidence: this.calculateConfidence(input, command.pattern),
          parameters: match.groups || {},
        };
      }
    } else {
      const match = input.match(command.pattern);
      if (match) {
        return {
          confidence: 0.9,
          parameters: match.groups || {},
        };
      }
    }

    return null;
  }

  private calculateConfidence(input: string, pattern: string): number {
    const patternWords = pattern
      .toLowerCase()
      .split(/\W+/)
      .filter((w) => w.length > 0);
    const inputWords = input
      .toLowerCase()
      .split(/\W+/)
      .filter((w) => w.length > 0);

    const matches = patternWords.filter((word) => inputWords.includes(word));
    return matches.length / patternWords.length;
  }

  private executeCommand(
    action: string,
    parameters: Record<string, string>
  ): void {
    const handler = this.commandHandlers.get(action);
    if (handler) {
      handler(parameters);
    }
  }

  getAvailableCommands(): VoiceCommand[] {
    return Array.from(this.commands.values());
  }
}
```

## Text-to-Speech System

### Speech Synthesis Manager

```typescript
interface SpeechSynthesisConfig {
  voice?: string;
  rate: number;
  pitch: number;
  volume: number;
  language: string;
}

interface SpeechQueueItem {
  text: string;
  config: SpeechSynthesisConfig;
  onStart?: () => void;
  onEnd?: () => void;
  onError?: (error: string) => void;
}

class SpeechSynthesisManager {
  private synthesis: SpeechSynthesis;
  private speechQueue: SpeechQueueItem[] = [];
  private isSpeaking = false;
  private availableVoices: SpeechSynthesisVoice[] = [];

  constructor() {
    this.synthesis = window.speechSynthesis;
    this.loadVoices();
  }

  async speak(
    text: string,
    config?: Partial<SpeechSynthesisConfig>
  ): Promise<void> {
    const fullConfig: SpeechSynthesisConfig = {
      rate: 1,
      pitch: 1,
      volume: 1,
      language: "en-US",
      ...config,
    };

    return new Promise((resolve, reject) => {
      this.speechQueue.push({
        text,
        config: fullConfig,
        onEnd: resolve,
        onError: reject,
      });

      if (!this.isSpeaking) {
        this.processQueue();
      }
    });
  }

  stop(): void {
    this.synthesis.cancel();
    this.speechQueue = [];
    this.isSpeaking = false;
  }

  pause(): void {
    this.synthesis.pause();
  }

  resume(): void {
    this.synthesis.resume();
  }

  getAvailableVoices(): SpeechSynthesisVoice[] {
    return this.availableVoices;
  }

  setDefaultVoice(voiceName: string): void {
    const voice = this.availableVoices.find((v) => v.name === voiceName);
    if (voice) {
      // Store as default voice
    }
  }

  private loadVoices(): void {
    this.availableVoices = this.synthesis.getVoices();

    if (this.availableVoices.length === 0) {
      this.synthesis.onvoiceschanged = () => {
        this.availableVoices = this.synthesis.getVoices();
      };
    }
  }

  private processQueue(): void {
    if (this.speechQueue.length === 0) {
      this.isSpeaking = false;
      return;
    }

    this.isSpeaking = true;
    const item = this.speechQueue.shift()!;

    const utterance = new SpeechSynthesisUtterance(item.text);
    utterance.rate = item.config.rate;
    utterance.pitch = item.config.pitch;
    utterance.volume = item.config.volume;
    utterance.lang = item.config.language;

    if (item.config.voice) {
      const voice = this.availableVoices.find(
        (v) => v.name === item.config.voice
      );
      if (voice) {
        utterance.voice = voice;
      }
    }

    utterance.onstart = () => {
      item.onStart?.();
    };

    utterance.onend = () => {
      item.onEnd?.();
      this.processQueue();
    };

    utterance.onerror = (event) => {
      item.onError?.(event.error);
      this.processQueue();
    };

    this.synthesis.speak(utterance);
  }
}
```

## Voice-Enabled Components

### Voice Assistant Component

```typescript
@Component
struct VoiceAssistant {
  @State private isListening: boolean = false
  @State private transcript: string = ''
  @State private confidence: number = 0
  @State private lastCommand: string = ''
  @State private responses: string[] = []
  @State private availableCommands: VoiceCommand[] = []

  private speechRecognition = new SpeechRecognitionManager({
    language: 'en-US',
    continuous: true,
    maxAlternatives: 3,
    confidenceThreshold: 0.7,
    noiseReduction: true
  })

  private commandProcessor = new VoiceCommandProcessor()
  private speechSynthesis = new SpeechSynthesisManager()

  build() {
    Column() {
      this.buildHeader()
      this.buildVoiceIndicator()
      this.buildTranscript()
      this.buildResponses()
      this.buildCommands()
      this.buildControls()
    }
    .padding(16)
    .width('100%')
    .height('100%')
  }

  aboutToAppear() {
    this.initializeVoiceCommands()
    this.setupEventListeners()
  }

  @Builder
  private buildHeader() {
    Text('Voice Assistant')
      .fontSize(24)
      .fontWeight(FontWeight.Bold)
      .margin({ bottom: 20 })
  }

  @Builder
  private buildVoiceIndicator() {
    Stack() {
      Circle({ width: 120, height: 120 })
        .fill(this.isListening ? '#007AFF' : '#E0E0E0')
        .animation({
          duration: 500,
          curve: Curve.EaseInOut
        })

      Text(this.isListening ? '🎤' : '🔇')
        .fontSize(48)
        .animation({
          duration: 300,
          curve: Curve.EaseInOut
        })

      if (this.isListening) {
        Circle({ width: 140, height: 140 })
          .stroke('#007AFF40', 2)
          .fillOpacity(0)
          .scale({ x: 1.2, y: 1.2 })
          .animation({
            duration: 1000,
            iterations: -1,
            curve: Curve.Linear
          })
      }
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildTranscript() {
    Column() {
      Text('Listening...')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      Text(this.transcript || 'Say something...')
        .fontSize(14)
        .fontColor(this.transcript ? '#000000' : '#666666')
        .textAlign(TextAlign.Center)
        .backgroundColor('#F8F9FA')
        .padding(16)
        .borderRadius(8)
        .width('100%')
        .minHeight(60)

      if (this.confidence > 0) {
        Progress({
          value: this.confidence * 100,
          total: 100,
          style: ProgressStyle.Linear
        })
        .color('#34C759')
        .backgroundColor('#E0E0E0')
        .margin({ top: 8 })
      }
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildResponses() {
    if (this.responses.length > 0) {
      Column() {
        Text('Assistant Responses')
          .fontSize(16)
          .fontWeight(FontWeight.Bold)
          .margin({ bottom: 8 })

        Scroll() {
          ForEach(this.responses.slice(-3), (response: string, index: number) => {
            Text(response)
              .fontSize(14)
              .fontColor('#007AFF')
              .backgroundColor('#F0F8FF')
              .padding(12)
              .borderRadius(8)
              .width('100%')
              .margin({ bottom: 8 })
          })
        }
        .height(120)
      }
      .margin({ bottom: 20 })
    }
  }

  @Builder
  private buildCommands() {
    Column() {
      Text('Available Commands')
        .fontSize(16)
        .fontWeight(FontWeight.Bold)
        .margin({ bottom: 8 })

      Scroll() {
        ForEach(this.availableCommands.slice(0, 5), (command: VoiceCommand) => {
          Column() {
            Text(command.description)
              .fontSize(14)
              .fontWeight(FontWeight.Medium)
              .margin({ bottom: 4 })

            Text(`Example: "${command.examples[0]}"`)
              .fontSize(12)
              .fontColor('#666666')
              .fontStyle(FontStyle.Italic)
          }
          .alignItems(HorizontalAlign.Start)
          .padding(12)
          .backgroundColor('#F8F9FA')
          .borderRadius(8)
          .width('100%')
          .margin({ bottom: 8 })
        })
      }
      .height(200)
    }
    .margin({ bottom: 20 })
  }

  @Builder
  private buildControls() {
    Row() {
      Button(this.isListening ? 'Stop Listening' : 'Start Listening')
        .onClick(() => this.toggleListening())
        .backgroundColor(this.isListening ? '#FF3B30' : '#34C759')
        .fontColor('#FFFFFF')
        .flexGrow(1)
        .margin({ right: 8 })

      Button('Clear')
        .onClick(() => this.clearResponses())
        .backgroundColor('#8E8E93')
        .fontColor('#FFFFFF')
        .width(80)
    }
  }

  private initializeVoiceCommands(): void {
    // Navigation commands
    this.commandProcessor.registerCommand('navigate', {
      pattern: 'go to {page}|navigate to {page}|open {page}',
      action: 'navigate',
      description: 'Navigate to a page',
      examples: ['Go to settings', 'Navigate to home', 'Open profile']
    })

    this.commandProcessor.registerHandler('navigate', (params) => {
      const page = params.page
      this.addResponse(`Navigating to ${page}`)
      this.speechSynthesis.speak(`Opening ${page} page`)
    })

    // Search commands
    this.commandProcessor.registerCommand('search', {
      pattern: 'search for {query}|find {query}|look for {query}',
      action: 'search',
      description: 'Search for content',
      examples: ['Search for restaurants', 'Find my photos', 'Look for contacts']
    })

    this.commandProcessor.registerHandler('search', (params) => {
      const query = params.query
      this.addResponse(`Searching for "${query}"`)
      this.speechSynthesis.speak(`Searching for ${query}`)
    })

    // Control commands
    this.commandProcessor.registerCommand('volume', {
      pattern: 'set volume to {level}|volume {level}',
      action: 'volume',
      description: 'Control volume',
      examples: ['Set volume to 50', 'Volume high', 'Volume low']
    })

    this.commandProcessor.registerHandler('volume', (params) => {
      const level = params.level
      this.addResponse(`Setting volume to ${level}`)
      this.speechSynthesis.speak(`Volume set to ${level}`)
    })

    // Time and date commands
    this.commandProcessor.registerCommand('time', {
      pattern: 'what time is it|current time|tell me the time',
      action: 'time',
      description: 'Get current time',
      examples: ['What time is it?', 'Current time', 'Tell me the time']
    })

    this.commandProcessor.registerHandler('time', () => {
      const now = new Date()
      const timeString = now.toLocaleTimeString()
      this.addResponse(`Current time is ${timeString}`)
      this.speechSynthesis.speak(`The time is ${timeString}`)
    })

    this.availableCommands = this.commandProcessor.getAvailableCommands()
  }

  private setupEventListeners(): void {
    this.speechRecognition.onResult((result) => {
      this.transcript = result.transcript
      this.confidence = result.confidence

      if (result.isFinal) {
        this.processVoiceCommand(result.transcript)
      }
    })

    this.speechRecognition.onError((error) => {
      this.addResponse(`Speech recognition error: ${error}`)
    })
  }

  private processVoiceCommand(transcript: string): void {
    const match = this.commandProcessor.processVoiceInput(transcript)

    if (match) {
      this.lastCommand = `${match.command.description} (${(match.confidence * 100).toFixed(0)}% confidence)`
    } else {
      this.addResponse(`Command not recognized: "${transcript}"`)
      this.speechSynthesis.speak("I didn't understand that command")
    }
  }

  private async toggleListening(): Promise<void> {
    if (this.isListening) {
      this.speechRecognition.stopListening()
      this.isListening = false
    } else {
      const success = await this.speechRecognition.startListening()
      this.isListening = success

      if (success) {
        this.speechSynthesis.speak("Voice assistant is now listening")
      }
    }
  }

  private clearResponses(): void {
    this.responses = []
    this.transcript = ''
    this.confidence = 0
    this.lastCommand = ''
  }

  private addResponse(response: string): void {
    this.responses.push(response)

    // Keep only last 10 responses
    if (this.responses.length > 10) {
      this.responses = this.responses.slice(-10)
    }
  }
}
```

## Audio Processing

### Audio Analysis Engine

```typescript
interface AudioFeatures {
  volume: number;
  pitch: number;
  spectralCentroid: number;
  zeroCrossingRate: number;
  mfcc: number[];
}

class AudioAnalysisEngine {
  private audioContext: AudioContext;
  private analyser: AnalyserNode;
  private mediaStream: MediaStream | null = null;
  private isAnalyzing = false;

  constructor() {
    this.audioContext = new AudioContext();
    this.analyser = this.audioContext.createAnalyser();
    this.analyser.fftSize = 2048;
  }

  async startAnalysis(): Promise<boolean> {
    try {
      this.mediaStream = await navigator.mediaDevices.getUserMedia({
        audio: true,
      });
      const source = this.audioContext.createMediaStreamSource(
        this.mediaStream
      );
      source.connect(this.analyser);

      this.isAnalyzing = true;
      this.processAudio();
      return true;
    } catch (error) {
      console.error("Failed to start audio analysis:", error);
      return false;
    }
  }

  stopAnalysis(): void {
    this.isAnalyzing = false;
    if (this.mediaStream) {
      this.mediaStream.getTracks().forEach((track) => track.stop());
      this.mediaStream = null;
    }
  }

  private processAudio(): void {
    if (!this.isAnalyzing) return;

    const bufferLength = this.analyser.frequencyBinCount;
    const dataArray = new Uint8Array(bufferLength);
    const timeDataArray = new Uint8Array(bufferLength);

    this.analyser.getByteFrequencyData(dataArray);
    this.analyser.getByteTimeDomainData(timeDataArray);

    const features = this.extractFeatures(dataArray, timeDataArray);
    this.onAudioFeatures(features);

    requestAnimationFrame(() => this.processAudio());
  }

  private extractFeatures(
    frequencyData: Uint8Array,
    timeData: Uint8Array
  ): AudioFeatures {
    return {
      volume: this.calculateVolume(timeData),
      pitch: this.calculatePitch(frequencyData),
      spectralCentroid: this.calculateSpectralCentroid(frequencyData),
      zeroCrossingRate: this.calculateZeroCrossingRate(timeData),
      mfcc: this.calculateMFCC(frequencyData),
    };
  }

  private calculateVolume(timeData: Uint8Array): number {
    let sum = 0;
    for (let i = 0; i < timeData.length; i++) {
      const sample = (timeData[i] - 128) / 128;
      sum += sample * sample;
    }
    return Math.sqrt(sum / timeData.length);
  }

  private calculatePitch(frequencyData: Uint8Array): number {
    let maxIndex = 0;
    let maxValue = 0;

    for (let i = 0; i < frequencyData.length; i++) {
      if (frequencyData[i] > maxValue) {
        maxValue = frequencyData[i];
        maxIndex = i;
      }
    }

    return (
      (maxIndex * this.audioContext.sampleRate) / (2 * frequencyData.length)
    );
  }

  private calculateSpectralCentroid(frequencyData: Uint8Array): number {
    let weightedSum = 0;
    let magnitudeSum = 0;

    for (let i = 0; i < frequencyData.length; i++) {
      const frequency =
        (i * this.audioContext.sampleRate) / (2 * frequencyData.length);
      const magnitude = frequencyData[i];

      weightedSum += frequency * magnitude;
      magnitudeSum += magnitude;
    }

    return magnitudeSum > 0 ? weightedSum / magnitudeSum : 0;
  }

  private calculateZeroCrossingRate(timeData: Uint8Array): number {
    let crossings = 0;
    for (let i = 1; i < timeData.length; i++) {
      if (timeData[i] >= 128 !== timeData[i - 1] >= 128) {
        crossings++;
      }
    }
    return crossings / timeData.length;
  }

  private calculateMFCC(frequencyData: Uint8Array): number[] {
    // Simplified MFCC calculation
    const mfccCount = 13;
    const mfcc: number[] = [];

    for (let i = 0; i < mfccCount; i++) {
      const start = Math.floor((i * frequencyData.length) / mfccCount);
      const end = Math.floor(((i + 1) * frequencyData.length) / mfccCount);

      let sum = 0;
      for (let j = start; j < end; j++) {
        sum += frequencyData[j];
      }

      mfcc.push(sum / (end - start));
    }

    return mfcc;
  }

  private onAudioFeatures(features: AudioFeatures): void {
    // Process audio features
    console.log("Audio features:", features);
  }
}
```

## Conclusion

Voice and speech integration in ArkUI applications enables:

- Advanced speech recognition with confidence scoring
- Intelligent voice command processing and pattern matching
- High-quality text-to-speech synthesis with voice selection
- Real-time audio analysis and feature extraction
- Conversational user interfaces with voice feedback
- Accessibility improvements through voice interaction

These capabilities allow developers to create natural, voice-driven user experiences that enhance accessibility and provide hands-free interaction modes for various application scenarios.
