# AI 引擎

> 🤖 HarmonyOS AI 能力与机器学习

## 📖 目录

1. [AI 能力概述](#ai-能力概述)
2. [HiAI Foundation](#hiai-foundation)
3. [ML Kit 机器学习](#ml-kit-机器学习)
4. [CV 计算机视觉](#cv-计算机视觉)
5. [NLP 自然语言处理](#nlp-自然语言处理)
6. [语音能力](#语音能力)
7. [AI 最佳实践](#ai-最佳实践)
8. [常见问题](#常见问题)

---

## AI 能力概述

HarmonyOS 提供全面的 AI 能力，包括机器学习、计算机视觉、自然语言处理等。

### AI 技术栈

```
AI 能力
├── HiAI Foundation
│   ├── HiAI Engine
│   ├── HiAI Model
│   └── HiAI HiAIPPT
├── ML Kit（机器学习）
│   ├── 人脸检测
│   ├── 文字识别
│   ├── 物体检测
│   └── 图像分割
├── CV（计算机视觉）
│   ├── 图像分类
│   ├── 目标检测
│   └── 图像增强
├── NLP（自然语言处理）
│   ├── 文本分析
│   ├── 机器翻译
│   └── 情感分析
└── 语音能力
    ├── 语音识别（ASR）
    ├── 语音合成（TTS）
    └── 声纹识别
```

---

## HiAI Foundation

### 初始化 HiAI

```typescript
import { hiAI } from '@kit.HiAIFoundation';

// 初始化 HiAI Engine
async function initHiAI(): Promise<void> {
  try {
    await hiAI.init();
    console.log('HiAI initialized successfully');
  } catch (error) {
    console.error('HiAI initialization failed:', error);
  }
}

// 检查 HiAI 能力
async function checkHiAICapability(): Promise<boolean> {
  try {
    const capability = await hiAI.getCapability();
    return capability.isAvailable;
  } catch {
    return false;
  }
}
```

### HiAI 模型管理

```typescript
// 加载模型
async function loadModel(modelPath: string): Promise<hiAI.Model> {
  const model = await hiAI.loadModel({
    path: modelPath,
    framework: hiAI.Framework.TENSORFLOW_LITE
  });
  return model;
}

// 执行推理
async function runInference(model: hiAI.Model, input: Tensor): Promise<Tensor> {
  const output = await model.predict(input);
  return output;
}
```

---

## ML Kit 机器学习

### 人脸检测

```typescript
import { faceDetect } from '@kit.MLKit';

// 检测人脸
async function detectFaces(imagePath: string): Promise<faceDetect.Face[]> {
  const detector = faceDetect.createFaceDetector({
    maxFaceNum: 10,
    minFaceSize: 0.1,
    isPose: true,
    isEyeOpen: true,
    isSmile: true
  });

  const faces = await detector.detect(imagePath);
  return faces;
}

// 使用示例
const faces = await detectFaces('path/to/image.jpg');
faces.forEach(face => {
  console.log('Face bounds:', face.bounds);
  console.log('Is smiling:', face.smilingProbability > 0.5);
  console.log('Left eye open:', face.leftEyeOpenProbability > 0.5);
});
```

### 文字识别 (OCR)

```typescript
import { textRecognition } from '@kit.MLKit';

// 识别文字
async function recognizeText(imagePath: string): Promise<string> {
  const recognizer = textRecognition.createTextRecognizer();
  const result = await recognizer.detect(imagePath);
  
  let fullText = '';
  result.blocks.forEach(block => {
    block.lines.forEach(line => {
      fullText += line.text + '\n';
    });
  });
  
  return fullText;
}

// 使用示例
const text = await recognizeText('path/to/document.jpg');
console.log('Recognized text:', text);
```

### 物体检测

```typescript
import { objectDetection } from '@kit.MLKit';

// 检测物体
interface DetectedObject {
  label: string;
  confidence: number;
  bounds: { left: number; top: number; right: number; bottom: number };
}

async function detectObjects(imagePath: string): Promise<DetectedObject[]> {
  const detector = objectDetection.createObjectDetector({
    modelName: 'mobilenet_v2'
  });

  const results = await detector.detect(imagePath);
  return results.map(result => ({
    label: result.label,
    confidence: result.confidence,
    bounds: result.bounds
  }));
}
```

### 图像分类

```typescript
import { imageClassification } from '@kit.MLKit';

// 分类图像
interface ClassificationResult {
  label: string;
  confidence: number;
}

async function classifyImage(imagePath: string): Promise<ClassificationResult[]> {
  const classifier = imageClassification.createClassifier({
    modelName: 'inception_v3',
    topK: 5
  });

  const results = await classifier.classify(imagePath);
  return results.map(result => ({
    label: result.label,
    confidence: result.score
  }));
}
```

### 图像分割

```typescript
import { imageSegmentation } from '@kit.MLKit';

// 图像分割
async function segmentImage(imagePath: string): Promise<ImageSegmentationResult> {
  const segmentor = imageSegmentation.createSegmentor({
    modelName: 'deeplab_v3'
  });

  const result = await segmentor.segment(imagePath);
  return {
    mask: result.mask,           // 分割掩码
    labels: result.labels,       // 语义标签
    confidence: result.confidence
  };
}
```

---

## CV 计算机视觉

### 图像增强

```typescript
import { imageProcessing } from '@kit.HiAIFoundation';

// 图像超分辨率
async function superResolution(imagePath: string, scale: number): Promise<string> {
  const processor = imageProcessing.createSuperResolution({
    scale: scale  // 2x, 4x
  });

  const result = await processor.process(imagePath);
  return result.outputPath;
}

// 图像去噪
async function denoiseImage(imagePath: string): Promise<string> {
  const processor = imageProcessing.createDenoiser();
  const result = await processor.process(imagePath);
  return result.outputPath;
}

// 图像风格迁移
async function styleTransfer(contentPath: string, stylePath: string): Promise<string> {
  const processor = imageProcessing.createStyleTransfer({
    modelPath: 'path/to/style_model'
  });

  const result = await processor.process({
    content: contentPath,
    style: stylePath
  });
  return result.outputPath;
}
```

---

## NLP 自然语言处理

### 文本分析

```typescript
import { nlp } from '@kit.HiAIFoundation';

// 分词
async function tokenize(text: string): Promise<string[]> {
  const tokenizer = nlp.createTokenizer({
    language: 'zh'
  });
  return await tokenizer.tokenize(text);
}

// 词性标注
async function posTagging(text: string): Promise<POSTag[]> {
  const tagger = nlp.createPOSTagger();
  return await tagger.tag(text);
}

// 命名实体识别
async function recognizeEntities(text: string): Promise<Entity[]> {
  const recognizer = nlp.createNERecognizer();
  return await recognizer.recognize(text);
}
```

### 机器翻译

```typescript
import { translation } from '@kit.HiAIFoundation';

// 翻译文本
async function translateText(
  text: string,
  from: string,
  to: string
): Promise<string> {
  const translator = translation.createTranslator({
    sourceLanguage: from,
    targetLanguage: to
  });

  const result = await translator.translate(text);
  return result.translatedText;
}

// 使用示例
const translated = await translateText('Hello, World!', 'en', 'zh');
console.log(translated);  // 你好，世界！
```

### 情感分析

```typescript
import { sentiment } from '@kit.HiAIFoundation';

// 分析情感
interface SentimentResult {
  score: number;      // -1 到 1
  label: string;      // positive, negative, neutral
  confidence: number;
}

async function analyzeSentiment(text: string): Promise<SentimentResult> {
  const analyzer = sentiment.createAnalyzer({
    language: 'zh'
  });

  const result = await analyzer.analyze(text);
  return {
    score: result.score,
    label: result.score > 0.3 ? 'positive' : result.score < -0.3 ? 'negative' : 'neutral',
    confidence: Math.abs(result.score)
  };
}
```

---

## 语音能力

### 语音识别 (ASR)

```typescript
import { speechRecognition } from '@kit.HiAIFoundation';

// 语音识别
class SpeechRecognizer {
  private recognizer: speechRecognition.Recognizer;

  constructor() {
    this.recognizer = speechRecognition.createRecognizer({
      language: 'zh-CN',
      engineType: speechRecognition.EngineType.ONLINE
    });
  }

  // 开始识别
  async start(): Promise<void> {
    await this.recognizer.startListening();
  }

  // 停止识别
  async stop(): Promise<string> {
    const result = await this.recognizer.stopListening();
    return result.text;
  }

  // 取消识别
  async cancel(): Promise<void> {
    await this.recognizer.cancel();
  }

  // 监听结果
  onResult(callback: (text: string) => void): void {
    this.recognizer.on('result', callback);
  }
}

// 使用示例
const recognizer = new SpeechRecognizer();
recognizer.onResult((text) => {
  console.log('Recognized:', text);
});
await recognizer.start();
```

### 语音合成 (TTS)

```typescript
import { textToSpeech } from '@kit.HiAIFoundation';

// 语音合成
class SpeechSynthesizer {
  private synthesizer: textToSpeech.Synthesizer;

  constructor() {
    this.synthesizer = textToSpeech.createSynthesizer({
      language: 'zh-CN',
      voice: 'female',
      speed: 1.0,
      pitch: 1.0
    });
  }

  // 合成并播放
  async speak(text: string): Promise<void> {
    await this.synthesizer.speak(text);
  }

  // 暂停
  async pause(): Promise<void> {
    await this.synthesizer.pause();
  }

  // 恢复
  async resume(): Promise<void> {
    await this.synthesizer.resume();
  }

  // 停止
  async stop(): Promise<void> {
    await this.synthesizer.stop();
  }
}

// 使用示例
const synthesizer = new SpeechSynthesizer();
await synthesizer.speak('你好，欢迎使用鸿蒙系统！');
```

### 声纹识别

```typescript
import { voicePrint } from '@kit.HiAIFoundation';

// 声纹注册
async function enrollVoicePrint(audioPath: string, userId: string): Promise<void> {
  const enroller = voicePrint.createEnroller();
  await enroller.enroll({
    audioPath: audioPath,
    userId: userId
  });
}

// 声纹验证
async function verifyVoicePrint(audioPath: string, userId: string): Promise<boolean> {
  const verifier = voicePrint.createVerifier();
  const result = await verifier.verify({
    audioPath: audioPath,
    userId: userId
  });
  return result.isMatch;
}
```

---

## AI 最佳实践

### 1. 模型优化

```typescript
// 使用量化模型减少内存占用
const model = await hiAI.loadModel({
  path: 'model_quantized.tflite',
  quantization: true
});

// 使用模型缓存
const cachedModel = await hiAI.loadModel({
  path: 'model.tflite',
  cacheEnabled: true
});
```

### 2. 异步推理

```typescript
// 异步执行推理，避免阻塞 UI
async function predictAsync(input: Tensor): Promise<Tensor> {
  return new Promise((resolve) => {
    setTimeout(async () => {
      const result = await model.predict(input);
      resolve(result);
    }, 0);
  });
}
```

### 3. 批量处理

```typescript
// 批量处理提高效率
async function batchPredict(inputs: Tensor[]): Promise<Tensor[]> {
  const batchSize = 8;
  const results: Tensor[] = [];
  
  for (let i = 0; i < inputs.length; i += batchSize) {
    const batch = inputs.slice(i, i + batchSize);
    const batchResults = await model.batchPredict(batch);
    results.push(...batchResults);
  }
  
  return results;
}
```

### 4. 错误处理

```typescript
async function safePredict(input: Tensor): Promise<Tensor | null> {
  try {
    const result = await model.predict(input);
    return result;
  } catch (error) {
    if (error.code === 'MODEL_NOT_LOADED') {
      await reloadModel();
      return await model.predict(input);
    }
    console.error('Prediction failed:', error);
    return null;
  }
}
```

---

## 常见问题

### Q1: AI 模型太大怎么办？

**A:** 使用模型压缩技术：
- 量化（INT8）
- 剪枝
- 知识蒸馏
- 模型分割

### Q2: 如何离线使用 AI？

**A:** 使用本地模型：
- ML Kit 支持离线推理
- 使用轻量级模型
- 预下载模型到本地

### Q3: AI 推理速度慢？

**A:** 优化方法：
- 使用 GPU 加速
- 减少输入尺寸
- 使用优化后的模型
- 批量处理

---

## 📚 延伸阅读

- [HiAI Foundation](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ai-overview)
- [ML Kit](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/ml-kit-overview)
- [语音能力](https://developer.huawei.com/consumer/cn/doc/harmonyos-guides/speech-overview)

---

> 📝 **最后更新**: 2026-05-03
> 📌 **适用版本**: HarmonyOS 6.0 (API 12)
