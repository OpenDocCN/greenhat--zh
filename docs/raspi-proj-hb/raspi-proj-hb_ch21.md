## 第二十二章：Raspberry Pi GPIO 引脚指南

本指南是所有现有 Raspberry Pi 主板的 GPIO 引脚参考。请使用这些表格查找引脚的位置、名称和功能。

以下表格展示了 Raspberry Pi 3 Model B、Raspberry Pi 2 Model B、Raspberry Pi 1 Model A+、Raspberry Pi 1 Model B+、Raspberry Pi Zero 和 Raspberry Pi Zero W 的 GPIO 引脚信息。

| **功能** | **名称** | **编号** | **编号** | **名称** | **功能** |
| --- | --- | --- | --- | --- | --- |
| DC 电源 | 3.3 V | 1 | 2 | 5 V | DC 电源 |
| SDA1, I²C | GPIO 2 | 3 | 4 | 5 V | DC 电源 |
| SCL1, I²C | GPIO 3 | 5 | 6 | GND |  |
| GPIO_GCLK | GPIO 4 | 7 | 8 | GPIO 14 | TXD0 |
|  | GND | 9 | 10 | GPIO 15 | RXD0 |
| GPIO_GEN0 | GPIO 17 | 11 | 12 | GPIO 18 | GPIO_GEN1 |
| GPIO_GEN2 | GPIO 27 | 13 | 14 | GND |  |
| GPIO_GEN3 | GPIO 22 | 15 | 16 | GPIO 23 | GPIO_GEN4 |
| DC 电源 | 3.3 V | 17 | 18 | GPIO 24 | GPIO_GEN5 |
| SPI_MOSI | GPIO 10 | 19 | 20 | GND |  |
| SPI_MISO | GPIO 9 | 21 | 22 | GPIO 25 | GPIO_GEN6 |
| SPI_CLK | GPIO 11 | 23 | 24 | GPIO 8 | SPI_CE0_N |
|  | GND | 25 | 26 | GPIO 7 | SPI_CE1_N |
| I²C ID EEPROM | DNC | 27 | 28 | DNC | I²C ID EEPROM |
|  | GPIO 5 | 29 | 30 | GND |  |
|  | GPIO 6 | 31 | 32 | GPIO 12 |  |
|  | GPIO 13 | 33 | 34 | GND |  |
|  | GPIO 19 | 35 | 36 | GPIO 16 |  |
|  | GPIO 26 | 37 | 38 | GPIO 20 |  |
|  | GND | 39 | 40 | GPIO 21 |  |

Raspberry Pi 1 Model A 和 Raspberry Pi 1 Model B Rev. 2 的引脚布局相同，但只有前 26 个引脚。

Raspberry Pi 1 Model B Rev. 1 是第一块发布的 Raspberry Pi 主板，其引脚布局与其他所有板子不同。这些板子已经不再销售，但如果你恰好拥有一块，这里是它的引脚布局。

| **功能** | **名称** | **编号** | **编号** | **名称** | **功能** |
| --- | --- | --- | --- | --- | --- |
| DC 电源 | 3.3 V | 1 | 2 | 5 V | DC 电源 |
| SDA0, I²C | GPIO 0 | 3 | 4 | 5 V | DC 电源 |
| SCL0, I²C | GPIO 1 | 5 | 6 | GND |  |
| GPIO_GCLK | GPIO 4 | 7 | 8 | GPIO 14 | TXD0 |
|  | GND | 9 | 10 | GPIO 15 | RXD0 |
| GPIO_GEN0 | GPIO 17 | 11 | 12 | GPIO 18 | GPIO_GEN1 |
| GPIO_GEN2 | GPIO 21 | 13 | 14 | GND |  |
| GPIO_GEN3 | GPIO 22 | 15 | 16 | GPIO 23 | GPIO_GEN4 |
| DC 电源 | 3.3 V | 17 | 18 | GPIO 24 | GPIO_GEN5 |
| SPI_MOSI | GPIO 10 | 19 | 20 | GND |  |
| SPI_MISO | GPIO 9 | 21 | 22 | GPIO 25 | GPIO_GEN6 |
| SPI_CLK | GPIO 11 | 23 | 24 | GPIO 8 | SPI_CE0_N |
|  | GND | 25 | 26 | GPIO 7 | SPI_CE1_N |
