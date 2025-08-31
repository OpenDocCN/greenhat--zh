# 第十六章：IoT 黑客工具

![](img/chapterart.png)

本附录列出了流行的 IoT 黑客软件和硬件工具。它包括了本书中讨论的工具，以及一些我们没有涉及但仍然很有用的工具。虽然这不是一个完整的 IoT 黑客工具目录，但它可以作为快速入门的指南。我们按照字母顺序列出了这些工具。为了方便参考，请查看第 414 页的“按章节分类的工具”部分，该部分包含了工具与使用这些工具的章节的对应表。

## Adafruit FT232H Breakout

Adafruit FT232H Breakout 可能是与 I²C、SPI、JTAG 和 UART 接口连接的最小且最便宜的设备。它的主要缺点是没有预先焊接插头。它基于 FT232H 芯片，这是 Attify Badge、Shikra 和 Bus Blaster 使用的芯片（尽管 Bus Blaster 使用的是双通道版本 FT2232H）。你可以在[`www.adafruit.com/product/2264`](https://www.adafruit.com/product/2264)购买它。

## Aircrack-ng

Aircrack-ng 是一套用于 Wi-Fi 安全测试的开源命令行工具。它支持数据包捕获、重放攻击和去认证攻击，还能破解 WEP 和 WPA PSK。我们在第十二章和第十五章中广泛使用了 Aircrack-ng 工具集中的各种程序。你可以在[`www.aircrack-ng.org/`](https://www.aircrack-ng.org/)找到所有工具。

## Alfa Atheros AWUS036NHA

Alfa Atheros AWUS036NHA 是一款无线（802.11 b/g/n）USB 适配器，我们在第十二章中使用它进行 Wi-Fi 攻击。Atheros 芯片组因支持 AP 监控模式和具有数据包注入能力而闻名，这两者在进行大多数 Wi-Fi 攻击时都至关重要。你可以在[`www.alfa.com.tw/products_detail/7.htm`](https://www.alfa.com.tw/products_detail/7.htm)了解更多信息。

## Android 调试桥

Android 调试桥（adb）是一款与 Android 设备进行通信的命令行工具。我们在第十四章中广泛使用它与易受攻击的 Android 应用进行交互。你可以在[`developer.android.com/studio/command-line/adb`](https://developer.android.com/studio/command-line/adb)了解所有关于它的信息。

## Apktool

Apktool 是一款用于静态分析 Android 二进制文件的工具。我们在第十四章展示了如何使用它分析 APK 文件。可以从[`ibotpeaches.github.io/Apktool/`](https://ibotpeaches.github.io/Apktool/)下载它。

## Arduino

Arduino 是一个廉价、易于使用的开源电子平台，允许你使用 Arduino 编程语言来编程微控制器。我们在第七章中使用 Arduino 编写了一个用于黑丸微控制器的易受攻击程序。第八章中使用 Arduino UNO 作为 I²C 总线上的控制器。在第十三章中，我们使用 Arduino 编程 Heltec LoRa 32 开发板作为 LoRa 发送器。Arduino 的官网是[`www.arduino.cc/`](https://www.arduino.cc/)。

## Attify Badge

Attify Badge 是一个可以通过 UART、1-WIRE、JTAG、SPI 和 I²C 进行通信的硬件工具。它支持 3.3V 和 5V 电流。它基于 FT232H 芯片，该芯片用于 Adafruit FT232H Breakout、Shikra 和 Bus Blaster（尽管 Bus Blaster 使用的是双通道版本 FT2232H）。你可以在[`www.attify-store.com/products/attify-badge-uart-jtag-spi-i2c-pre-soldered-headers`](https://www.attify-store.com/products/attify-badge-uart-jtag-spi-i2c-pre-soldered-headers)找到预焊接头的 badge。

## Beagle I2C/SPI 协议分析仪

Beagle I2C/SPI 协议分析仪是一种硬件工具，用于高性能监控 I²C 和 SPI 总线。你可以在[`www.totalphase.com/products/beagle-i2cspi/`](https://www.totalphase.com/products/beagle-i2cspi/)购买它。

## Bettercap

Bettercap 是一个用 Go 语言编写的开源多功能工具。你可以使用它进行 Wi-Fi、BLE 和无线 HID 设备的侦察，以及以太网中间人攻击。我们在第十一章中使用它进行 BLE 破解。你可以在[`www.bettercap.org/`](https://www.bettercap.org/)下载它。

## BinaryCookieReader

BinaryCookieReader 是一个用于解码 iOS 应用程序二进制 Cookie 的工具。我们在第十四章中使用它做了相关操作。可以在[`github.com/as0ler/BinaryCookieReader/`](https://github.com/as0ler/BinaryCookieReader/)找到它。

## Binwalk

Binwalk 是一个用于分析和提取固件的工具。它可以使用自定义签名识别固件镜像中嵌入的文件和代码，这些签名通常用于固件镜像中的文件（如档案、头文件、引导加载程序、Linux 内核和文件系统）。我们在第九章中使用 Binwalk 分析了 Netgear D600 路由器的固件，并在第四章中提取了 IP 摄像头固件的文件系统。你可以在[`github.com/ReFirmLabs/binwalk/`](https://github.com/ReFirmLabs/binwalk/)下载它。

## BladeRF

BladeRF 是一个 SDR 平台，类似于 HackRF One、LimeSDR 和 USRP。它有两个版本，更新且更贵的 bladeRF 2.0 micro 支持更广泛的频率范围，从 47 MHz 到 6 GHz。你可以在[`www.nuand.com/.`](https://www.nuand.com/.)了解更多关于 bladeRF 产品的信息。

## BlinkM LED

BlinkM LED 是一个全彩 RGB LED，可以通过 I²C 进行通信。第八章将 BlinkM LEDs 作为 I²C 总线上的外设使用。你可以在[`www.sparkfun.com/products/8579/`](https://www.sparkfun.com/products/8579/)找到该产品的 datasheet 或从中购买。

## Burp Suite

Burp Suite 是用于 Web 应用程序安全测试的标准工具。它包括一个代理服务器、Web 漏洞扫描器、爬虫和其他高级功能，你还可以通过 Burp 扩展来扩展它的功能。你可以在[`portswigger.net/burp/`](https://portswigger.net/burp/)免费下载社区版。

## Bus Blaster

Bus Blaster 是一款兼容 OpenOCD 的高速 JTAG 调试器，基于双通道 FT2232H 芯片。我们在第七章中使用 Bus Blaster 与 STM32F103 目标设备上的 JTAG 进行接口连接。你可以在[`dangerousprototypes.com/docs/Bus_Blaster`](http://dangerousprototypes.com/docs/Bus_Blaster)下载。

## Bus Pirate

Bus Pirate 是一款开源多功能工具，专用于编程、分析和调试微控制器。它支持多种总线模式，例如 bitbang、SPI、I²C、UART、1-Wire、原始线和通过特殊固件支持的 JTAG。你可以在[`dangerousprototypes.com/docs/Bus_Pirate`](http://dangerousprototypes.com/docs/Bus_Pirate)找到更多信息。

## CatWAN USB Stick

CatWAN USB Stick 是一款开源 USB 启动器，设计用于 LoRa/LoRaWAN 收发器。在第十三章中，我们将它用作嗅探器，捕获 Heltec LoRa 32 和 LoStik 之间的 LoRa 流量。你可以在[`electroniccats.com/store/catwan-usb-stick/`](https://electroniccats.com/store/catwan-usb-stick/)购买。

## ChipWhisperer

ChipWhisperer 项目是一款用于进行侧信道功耗分析和对硬件目标进行故障攻击的工具。它包含开源硬件、固件和软件，提供多种开发板和示例目标设备用于练习。你可以在[`www.newae.com/chipwhisperer/`](https://www.newae.com/chipwhisperer/)购买。

## CircuitPython

CircuitPython 是一种易于使用的开源语言，基于 MicroPython，这是一种优化用于在微控制器上运行的 Python 版本。我们在第十三章中使用 CircuitPython 将 CatWAN USB 启动器编程为 LoRa 嗅探器。它的网站在[`circuitpython.org/.`](https://circuitpython.org/.)

## Clutch

Clutch 是一个用于解密 iOS 设备内存中 IPA 文件的工具。我们在第十四章中简要提到过它。你可以在[`github.com/KJCracks/Clutch/`](https://github.com/KJCracks/Clutch/)下载。

## CubicSDR

CubicSDR 是一款跨平台的 SDR 应用程序。我们在第十五章中使用它将无线电频谱转换成我们可以分析的数字流。你可以在[`github.com/cjcliffe/CubicSDR/`](https://github.com/cjcliffe/CubicSDR/)找到它。

## Dex2jar

Dex2jar 是一款用于将 Android 包中的 DEX 文件转换为更易阅读的 JAR 文件的工具。我们在第十四章中使用它对 APK 文件进行反编译。你可以在[`github.com/pxb1988/dex2jar/`](https://github.com/pxb1988/dex2jar/)下载。

## Drozer

Drozer 是一款 Android 安全测试框架。我们在第十四章中使用它对一个易受攻击的 Android 应用进行了动态分析。你可以在[`github.com/FSecureLABS/drozer/`](https://github.com/FSecureLABS/drozer/)获取它。

## FIRMADYNE

FIRMADYNE 是一款用于模拟和动态分析基于 Linux 的嵌入式固件的工具。我们在第九章中展示了 FIRMADYNE，模拟了 Netgear D600 路由器的固件。你可以在[`github.com/firmadyne/firmadyne/`](https://github.com/firmadyne/firmadyne/)找到 FIRMADYNE 的源代码和文档。

## Firmwalker

Firmwalker 在提取或挂载的固件文件系统中搜索有趣的数据，如密码、加密密钥等。在第九章中，我们展示了如何使用 Firmwalker 进行 Netgear D600 固件分析。你可以在 [`github.com/craigz28/firmwalker/`](https://github.com/craigz28/firmwalker/) 找到它。

## 固件分析和比较工具（FACT）

FACT 是一个用于自动化固件分析过程的工具，能够解包固件文件并在其中搜索敏感信息，如凭证、加密材料等。你可以在 [`github.com/fkie-cad/FACT_core/`](https://github.com/fkie-cad/FACT_core/) 找到它。

## Frida

Frida 是一个动态二进制插桩框架，用于分析正在运行的进程并生成动态钩子。我们在第十四章中使用它来避免 iOS 应用中的越狱检测，并且避免 Android 应用中的 root 检测。我们还在第十五章中使用它来破解控制智能跑步机的按钮。你可以在 [`frida.re/.`](https://frida.re/) 了解更多信息。

## FTDI FT232RL

FTDI FT232RL 是一个 USB 到串行 UART 适配器。我们在第七章中使用它与黑色小药丸微控制器的 UART 端口进行接口连接。我们使用的是 [`www.amazon.com/Adapter-Serial-Converter-Development-Projects/dp/B075N82CDL/`](https://www.amazon.com/Adapter-Serial-Converter-Development-Projects/dp/B075N82CDL/)，但也有更便宜的替代品。

## GATTTool

通用属性配置文件工具（GATTTool）用于发现、读取和写入 BLE 属性。我们在第十一章中广泛使用它来演示各种 BLE 攻击。GATTTool 是 BlueZ 的一部分，你可以在 [`www.bluez.org/.`](http://www.bluez.org/) 找到它。

## GDB

GDB 是一个便携的、成熟的、功能完整的调试器，支持多种编程语言。在第七章中，我们与 OpenOCD 一起使用它通过 SWD 对设备进行攻击。你可以在 [`www.gnu.org/software/gdb/.`](https://www.gnu.org/software/gdb/) 了解更多信息。

## Ghidra

Ghidra 是由美国国家安全局（NSA）开发的一个免费的开源逆向工程工具。它常常与 IDA Pro 相比较，后者是闭源的且价格昂贵，但具有 Ghidra 所不具备的功能。你可以在 [`github.com/NationalSecurityAgency/ghidra/`](https://github.com/NationalSecurityAgency/ghidra/) 下载 Ghidra。

## HackRF One

HackRF One 是一个流行的开源 SDR 硬件平台。它支持从 1 MHz 到 6 GHz 的无线电信号。你可以将其用作独立工具或作为 USB 2.0 外设。类似的工具包括 bladeRF、LimeSDR 和 USRP。HackRF 仅支持半双工通信，而其他工具则支持全双工通信。你可以通过 Great Scott Gadgets 了解更多信息，网址为 [`greatscottgadgets.com/hackrf/one/.`](https://greatscottgadgets.com/hackrf/one/)。

## Hashcat

Hashcat 是一个快速的密码恢复工具，能够利用 CPU 和 GPU 加速破解速度。我们在第十二章中使用它恢复了 WPA2 PSK。它的网站是 [`hashcat.net/hashcat/.`](https://hashcat.net/hashcat/.)

## Hcxdumptool

Hcxdumptool 是一个用于从无线设备捕获数据包的工具。我们在第十二章中使用它捕获 Wi-Fi 流量，然后分析这些数据来通过 PMKID 攻击破解 WPA2 PSK。可以从 [`github.com/ZerBea/hcxdumptool/`](https://github.com/ZerBea/hcxdumptool/) 获取。

## Hcxtools

Hcxtools 是一套用于将捕获的数据包转换为与 Hashcat 或 John the Ripper 等工具兼容的格式的工具套件，用于破解。我们在第十二章中使用它通过 PMKID 攻击破解了 WPA2 PSK。可以从 [`github.com/ZerBea/hcxtools/`](https://github.com/ZerBea/hcxtools/) 获取。

## Heltec LoRa 32

Heltec LoRa 32 是一个低成本的基于 ESP32 的 LoRa 开发板。我们在第十三章中使用它发送 LoRa 无线流量。你可以从 [`heltec.org/project/wifi-lora-32/`](https://heltec.org/project/wifi-lora-32/) 获取。

## Hydrabus

Hydrabus 是另一个开源硬件工具，支持诸如原始线路、I²C、SPI、JTAG、CAN、PIN、NAND Flash 和 SMARTCARD 等模式。它用于调试、分析和攻击通过支持的协议连接的设备。你可以在 [`hydrabus.com/`](https://hydrabus.com/) 找到 Hydrabus。

## IDA Pro

IDA Pro 是最流行的二进制分析和逆向工程反汇编工具。商业版可在 [`www.hex-rays.com/`](http://www.hex-rays.com/) 获取，免费版可以在 [`www.hex-rays.com/products/ida/support/download_freeware.shtml`](http://www.hex-rays.com/products/ida/support/download_freeware.shtml) 下载。作为 IDA Pro 的免费开源替代品，可以看看 Ghidra。

## JADX

JADX 是一个将 DEX 转换为 Java 的反编译工具。它允许你轻松查看来自 Android DEX 和 APK 文件的 Java 源代码。我们在第十四章中简要展示过它。你可以在 [`github.com/skylot/jadx/`](https://github.com/skylot/jadx/) 下载它。

## JTAGulator

JTAGulator 是一个开源硬件工具，帮助识别目标设备上的芯片内调试（OCD）接口，方法是从测试点、通孔或元件焊盘中提取信息。我们在第七章中提到过它。你可以在 [`www.jtagulator.com/`](http://www.jtagulator.com/) 获取更多有关如何使用和购买 JTAGulator 的信息。

## John the Ripper

John the Ripper 是最流行的免费开源跨平台密码破解工具。它支持字典攻击和暴力破解模式，可以破解多种加密的密码格式。我们经常用它来破解 IoT 设备中的 Unix shadow 哈希，如第九章中所示。它的网站是 [`www.openwall.com/john/`](https://www.openwall.com/john/).

## LimeSDR

LimeSDR 是一个低成本的开源软件定义无线电（SDR）平台，能够与 Snappy Ubuntu Core 集成，允许你下载和使用现有的 LimeSDR 应用程序。它的频率范围是 100 kHz 到 3.8 GHz。你可以在 [`www.crowdsupply.com/lime-micro/limesdr/`](https://www.crowdsupply.com/lime-micro/limesdr/) 获取它。

## LLDB

LLDB 是一个现代的开源调试器，是 LLVM 项目的一部分。它专门用于调试 C、Objective-C 和 C++ 程序。我们在第十四章中介绍了它，用于利用 iGoat 移动应用程序。可以在 [`lldb.llvm.org/`](https://lldb.llvm.org/) 找到它。

## LoStik

LoStik 是一个开源的 USB LoRa 设备。我们在第十三章中使用它作为 LoRa 无线电流量的接收器。你可以在 [`ronoth.com/lostik/`](https://ronoth.com/lostik/) 获取它。

## Miranda

Miranda 是一个用于攻击 UPnP 设备的工具。我们在第六章中使用 Miranda 穿透了一个易受攻击的启用 UPnP 的 OpenWrt 路由器的防火墙。Miranda 位于 [`code.google.com/archive/p/mirandaupnptool/`](https://code.google.com/archive/p/mirandaupnptool/)。

## 移动安全框架（MobSF）

MobSF 是一个用于执行移动应用程序二进制文件静态和动态分析的工具。可以在 [`github.com/MobSF/Mobile-Security-Framework-MobSF/`](https://github.com/MobSF/Mobile-Security-Framework-MobSF/) 获取它。

## Ncrack

Ncrack 是一个高速网络认证破解工具，作为 Nmap 套件的一部分进行开发。我们在第四章中详细讨论了 Ncrack，展示了如何为 MQTT 协议编写模块。Ncrack 托管在 [`nmap.org/ncrack/`](https://nmap.org/ncrack/)。

## Nmap

Nmap 可能是最受欢迎的免费开源网络发现和安全审计工具。Nmap 套件包括 Zenmap（Nmap 的图形界面）、Ncat（网络调试工具和现代实现的 netcat）、Nping（一个类似 Hping 的数据包生成工具）、Ndiff（用于比较扫描结果）、Nmap 脚本引擎（NSE；用于通过 Lua 脚本扩展 Nmap）、Npcap（基于 WinPcap/Libpcap 的数据包嗅探库）和 Ncrack（网络认证破解工具）。你可以在 [`nmap.org/`](https://nmap.org/) 找到 Nmap 套件的工具。

## OpenOCD

OpenOCD 是一个免费且开源的工具，旨在通过 JTAG 和 SWD 调试 ARM、MIPS 和 RISC-V 系统。我们在第七章中使用 OpenOCD 通过 SWD 接口与我们的目标设备（黑色小板）进行交互，并在 GDB 的帮助下进行利用。你可以在 [`openocd.org/`](http://openocd.org/) 了解更多信息。

## Otool

Otool 是一个用于 macOS 环境的目标文件显示工具。我们在第十四章中简要使用了它。它是 Xcode 包的一部分，你可以在 [`developer.apple.com/downloads/index.action`](https://developer.apple.com/downloads/index.action) 访问。

## OWASP Zed Attack Proxy

OWASP Zed Attack Proxy (ZAP) 是一个开源的 Web 应用安全扫描器，由 OWASP 社区维护。它是 Burp Suite 的完全免费替代品，尽管它没有那么多高级功能。你可以在 [`www.zaproxy.org/`](https://www.zaproxy.org/) 上找到它。

## Pholus

Pholus 是一款 mDNS 和 DNS-SD 安全评估工具，我们在第六章中演示了它。你可以从 [`github.com/aatlasis/Pholus`](https://github.com/aatlasis/Pholus) 下载它。

## Plutil

Plutil 是一款用于将属性列表（*.plist*）文件从一种格式转换为另一种格式的工具。我们在第十四章中使用它从一个易受攻击的 iOS 应用程序中提取凭证。Plutil 是为 macOS 环境构建的。

## Proxmark3

Proxmark3 是一款通用的 RFID 工具，配备强大的 FPGA 微控制器，能够读取和仿真低频和高频标签。第十章中的 RFID 和 NFC 攻击 heavily 依赖 Proxmark3 的硬件和软件。我们在第十五章中也使用了该工具克隆钥匙锁系统的 RFID 标签。你可以在 [`github.com/Proxmark/proxmark3/wiki/`](https://github.com/Proxmark/proxmark3/wiki/) 上了解更多信息。

## Pupy

Pupy 是一个开源的跨平台后渗透工具，使用 Python 编写。我们在第十五章中使用它在基于 Android 的跑步机上设置远程 shell。你可以在 [`github.com/n1nj4sec/pupy/`](https://github.com/n1nj4sec/pupy/) 上获取它。

## Qark

Qark 是一款用于扫描 Android 应用程序漏洞的工具。我们在第十四章中简要使用了它。你可以在 [`github.com/linkedin/qark/`](https://github.com/linkedin/qark/) 上下载它。

## QEMU

QEMU 是一个开源的硬件虚拟化仿真器，具有完整的系统和用户模式仿真功能。在物联网黑客攻击中，它非常适合仿真固件二进制文件。第九章中讨论的固件分析工具，如 FIRMADYNE，就依赖于 QEMU。其官网地址为 [`www.qemu.org/`](https://www.qemu.org/)。

## Radare2

Radare2 是一个功能齐全的反向工程和二进制分析框架。我们在第十四章中使用它分析了一个 iOS 二进制文件。你可以在 [`rada.re/n/`](https://rada.re/n/) 上找到它。

## Reaver

Reaver 是一款用于暴力破解 WPS PIN 的工具。我们在第十二章中演示了 Reaver。你可以在 [`github.com/t6x/reaver-wps-fork-t6x/`](https://github.com/t6x/reaver-wps-fork-t6x/) 上找到它。

## RfCat

RfCat 是一个开源的无线电狗头固件，允许你使用 Python 控制无线收发器。你可以在 [`github.com/atlas0fd00m/rfcat/`](https://github.com/atlas0fd00m/rfcat/) 上获取它。

## RFQuack

RFQuack 是一款用于射频操作的库固件，支持多种无线电芯片（CC1101、nRF24 和 RFM69HW）。你可以在 [`github.com/trendmicro/RFQuack/`](https://github.com/trendmicro/RFQuack/) 获取它。

## Rpitx

Rpitx 是一款开源软件，你可以使用它将 Raspberry Pi 转换为一个 5 kHz 到 1500 MHz 的射频发射器。我们在第十五章中使用它来干扰无线报警器。你可以从 [`github.com/F5OEO/rpitx/`](https://github.com/F5OEO/rpitx/) 下载它。

## RTL-SDR DVB-T Dongle

RTL-SDR DVB-T Dongle 是一款低成本的 SDR，配备 Realtek RTL2832U 芯片，可以用于接收（但不能发射）无线电信号。我们在第十五章中使用它捕获无线报警器的无线电流，随后进行了干扰。你可以在 [`www.rtl-sdr.com/`](https://www.rtl-sdr.com/) 了解更多关于 RTL-SDR Dongle 的信息。

## RTP Tools

RTP Tools 是一套用于处理 RTP 数据的程序。我们在第十五章中使用它回放通过网络流式传输的 IP 摄像头视频。你可以在 [`github.com/irtlab/rtptools/`](https://github.com/irtlab/rtptools/) 找到它。

## Scapy

Scapy 是最受欢迎的封包构造工具之一。它是用 Python 编写的，能够解码或伪造多种网络协议的数据包。我们在第四章中使用它创建自定义 ICMP 数据包，以帮助进行 VLAN 跳跃攻击。你可以在 [`scapy.net/`](https://scapy.net/) 获取它。

## Shikra

Shikra 是一款硬件黑客工具，声称可以克服 Bus Pirate 的不足，不仅可以进行调试，还能执行诸如位突发或模糊测试等攻击。它支持 JTAG、UART、SPI、I²C 和 GPIO。它基于 FT232H 芯片，后者用于 Attify Badge、Adafruit FT232H Breakout 和 Bus Blaster（Bus Blaster 使用双通道版本的 FT2232H）。你可以在 [`int3.cc/products/the-shikra/`](https://int3.cc/products/the-shikra/) 获取它。

## STM32F103C8T6（黑色药丸）

黑色药丸是一款广受欢迎且价格便宜的微控制器，采用 ARM Cortex-M3 32 位 RISC 核心。我们在第七章中使用黑色药丸作为 JTAG/SWD 漏洞攻击的目标设备。你可以通过多个在线平台购买黑色药丸，包括在 [`www.amazon.com/RobotDyn-STM32F103C8T6-Cortex-M3-Development-bootloader/dp/B077SRGL47`](https://www.amazon.com/RobotDyn-STM32F103C8T6-Cortex-M3-Development-bootloader/dp/B077SRGL47)/ 购买。

## S3Scanner

S3Scanner 是一款用于枚举目标 Amazon S3 存储桶的工具。我们在第九章中使用它来查找 Netgear 的 S3 存储桶。你可以在 [`github.com/sa7mon/S3Scanner/`](https://github.com/sa7mon/S3Scanner/) 下载它。

## Ubertooth One

Ubertooth One 是一款流行的开源硬件和软件工具，用于蓝牙和 BLE 攻击。你可以在 [`greatscottgadgets.com/ubertoothone/`](https://greatscottgadgets.com/ubertoothone/) 上了解更多信息。

## Umap

Umap 是一款通过 WAN 接口远程攻击 UPnP 的工具。我们在第六章中描述并使用了 Umap。你可以从 [`toor.do/umap-0.8.tar.gz`](https://toor.do/umap-0.8.tar.gz) 下载它。

## USRP

USRP 是一系列具有广泛应用的 SDR 平台。你可以在 [`www.ettus.com/`](https://www.ettus.com/) 上了解更多信息。

## VoIP Hopper

VoIP Hopper 是一个开源工具，用于进行 VLAN 跳跃安全测试。VoIP Hopper 可以模拟 Cisco、Avaya、Nortel 和 Alcatel-Lucent 环境中的 VoIP 电话行为。我们在第四章中使用它模拟 Cisco 的 CDP 协议。你可以在[`voiphopper.sourceforge.net/`](http://voiphopper.sourceforge.net/)下载它。

## Wifiphisher

Wifiphisher 是一个用于进行 Wi-Fi 关联攻击的恶意接入点框架。我们在第十二章中使用 Wifiphisher 对 TP Link 接入点和受害者移动设备进行已知信标攻击。你可以在[`github.com/wifiphisher/wifiphisher/`](https://github.com/wifiphisher/wifiphisher/)下载 Wifiphisher。

## Wireshark

Wireshark 是一款开源网络数据包分析工具，也是最受欢迎的免费数据包捕获工具。我们在全书中广泛使用并讨论了 Wireshark。你可以从[`www.wireshark.org/`](https://www.wireshark.org/)下载它。

## Yersinia

Yersinia 是一款开源工具，用于执行第二层攻击。我们在第四章中使用 Yersinia 发送 DTP 数据包并进行交换机欺骗攻击。你可以在[`github.com/tomac/yersinia/`](https://github.com/tomac/yersinia/)找到它。

## 按章节工具

| **章节** | **工具** |
| --- | --- |
| **1: 物联网安全世界** | 无 |
| **2: 威胁建模** | 无 |
| **3: 安全测试方法学** | 无 |
| **4: 网络评估** | Binwalk, Nmap, Ncrack, Scapy, VoIP Hopper, Yersinia |
| **5: 分析网络协议** | Wireshark, Nmap / NSE |
| **6: 利用零配置网络** | Wireshark, Miranda, Umap, Pholus, Python |
| **7: UART、JTAG 和 SWD 利用** | Arduino, GDB, FTDI FT232RL, JTAGulator, OpenOCD, ST-Link v2 编程器, STM32F103C8T6 |
| **8: SPI 和 I²C** | Bus Pirate, Arduino UNO, BlinkM LED |
| **9: 固件破解** | Binwalk, FIRMADYNE, Firmwalker, Hashcat, S3Scanner |
| **10: 短程无线电：滥用 RFID** | Proxmark3 |
| **11: 低功耗蓝牙** | Bettercap, GATTTool, Wireshark, BLE USB 加密狗（如：Ubertooth One） |
| **12: 中程无线电：Wi-Fi 破解** | Aircrack-ng, Alfa Atheros AWUS036NHA, Hashcat, Hcxtools, Hcxdumptool, Reaver, Wifiphisher, |
| **13: 长程无线电：LPWAN** | Arduino, CircuitPython, Heltec LoRa 32, CatWAN USB, LoStik |
| **14: 攻击移动应用** | Adb, Apktool, BinaryCookieReader, Clutch, Dex2jar, Drozer, Frida, JADX, Plutil, Otool, LLDB, Qark, Radare2 |
| **15: 智能家居攻击** | Aircrack-ng, CubicSDR, Frida, Proxmark3, Pupy, Rpitx, RTL-SDR DVB-T, Rtptools |
