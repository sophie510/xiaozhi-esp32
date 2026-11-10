# 产品相关介绍网址

```http
https://e.tb.cn/h.6Gl2LC7rsrswQZp?tk=qFuaV9hzh0k CZ356
```

# 本仓库变体说明（实验分支）

当前 `config.json` 默认构建名为 **`zhengchen-1.54tft-wifi-voice-barge-in-exp`**，是 **语音打断实验固件**，不是稳定回退版。

| 项目 | 值 |
|---|---|
| 板类型 | `zhengchen-1.54tft-wifi`（`CONFIG_BOARD_TYPE_ZHENGCHEN_1_54TFT_WIFI=y`） |
| Flash | 16MB（工程默认）+ DIO + 80MHz；分区 `partitions/v2/16m.csv` |
| 语言 | zh-CN（工程默认） |
| OTA | `https://robot.onelinglabs.com/xiaozhi/ota/` |
| 开机自动聆听 | `CONFIG_AUTO_LISTEN_ON_BOOT=y`（保留） |
| 实验开关 | `CONFIG_USE_SERVER_AEC=y`（上游标注 Unstable） |

## 实验目标（TTS 播放中真人语音打断）

最小可行路径（不改硬件）：

1. Hello 携带 `features.aec=true`
2. 默认监听模式变为 `realtime`
3. 进入 Speaking 时 **不关闭** 语音上行，并把播放包时间戳打到上行 Opus，供服务端做 AEC / barge-in

**启用 `CONFIG_USE_SERVER_AEC` 本身不能保证打断成功。** 服务端至少需要：

- 接受 `features.aec`
- 正确处理 `listen.mode=realtime`
- 在 TTS 期间用上行时间戳消除扬声器回声，并在检测到真人语音时停止 TTS / 转入新一轮识别
- 若服务端忽略 AEC：上行会混入 TTS 回声 → 误识别 / 自说自话风险升高

本板是 `NoAudioCodecSimplex`，**无扬声器参考回路**，设备侧 AEC（`CONFIG_USE_DEVICE_AEC`）不适用，实验不开启。

按键打断与唤醒词路径保留，可作本地兜底。开机一次自动聆听行为保留。

## 采样率说明（与 AEC 时间戳对齐）

- 上行 Opus 固定按 **16000 Hz** 编码（hello `audio_params.sample_rate=16000`）
- 本板扬声器输出 **24000 Hz**；若服务端下行是 16000，设备会重采样到 24000（日志可能出现 sample rate mismatch 警告）
- 服务端 AEC 对齐依赖 **播放包 timestamp**，不是设备回传参考波形；重采样额外延迟可能降低 AEC 质量，属已知限制，本实验不扩大改采样链路

## 烧录（只改 app，保护 NVS）

**禁止** 对已配网设备使用整片 `merged-binary.bin` @ `0x0`（未覆盖区域填 `0xFF` 会清掉 NVS）。

仅写 `ota_0` 应用分区：

```bash
esptool.py --chip esp32s3 -b 460800 write-flash \
  --flash-mode dio --flash-size 16MB --flash-freq 80m \
  0x20000 xiaozhi.bin
```

分区兼容：本实验不改 bootloader / 分区表 / NVS / otadata / phy / assets。

## 回退到上一版「仅自动聆听」应用

上一版（无 SERVER_AEC）feature commit：`42bfa28`  
稳定回退 app（只写 `0x20000`）SHA256：

`c3a7737f9aa2b1a1aba3afde97b8ecbbfada3cfca29d845d79cd795588f7772b`

```bash
esptool.py --chip esp32s3 write-flash \
  --flash-mode dio --flash-size 16MB --flash-freq 80m \
  0x20000 zhengchen-1.54tft-wifi-app-0x20000.bin
```

Mac 上的全片 16MB 原厂备份（含 NVS）请本地保留，**不要上传**。

## 编译

```bash
source /path/to/esp-idf/export.sh   # ESP-IDF v6.1
python3 scripts/build.py zhengchen/1.54tft-wifi --name zhengchen-1.54tft-wifi-voice-barge-in-exp
```
