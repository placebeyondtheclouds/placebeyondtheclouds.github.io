---
layout: post
title:  "Re-building 1S 80mm freestyle quad"
lang: en
tags: [en, 1s, tinywhoop, quad, fpv, diy, 80mm, analog ]
published: true
---

I damaged ESC #2 on the 1S Matrix AIO (Meteor75 Pro), either by running a motor with damaged windings or from a voltage spike in a crash. It's overheating, not giving full power to the motor, and the video feed has white washouts during high throttle. So I'm replacing the AIO with JHEMCU G474ELRS and [HGLRC Zeuz nano 350mw VTX](https://hglrc.freshdesk.com/support/solutions/articles/61000307667-zeus-350mw-vtx). New AIO is 1-2s, 12A ESCs with Bluejay, has a better ELRS antenna (with IPEX/UF.L connector), 4 UARTS, runs at 170 MHz, 8MB blackbox (sadly). 
Meteor75 frame is scraping the battery and motor screws against the ground, so I am replacing it with a clone of Mobula7 but for 45mm props (80mm base and 47mm ducts instead of 75mm and 43mm). It has 2S battery tray, and with 1S battery being propped by a dense foam material glued to the bottom the whoop will land on the lower part of the frame without the battery or motor screws touching the ground. Another solution to the problem would be keep the meteor75pro frame and printing [the battery bumper](https://www.thingiverse.com/thing:7056235).
camera - [Caddx Ant Nano Lite](https://caddxfpv.com/products/caddxfpv-ant-lite-4-3-fpvcycle-edition). ~~the motors are 1102 22000kv left from the Meteor75~~ changed the motors to noname 1103 15000KV 1-2S (3 screw 6mm base, 1.5 shaft, same as the betafpv 1102 motors). the props are gemfan 45mm-3. 20AVG battery lead with BT2.0. batteries batches of 高能 100C 550mAh LiHV 1S A30 (new, untested) and 格氏 95C 550mAh 1S LiHV BT2.0 (old, not very good life expectancy).

## pictures


work in progress


## the process

- connect the FC, check that it is working, backup the config (`dump`, `diff all showdefaults`)

- UARTs:
```
UART1:SBUS
UART2:VTX
UART3:onboard ELRS
UART4:(GPS)
```

- pad connections to the FC:
```
VTX: 5V(red) - 5V, GND(black) - GND, video(yellow) - VTX, RX(purple) - TX2
caddx ant nano lite pinout: 5-25V(red) - 5V, GND - GND, VIDEO(yellow) - CAM, OSD GND(can be disconnected), OSD signal (green) - through 150R to LED_STRIP pad on the FC.
buzzer: red - BZ+, black - BZ-
```

- apply `P-1025` conformal coating to the FC and VTX boards (the camera is already coated)

- install the components into the frame, secure what needed with zip ties, add `Kafuter K-705` silicon sealant to the places where wires are soldered to FC pads, to the U.FL connectors on FC and VTX

- recompile betaflight v2025, target `JHEG474`, analog, add features: camera control

- reflash bluejay, target `Z-H-30`, PWM 48kHz (it runaways on 96kHz for some reason), adjust the direction of the motors (props out)
```
startup sliders 1100 and 1200
rampup x3
motor timing  15 degrees
```

- flash the elrs receiver (target `BETAFPV 2.4GHz Lite RX`)
 
- calibrate accelerometer

- adjust voltage sensor calibration values

- in motors tab, turn on _motor direction is reversed_. it makes the mixer to expect that _some_ of the motors are reversed
```
set yaw_motors_reversed = ON
```

- (not working, probably needs a 10uF cap between the OSD and GND) [camera control](https://oscarliang.com/fpv-camera-control-fc/). measure the OSD pin voltage.
```
resource
resource LED_STRIP 1 none
resource camera_control 1 B02
set camera_control_ref_voltage = 337
```

- load elrs 150Hz rate profile (although some of the values will be changed with the filters tuning later)

- misc

```
feature -AIRMODE
feature TELEMETRY
feature OSD
beacon RX_LOST
beacon RX_SET
set baro_hardware = NONE
set gps_rescue_allow_arming_without_fix = ON
set pos_hold_without_mag = ON
set failsafe_procedure = DROP
set pidsum_limit_yaw = 1000
```

- modes

```
# aux
aux 0 0 0 1900 2100 0 0
aux 1 0 2 1900 2100 0 0
aux 2 1 1 1900 2100 0 0
aux 3 3 1 1400 1600 0 0
aux 4 27 3 1900 2100 0 0
aux 5 11 1 1400 1600 0 0
aux 6 13 4 1900 2100 0 0
aux 7 26 5 1900 2100 0 0
aux 8 28 1 900 1100 0 0
aux 9 35 2 1925 2100 0 0
```

- vtx table

```
vtxtable bands 5
vtxtable channels 8
vtxtable band 1 BOSCAM_A A CUSTOM 5865 5845 5825 5805 5785 5765 5745 5725
vtxtable band 2 BOSCAM_B B CUSTOM 5733 5752 5771 5790 5809 5828 5847 5866
vtxtable band 3 BOSCAM_E E CUSTOM 5705 5685 5665 5645 5885 5905 5925 5945
vtxtable band 4 FATSHARK F CUSTOM 5740 5760 5780 5800 5820 5840 5860 5880
vtxtable band 5 RACEBAND R CUSTOM 5658 5695 5732 5769 5806 5843 5880 5917
vtxtable powerlevels 4
vtxtable powervalues 25 100 200 400
vtxtable powerlabels 25 100 200 350
set vtx_band = 5
set vtx_channel = 8
set vtx_power = 1
set vtx_low_power_disarm = UNTIL_FIRST_ARM
set vtx_freq = 5917
set vcd_video_system = PAL
```

-  PIDs

```
profile 0

# profile 0
set profile_name = working
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
set throttle_boost = 0
set pidsum_limit = 1000
set p_pitch = 87
set i_pitch = 157
set d_pitch = 44
set f_pitch = 35
set p_roll = 70
set i_roll = 124
set d_roll = 39
set f_roll = 28
set p_yaw = 70
set i_yaw = 124
set f_yaw = 28
set d_max_roll = 52
set d_max_pitch = 60
set d_max_advance = 0
set auto_profile_cell_count = 1
set motor_output_limit = 95
set thrust_linear = 20
set dyn_idle_min_rpm = 75
set simplified_master_multiplier = 120
set simplified_d_gain = 110
set simplified_pi_gain = 130
set simplified_feedforward_gain = 20
set simplified_pitch_pi_gain = 120
set simplified_dterm_filter_multiplier = 110
set tpa_rate = 80
set tpa_breakpoint = 1500
```

-rates 
```
rateprofile 0

# rateprofile 0
set rateprofile_name = sasha
set roll_rc_rate = 16
set pitch_rc_rate = 16
set yaw_rc_rate = 16
set roll_srate = 90
set pitch_srate = 90
set yaw_srate = 90
```

- filters:
```
set gyro_lpf1_static_hz = 0
set gyro_lpf2_static_hz = 1000
set dyn_notch_q = 500
set dyn_notch_min_hz = 160
set dyn_notch_max_hz = 370
set gyro_lpf1_dyn_min_hz = 0
set rc_smoothing_auto_factor = 25
set rc_smoothing_auto_factor_throttle = 25
set rpm_filter_weights = 100,20,20
set rpm_filter_min_hz = 140
```

- motors (important) and battery:
```
set dshot_bidir = ON
set dshot_bitbang = AUTO
set motor_pwm_protocol = DSHOT300
set motor_poles = 12
set bat_capacity = 550
set vbat_max_cell_voltage = 440
set vbat_full_cell_voltage = 420
set vbat_min_cell_voltage = 320
set vbat_warning_cell_voltage = 340
set beeper_dshot_beacon_tone = 3
set small_angle = 180
```

- OSD. if BF is set to NTSC and the camera outputs PAL, the osd elements will not be visible
```
set report_cell_voltage = ON
set osd_rssi_dbm_alarm = -90
set osd_cap_alarm = 550
set osd_alt_alarm = 120
set osd_vbat_pos = 396
set osd_rssi_pos = 448
set osd_link_quality_pos = 14821
set osd_link_tx_power_pos = 4549
set osd_rssi_dbm_pos = 14816
set osd_rsnr_pos = 64
set osd_tim_1_pos = 385
set osd_tim_2_pos = 14839
set osd_remaining_time_estimate_pos = 472
set osd_flymode_pos = 14835
set osd_g_force_pos = 375
set osd_throttle_pos = 289
set osd_vtx_channel_pos = 4128
set osd_crosshairs_pos = 237
set osd_ah_sbar_pos = 4334
set osd_ah_pos = 4206
set osd_current_pos = 14827
set osd_mah_drawn_pos = 6616
set osd_craft_name_pos = 394
set osd_gps_speed_pos = 4323
set osd_gps_lon_pos = 17
set osd_gps_lat_pos = 0
set osd_gps_sats_pos = 14784
set osd_home_dir_pos = 4206
set osd_home_dist_pos = 4172
set osd_flight_dist_pos = 407
set osd_compass_bar_pos = 42
set osd_altitude_pos = 20726
set osd_debug_pos = 385
set osd_avg_cell_voltage_pos = 14796
set osd_battery_usage_pos = 393
set osd_disarmed_pos = 14603
set osd_nheading_pos = 397
set osd_up_down_reference_pos = 4348
set osd_ready_mode_pos = 14508
set osd_nvario_pos = 4374
set osd_esc_tmp_pos = 4150
set osd_esc_rpm_pos = 288
set osd_flip_arrow_pos = 14574
set osd_core_temp_pos = 4118
set osd_rate_profile_name_pos = 20
set osd_pid_profile_name_pos = 374
set osd_sys_vtx_temp_pos = 407
set osd_stat_bitmask = 8521444
```

## references

- https://oscarliang.com
- https://www.youtube.com/@ChrisRosser
- https://www.youtube.com/@JoshuaBardwell
- https://www.youtube.com/@MediocreNerd