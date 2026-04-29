# 固件编译与烧录指南

## 修改服务器地址

固件中的接口地址在 `main/Kconfig.projbuild` 第 5 行：

```
config OTA_URL
    string "Default OTA URL"
    default "http://192.168.3.33:8003/xiaozhi/ota/"
```

修改 `default` 后面的地址即可，格式为 `http://<IP>:<PORT>/xiaozhi/ota/`。

修改后需同步更新 `sdkconfig` 中对应的值（第 602 行）：

```
CONFIG_OTA_URL="http://192.168.3.33:8003/xiaozhi/ota/"
```

> 也可以通过 `idf.py menuconfig` → Xiaozhi Assistant → Default OTA URL 在 UI 中修改，修改后 sdkconfig 会自动同步。

## 环境准备

```bash
# 激活 ESP-IDF 环境（每次新开终端都需要执行）
source /Users/lldd/codes/esp/esp-idf/export.sh

# 进入项目目录
cd /Users/lldd/codes/esp/xiaozhi-esp32
```

如果激活报错（如 `No module named 'rich'`），先重新安装依赖：

```bash
cd /Users/lldd/codes/esp/esp-idf
./install.sh esp32s3
source export.sh
```

## 编译

```bash
cd /Users/lldd/codes/esp/xiaozhi-esp32
idf.py build
```

编译成功后输出：`build/xiaozhi.bin`，固件大小约 2.8MB，占 app 分区 69%。

## 烧录

### 方式一：网页烧录（推荐）

先合并所有分区为单文件：

```bash
python -m esptool --chip esp32s3 merge_bin \
  -o build/xiaozhi-merged.bin \
  --flash_mode dio --flash_size 16MB --flash_freq 80m \
  0x0       build/bootloader/bootloader.bin \
  0x8000    build/partition_table/partition-table.bin \
  0xd000    build/ota_data_initial.bin \
  0x20000   build/xiaozhi.bin \
  0x800000  build/generated_assets.bin
```

输出文件：`build/xiaozhi-merged.bin`

在网页烧录工具中选择该文件，烧录地址填 `0x0`。

### 方式二：命令行烧录

```bash
# 替换 /dev/cu.usbserial-xxx 为实际串口
idf.py -p /dev/cu.usbserial-xxx flash
```

或手动指定所有分区：

```bash
python -m esptool --chip esp32s3 -b 460800 \
  --before default_reset --after hard_reset write_flash \
  --flash_mode dio --flash_size 16MB --flash_freq 80m \
  0x0       build/bootloader/bootloader.bin \
  0x8000    build/partition_table/partition-table.bin \
  0xd000    build/ota_data_initial.bin \
  0x20000   build/xiaozhi.bin \
  0x800000  build/generated_assets.bin
```

## 目标芯片

当前配置为 `esp32s3`，如需切换芯片：

```bash
idf.py set-target esp32s3   # 或 esp32, esp32c3, esp32c6 等
idf.py build
```
