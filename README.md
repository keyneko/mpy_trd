# 烧录固件
https://micropython.org/download/
```bash
$ wget https://micropython.org/resources/firmware/ESP32_GENERIC_C3-20240222-v1.22.2.bin

# 使用esptool
$ source '/home/keyneko/Documents/GitHub/esp-idf/export.sh'

$ ls /dev/ttyUSB* /dev/ttyS*

$ esptool.py --chip esp32c3 --port /dev/ttyUSB0 erase_flash
$ esptool.py --chip esp32c3 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x0 ESP32_GENERIC_C3-20240222-v1.22.2.bin

$ esptool.py --chip esp32 --port /dev/ttyUSB0 erase_flash
$ esptool.py --chip esp32 --port /dev/ttyUSB0 --baud 460800 write_flash -z 0x1000 ESP32_GENERIC-20240222-v1.22.2.bin
```

# 编译固件
我这里编译的是绑定lvgl的lv_micropython版本，micropython编译也是一样的操作
```bash
$ sudo apt-get install build-essential libreadline-dev libffi-dev git pkg-config libsdl2-2.0-0 libsdl2-dev python3.8 parallel

$ git clone https://github.com/lvgl/lv_micropython.git
$ cd lv_micropython
	$ git checkout release/v8
$ git submodule update --init --recursive lib/lv_bindings
$ make -C mpy-cross

fix: lv_micropython/lib/lv_bindings/lvgl/src/drivers/sdl/lv_sdl_window.c
  SDL_PixelFormatEnum px_format -> Uint32 px_format


# ESP32 port
# ESP-IDF工具链安装（MicroPython最大支持到esp-idf v4.4）
$ git clone -b v4.4 --recursive https://github.com/espressif/esp-idf.git
$ cd esp-idf
$ git checkout v4.4
$ git submodule update --init --recursive
$ ./install.sh 
  # 如果./install.sh报错
  $ sudo rm ~/.espressif/idf-env.json 
  $ ./install.sh 
$ source '/home/keyneko/Documents/GitHub/esp-idf/export.sh'

# 如果是v4.4.7版本, 修改lv_micropython/ports/esp32/network_common.c
	#if ESP_IDF_VERSION >= ESP_IDF_VERSION_VAL(4, 4, 7)
	#define TEST_WIFI_AUTH_MAX 10
	#elif ESP_IDF_VERSION >= ESP_IDF_VERSION_VAL(4, 3, 0)
	#define TEST_WIFI_AUTH_MAX 9
	#else
	#define TEST_WIFI_AUTH_MAX 8
	#endif

$ cd ports/esp32
$ make submodules
$ make LV_CFLAGS="-DLV_COLOR_DEPTH=16 -DLV_COLOR_16_SWAP=1" BOARD=GENERIC
	# 如果编译带PSRAM的ESP32，需要在编译前设置对应的PSRAM大小
	idf.py set-target esp32
	idf.py menuconfig
		CONFIG_ESP32_SPIRAM_SUPPORT=y
	make BOARD=GENERIC_SPIRAM
$ make erase
$ make deploy

# 使用picocom进入REPL
$ sudo apt-get update
$ sudo apt-get install picocom
$ picocom -b 115200 /dev/ttyUSB0


# ESP8266 prot
# 安装esp8266工具链
$ sudo apt-get install libtool-bin
  # 安装可能缺少的依赖包
	$ sudo apt-get install help2man

$ git clone https://github.com/pfalcon/esp-open-sdk.git
$ cd esp-open-sdk
# To build the self-contained, standalone toolchain+SDK
$ make STANDALONE=y
	$ sudo apt-get install python-dev
	# 网络问题，需要手动下载tarballs包
	$ cd crosstool-NG/.build/tarballs
	$ wget https://libisl.sourceforge.io/isl-0.14.tar.xz
	$ wget http://mirror.opencompute.org/onie/crosstool-NG/expat-2.1.0.tar.gz
	$ wget https://wii.leseratte10.de/devkitPro/buildscripts/sources/newlib-2.0.0.tar.gz

$ export PATH=$PATH:/home/keyneko/Documents/GitHub/esp-open-sdk/xtensa-lx106-elf/bin
$ cd ports/esp8266
$ make submodules
$ make 
$ esptool.py --port /dev/ttyXXX erase_flash
$ make deploy
```
