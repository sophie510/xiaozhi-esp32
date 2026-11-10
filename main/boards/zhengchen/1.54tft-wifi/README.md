# 产品相关介绍网址

```http
https://e.tb.cn/h.6Gl2LC7rsrswQZp?tk=qFuaV9hzh0k CZ356
```

# 本变体行为

- 目标板：`zhengchen-1.54tft-wifi`（`CONFIG_BOARD_TYPE_ZHENGCHEN_1_54TFT_WIFI=y`）
- Flash：16MB，DIO，80MHz；分区表保持 `partitions/v2/16m.csv`
- 语言：zh-CN
- OTA：`https://robot.onelinglabs.com/xiaozhi/ota/`
- **开机自动聆听**：物理开机且联网激活成功后，自动进入聆听（无需先喊「你好小智」）
  - 仅本次开机激活成功触发一次
  - 告别 / 关闭音频通道后回到待命，**不会**强制重新打开会话
  - 保留 Boot 键切换会话、唤醒词唤醒
  - 未配网 / 配网流程不受影响（激活未完成不会自动开会话）
  - 聆听状态仍通过状态栏「聆听中...」等既有 UI 展示
- 不开启不稳定的服务端 AEC（`CONFIG_USE_SERVER_AEC=n`）

# 编译配置命令

**推荐（仓库入口）：**

```bash
source /path/to/esp-idf/export.sh   # ESP-IDF v6.1 优先，最低 v6.0.1
python3 scripts/build.py zhengchen/1.54tft-wifi --name zhengchen-1.54tft-wifi
```

**或 menuconfig：**

```bash
idf.py set-target esp32s3
idf.py menuconfig
```

选择板子：

```
Xiaozhi Assistant -> Board Type -> Zhengchen 1.54-inch TFT Wi-Fi (征辰科技)
```

并启用：

```
Xiaozhi Assistant -> Auto enter listening after boot activation
```

**编译 / 烧录：**

```bash
idf.py build
idf.py -p <PORT> flash monitor
```

**合并固件：**

```bash
idf.py merge-bin
```

合并产物为 `build/merged-binary.bin`（整片镜像，偏移 `0x0`）。
