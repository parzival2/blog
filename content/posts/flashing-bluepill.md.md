---
author: "Kalyan"
title: "Flashing bluepill using ST-Link board"
date: "2021-03-05"
description: "Flashing bluepill using ST-Link board"
tags: ["st", "bluepill"]
ShowToc: true
---
I want to get into programming [Blue-pill](https://stm32-base.org/boards/STM32F103C8T6-Blue-Pill.html) using [libopencm3](https://libopencm3.org/) and [FreeRTOS](https://www.freertos.org/). To get started I wanted to compile few test programs to get the feel for it. I needed a flasher as the small board doesn't have an inbuilt programmer like [STM32F4Discovery](https://www.st.com/en/evaluation-tools/stm32f4discovery.html)
## ST-Link
 I have ordered a cheap [ST-Link](https://www.st.com/en/development-tools/st-link-v2.html) that I have found online specifically [this](https://de.aliexpress.com/item/32676015777.html). The pinout for this one is
<a data-flickr-embed="true" href="https://www.flickr.com/photos/192459396@N08/51012185817/in/dateposted-public/" title="StLink_Cheap_Pinout"><img src="https://live.staticflickr.com/65535/51012185817_297ab3b9a9_h.jpg" width="480" height="480" alt="StLink_Cheap_Pinout"></a><script async src="//embedr.flickr.com/assets/client-code.js" charset="utf-8"></script>
## Bluepill
The bluepill connections are as follows from the top right
![Bluepill](https://stm32-base.org/assets/img/boards/STM32F103C8T6_Blue_Pill-2.jpg)
| Pin |
|--|
| GND |
| CLK |
| SDIO |
| 3V3 |
So you just have to connect them according to the name of the pin.
```
3V3			->			3V3
SDIO		->			SDIO
CLK			->			CLK
GND			->			GND
```
After they are connected properly, the LED should start blinking. There is already a blinking program on all the boards. 
## Flashing
For flashing you can use either OpenOCD or [stlink-utils](https://github.com/stlink-org/stlink). You can also check the flash size by using `st-info --probe¸ 





 
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTg0MTkyOTg2M119
-->