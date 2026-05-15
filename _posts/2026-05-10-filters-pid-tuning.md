---
layout: post
title:  "Filters and PID tuing in Betaflight"
lang: en
tags: [en, pid, filters, quad]
category: notes
published: true
---

How I tune filters and PIDs in Betaflight using PIDtoolbox pro (a musthave, current v.0.93)

## radio

- ADC Filter OFF
- global functions -> `GF1 ON Lua Script bfbkgd` for time sync from radio's RTC to the betaflight flight controller

## quad

- must have a capacitor
- must have new props installed, all screws tightened, no dangling antennas or XT60

## ESC

- [ESC tuning](https://www.youtube.com/watch?v=EhYKeZfSQIw&list=PLFPBjpbd5xKT9eCWvtuFJ-qe3BxTDFvad)
- https://esc-configurator.com or https://am32.ca, or [locally]({% post_url 2025-11-23-bf-local %})
- for 2,5-inch and smaller set PWM frequency to 48 kHz, for larger quads set it to 24 kHz
- set type to 2S+ if it is not 1S FC, disable temperature protection
- motor settings in am32 must be set close to the actual ones. manual protocol selection `DSHOT`

## Betaflight preparation

- backup previous config (`dump` and `diff all showdefaults`), flash the latest version of Betaflight using [online app](https://app.betaflight.com/) or [locally]({% post_url 2025-11-23-bf-local %})
- load ELRS profile, RC smoothing should fit the flying style. less smoothing will result in hotter motors when the tune is pushed to extremes
- enable bidirectional dshot, set correct dynamic idle value is based on the prop size/pitch, [here](https://oscarliang.com/how-to-enable-and-configure-betaflight-dynamic-idle/) and [here](https://youtu.be/1oYoVE4xu1U?si=yH7NVtL8CJaB1tvT&t=798)
- the two `pidsum_limit` can be used to set PID authority to 100% from 50% default (**do not** change this after PID tuning). 
```
set pidsum_limit = 1000
set pidsum_limit_yaw = 1000
```

- if MCU has enough compute power, set ESCs to DSHOT600, `set pid_process_denom = 1` and disable gyro low pass 2
- `motor_poles` must be set correctly (usually 12 for small motors and 14 for large motors)
- set blackbox to 2 kHz, debug fft_freq


## PID and filter tuning process

- https://www.youtube.com/watch?v=FsJNHI2HWlg
- https://www.youtube.com/watch?v=sqT4MACi3d8
- https://www.youtube.com/watch?v=ehvQm8Rqrzk
- if needed, set linear rates, angle strength 50, angle limit 30
- for tuning filters only (stepresponse tool ignores any movement less than 20 degrees per second): trim logs each time to remove takeoff and landing
- the process
  - on the default PIDs, record hover
  - use PIDtoolbox to adjust the filters (rpm filter weights, q, dynamic notches count, q). keep the noise under -40 db (the noise is mostly meaningless below -30). turn off gyro lowpass 1 if there is no noise after 500hz
    - adjust
      ```
      set rpm_filter_weights = 100,100,100
      set rpm_filter_fade_range_hz = 50
      set rpm_filter_q = 500
      ```
  - set ff and dmax to zero, iterm gain to 0.2
  - in angle mode, record wiggle with **dterm** gain 0.6-1.8 step 0.2, in step response tool find **the best curve without overshoot** (to have a headroom for mm)
  - in angle mode, record wiggle with **master** multiplier 0.6-1.8 step 0.2, choose **the lowest latency before oscillations begin** in step response tool, compare roll and pitch latency and adjust roll:pitch balance
  - in acro mode, record **iterm** gain 0.5-2 step 0.5, choose  **the best curve without overshoot or oscillations** in step response tool
  - in acro mode, record **ff** gain 0.5-2 step 0.5, choose the closest following curve without overshoot, lowest latency (shortest period) between setpoint and gyro in the main window plots.

- if there is a slow bounceback after a roll or a flip, lower I term gain 
- iterm https://www.youtube.com/watch?v=Sq_DFjmvVDE


## (optional) use blackbox explorer to adjust filters

- https://www.youtube.com/watch?v=E3s5XYk3M74
- UAVTech's workspace for blackbox viewer is [here](https://theuavtech.com/wp-content/uploads/2024/10/UAVtech-BF-BBE-4.0-Version.json)


## after tuning

- turn on anti-gravity 5, voltage sag compensation (depends on `vbat_warning_cell_voltage`), [thrust linearization](https://oscarliang.com/fpv-drone-tuning/#Thrust-Linearization)
- `set motor_output_limit = 90` for a 5-inch
- do hover, punchouts, forward flight, test for propwash (split-s, sharp 180 turns, dives), throttle chops to test antigravity (nose dives - bump up, throbbles - lower)
- watch for motor temperature

## problems

- [about](https://www.youtube.com/watch?v=7GweG0RnCfc) the `washout` problem with ducted frames, when quad not descending with lowered throttle
- hot motors [1](https://www.youtube.com/watch?v=fU7P90sKScA) [2](https://www.youtube.com/watch?v=omat80ZiGHA) . [Too Much D-Term, Too Little Filtering, or Both](https://oscarliang.com/fpv-drone-motors-get-hot/#Too-Much-D-Term-Too-Little-Filtering-or-Both). hot motors might be caused by RC smoothing being too low for the tune
- [wobbles](https://www.youtube.com/watch?v=YaHpP1YEhX0)

## references

- https://oscarliang.com/fpv-drone-tuning/
- https://oscarliang.com/pid/
- https://www.youtube.com/playlist?list=PLFPBjpbd5xKQAzyblBStGKYtNdP9FCpRe