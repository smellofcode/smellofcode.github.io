---
title: Weather Station with ESP 8266 (part 1)
description: Introduction to the battery powered, ESP 8266 IoT project.
layout: post
post-url: weather-station
categories: smarthome iot esp8266 mcu
pic: /sh/img/weather-inside.jpg
---
The ESP 8266 has relatively high power consumption (around 70-80mA) which makes battery powered devices difficult to build, unless we accept a very short battery lifetime. Reasons for this are apparently connected with WiFi communication model, which is not especially economical, big processing power of the ESP chip itself has also an influence on this.

But I really like 8266 because setting up the network topology is trivial - there is nothing to be set as I will use my existing home network. These devices also need to be battery powered, because I don't want to put wires. So I have checked several options and this is what I found. The situation is not that bad as it looks, as there are a few power saving modes of the ESP 8266, from which the so-called deep sleep mode seems to be a very promising. In deep sleep mode the power consumption can be vastly reduced to around 10uA, which is a very small number.

![Weather station interior]({{  "/sh/img/weather-inside.jpg" | absolute_url }})

The weather station seems to be an ideal project for deep sleep mode, because we usually don't need to take measurements all the time, but instead we can wait a certain amount of minutes. The longer the wait interval is, the longer deep sleep mode can be used thus I can lower the overall power consumption.

The ESP 8266 itself operates in 3.0-3.6V area and nominal voltage should be around 3.3V. After some experiments I decided to use a single 18650 Li-ion rechargeable battery, and an LDO voltage stabilizer with limits the voltage of the power source to the safe 3.3V level. It is noteworthy the best 18650 cells are around 3400mAh. If you see much better (as for 2020), i.e. over 10000mAh, there are fake for sure. Just don't buy it.

Originally I have also decided to use a cheap DHT11 sensor, but it seems to be unreliable with humidity measurements, so I reverted to use more expensive BME280. As a bonus I got a pressure meter and much lower idle state power consumption of the sensor.

I describe the project with details on GitHub: [https://github.com/maciejmalecki/weather].

[https://github.com/maciejmalecki/weather]: https://github.com/maciejmalecki/weather

