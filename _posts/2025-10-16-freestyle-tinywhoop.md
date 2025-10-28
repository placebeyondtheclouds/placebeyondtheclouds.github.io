---
layout: post
title:  "Re-building 1S 80mm freestyle quad"
lang: en
tags: [en, 1s, tinywhoop, quad, fpv, diy, 80mm, analog ]
published: true
---

I damaged ESC #2 on the 1S Matrix AIO (Meteor75 Pro), either by running a motor with damaged windings or from a voltage spike in a crash. It's overheating, not giving full power to the motor, and the video feed has white washouts during high throttle. So I'm replacing the AIO with JHEMCU G474ELRS and [HGLRC Zeuz nano 350mw VTX](https://hglrc.freshdesk.com/support/solutions/articles/61000307667-zeus-350mw-vtx). New AIO is 1-2s, 12A ESCs with Bluejay, has a better ELRS antenna (with IPEX/UF.L connector), 4 UARTS, runs at 170 MHz, 8MB blackbox (sadly). 
Meteor75 frame is scraping the battery and motor screws against the ground, so I am replacing it with a clone of Mobula7 but for 45mm props (80mm base and 47mm ducts instead of 75mm and 43mm). It has 2S battery tray, and with 1S battery ~~being propped by a dense foam material (bad idea) glued to the bottom~~ the whoop will land on the lower part of the frame without the battery or motor screws touching the ground. Another solution to the problem would be keep the meteor75pro frame and printing [the battery bumper](https://www.thingiverse.com/thing:7056235).
camera - [Caddx Ant Nano Lite](https://caddxfpv.com/products/caddxfpv-ant-lite-4-3-fpvcycle-edition). the motors are 1102 22000kv left from the Meteor75. the props are gemfan 45mm-3. 20AVG battery lead with BT2.0. batteries batches of 高能 100C 550mAh LiHV 1S A30 and 格氏 95C 550mAh 1S LiHV BT2.0.

## video


work in progress


## the process

- connect the FC, check that it is working, backup the config (`dump`, `diff all showdefaults` and save the outputs)

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
caddx ant nano lite pinout: 5-25V(red) - 5V, GND - GND, VIDEO(yellow) - CAM, OSD GND(can be disconnected), OSD signal (green) - through 150R to LED_STRIP pad on the FC. add 10uf cap (106 10uf 50v) between OSD signal and GND on the camera side.
buzzer: red - BZ+, black - BZ-
```

- install the components into the frame, secure what needed with zip ties

- Caddx Ant Nano Lite camera settings (using OSD menu board): AE mode to BLC=3, brightness=35, contrast auto, saturation manual=20

- flash betaflight v2025.12, target `JHEG474`, analog OSD, add features: camera control. restore the original backup

- reflash bluejay, target `Z-H-30`, PWM 48kHz (I get runaways on 96kHz for some reason), adjust the direction of the motors (props out)
```
startup sliders to the max
rampup x3
motor timing  15 degrees
ESC power rating 2S+
temperature protection 140
beacon delay 1 min
```

- flash the elrs receiver (target `BETAFPV 2.4GHz Lite RX`)

- ~~(optional) connect to the FC using elrs wifi. in BF configurator options enable manual connection, connect to the RX wifi, then use port `tcp://10.0.0.1`~~
 
- calibrate accelerometer

- [adjust](https://oscarliang.com/current-sensor-calibration/) current sensor calibration values
```
set ibata_scale = 598
```

- in motors tab, turn on _motor direction is reversed_. it makes the mixer to expect that _some_ of the motors are reversed
```
set yaw_motors_reversed = ON
```

## restore my settings

- load elrs 150Hz rate profile (although some of the values will be changed with the filters tuning later)

- (not working) [camera control](https://oscarliang.com/fpv-camera-control-fc/). measure the OSD pin voltage.

```
resource
resource LED_STRIP 1 none
resource camera_control 1 B02
set camera_control_ref_voltage = 326
```

- misc

```
feature -AIRMODE
feature TELEMETRY
feature OSD
beacon RX_LOST
beacon RX_SET
set baro_hardware = NONE
set pidsum_limit_yaw = 1000
```

- modes (for my radio setup)

```
# aux
aux 0 0 0 1900 2100 0 0
aux 1 0 2 1900 2100 0 0
aux 2 1 1 1900 2100 0 0
aux 3 13 4 1900 2100 0 0
aux 4 26 5 1900 2100 0 0
aux 5 28 1 900 1100 0 0
aux 6 32 6 1850 2100 0 0
aux 7 35 2 1925 2100 0 0
```



- corrected vtx table for HGLRC Zeus nano 350mw

```
vtxtable
vtxtable bands 5
vtxtable channels 8
vtxtable band 1 BOSCAM_A A CUSTOM 5865 5845 5825 5805 5785 5765 5745 5725
vtxtable band 2 BOSCAM_B B CUSTOM 5733 5752 5771 5790 5809 5828 5843 5866
vtxtable band 3 BOSCAM_E E CUSTOM 5705 5685 5665 5645 5885 5905 5925 5945
vtxtable band 4 FATSHARK F CUSTOM 5740 5760 5780 5800 5820 5840 5860 5880
vtxtable band 5 RACEBAND R CUSTOM 5658 5695 5732 5769 5806 5843 5880 5917
vtxtable powerlevels 4
vtxtable powervalues 25 100 200 400
vtxtable powerlabels 25 100 200 350
```

official [vtx table for HGLRC Zeus nano 350mw](https://www.rotorama.cz/cms/assets/docs/d0c22322f24f3bf72e2e66bab648f238/13272-1/zeus-nano-350mw-vtx.json). [review.](https://www.multirotorguide.com/reviews/review-hglrc-zeus-vtx-nano/)


- ZENCHANSI 棕熊 007 400mw vtx. it uses smartaudio 2.1 protocol, so the power values will be in dBm. BF CLI has the command `vtx_info` to show the power levels that the VTX supports

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

- vtx settings

```
set vtx_band = 6
set vtx_channel = 8
set vtx_power = 1
set vtx_low_power_disarm = UNTIL_FIRST_ARM
set vcd_video_system = PAL
```

- vtx power on a pot (BF:aux4, radio:ch8). 
`<index> <aux_channel> <vtx_band> <vtx_channel> <vtx_power> <start_range> <end_range>`
```
vtx 0 3 0 0 1 999 1300
vtx 1 3 0 0 2 1300 1600
vtx 2 3 0 0 3 1600 2000

```

- PIDs

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

- rates 

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
set osd_vbat_pos = 396
set osd_rssi_pos = 448
set osd_link_quality_pos = 6629
set osd_link_tx_power_pos = 4549
set osd_rssi_dbm_pos = 6624
set osd_rsnr_pos = 64
set osd_tim_1_pos = 385
set osd_tim_2_pos = 6647
set osd_remaining_time_estimate_pos = 472
set osd_flymode_pos = 6643
set osd_anti_gravity_pos = 341
set osd_g_force_pos = 375
set osd_throttle_pos = 289
set osd_vtx_channel_pos = 4128
set osd_crosshairs_pos = 237
set osd_ah_sbar_pos = 4334
set osd_ah_pos = 4206
set osd_current_pos = 6635
set osd_mah_drawn_pos = 6616
set osd_wh_drawn_pos = 341
set osd_motor_diag_pos = 341
set osd_craft_name_pos = 394
set osd_pilot_name_pos = 341
set osd_gps_speed_pos = 4323
set osd_gps_lon_pos = 17
set osd_gps_lat_pos = 0
set osd_gps_sats_pos = 6592
set osd_home_dir_pos = 4206
set osd_home_dist_pos = 4172
set osd_flight_dist_pos = 407
set osd_compass_bar_pos = 42
set osd_altitude_pos = 20726
set osd_pid_roll_pos = 341
set osd_pid_pitch_pos = 341
set osd_pid_yaw_pos = 341
set osd_debug_pos = 385
set osd_debug2_pos = 234
set osd_power_pos = 341
set osd_pidrate_profile_pos = 341
set osd_warnings_pos = 14729
set osd_avg_cell_voltage_pos = 14796
set osd_pit_ang_pos = 341
set osd_rol_ang_pos = 341
set osd_battery_usage_pos = 393
set osd_disarmed_pos = 14603
set osd_nheading_pos = 397
set osd_up_down_reference_pos = 4348
set osd_ready_mode_pos = 14508
set osd_nvario_pos = 4374
set osd_esc_tmp_pos = 4150
set osd_esc_rpm_pos = 288
set osd_esc_rpm_freq_pos = 341
set osd_rtc_date_time_pos = 341
set osd_adjustment_range_pos = 341
set osd_flip_arrow_pos = 14574
set osd_core_temp_pos = 4118
set osd_log_status_pos = 341
set osd_stick_overlay_left_pos = 341
set osd_stick_overlay_right_pos = 341
set osd_stick_overlay_radio_mode = 2
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

- blackbox for filter tuning

```
blackbox_sample_rate = 1/2
blackbox_device = SPIFLASH
blackbox_disable_pids = OFF
blackbox_disable_rc = ON
blackbox_disable_setpoint = ON
blackbox_disable_bat = ON
blackbox_disable_alt = ON
blackbox_disable_rssi = ON
blackbox_disable_gyro = OFF
blackbox_disable_gyrounfilt = OFF
blackbox_disable_acc = ON
blackbox_disable_attitude = ON
blackbox_disable_debug = OFF
blackbox_disable_motors = OFF
blackbox_disable_rpm = OFF
blackbox_disable_gps = ON
blackbox_mode = NORMAL
blackbox_high_resolution = OFF
```

## testing

- _after_ a successful test flight, apply `P-1025` conformal coating to the FC and VTX boards (I already applied it to the camera board), add `Kafuter K-705` silicon sealant to the places where wires are soldered to the FC pads, U.FL connectors on FC and VTX

## references

- https://oscarliang.com
- https://www.youtube.com/@ChrisRosser
  - https://www.youtube.com/watch?v=EhYKeZfSQIw
- https://www.youtube.com/@JoshuaBardwell
- https://www.youtube.com/@MediocreNerd
- https://speedybee.zendesk.com/hc/en-us/articles/18769825525531-Experiencing-a-Runaway-takeoff-During-Drone-s-First-Flight
- https://www.team-blacksheep.com/media/files/vtx-table-for-betaflight.txt
- https://www.betaflight.com/docs/wiki/guides/current/VTX-CLI-Settings