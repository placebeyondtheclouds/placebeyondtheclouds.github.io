---
layout: post
title:  "4K analog 2-inch 2S cinewhoop build start to finish"
lang: en
tags: [en, 2s, cinewhoop, quad, fpv, diy, 85mm, analog, 4k, 2inch]
category: tutorial
published: true
---


I want to build the smallest quad possible that can carry a 4K camera onboard, have prop guards and have enough thrust to be able to do light freestyle. Here's how I built a cinewhoop using Mobula8 frame and Runcam Thumb 2. I decided to build it on a FC with fast modern MCU from [ArteryTek](https://oscarliang.com/at32-flight-controllers/) and new industry standard gyro [ICM42688P](https://invensense.tdk.com/wp-content/uploads/2022/12/DS-000347-ICM-42688-P-v1.7.pdf). the Runcam camera will act as ~~both the fpv camera and~~ the 4K cine camera. having a live `preview` in the goggles is super convenient for dialing in Runcam's manual exposure settings. also useful for checking if I lost the action camera in a crash or not. **highlights of the build**: live switching between the cameras, Runcam camera recording start/stop from the radio, VTX power level adjustment on a pot, turtle mode without arming, RHCP antenna for VTX, whip-style antenna for RX, low esr capacitor.

## results and updates (mobula8 frame)

- dry weight: 85.2g, with 550mah batteries: 111.4g. flight time 2 minutes (winter).
- there is a significant delay in video from the Runcam camera, which makes it very difficult to fly, so I have to add an fpv camera and a video source switcher to still be able to view the preview of the runcam exposure in the goggles but fly with the regular fpv camera. the switcher is controlled with PWM by remapping LED_STRIP pad resource to a servo.
- FC restarts because of a voltage sag caused by current spike caused by throttle bump or a crash (`crash_recovery = ON`). when FC restarts, camera comes back on but without video feed, last 10-20 seconds of video recording is lost, gyro data is lost completely. powering the camera not from the FCs onboard 5V but from the battery directly through a 5V BEC (to alleviate some load from the FC) did not solve the problem. `set motor_output_limit = 80` ~~seemd to help~~. ~~maybe just need to swap the two A30 for one XT30 and a proper 2S battery~~
- the final part of the video and gyro data is lost if Runcam camera loses power before recording is stopped. if the camera is powered from the FC and the FC reboots, the camera loses power. powering the camera from the battery directly through the UBEC acts as a protection against that.
- ND filters are easy to lose in a crash (lost one ND8, 30元)
- ~~it seems like the Runcam camera reads (sometimes?) date and time from the flight controller~~ no, it takes the time from the QR code
- tried to use 3S with `set motor_output_limit = 60`, burned those the 1103 15000KV motors right after takeoff.
- replaced the motors with **Happymodel EX1103 11000KV 2S**, at `set motor_output_limit = 80` there is no more voltage sag, the problem was with the generic motors not suitable for a quad this heavy (or them being close to 1S rather than 2S) and drawing too much current. the current at hovering dropped from 4.5A@34% to 3A@41% (at `ibata_scale = 400`). `set motor_output_limit = 100` also works without FC restarts.
- got motor desync event at 15 degrees timing, moved ESCs back to 22.5 degrees
- flight time at 96kHz PWM, 22.5 degrees motor timing, 90% motor limit, outside temperature around 0 degrees celsius, two LiHV 550mah 100C A30 batteries in series: around 2 minutes down to 3.3v
- the 5GHz antenna requires putting a heatshrink to the place where the cable is soldered to the antenna itself, otherwise the shielding gradually breaks and loses contact with the antenna

## update 2: transplant

