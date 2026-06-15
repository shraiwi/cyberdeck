# cyberdeck

**Basics:**
- Model: `R36XX-V21 2024-12-18`, Panel 4
- Base image: [dArkOSRE-R36](https://github.com/southoz/dArkOSRE-R36) deROMified (use chroot+qemu to run builds)
	- [Installation Guide](https://github.com/southoz/dArkOSRE-R36/wiki/Firmware-Installation#genuine-r36s)
- [Handhelds wiki listing](https://handhelds.wiki/R36XX)
- [ROCKNIX Wiki Listing](https://rocknix.org/devices/unbranded/game-console-r35s-r36s)
## todo
- [x] Switch to mainline [dArkOS](https://github.com/christianhaitian/dArkOS) w/ DTBs from [AeolusUX/R36S-DTB](https://github.com/AeolusUX/R36S-DTB/)
	- [Wiki says](https://handhelds.wiki/R36XX#ArkOS) to use `Panel 4` DTBs.
- [ ] Get USB OTG Keyboard working
	- Hunch: `libusb` is not loaded; can mount usb drives in darkOSRE but Rocknix tbd. (shouldn’t rocknix work?)

## programs
- LLM: `Qwen3:0.6B` 4b quant
	- Tools: qalc (calculator) ddgr (browser)
- Terminal emulator: on-screen keyboard using SDL

## poking and prodding

**How to get SSH:**
1. Boot
2. Input Wifi Credentials AND country
3. SSH should be on? user: `root`, pwd: `rocknix`

### usb devices
```
RK3326:~ # lsusb
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 002: ID 0bda:0179 Realtek Semiconductor Corp. RTL8188ETV Wireless LAN 802.11n Network Adapter
```
### systemctl services
Notably, `essway.service` and `sway.service`
```
RK3326:~ # systemctl list-units --type=service --state=running
  UNIT                      LOAD   ACTIVE SUB     DESCRIPTION
  avahi-daemon.service      loaded active running Avahi mDNS/DNS-SD Stack
  dbus.service              loaded active running D-Bus System Message Bus
  debug-shell.service       loaded active running Early root shell on /dev/ttyS2 FOR DEBUGGING ONLY
  essway.service            loaded active running EmulationStation-with-Sway
  input.service             loaded active running Volume button service
  iwd.service               loaded active running Wireless service
  NetworkManager.service    loaded active running Network Manager
  nmbd.service              loaded active running Samba NMB Daemon
  pipewire-pulse.service    loaded active running PipeWire PulseAudio Service
  pipewire.service          loaded active running PipeWire Multimedia Service
  powerstate.service        loaded active running AC/Battery Performance Service
  rpcbind.service           loaded active running RPC Bind
  seatd.service             loaded active running Seat Management Daemon
  smbd.service              loaded active running Samba SMB Daemon
  sshd.service              loaded active running OpenSSH server daemon
  sway.service              loaded active running Sway Wayland Compositor
  systemd-journald.service  loaded active running Journal Service
  systemd-logind.service    loaded active running User Login Management
  systemd-resolved.service  loaded active running Network Name Resolution
  systemd-timesyncd.service loaded active running Network Time Synchronization
  systemd-udevd.service     loaded active running Rule-based Manager for Device Events and Files
  touchkeyboard.service     loaded active running touchscreen keyboard service
  wireplumber.service       loaded active running Multimedia Service Session Manager
  wsdd2.service             loaded active running WSD/LLMNR Discovery/Name Service Daemon

Legend: LOAD   → Reflects whether the unit definition was properly loaded.
        ACTIVE → The high-level unit activation state, i.e. generalization of SUB.
        SUB    → The low-level unit activation state, values depend on unit type.
```
### sway
The device ships with `sway` so that means i dont have to deal with any bs like DRM/KMS drivers. we can write the terminal emulator in macroquad/rust and then just boot straight into that instead of emulationstation. easy dubs. Input is already handled in SDL so this should just work.

```
RK3326:~ # systemctl stop essway.service
RK3326:~ # find /run /tmp -name "wayland-*" 2>/dev/null
/run/0-runtime-dir/wayland-1
/run/0-runtime-dir/wayland-1.lock
```

### touchscreen keyboard?
Seems interesting.

```
RK3326:~ # systemctl cat touchkeyboard.service
# /usr/lib/systemd/system/touchkeyboard.service
[Unit]
Description=touchscreen keyboard service
After=input.service
Before=rocknix.target
StartLimitIntervalSec=0

[Service]
Environment=HOME=/storage
ExecStart=/usr/bin/rocknix-touchscreen-keyboard
Restart=always
RestartSec=1

[Install]
WantedBy=multi-user.target
```

### idle htop (no ess)
Pretty light. yummy.

```
RK3326:~ # htop

    0[|                            2.6%] Tasks: 44, 18 thr, 87 kthr; 2 running
    1[|                            1.3%] Load average: 0.05 0.07 0.10
    2[|                            1.3%] Uptime: 00:20:19
    3[||                           3.9%]
  Mem[||||||||||||||||||||||||152M/982M]
  Swp[                            0K/0K]

  [Main] [I/O]
    PID USER       PRI  NI  VIRT   RES   SHR S  CPU%-MEM%   TIME+  Command
   6056 root        20   0  4280  2820  2252 R   2.0  0.3  0:00.33 htop
      1 root        20   0 21180 12688  8388 S   0.0  1.3  0:07.35 /usr/lib/syste
    191 root        20   0  4368  3408  2740 S   0.0  0.3  0:00.13 /bin/sh
    223 root        20   0 25824  7660  6376 S   0.0  0.8  0:00.87 /usr/lib/syste
    228 systemd-re  20   0 19408 12160  9684 S   0.0  1.2  0:04.40 /usr/lib/syste
    276 root        20   0  6192  2808  2520 S   0.0  0.3  0:00.06 /usr/sbin/rpcb
    279 systemd-ti  20   0 89932  6936  5972 S   0.0  0.7  0:00.29 /usr/lib/syste
    295 systemd-ti  20   0 89932  6936     0 S   0.0  0.7  0:00.01 /usr/lib/syste
    324 root        20   0 29960  8208  6404 S   0.0  0.8  0:00.71 /usr/lib/syste
    353 avahi       20   0  7424  3296  2884 S   0.0  0.3  0:00.23 avahi-daemon:
    354 dbus        20   0  8476  3952  3048 S   0.0  0.4  0:01.31 /usr/bin/dbus-
F1Help  F2Setup F3SearchF4FilterF5Tree  F6SortByF7Nice -F8Nice +F9Kill  F10Quit

```
