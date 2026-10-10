# 固件烧录工具集（Web Flasher）

SunFounder 固件烧录工具集：**首页是一份工具清单**，点进每个工具才是对应的烧录页面。
全部基于浏览器 Web Serial 直连设备，**Chrome / Edge 打开即用**，无需安装驱动与软件。
支持两类目标：**ESP32 系列**（esptool-js，整包镜像）与 **AVR / Arduino UNO**（STK500v1，Intel HEX，直刷主板 bootloader）。

在线地址：https://sunfounder.github.io/web-flasher/

## 页面结构
```
index.html          工具清单页（读 tools.json 渲染列表，点条目进对应工具）
tools.json          工具清单：{ name, version, target, desc, path }
favicon.png         站点图标（SunFounder logo）
vendor/             共享库（esptool-js 打包件；webserial-flasher = AVR/STK500v1）
tools/<工具id>/
  index.html        该工具的烧录页面（选版本 → 连接刷机）
  firmwares.json    该工具的固件版本清单
  firmwares/*.bin   固件镜像（ESP32）
  firmwares/*.hex   固件镜像（AVR）
```

## 使用
1. 用 Chrome 或 Edge 打开 https://sunfounder.github.io/web-flasher/ （Web Serial 必须 https 或 localhost）。
2. 在清单里点进要烧录的工具 → 选择固件版本 → 「连接并刷机」→ 弹窗里选设备串口。
3. 进度到 100% 弹窗提示即完成；若设备没有自动重启，拔下 USB 再插上。

## 新增一个工具
1. 建目录 `tools/<工具id>/`，把 `tools/qc-tool/index.html` 复制过去（只需改标题、说明、返回链接即可，逻辑通用）。
2. 固件镜像放进 `tools/<工具id>/firmwares/`（推荐整包镜像，从 0x0 起烧）。
3. 编辑 `tools/<工具id>/firmwares.json` 登记版本：

```json
{
  "firmwares": [
    {
      "id": "xxx-v1.2.3",
      "version": "1.2.3",
      "date": "2026-01-01",
      "note": "出厂整包镜像",
      "flashMode": "dio",
      "flashFreq": "80m",
      "flashSize": "16MB",
      "eraseAll": true,
      "segments": [
        { "file": "firmwares/xxx-v1.2.3-merged.bin", "address": 0 }
      ]
    }
  ]
}
```
   - `address` 用十进制（0x10000 写 65536）；多段烧录就写多条 segment。
   - `flashMode/freq/size` 必须与目标板一致（本项目 QC 工具板子为 **dio / 80m / 16MB**，用 qio 会黑屏起不来）。

4. 在根目录 `tools.json` 的 `tools` 数组加一条：
```json
{ "id": "xxx", "name": "产品名 · 固件烧录", "version": "1.2.3", "target": "ESP32-S3（板子/Flash）", "desc": "说明", "path": "tools/xxx/" }
```
5. 提交推送，GitHub Pages 自动更新。

## 制作整包镜像（示例）
```
esptool --chip esp32s3 merge_bin --flash_mode dio --flash_freq 80m --flash_size 16MB \
  -o out.bin 0x0 bootloader.bin 0x8000 partitions.bin 0xe000 boot_app0.bin 0x10000 firmware.bin
```

## 多语言（面向客户）

对客户的产品页按 `sunfounder/download` 同一套写法做中英双语，**不引入任何框架**：

- 文案写在 `data-en` / `data-zh` 两个属性上，脚本按语言 `innerHTML` 落地；
- `getLang()`：URL 的 `?lang=zh|en` 优先，没有就 `navigator.language`（`zh*` → 中文，其余 → 英文）；
- 右上角语言按钮只改 URL 的 `lang` 参数并刷新；客户页之间互相跳转时要把 `lang` 带下去；
- `tools.json` 里 `name_en` / `target_en` / `desc_en` 缺省就回落到中文，索引页两种语言都能显示；
- 固件清单里文案字段用 `note_en`/`note_zh`；
- JS 里跑出来的状态文字（"写入固件… 42%"、弹窗）用页面里的 `T(en, zh)` 取，不能只写一份。