- transplanting everything into a carbon fiber [AstroRC Carbonfly 80 (薯片80) frame](https://astrorc.net/products/astrorc-carbonfly-80-1-8inch-frame-o4-version). I couldn't choose the 2inch frame because the motor mounts are different (4 holes on 9mm radius) and I would like to keep my motors. [AstroRC official frame assembly tutorial](https://www.youtube.com/watch?v=75qipfHU6d8), changing the props to gemfan 45mm-3 because 2023 do not fit into the frame without modifications (that will follow)
- dry weight: 95.6 grams, with batteries: 121.8 grams. flight time 3 to 3.5 minutes (winter).
- looking for the explanation to the extended flight time
  - the 1103 motors with Gemfan 45mm-3 max RPM at full throttle reach 53000rpm, compared to 38000rpm with 2023 props in mobula8 frame
  - **hypothesis**: the motors do not have enough torque to spin the 2023 props (high pitch) at high RPM on a quad this heavy, so they draw more current, draining the battery while not producing enough thrust. and the smaller props, on contrary, are able to spin to higher RPM, producing more thrust at lower current draw. so presumably on a lighter quad like the original analog mobula8 with 1103 motors the 2023 props would work just fine.
- 2023 props test: motors at full throttle rpm reach 40000rpm, overloads the battery on high throttle maneuvers (FC restarts on crash but not in flight). hovering at 3.4A@40%. flight time feels a bit shorter (around 2.5 to 3 minutes)

## update 3

- try gemfan [2023s props](https://www.gemfanhobby.com/2023s-hurricane-pc-3-blade.html) (50.8mm, pitch 2in)

## todo

- [x] tune filters
- [x] tune PIDs
- [ ] adjust the current sensor calibration value

## initial build video


<iframe width="560" height="315" src="https://www.youtube.com/embed/w96JuggRIIU" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## video about transplanting into a carbon fiber frame (and changing props to 45mm-3)

<iframe width="560" height="315" src="https://www.youtube.com/embed/JF7ALfBuW54" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## video about trying the original 2023 props instead of 45mm-3

<iframe width="560" height="315" src="https://www.youtube.com/embed/J0_ydPTKdlM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>

## pictures (mobula8 frame)

| - | - | - |
| ![1](/assets/images/not-mobula8-01.jpg) | ![2](/assets/images/not-mobula8-02.jpg) |![2](/assets/images/not-mobula8-03.jpg) |
| ![4](/assets/images/not-mobula8-04.jpg) | ![5](/assets/images/not-mobula8-05.jpg) |![6](/assets/images/not-mobula8-06.jpg) |
| ![7](/assets/images/not-mobula8-07.jpg) | ![8](/assets/images/not-mobula8-08.jpg) |![9](/assets/images/not-mobula8-09.jpg) |
| ![10](/assets/images/not-mobula8-10.jpg) | ![11](/assets/images/not-mobula8-11.jpg) |![12](/assets/images/not-mobula8-12.jpg) |
| ![13](/assets/images/not-mobula8-13.jpg) | ![14](/assets/images/not-mobula8-14.jpg) |![15](/assets/images/not-mobula8-15.jpg) |
| ![16](/assets/images/not-mobula8-16.jpg) | ![17](/assets/images/not-mobula8-17.jpg) |![18](/assets/images/not-mobula8-18.jpg) |
| ![19](/assets/images/not-mobula8-19.jpg) | ![20](/assets/images/not-mobula8-20.jpg) |![21](/assets/images/not-mobula8-21.jpg) |
| ![22](/assets/images/not-mobula8-22.jpg) | ![23](/assets/images/not-mobula8-23.jpg) |![24](/assets/images/not-mobula8-24.jpg) |
| ![25](/assets/images/not-mobula8-25.jpg) |  ![34j](/assets/images/not-mobula8-34.jpg) |![35](/assets/images/not-mobula8-35.jpg) |
| ![31](/assets/images/not-mobula8-31.png)   | ![27](/assets/images/not-mobula8-27.jpg) |![28](/assets/images/not-mobula8-28.jpg) |
| ![29](/assets/images/not-mobula8-29.png) |   ![30](/assets/images/not-mobula8-30.png) | ![34](/assets/images/not-mobula8-34.png) |
| ![32](/assets/images/not-mobula8-32.png) |   ![33](/assets/images/not-mobula8-33.png) |  |

## pictures (carbonfly80 frame + Gemfan 45mm-3 props)


| - | - | - | - |
| ![36](/assets/images/not-mobula8-36.jpg) | ![37](/assets/images/not-mobula8-37.jpg) | ![38](/assets/images/not-mobula8-38.jpg) |
| ![40](/assets/images/not-mobula8-40.jpg) | ![41](/assets/images/not-mobula8-41.jpg) | ![39](/assets/images/not-mobula8-39.jpg) | 
| ![42](/assets/images/not-mobula8-42.jpg) | ![43](/assets/images/not-mobula8-43.jpg) |  |
| ![44](/assets/images/not-mobula8-44.png) | ![45](/assets/images/not-mobula8-45.png) |    |


## parts list (mobula8 frame)

- ‌衢州市云端智能科技 (Happymodel) [Mobula8 2inch frame](https://www.happymodel.cn/index.php/2023/04/28/mobula8-frame-85mm-brushless-whoop-frame/)
- props 乾丰 (Gemfan) [2023](https://www.gemfanhobby.com/2023-hurricane-pc-3-blade.html) 2 inch (52.17mm), pitch 2.3 inch,  3-blade (1.5mm shaft)
- [Runcam Thumb 2](https://shop.runcam.com/runcam-thumb-2/), IMX586 sensor, gyro, UART control, type C connector
- 3 M1.4x5 or M1.4x7 screws (with coarse thread for plastic). there is [a mount to use with FPV camera](https://www.thingiverse.com/thing:6090638) that would lift the runcam higher above the frame. I used `Removable_V2_BlueJay_Canopy_with_Thumb_Pro.stl` from that thing and the top part of [the original Runcam Thumb 2 mount](https://www.thingiverse.com/thing:6807624). 
- ~~generic 1103 15000KV 2S (3 hole M1.4 base with 6mm distance, same as the betafpv 1102 motors) or better, more expensive motors like `DarwinFPV bling 1103 8000KV`, `Happymodel EX1103 11000KV`, `sparkhobby xspeed 1103 8500KV`~~
- ‌衢州市云端智能科技 (Happymodel) [EX1103 11000KV 2S](https://www.happymodel.cn/index.php/2022/09/05/bassline-spare-part-ex1103-kv11000-brushless-motor/) motors. the blueprint says the motor mounting holes are for M1.4 screws, but the actual size is more like M1.6
- 津航电子 (JHEMCU) [GHF435AIO](https://www.jhemcu.com/e_productshow/?84-JHEMCU-GHF435AIO-2-4S-20A-w-Built-in-24G-ELRSIPEXSMD-84.html) V2 - 25mm mount, 2-4s, 4 UARTS, 20A Bluejay DSHOT600 ESCs, ArteryTek AT32F435: 288MHz core 1MB flash 384KB RAM, BETAFPV 2.4GHz Lite RX (serial) IPEX gen1, 16MB blackbox, ICM-42688 IMU, AT7656E OSD
- 衢州市云端智能科技 (Happymodel) [5.8G Crown LDS antenna RHCP](https://www.happymodel.cn/index.php/2025/08/07/happymodel-5-8g-crown-lds-antenna-rhcp-lhcp-for-micro-fpv-whoops/), [3.5dBi, 5500-6000MHz](https://www.happymodel.cn/wp-content/uploads/2025/08/5.8G-Crown-antenna-RHCP-testing-data.xls.pdf), IPEX gen1
- 化骨龙航模 (HGLRC) [Zeuz nano 350mw VTX](https://hglrc.freshdesk.com/support/solutions/articles/61000307667-zeus-350mw-vtx). [review](https://www.multirotorguide.com/reviews/review-hglrc-zeus-vtx-nano/).
- two A30 female connectors (because I will use pairs of 1S batteries with A30 connectors, connected in series)
- heatshrink, 2mm zip ties, 18AWG wires for the battery lead
- batteries: pairs of 高能 (GNB) 100C 550mAh LiHV 1S A30 (18mmX6mmX66mm) in series
- 飞翼创新 FOXEER FP1112 video source switcher
- 卡德克斯技术 CADDXFPV Ant (f/1.2 lens) camera
- 5V 2A(continuous) 3A(spike) UBEC (input: 6.5-28V) for the Runcam

## additional parts list (carbonfly 80 frame transplant)

- [AstroRC Carbonfly 80 (薯片80) frame](https://astrorc.net/products/astrorc-carbonfly-80-1-8inch-frame-o4-version)
- [乾丰 (gemfan) 45mm-3](https://www.gemfanhobby.com/45mm-pc-3-blade.html) props (1.5 inch pitch, 1.5mm shaft)

## wiring and assembly

- wiring diagrams for the FC are [here](https://jhemcu.work:6/sharing/3c1SjKuS9). 

| wire color | GHF435AIO pad | Runcam Thumb 2 (type C pin) |  FOXEER FP1112 |  Caddxfpv Ant |
|------------|---------------|----------------------------|----------------|-------------|
| purple      | RX3           | TX (A3)                 |                 |           |
| green      | TX3           | RX (A2)                  |                 |           |
| orange      | CAM           |                       | VIDEO OUT         |           |
| green      | LED_STRIP       |                       | PWM              |           |
| yellow      |              |         CVBS (A11)     | VIDEO 2        |           |
| red          |             |                    | cam1 VCC        |     5V      |
| black      |              |                    | cam1 GND        |    GND       |
| yellow      |              |                   | VIDEO 1        |    VIDEO     |
   

| wire color | 2S 5V BEC | Runcam Thumb 2 (type C pin) | battery|
|------------|---------------|----------------|----------------|
| red        | 5V OUT            | 5V (A4 or A9)         |       |
| black      | GND OUT           | GND (A1 or A12)       |       |
| red        | 6.4-24V IN           |               |  +       |
| black      | GND IN           |                   |  -       |


| wire color | GHF435AIO pad | VTX pad |
|------------|---------------|----------------|
| red        | 5V            | 5V             |
| black      | GND           | GND            |
| yellow     | VTX           | video          |
| purple     | TX5           | RX             |

- use rosin-free flux paste and 63% tin soldering thread (melts at around 183 degrees), clean the board with isopropyl alcohol after soldering. apply `P-1025` conformal coating to the FC and VTX boards. add `Kafuter K-705` silicon sealant to the places where wires are soldered to the FC pads, U.FL connectors on FC and VTX. apply blue `Loctite-243` onto last threads of the motor screws.

- UARTs configuration:
```
UART1: SBUS
UART2: onboard ELRS
UART3: Runcam
UART5: VTX (IRC Tramp)
```

- Caddx Ant camera settings (using OSD menu board): AE mode to BLC=3, brightness=35, contrast auto, saturation manual=20


## radio/modes setup

- ADC Filter OFF
- send radio's RTC data to the flight controller to have correct time in blackbox files and on the OSD: go to special functions, add `ON Lua bfbkgd On` and turn the checkmark on
- ch5 inverted, SB - arm
- ch6 inverted, SA - air / acro / angle 
- ch7 - turtle mode (+ blackbox erase)
- ch8 (aux4 is 3 in vtx CLI command), S2 - VTX power control
- ch9 - SW5 toggle (aux5) - switch between the cameras
- ch10, SW6 toggle - Runcam button
- CH11 SD (aux7) beeper
- CH12 SW2 2pos (aux8) blackbox
- add special function `SW6 playtrk vtx`

## ESCs configuration
- [ESC Configurator](https://esc-configurator.com/) or [run it locally]({% post_url 2025-11-23-bf-local %})

- **connect the battery**, reflash bluejay, target `G-H-30`, PWM 96kHz

```
startup boost min 1025, max 1050
rampup x3
motor timing  22.5 degrees
beep strength 80
beacon strength 80
ESC power rating 2S+
force EDT arm OFF
temperature protection Disabled (because it relies on EDT and the EDT is not working correctly here!)
beacon delay 1 min
```

[settings explanation here](https://github.com/bird-sanctuary/bluejay/wiki/Setup), [here](https://github.com/bird-sanctuary/bluejay/wiki/FAQ), and [here](https://oscarliang.com/bluejay-blheli-s/#Best-Settings). I believe I got a [motor desync](https://oscarliang.com/fix-esc-desync/) once at 15 deg timing, so I changed it back to 22.5

## RX

- reflash the RX with the latest ELRS 3.6 `BETAFPV 2.4GHz Lite RX` (Unified_ESP8285_2400_RX). set up model match.

## FC Betaflight configuration

- [connect the FC](https://app.betaflight.com/). can also [run it locally]({% post_url 2025-11-23-bf-local %})
- check that it is working, save the output of `dump` and `diff all showdefaults` to separate files
- `status`: GYRO=ICM42688P, ACC=ICM42688P, BARO=DPS310
- find out the firmware target: `JHEF435`. 
- build and flash betaflight v2025.12, analog OSD, add features: camera control.
  - [can be built locally]({% post_url 2025-11-23-bf-local %}). local build command: `make JHEF435 EXTRA_FLAGS=" -D'RELEASE_NAME=2025.12.0-RC2' -DCLOUD_BUILD -DUSE_ACRO_TRAINER -DUSE_CAMERA_CONTROL -DUSE_DSHOT -DUSE_GPS -DUSE_GPS_PLUS_CODES -DUSE_LED_STRIP -DUSE_OSD -DUSE_OSD_SD -DUSE_PINIO -DUSE_SERIALRX -DUSE_SERIALRX_CRSF -DUSE_TELEMETRY -DUSE_TELEMETRY_CRSF -DUSE_VTX -DUSE_SERVOS" -j`
- calibrate the accelerometer. fly in angle mode and use [the stick commands](https://oscarliang.com/stick-commands/) to adjust trim: disarm, throttle up with yaw in the center and use the right stick to add roll or pitch trim iteratively with test flights until the quad hovers level.
- load elrs 150Hz rate profile
- UARTS:

```
# serial
serial VCP 1 115200 57600 0 115200
serial UART1 0 115200 57600 0 115200
serial UART2 64 115200 57600 0 115200
serial UART3 16384 115200 57600 0 115200
serial UART5 8192 115200 57600 0 115200
```

- [adjust](https://oscarliang.com/current-sensor-calibration/) the current sensor calibration value

```
set ibata_scale = 400
```

- tell the mixer that we are using props out

```
set yaw_motors_reversed = ON
```

- `pid_process_denom = 1` means we run PID loop at the same frequency as the gyro refresh rate (no aliasing). `pidsum_limit_yaw` sets PID authority to 100% on yaw axis.

```
feature -AIRMODE
feature TELEMETRY
feature OSD
beacon RX_LOST
beacon RX_SET
set pidsum_limit_yaw = 1000
set pid_process_denom = 1
```

- `set baro_hardware = DPS310` to skip barometer auto detection

- modes (for my radio setup). IDs are [here](https://github.com/betaflight/betaflight/blob/master/src/main/msp/msp_box.c). add mode `camera power` to AUX6

```
# mode / function ID / channel 0-7 (AUX1-8 in configurator) / range
aux 0 0 0 1900 2100 0 0
aux 1 0 2 1900 2100 0 0
aux 2 1 1 1900 2100 0 0
aux 3 13 6 1900 2100 0 0
aux 4 26 7 1900 2100 0 0
aux 5 28 1 900 1100 0 0
aux 6 31 2 1925 2100 0 0
aux 7 33 5 1900 2100 0 0
aux 8 35 2 1925 2100 0 0
```


- video source switcher control with servo output. the switcher's manual says it uses PWM 3.3V, 50Hz, duty 5-10%

```
resource
resource LED_STRIP 1 none
resource servo 1 B01
feature LED_STRIP
feature SERVO_TILT
feature -CHANNEL_FORWARDING
set servo_pwm_rate = 50
servo 0 1000 2000 1500 100 8
```

this maps LED_STRIP pad resource to the first servo. servo0 is activated with AUX5 (`8` here, count from 1 with 1-4 being the stick axes, 5 being AUX1 and so on). mode value 2000 on the CH10 (toggle on the radio) sets LED_STRIP pad to HIGH (3.3V). this will tell the video channel switcher to switch to VIDEO2 (runcam). the other state is LOW, which means the fpv camera (VIDEO1) will be connected to the FC by default.

-  HGLRC Zeus nano 350mw [vtxtable](https://www.rotorama.cz/cms/assets/docs/d0c22322f24f3bf72e2e66bab648f238/13272-1/zeus-nano-350mw-vtx.json).

```
vtxtable bands 5
vtxtable channels 8
vtxtable band 1 BOSCAM_A A CUSTOM  5865 5845 5825 5805 5785 5765 5745 5725
vtxtable band 2 BOSCAM_B B CUSTOM  5733 5752 5771 5790 5809 5828 5847 5866
vtxtable band 3 FATSHARK F CUSTOM  5740 5760 5780 5800 5820 5840 5860 5880
vtxtable band 4 RACEBAND R CUSTOM  5658 5695 5732 5769 5806 5843 5880 5917
vtxtable band 5 IMD6     I CUSTOM  5362 5399 5436 5473 5510 5547 5584 5621
vtxtable powerlevels 4
vtxtable powervalues 25 100 200 400
vtxtable powerlabels 25 100 200 350
```

- vtx settings

```
set vtx_band = 4
set vtx_channel = 8
set vtx_power = 1
set vtx_low_power_disarm = UNTIL_FIRST_ARM
set vcd_video_system = PAL
```

- in-flight VTX power switching on a pot, used S2 (BF:aux4, radio:ch8). 0 means no change. `<index> <aux_channel> <vtx_band> <vtx_channel> <vtx_power> <start_range> <end_range>`.

```
vtx 0 3 0 0 1 900 1249
vtx 1 3 0 0 2 1250 1499
vtx 2 3 0 0 3 1500 1749
vtx 3 3 0 0 4 1750 2100
```

the radio reporting current VTX power level with audio messages can be set up like this: 

| - | - | - |
| ![29](/assets/images/not-mobula8-29.png) |   ![30](/assets/images/not-mobula8-30.png) | ![34](/assets/images/not-mobula8-34.png) |


- filters for mobula8 frame and gemfan 2023 props. [gyro low pass 2 can be disabled](https://youtu.be/E3s5XYk3M74?si=tRDyE5hmXNsq65gD&t=344) because the PID loop frequency is equal to the gyro update rate (8KHz), there is no antialiasing needed. also set these parameters to be profile-independent (`set simplified_gyro_filter = ON` and `set simplified_dterm_filter = OFF`):

```
set rpm_filter_weights = 100,20,20
set rpm_filter_min_hz = 120
set rpm_filter_fade_range_hz = 0

set simplified_gyro_filter = ON
set gyro_lpf1_static_hz = 0
set gyro_lpf2_static_hz = 0
set gyro_lpf1_dyn_min_hz = 0

set simplified_dterm_filter = OFF
set dterm_lpf1_dyn_min_hz = 82
set dterm_lpf1_dyn_max_hz = 165
set dterm_lpf1_static_hz = 82
set dterm_lpf2_static_hz = 165
```

- changes to the rpm filters for Carbonfly80 frame and gemfan 45mm-3 props

```
set rpm_filter_min_hz = 160
set rpm_filter_fade_range_hz = 40
```

- PIDs. dynamic idle value is based on the prop size/pitch, [here](https://oscarliang.com/how-to-enable-and-configure-betaflight-dynamic-idle/) and [here](https://youtu.be/1oYoVE4xu1U?si=yH7NVtL8CJaB1tvT&t=798)

```
profile 2

# profile 2
set profile_name = mob8
set dterm_lpf1_dyn_min_hz = 82
set dterm_lpf1_dyn_max_hz = 165
set dterm_lpf1_static_hz = 82
set dterm_lpf2_static_hz = 165
set vbat_sag_compensation = 100
set anti_gravity_gain = 40
set crash_dthreshold = 80
set crash_gthreshold = 600
set crash_setpoint_threshold = 500
set crash_recovery_rate = 150
set crash_recovery = ON
set iterm_relax_type = GYRO
set iterm_windup = 85
set pidsum_limit = 1000
set pidsum_limit_yaw = 1000
set throttle_boost = 0
set p_pitch = 56
set i_pitch = 80
set d_pitch = 48
set f_pitch = 89
set i_roll = 63
set d_roll = 35
set f_roll = 71
set i_yaw = 63
set f_yaw = 71
set d_max_roll = 35
set d_max_pitch = 48
set d_max_advance = 0
set launch_control_mode = NORMAL
set thrust_linear = 20
set feedforward_averaging = OFF
set feedforward_smooth_factor = 30
set feedforward_jitter_factor = 9
set dyn_idle_min_rpm = 100
set simplified_i_gain = 80
set simplified_d_gain = 120
set simplified_d_max_gain = 0
set simplified_feedforward_gain = 80
set simplified_pitch_d_gain = 120
set simplified_pitch_pi_gain = 120
set simplified_dterm_filter = OFF
set simplified_dterm_filter_multiplier = 110

profile 3

# profile 3
set profile_name = carb45
set dterm_lpf1_dyn_min_hz = 82
set dterm_lpf1_dyn_max_hz = 165
set dterm_lpf1_static_hz = 82
set dterm_lpf2_static_hz = 165
set vbat_sag_compensation = 100
set anti_gravity_gain = 40
set crash_dthreshold = 80
set crash_gthreshold = 600
set crash_setpoint_threshold = 500
set crash_recovery_rate = 150
set crash_recovery = ON
set iterm_relax_type = GYRO
set iterm_windup = 85
set pidsum_limit = 1000
set pidsum_limit_yaw = 1000
set throttle_boost = 0
set p_pitch = 56
set i_pitch = 80
set d_pitch = 44
set f_pitch = 149
set i_roll = 63
set i_yaw = 63
set d_max_roll = 30
set d_max_pitch = 44
set d_max_advance = 0
set launch_control_mode = NORMAL
set thrust_linear = 20
set feedforward_averaging = OFF
set feedforward_smooth_factor = 30
set feedforward_jitter_factor = 9
set dyn_idle_min_rpm = 100
set simplified_i_gain = 100
set simplified_d_max_gain = 0
set simplified_feedforward_gain = 80
set simplified_pitch_d_gain = 130
set simplified_pitch_pi_gain = 120
set simplified_dterm_filter = OFF
set simplified_dterm_filter_multiplier = 110


```

[use master multiplier](https://www.youtube.com/watch?v=u74tDug6lpc) to adjust overall aggressiveness (sluggish, unpredictable <==> twitchy, motor heat, motor noise).

- rates 

```
rateprofile 0

set rateprofile_name = sasha
set roll_rc_rate = 16
set pitch_rc_rate = 16
set yaw_rc_rate = 16
set roll_srate = 90
set pitch_srate = 90
set yaw_srate = 90

```



- motors (important) and battery:

```
set dshot_bidir = ON
set dshot_bitbang = AUTO
set motor_pwm_protocol = DSHOT600
set motor_poles = 12
set bat_capacity = 550
set vbat_max_cell_voltage = 440
set vbat_full_cell_voltage = 420
set vbat_min_cell_voltage = 320
set vbat_warning_cell_voltage = 340
set beeper_dshot_beacon_tone = 3
set small_angle = 180
set dshot_edt = ON
set force_battery_cell_count = 2
```

**props off**, check that [Extended DSHOT Telemetry](https://github.com/bird-sanctuary/bluejay/wiki/Setup#extended-dshot-telemetry---edt) is working:

```
dshotprog 255 13
motor 255 1060
dshot_telemetry_info
```

can enable temperature protection in the ESCs if the temperature readings are sane (there were none in my case)

- OSD. if BF is set to NTSC and the camera outputs PAL, the osd elements will not be visible

```
set osd_units = METRIC
set osd_warn_bitmask = 270335
set osd_rssi_alarm = 20
set osd_link_quality_alarm = 80
set osd_rssi_dbm_alarm = -90
set osd_rsnr_alarm = 4
set osd_cap_alarm = 550
set osd_alt_alarm = 120
set osd_distance_alarm = 0
set osd_esc_temp_alarm = 0
set osd_esc_rpm_alarm = -1
set osd_esc_current_alarm = -1
set osd_core_temp_alarm = 70
set osd_ah_max_pit = 20
set osd_ah_max_rol = 40
set osd_ah_invert = OFF
set osd_logo_on_arming = OFF
set osd_logo_on_arming_duration = 5
set osd_arming_logo = 0
set osd_tim1 = 2560
set osd_tim2 = 2561
set osd_vbat_pos = 4524
set osd_rssi_pos = 6592
set osd_link_quality_pos = 6629
set osd_link_tx_power_pos = 6597
set osd_rssi_dbm_pos = 6624
set osd_rsnr_pos = 6560
set osd_tim_1_pos = 385
set osd_tim_2_pos = 14839
set osd_remaining_time_estimate_pos = 4536
set osd_flymode_pos = 6643
set osd_anti_gravity_pos = 341
set osd_g_force_pos = 375
set osd_throttle_pos = 6611
set osd_vtx_channel_pos = 4128
set osd_crosshairs_pos = 237
set osd_ah_sbar_pos = 238
set osd_ah_pos = 110
set osd_current_pos = 6635
set osd_mah_drawn_pos = 6616
set osd_wh_drawn_pos = 341
set osd_motor_diag_pos = 341
set osd_craft_name_pos = 394
set osd_pilot_name_pos = 341
set osd_gps_speed_pos = 227
set osd_gps_lon_pos = 17
set osd_gps_lat_pos = 0
set osd_gps_sats_pos = 6528
set osd_home_dir_pos = 4206
set osd_home_dist_pos = 4172
set osd_flight_dist_pos = 407
set osd_compass_bar_pos = 42
set osd_altitude_pos = 16630
set osd_pid_roll_pos = 341
set osd_pid_pitch_pos = 341
set osd_pid_yaw_pos = 341
set osd_debug_pos = 321
set osd_debug2_pos = 352
set osd_power_pos = 341
set osd_pidrate_profile_pos = 13
set osd_warnings_pos = 14729
set osd_avg_cell_voltage_pos = 14796
set osd_pit_ang_pos = 341
set osd_rol_ang_pos = 341
set osd_battery_usage_pos = 393
set osd_disarmed_pos = 14603
set osd_nheading_pos = 397
set osd_up_down_reference_pos = 252
set osd_ready_mode_pos = 14508
set osd_nvario_pos = 278
set osd_esc_tmp_pos = 54
set osd_esc_rpm_pos = 288
set osd_esc_rpm_freq_pos = 341
set osd_rtc_date_time_pos = 0
set osd_adjustment_range_pos = 341
set osd_flip_arrow_pos = 14574
set osd_core_temp_pos = 4150
set osd_log_status_pos = 341
set osd_stick_overlay_left_pos = 258
set osd_stick_overlay_right_pos = 276
set osd_stick_overlay_radio_mode = 2
set osd_rate_profile_name_pos = 20
set osd_pid_profile_name_pos = 13
set osd_profile_name_pos = 341
set osd_rcchannels_pos = 341
set osd_camera_frame_pos = 142
set osd_efficiency_pos = 341
set osd_total_flights_pos = 341
set osd_aux_pos = 341
set osd_sys_goggle_voltage_pos = 341
set osd_sys_vtx_voltage_pos = 341
set osd_sys_bitrate_pos = 341
set osd_sys_delay_pos = 341
set osd_sys_distance_pos = 341
set osd_sys_lq_pos = 341
set osd_sys_goggle_dvr_pos = 341
set osd_sys_vtx_dvr_pos = 341
set osd_sys_warnings_pos = 341
set osd_sys_vtx_temp_pos = 407
set osd_sys_fan_speed_pos = 341
set osd_stat_bitmask = 8521444
set osd_profile = 1
set osd_profile_1_name = -
set osd_profile_2_name = -
set osd_profile_3_name = -
set osd_gps_sats_show_pdop = OFF
set osd_displayport_device = AUTO
set osd_rcchannels = -1,-1,-1,-1
set osd_camera_frame_width = 24
set osd_camera_frame_height = 11
set osd_stat_avg_cell_value = OFF
set osd_framerate_hz = 12
set osd_menu_background = TRANSPARENT
set osd_aux_channel = 1
set osd_aux_scale = 200
set osd_aux_symbol = 65
set osd_craftname_msgs = OFF
```

- blackbox for filter tuning (2kHz). for PID tuning it will be `set blackbox_sample_rate = 1/2` (4kHz). UAVTech's workspace for blackbox viewer is [here](https://theuavtech.com/wp-content/uploads/2024/10/UAVtech-BF-BBE-4.0-Version.json)

```
set blackbox_sample_rate = 1/4

set blackbox_disable_gps = ON
set blackbox_disable_servos = ON
set blackbox_disable_attitude = ON
set blackbox_disable_acc = ON
set blackbox_disable_bat = ON
set blackbox_disable_alt = ON
set blackbox_disable_rssi = ON

set blackbox_disable_pids = OFF
set blackbox_disable_rc = OFF
set blackbox_disable_setpoint = OFF
set blackbox_disable_gyro = OFF
set blackbox_disable_gyrounfilt = OFF
set blackbox_disable_debug = OFF
set blackbox_disable_motors = OFF
set blackbox_disable_rpm = OFF

set blackbox_mode = NORMAL
set blackbox_high_resolution = OFF
set debug_mode = GYRO_RAW
```

I set `debug_mode = BATTERY` midway through the first part of the build to debug the voltage sag problem

```
save
```


## runcam configuration 

- [update the firmware](https://www.runcam.com/download/thumb2/) to `1.2.2`
- QR code: set the camera to power on automatically when it receives power, but manual start of recording. set it to use gyro, no geometry correction, 4:3 4K 30FPS, PAL, daylight white balance, shutter 1/60, ISO manual, saturation 3, contrast 1, sharpness 1. use ISO and ND filters to adjust the exposure. stop recording before cutting the power, otherwise the end of the video file and the gyro data will be lost.
- [gyroflow best practices for older cameras, does not directly apply to thumb2](https://docs.gyroflow.xyz/app/getting-started/supported-cameras/runcam)
- [lens profile](https://github.com/gyroflow/lens_profiles/blob/main/RunCam/Runcam_Thumb2_4by3.json) for Gyroflow

## changes after transplanting into AstroRC Carbonfly 80 frame

- the board is installed upside down in this frame. wrong alignment will trigger runaway protection on takeoff.

```
set align_board_roll = 180
set align_board_yaw = 45
```

- reorder the motors (do not touch the resources), set motors direction (using Betaflight wizard or directly in Bluejay). the motors are in pusher props out configuration

```
set motor_output_reordering = 2,3,0,1,4,5,6,7
```

- 2023 props are too big for this frame, I had to switch to Gemfan 45mm-3. will also try 2023 props with shims between the motors and the frame and longer bolts



## references

- [motor size / motor kv / battery size / prop size lookup table](https://oscarliang.com/table-prop-motor-lipo-weight/)
- https://jhemcu.work:6/sharing/3c1SjKuS9
- https://store-m8o52p.mybigcommerce.com/product_images/img_runcam_thumb2/runcam_thumb2_manual_en.pdf
- https://flying-rabbit-fpv.com/2020/11/07/creating-a-betaflight-target/
- https://github.com/bird-sanctuary/bluejay/wiki/Setup