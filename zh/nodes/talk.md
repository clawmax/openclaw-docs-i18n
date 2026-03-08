title: "OpenClaw AI 的对话模式连续语音对话"
description: "了解如何使用对话模式与AI进行连续语音对话。配置语音指令、管理行为，并设置ElevenLabs TTS以实现低延迟流式播放。"
keywords: ["对话模式", "语音对话", "elevenlabs tts", "连续语音", "语音指令", "openclaw ai", "语音配置", "流式播放"]
---

  媒体与设备

  
# 对话模式

对话模式是一个连续的语音对话循环：

1.  监听语音
2.  将转录文本发送给模型（主会话，chat.send）
3.  等待响应
4.  通过 ElevenLabs 说出响应（流式播放）

## 行为 (macOS)

-   启用对话模式时，**始终显示覆盖层**。
-   **监听 → 思考 → 说话** 的阶段转换。
-   在**短暂停顿**（静音窗口）后，发送当前转录文本。
-   回复会**写入 WebChat**（与键入相同）。
-   **语音打断**（默认开启）：如果助手正在说话时用户开始讲话，则停止播放并记录打断时间戳，用于下一个提示。

## 回复中的语音指令

助手可能会在其回复前添加**单行 JSON** 来控制语音：

```json
{ "voice": "<voice-id>", "once": true }
```

规则：

-   仅第一行非空行有效。
-   未知键被忽略。
-   `once: true` 仅适用于当前回复。
-   没有 `once` 时，该语音将成为对话模式的新默认语音。
-   JSON 行在 TTS 播放前会被剥离。

支持的键：

-   `voice` / `voice_id` / `voiceId`
-   `model` / `model_id` / `modelId`
-   `speed`, `rate` (WPM), `stability`, `similarity`, `style`, `speakerBoost`
-   `seed`, `normalize`, `lang`, `output_format`, `latency_tier`
-   `once`

## 配置 (~/.openclaw/openclaw.json)

```json
{
  talk: {
    voiceId: "elevenlabs_voice_id",
    modelId: "eleven_v3",
    outputFormat: "mp3_44100_128",
    apiKey: "elevenlabs_api_key",
    interruptOnSpeech: true,
  },
}
```

默认值：

-   `interruptOnSpeech`: true
-   `voiceId`: 回退到 `ELEVENLABS_VOICE_ID` / `SAG_VOICE_ID`（或当 API 密钥可用时，使用第一个 ElevenLabs 语音）
-   `modelId`: 未设置时默认为 `eleven_v3`
-   `apiKey`: 回退到 `ELEVENLABS_API_KEY`（或网关 shell 配置文件，如果可用）
-   `outputFormat`: 在 macOS/iOS 上默认为 `pcm_44100`，在 Android 上默认为 `pcm_24000`（设置为 `mp3_*` 以强制使用 MP3 流）

## macOS 用户界面

-   菜单栏切换：**对话**
-   配置选项卡：**对话模式** 组（语音 ID + 打断切换）
-   覆盖层：
    -   **监听中**：云朵随麦克风电平脉动
    -   **思考中**：下沉动画
    -   **说话中**：辐射状圆环
    -   点击云朵：停止说话
    -   点击 X：退出对话模式

## 注意事项

-   需要语音 + 麦克风权限。
-   使用 `chat.send` 针对会话键 `main`。
-   TTS 使用 ElevenLabs 流式 API 和 `ELEVENLABS_API_KEY`，并在 macOS/iOS/Android 上使用增量播放以实现更低延迟。
-   `eleven_v3` 的 `stability` 验证为 `0.0`、`0.5` 或 `1.0`；其他模型接受 `0..1`。
-   `latency_tier` 在设置时验证为 `0..4`。
-   Android 支持 `pcm_16000`、`pcm_22050`、`pcm_24000` 和 `pcm_44100` 输出格式，用于低延迟的 AudioTrack 流式播放。

[摄像头捕获](./camera.md)[语音唤醒](./voicewake.md)