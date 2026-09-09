# 固件烧录工具（Web Flasher）

通用浏览器固件烧录页：**Chrome / Edge 打开即用**（Web Serial 直连设备，无需安装驱动与软件），
可烧录任意 ESP 系列设备（ESP32 / S2 / S3 / C3 等）。

在线地址：https://sunfounder.github.io/web-flasher/

## 使用
1. 用 Chrome 或 Edge 打开本页（必须 https 或 localhost）。
2. 选择固件 → 「连接并刷机」→ 弹窗里选设备串口。
3. 进度到 100% 即烧录完成；若设备没有自动重启，拔下 USB 再插上。

## 如何加入一个新固件
1. 把合并好的镜像（或 bootloader/分区表/应用等分段 bin）放进 `firmwares/` 目录。
   - 最简单：用 esptool 把多段合并成一个整包再从 0x0 烧（例如：`esptool merge_bin -o out.bin 0x0 bootloader.bin 0x8000 partitions.bin 0xe000 boot_app0.bin 0x10000 app.bin`），注意 `--flash_mode/freq/size` 要和目标板一致（本项目板子为 dio/80m/16MB）。
2. 在 `firmwares.json` 的 `firmwares` 数组里加一条记录：

```json
{
  "id": "xxx-v1.2.3",
  "name": "产品名",
  "version": "1.2.3",
  "date": "2026-01-01",
  "target": "ESP32-S3（板子型号，Flash 容量）",
  "note": "备注（出厂/OTA 等）",
  "flashMode": "dio",
  "flashFreq": "80m",
  "flashSize": "16MB",
  "eraseAll": true,
  "segments": [
    { "file": "firmwares/xxx-v1.2.3.bin", "address": 0 }
  ]
}
```

多段烧录时 `segments` 写多条即可（address 是十进制，如 0x10000 写 65536）。

## 内置固件
| 名称 | 版本 | 目标 |
| --- | --- | --- |
| 思天 QC 工具 | 1.0.0 | ESP32-S3 / T-Display-S3 16MB |
