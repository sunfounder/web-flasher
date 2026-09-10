# 固件烧录工具集（Web Flasher）

SunFounder 固件烧录工具集：**首页是一份工具清单**，点进每个工具才是对应的烧录页面。
全部基于浏览器 Web Serial 直连设备，**Chrome / Edge 打开即用**，无需安装驱动与软件，支持 ESP32 / S2 / S3 / C3 等系列。

在线地址：https://sunfounder.github.io/web-flasher/

## 页面结构
```
index.html          工具清单页（读 tools.json 渲染列表，点条目进对应工具）
tools.json          工具清单：{ name, version, target, desc, path }
vendor/             共享库（esptool-js 打包件）
tools/<工具id>/
  index.html        该工具的烧录页面（选版本 → 连接刷机）
  firmwares.json    该工具的固件版本清单
  firmwares/*.bin   固件镜像
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

## 现有工具
| 工具 | 版本 | 目标设备 |
| :--: | :--: | :-- |
| 思天 QC 工具 | 1.0.0 | ESP32-S3（LilyGO T-Display-S3，16MB） |
