---
layout: post
title:  "Re-building freestyle tinywhoop"
lang: en
tags: [en, 1s, tinywhoop, quad, fpv, diy, 80mm, analog, 85mm, 2s ]
category: notes
published: true
---

Originally I was flying Meteor75 Pro, which was gradually rebuilt into this: 2-inch 2s tinywhoop. The following are my notes on the process.

~~I damaged ESC #2 on the 1S Matrix AIO (Meteor75 Pro), either by running a motor with damaged windings or from a voltage spike in a crash. It's overheating, not giving full power to the motor, and the video feed has white washouts during high throttle. So I'm replacing the AIO with JHEMCU G474ELRS and HGLRC Zeuz nano 350mw VTX. ~~

~~Meteor75 frame is scraping the battery and motor screws against the ground, so I am replacing it with a clone of Mobula7 but for 45mm props (80mm base and 47mm ducts instead of 75mm and 43mm respectively). It has 2S battery tray, and with 1S battery the whoop will land on the lower part of the frame without the battery or motor screws touching the ground. Another solution to the problem would be keep the meteor75pro frame and printing [the battery bumper](https://www.thingiverse.com/thing:7056235).~~

~~The battery is mounted using rubber bands with zip ties, occupying the lower part of the 2S holder and pressing the 1S battery to the frame.~~

~~Camera - [Caddx Ant](https://caddxfpv.com/collections/caddxfpv-tiny-camera/products/caddx-ant-analog-camera) with f/1.2 lens or [Caddx Ant Lite](https://caddxfpv.com/products/caddxfpv-ant-lite-4-3-fpvcycle-edition) with f/2.5 lens. [the canopy for the Ant Lite](https://www.thingiverse.com/thing:6201941) is 3D printed, needs to be modified for the regular Ant edition. the motors are 1102 22000kv left from the Meteor75. the props are 乾丰 (gemfan) 45mm-3 (1.5mm shaft). 20AVG battery lead. ~~

~~batteries: batches of 高能 (GNB) 100C 550mAh LiHV 1S A30 and 格氏 (Tattu) 95C 550mAh 1S LiHV (resoldered A30).~~

**Highlights of this build's final configuration**: 

2s, PAL analog, OSD profile change on a pot, VTX power change on a pot, turtle mode without arming, full weather protection, RHCP antenna for VTX, whip-style antenna for RX, buzzer, 220uf 16v cap. VTX is set to whatever power setting S2 pot at the moment the RX connects to the radio, but VTX keeps low power before the first arm. Crash recovery enabled. 


> read the updates! the quad ended up being very different 
{: .prompt-warning }



## update 1

the ELRS receiver on the AIO died after a week or so. ~~Update: using another VTX (ZENCHANSI 棕熊 W007 400mw) temporarily~~ dry weight before: 39.8 grams.

## update 2

- with a dedicated VTX and added ELRS receiver the quad became too heavy and feels very underpowered now. so I decided to swap it into Mobula8 2 inch frame (85 mm motorpost to motorpost), swap motors to 1103 11000KV, props gemfan 2023 or 2023s. resolder xt30 to the FC and use a proper 2s battery ~~GNB 550mAh 2s 100C HV XT30~~ 高能 (GNB) 70C 350mAh LiHV 2S XT30
- dry weight: 50.8 g, with two 1s batteries: 68.7 g


## update 3

- there is this problem when the quad hangs in the air with zero throttle, the `washout` problem with ducted frames described [here](https://www.youtube.com/watch?v=7GweG0RnCfc). changed props out to props in, PID profile mode was set to `RP`, bumped P and I from 90 to 100 and FF to 150. problem solved

## update 4

- changed to a bit smaller and lighter gemfan 2023s-3 props. dry weight: 50.3 g
- hover at 3.15A 30% throttle. 
- flight time to 3.4v is around 3-5 minutes at 13 degrees ambient, max current 24A

## update 5

- the VTX was placed between the battery and the FC and the video feed had white washout pulsations. I moved the VTX to the place under the canopy behind the camera, shortened the VTX and camera wires, moved the VTX GND to the pad near the camera GND, placed the RX below the FC where the VTX was before. now there are no washouts.

## update 6

- try Gemfan D2.2-3, 1.5inch, for ducted quads

## components

- 津航电子 (JHEMCU) G474ELRS - 1s-2s, 4 UARTS, 12A bluejay DSHOT300 ESCs, STM32G474: 170MHz core 512KB flash 128KB RAM, 8MB blackbox, BETAFPV 2.4GHz Lite RX (serial) IPEX gen1, no baro - ELRS died
- 衢州市云端智能科技 (Happymodel) [5.8G Crown LDS antenna RHCP](https://www.happymodel.cn/index.php/2025/08/07/happymodel-5-8g-crown-lds-antenna-rhcp-lhcp-for-micro-fpv-whoops/), [3.5dBi, 5500-6000MHz](https://www.happymodel.cn/wp-content/uploads/2025/08/5.8G-Crown-antenna-RHCP-testing-data.xls.pdf), IPEX gen1
- 卡德克斯技术 (Caddx) Ant lite (f/2.5 lens). Update: swapped to Caddx Ant (f/1.2 lens), not in the pictures
- ~~Mobula7 frame (80mm clone) - 45mm props, 80mm base, 47mm ducts~~
- ~~ZENCHANSI 棕熊 W007 400mw vtx~~
- 化骨龙航模 (HGLRC) [Zeuz nano 350mw VTX](https://hglrc.freshdesk.com/support/solutions/articles/61000307667-zeus-350mw-vtx). [review](https://www.multirotorguide.com/reviews/review-hglrc-zeus-vtx-nano/).
- ~~哈鸣科技 (BETAFPV) 1102 22000kv motors (left from Meteor75 Pro)~~
- ~~乾丰 (gemfan) 45mm-3 props (1.5mm shaft)~~
- 220uF 16v capacitor
- ~~20AWG wires for the battery lead, A30 connector~~
- added: cyclone EP1 ELRS 2.4ghz nano rx (IPEX gen1), whip antenna 

## components (mobula8 85mm frame)

- ‌衢州市云端智能科技 (Happymodel) [Mobula8 2inch frame](https://www.happymodel.cn/index.php/2023/04/28/mobula8-frame-85mm-brushless-whoop-frame/) (85mm base, 51mm props, 3 or 4 hole motor mounts)
- ~~props 乾丰 (Gemfan) [2023](https://www.gemfanhobby.com/2023-hurricane-pc-3-blade.html) 2 inch (52.17mm), pitch 2.3 inch, 3-blade (M2x6 screws, 1.5mm shaft)~~
- ‌衢州市云端智能科技 (Happymodel) [EX1103 11000KV 2S](https://www.happymodel.cn/index.php/2022/09/05/bassline-spare-part-ex1103-kv11000-brushless-motor/) motors. the blueprint says the motor mounting holes are for M1.4 screws, but the actual size is M1.6x4
- ~~高能 (GNB) 100C 550mAh LiHV 2S XT30, 18mmX12mmX69mm (GNB5502S100AHV), 29g~~
- 高能 (GNB) 70C 350mAh LiHV 2S XT30, 16mmX12mmX49mm (GNB3502S70AHV), 18g
- 18AWG leads, XT30 male connector
- 乾丰 (Gemfan) [2023S props](https://www.gemfanhobby.com/2023s-hurricane-pc-3-blade.html) (50.8mm, pitch 2inch, 3-blade, 1.5mm shaft only)

## pictures


| - | - | - |
| ![1](/assets/images/rebuild01.jpg) | ![2](/assets/images/rebuild02.jpg) |![3](/assets/images/rebuild03.jpg) |
|  ![4](/assets/images/rebuild04.jpg) |![5](/assets/images/rebuild05.jpg) |  ![6](/assets/images/rebuild06.jpg) |
| ![7](/assets/images/rebuild07.jpg) | ![8](/assets/images/rebuild08.jpg) | ![9](/assets/images/rebuild09.jpg) |
| ![10](/assets/images/rebuild10.jpg) |![11](/assets/images/rebuild11.jpg) | ![12](/assets/images/rebuild12.jpg) |
| ![13](/assets/images/rebuild13.jpg) | ![14](/assets/images/rebuild14.jpg) |![15](/assets/images/rebuild15.jpg) |
| ![29](/assets/images/not-mobula8-29.png) |   ![30](/assets/images/not-mobula8-30.png) | ![34](/assets/images/not-mobula8-34.png) |


## mobula 8 85mm transplant

| - | - | - |
| ![1](/assets/images/rebuild16.jpg) | ![2](/assets/images/rebuild17.jpg) |![3](/assets/images/rebuild18.jpg) |
| ![4](/assets/images/rebuild19.jpg) | ![5](/assets/images/rebuild20.png) | ![6](/assets/images/rebuild21.png) |
| ![7](/assets/images/rebuild22.jpg) | ![8](/assets/images/rebuild23.jpg) | ![9](/assets/images/rebuild24.jpg) |
| ![10](/assets/images/rebuild-25.png) |   ![11](/assets/images/rebuild-26.png) | - |


## the process

- wiring diagrams for the FC are [here](https://jhemcu.work:6/sharing/3c1SjKuS9)
- connect the FC, check that it is working, backup the config (`dump`, `diff all showdefaults` and save the outputs into separate files)

- UARTs:
```
UART1: SBUS
UART2: VTX (IRC Tramp)
UART3: onboard ELRS (died), the pads are too tiny to work with
UART4: external ELRS RX
```

- wiring:


| VTX pads | FC pads | wire color |
|----------|---------|------------|
| 5V       | 5V      | red        |
| GND      | GND     | black      |
| video   | VTX     | yellow     |
| RX       | TX2     | purple     |

| Caddx Ant | FC pads         | wire color |
|-----------|-----------------|------------|
| 5-25V     | 5V              | red        |
| GND       | GND             | black      |
| VIDEO     | CAM             | yellow     |


| ELRS RX | FC pads | wire color |
|---------|---------|------------|
| 5V     | 5V      | red        |
| GND     | GND     | black      |
| TX      | RX4     | yellow      |
| RX      | TX4     | white      |

- install the components into the frame, secure what needed with zip ties

- Caddx Ant camera settings (using OSD menu board): AE mode to BLC=3, brightness=35, contrast auto, saturation manual=20

- flash betaflight v2025.12, target `JHEG474`, analog OSD. restore the original backup
  - [can be built locally]({% post_url 2025-11-23-bf-local %}). local build command: `make JHEG474 EXTRA_FLAGS=" -D'RELEASE_NAME=2025.12.2' -DCLOUD_BUILD -DUSE_ACRO_TRAINER -DUSE_DSHOT -DUSE_LED_STRIP -DUSE_OSD -DUSE_OSD_SD -DUSE_PINIO -DUSE_SERIALRX -DUSE_SERIALRX_CRSF -DUSE_TELEMETRY -DUSE_TELEMETRY_CRSF -DUSE_VTX" -j`

- reflash bluejay, target `Z-H-30`, PWM 48kHz (I get runaways on 96kHz for some reason)
```
startup sliders to the max
rampup x3
motor timing  22.5 degrees
ESC power rating 2S+
temperature protection Disabled
beacon delay 1 min
```

- flash the onboard ELRS receiver (target `BETAFPV 2.4GHz Lite RX`)

- ~~(optional) connect to the FC using elrs wifi. in BF configurator options enable manual connection, connect to the RX wifi, then use port `tcp://10.0.0.1`~~
 
- calibrate accelerometer

- [adjust](https://oscarliang.com/current-sensor-calibration/) current sensor calibration values
```
set ibata_scale = 487
```

- in motors tab, turn off _motor direction is reversed_ for the **props in** configuration. it makes the mixer to expect that motors 1 and 2 are not reversed (spinning counterclockwise). 
```
set yaw_motors_reversed = OFF
```

use `motor direction` to adjust motors rotation direction individually. this changes settings of the ESCs

- (not using) [overclock](https://oscarliang.com/overclock-fc-betaflight/) the MCU from 170 MHz to 192MHz

```
set cpu_overclock = 192MHZ
```

## radio setup

- ADC Filter OFF
- send radio's RTC data to the flight controller to have correct time in blackbox files and on the OSD: go to special functions, add `ON Lua bfbkgd On` and turn the checkmark on
- ch5 inverted - arm
- ch6 inverted, SA - air / acro / angle
- ch7 - turtle mode
- ch8 (aux4), S2 - VTX power switching
- ch9 - SD - buzzer
- ch10 - aux6 
- CH11 (AUX7), S1 - OSD profile switching
- ch12 - aux8 
- add logical switch `L04 a>x tpwr 0mw`, add special function `L04 playval tpwr - enable`
- add logical switch `L05 a<x rxbt 3.4V AND SB1up delay 1`, add special function `L05 playval rxbt 5 enable`


set elrs 150Hz, telemetry std, switch mode: wide, link mode: normal, model match: **ID:3**, tx power dyn 500mw

## restore my settings

- load elrs 150Hz rate profile (although some of the values will be replaced with the filters tuning later). 

- UARTs:

```
serial UART2 8192 115200 57600 0 115200
serial UART4 64 115200 57600 0 115200
```

- [camera control](https://oscarliang.com/fpv-camera-control-fc/). measure the OSD pin voltage. set mode `camera control 1` to a channel. enable led_strip feature. flip the channel, activate using `throttle=0, yaw=100`

> not working 
{: .prompt-warning }

```
resource
resource LED_STRIP 1 none
resource camera_control 1 B02
set camera_control_ref_voltage = 330
```

- misc. the `beacon` settings saved me once, the buzzer wire got disconnected in a crash but I didn't notice. later the quad dived into a deep bush and I could find it only thanks to the faint beeping sound from the motors.

```
feature -AIRMODE
feature TELEMETRY
feature -LED_STRIP
feature OSD
beeper -ON_USB
beacon RX_LOST
beacon RX_SET
set baro_hardware = NONE
```

- modes (for my radio setup)

```
# aux
aux 0 0 0 1900 2100 0 0
aux 1 0 2 1900 2100 0 0
aux 2 1 1 1900 2100 0 0
aux 3 13 4 1900 2100 0 0
aux 4 28 1 900 1100 0 0
aux 5 35 2 1925 2100 0 0
```



- ZENCHANSI 棕熊 W007 400mw vtx. 

> not using this VTX, keeping this part for a reference
{: .prompt-warning }

it uses smartaudio 2.1 protocol, so the power values will be in dBm. BF CLI has the command `vtx_info` to show the power levels that the VTX supports

```
# vtx_info
# level 23 dBm, power 200 mW
# level 26 dBm, power 400 mW
# level 0 dBm, power 1 mW
```

this is the table that I made based off the pictures in the listing, the TBS webpage and the output of `vtx_info`. 

```
vtxtable bands 6
vtxtable channels 8
vtxtable band 1 BOSCAM_A A FACTORY 5865 5845 5825 5805 5785 5765 5745 5725
vtxtable band 2 BOSCAM_B B FACTORY 5733 5752 5771 5790 5809 5828 5843 5866
vtxtable band 3 BOSCAM_E E FACTORY 5705 5685 5665 5645 5885 5905 5925 5945
vtxtable band 4 FATSHARK F FACTORY 5740 5760 5780 5800 5820 5840 5860 5880
vtxtable band 5 RACE_LOW L FACTORY 5362 5399 5436 5473 5510 5543 5584 5621
vtxtable band 6 RACEBAND R FACTORY 5658 5695 5732 5769 5806 5843 5880 5917
vtxtable powerlevels 3
vtxtable powervalues 14 23 26
vtxtable powerlabels 25 200 400
```


the following official vtxtable which their tech support sent to me is **incorrect**:

```
vtxtable bands 5
vtxtable channels 8
vtxtable band 1 BOSCAM_A A FACTORY 5865 5845 5825 5805 5785 5765 5745 5725
vtxtable band 2 BOSCAM_B B FACTORY 5733 5752 5771 5790 5809 5828 5843 5866
vtxtable band 3 BOSCAM_E C FACTORY 5705 5685 5665 5645 5885 5905 5925 5945
vtxtable band 4 FATSHARK D FACTORY 5740 5760 5780 5800 5820 5840 5860 5880
vtxtable band 5 RACEBAND E FACTORY 5658 5695 5732 5769 5806 5843 5880 5917
vtxtable powerlevels 3
vtxtable powervalues 14 23 26
vtxtable powerlabels 25 200 400
```

-  HGLRC Zeus nano 350mw [vtxtable](https://www.rotorama.cz/cms/assets/docs/d0c22322f24f3bf72e2e66bab648f238/13272-1/zeus-nano-350mw-vtx.json). [review.](https://www.multirotorguide.com/reviews/review-hglrc-zeus-vtx-nano/)

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


four levels:
```
vtx 0 3 0 0 1 900 1249
vtx 1 3 0 0 2 1250 1499
vtx 2 3 0 0 3 1500 1749
vtx 3 3 0 0 4 1750 2100
```

three levels:
```
vtx 0 3 0 0 1 900 1249
vtx 1 3 0 0 2 1250 1499
vtx 2 3 0 0 3 1500 1749
vtx 3 3 0 0 3 1750 2100
```

the radio reporting current VTX power level with audio messages can be set up like this: 

| - | - | - |
| ![29](/assets/images/not-mobula8-29.png) |   ![30](/assets/images/not-mobula8-30.png) | ![34](/assets/images/not-mobula8-34.png) |



- rates 

```
rateprofile 0

set rateprofile_name = sasha
rates_type = ACTUAL
set roll_rc_rate = 16
set pitch_rc_rate = 16
set yaw_rc_rate = 16
set roll_srate = 90
set pitch_srate = 90
set yaw_srate = 90
```

- [filters and PID tuning]({% post_url 2026-05-10-filters-pid-tuning %})

- filters:

filters are super important. [tuning for performance](https://www.youtube.com/watch?v=TiwaQEkOdyo). `set dyn_notch_count = 1` gives me a flyaway in air mode. because 1 notch is not enough to filter the frame resonance, unfiltered gyro signal in combination of full control autority over the PID loop in air mode causes PID rampup. this build needs `3` notches

```

# master
set gyro_lpf1_static_hz = 0
set dyn_notch_count = 4
set dyn_notch_q = 300
set dyn_notch_min_hz = 110
set gyro_lpf1_dyn_min_hz = 0
set acc_trim_pitch = 1
set acc_trim_roll = 5
set acc_calibration = -17,-45,37,1

set rpm_filter_weights = 100,80,40
set rpm_filter_q = 400
set rpm_filter_min_hz = 180
set rpm_filter_fade_range_hz = 0
```

- PIDs tuned with PIDtoolbox using [this method](https://www.youtube.com/watch?v=ehvQm8Rqrzk). set `pidsum_limit` before tuning. also about [tuning](https://oscarliang.com/fpv-drone-tuning/) and [about pids](https://oscarliang.com/pid/). [about](https://www.youtube.com/watch?v=7GweG0RnCfc) the `washout` problem with ducted frames


```
profile 3

# profile 3
set profile_name = tune
set dterm_lpf1_dyn_min_hz = 82
set dterm_lpf1_dyn_max_hz = 165
set dterm_lpf1_static_hz = 82
set dterm_lpf2_static_hz = 165
set vbat_sag_compensation = 100
set iterm_relax_type = GYRO
set p_pitch = 56
set i_pitch = 100
set d_pitch = 57
set f_pitch = 119
set p_roll = 53
set i_roll = 95
set d_roll = 50
set f_roll = 115
set p_yaw = 100
set i_yaw = 100
set f_yaw = 143
set d_max_roll = 50
set d_max_pitch = 57
set thrust_linear = 20
set feedforward_averaging = OFF
set feedforward_smooth_factor = 30
set feedforward_jitter_factor = 9
set dyn_idle_min_rpm = 80
set simplified_pids_mode = RP
set simplified_master_multiplier = 120
set simplified_d_gain = 140
set simplified_d_max_gain = 0
set simplified_feedforward_gain = 120
set simplified_dterm_filter = OFF
set simplified_dterm_filter_multiplier = 110
```

- motors (important) and battery:

```
set dshot_bidir = ON
set dshot_bitbang = AUTO
set motor_pwm_protocol = DSHOT300
set motor_poles = 12
set bat_capacity = 350
set vbat_max_cell_voltage = 435
set vbat_full_cell_voltage = 420
set vbat_min_cell_voltage = 320
set vbat_warning_cell_voltage = 340
set beeper_dshot_beacon_tone = 3
set small_angle = 180
set force_battery_cell_count = 2


```

- OSD. if BF is set to NTSC and the camera outputs PAL, the osd elements will not be visible

```
set osd_warn_bitmask = 270335
set osd_rssi_dbm_alarm = -90
set osd_cap_alarm = 350
set osd_alt_alarm = 120
set osd_vbat_pos = 6611
set osd_rssi_pos = 6592
set osd_link_quality_pos = 6629
set osd_link_tx_power_pos = 6597
set osd_rssi_dbm_pos = 6624
set osd_rsnr_pos = 448
set osd_tim_1_pos = 385
set osd_tim_2_pos = 14839
set osd_remaining_time_estimate_pos = 6584
set osd_flymode_pos = 6643
set osd_anti_gravity_pos = 341
set osd_g_force_pos = 375
set osd_throttle_pos = 6579
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
set osd_gps_sats_pos = 416
set osd_home_dir_pos = 110
set osd_home_dist_pos = 76
set osd_flight_dist_pos = 407
set osd_compass_bar_pos = 42
set osd_altitude_pos = 247
set osd_pid_roll_pos = 341
set osd_pid_pitch_pos = 341
set osd_pid_yaw_pos = 341
set osd_debug_pos = 385
set osd_power_pos = 341
set osd_pidrate_profile_pos = 341
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
set osd_rate_profile_name_pos = 20
set osd_pid_profile_name_pos = 374
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
set osd_profile = 3
```

- in-flight OSD profile switching on a pot. I use S1, set it to CH11 (AUX7 in BF)

```
adjrange 0 0 6 900 2100 29 6 0 0
```

- `set crash_recovery = OFF`. `crash_recovery = ON` increases the risk of burning an ESC.


- blackbox for filter tuning

[debug modes](https://betaflight.com/docs/wiki/guides/current/Debug-Modes). `GYRO_SAMPLE` is useful for tuning filters as it allows to observe filtering at different stages and see the effects of different groups of filters. `AC_ERROR` shows Absolute Control Error by each axis in debug1. `FFT_FREQ` shows center frequency for the 3 dynamic notches.
```
set blackbox_sample_rate = 1/2
set blackbox_disable_bat = ON
set blackbox_disable_alt = ON
set blackbox_disable_rssi = ON
set blackbox_disable_acc = ON
set blackbox_disable_attitude = ON
set blackbox_disable_debug = ON
```

## testing

- _after_ successful test flights, apply `P-1025` conformal coating to the FC and VTX boards (I already applied it to the camera board), add ~~`Kafuter K-705` silicon sealant~~ `B-7000` to the places where wires are soldered to the FC pads, U.FL connectors on FC and VTX. apply blue `Loctite-243` onto last threads of the motor screws.

## Update on the AIO's RX failure

after a week or so flying, the RSSI suddenly got way too low, like there is no antenna connected. the flight before it was normal, I changed the battery for a fresh one and RSSI was low. I removed the ELRS antenna's IPEX connector from the AIO board and soldered the antenna directly, but it didn't help. reflashing also did nothing. I don't know if it's a result of a crash or a faulty AIO board. now I need to disable the integrated RX by shorting the two pads located between the battery's negative pad and the ELRS WiFi antenna, then connect an external RX to the UART3 pads located between battery's positive pad and motor 3 pads, the pad closer to the edge of the board being the R3 pad. Update: turns out these tiny pads are very fragile and one came off of the board after I tried to solder the wire to it. So I had to use UART4 to connect the new RX. No GPS for this build will be possible, except for if I remap the resources of SCL and SDA pads to UART3 RX and TX, or use soft serial. if the FC has other devices (like a barometer) using I2C, **they will stop working after remapping** scl and sda:

the original mapping:
```
resource SERIAL_TX 3 B10
resource SERIAL_RX 3 B11
resource I2C_SCL 1 A15
resource I2C_SDA 1 B07
```

new mapping:
```
resource I2C_SCL 1 none
resource I2C_SDA 1 none
resource SERIAL_TX 3 A15
resource SERIAL_RX 3 B07
```


The new RX is:
```
BETAFPV 2.4GHz Nano RX
Firmware Rev. 3.5.3 (40555e) ISM2G4
```



## references

- https://oscarliang.com
- https://www.youtube.com/@ChrisRosser
  - https://www.youtube.com/watch?v=EhYKeZfSQIw
- https://www.youtube.com/@JoshuaBardwell
- https://www.youtube.com/@MediocreNerd
- https://speedybee.zendesk.com/hc/en-us/articles/18769825525531-Experiencing-a-Runaway-takeoff-During-Drone-s-First-Flight
- https://www.team-blacksheep.com/media/files/vtx-table-for-betaflight.txt
- https://www.betaflight.com/docs/wiki/guides/current/VTX-CLI-Settings
- https://oscarliang.com/f1-f3-f4-flight-controller/