## 新增一个 AVR（Arduino UNO）工具

AVR 固件走的是 **STK500v1**（optiboot），不是 esptool，所以用 `vendor/webserial-flasher/`，
不能 import 它的 `index.js`（会连带拉 Node 专用的 `serialport` 裸包名，浏览器直接加载失败），
要从具体模块入口 import：

```js
import { STK500 } from "../../vendor/webserial-flasher/stk500.js";
import { WebSerialTransport } from "../../vendor/webserial-flasher/transport/WebSerialTransport.js";
```

`firmwares.json`（AVR 版）字段：

```json
{
  "board": {
    "name": "Arduino UNO (ATmega328P)",
    "chip": "atmega328p",
    "signature": "1e950f",
    "pageSize": 128,
    "flashSize": 32256,
    "baudRate": 115200,
    "resetDelayMs": 500
  },
  "firmwares": [
    { "id": "xxx-1.0.0", "version": "1.0.0", "date": "2026-01-01",
      "note_en": "For the XYZ app", "note_zh": "配合 XYZ APP 使用",
      "file": "firmwares/xxx-1.0.0.hex" }
  ]
}
```

要点：

- 用 **应用 hex**（`xxx.ino.X.Y.Z.hex`），**不要**用 `with_bootloader.hex`——后面那个是给 ISP 修 bootloader 用的。
- 页面**不做 chip erase**：optiboot 每次 `PROG_PAGE` 都会先擦该页（见 optiboot.c 的 `writebuffer()`），
  官方 avrdude 脚本用的也是 `-D`（不自动整片擦）。整片擦只会多一次风险，没有收益。
- DTR 复位要**自己发**：库的 `WebSerialTransport.setSignals()` 传的是 `{dtr,rts}`，
  而 Web Serial 规范要的是 `{dataTerminalReady,requestToSend}`，交给它等于没复位（页面里 `resetMethod:'none'` + 自己发脉冲）。
- 板上有 Upload/Run 拨动开关的（如 GalaxyRVR 扩展板），页面说明里要写清楚。
- 客户页的固件下拉行**只写版本号和日期**（如 `2.0.0 (2026-09-08)`），不要塞产品代次之类的标签；
  该选哪个版本、连不上、占用串口这些排查一律放页面下方的 FAQ。
- 工具页是**相对独立**的：不放「返回工具列表」链接。

## 串口监视器（tools/serial-monitor/）

不带固件的工具，页面自己就是全部内容（没有 `firmwares.json`，`tools.json` 里也不用写 `version`）。
功能：连串口收发数据、十六进制视图、时间戳、自动滚动、保存日志；波特率默认 **115200**（火星车固件的日志波特率）。

几个实现点：

- 文本视图用 **流式 `TextDecoder`**（`decode(bytes, {stream:true})`），否则多字节字符跨 chunk 会碎；收尾要再 `decode()` 冲一次。
- 原始字节按 `{t, b}` 存着（上限 256 KB，超了从头部丢），切换文本/十六进制视图时**整体重渲染**，两个视图才对得上。
- 十六进制视图用全局列号 `hexCol` 控制每行 16 字节，增量追加和整体重渲染结果一致。
- 默认 `115200`；打开串口会拉动 DTR 让 Arduino 复位，这属于正常现象，页面 FAQ 里写明。

## 现有工具
| 工具 | 版本 | 目标设备 |
| :--: | :--: | :-- |
| 思天 QC 工具 | 1.0.4 | ESP32-S3（LilyGO T-Display-S3，16MB） |
| GalaxyRVR 火星车 | 2.0.2 / 1.1.0 | Arduino UNO（ATmega328P；官方 UNO R3 或 CH340 板）；烧录后自动开串口 115200 |
| 串口监视器 | — | 任意 USB 串口设备（Web Serial） |