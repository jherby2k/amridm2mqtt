# AMRIDM2MQTT: Send AMR/ERT Power Meter Data Over MQTT

##### Copyright (c) 2018 Ben Johnson. Distributed under MIT License.

Using an [inexpensive rtl-sdr dongle](https://www.amazon.com/s/ref=nb_sb_noss?field-keywords=RTL2832U), it's possible to listen for signals from ERT compatible smart meters using rtlamr. This script runs as a daemon, launches rtl_tcp and rtlamr, and parses the output from rtlamr. If this matches your meter, it will push the data into MQTT for consumption by Home Assistant, OpenHAB, or custom scripts.

TODO: Video for Home Assistant

## Changes from Original Project

This fork is specifically for tuned for reading SCM messages.

- symbol lengths are tuned (and can be adjusted) to reduce CPU consumption

Example Topic:
`meters/32109876/reading`

## Requirements

Tested under Raspberry Pi OS (32-bit) (Debian Bullseye)

### rtl-sdr package

Install RTL-SDR package

`sudo apt-get install rtl-sdr`

Set permissions on rtl-sdr device

/etc/udev/rules.d/rtl-sdr.rules

`SUBSYSTEMS=="usb", ATTRS{idVendor}=="0bda", ATTRS{idProduct}=="2838", MODE:="0666"`

Prevent tv tuner drivers from using rtl-sdr device

/etc/modprobe.d/rtl-sdr.conf

`blacklist dvb_usb_rtl28xxu`

### git

`sudo apt-get install git`

### pip3 and paho-mqtt

Install pip for python 3

`sudo apt-get install python3-pip`

Install paho-mqtt package for python3

`sudo pip3 install paho-mqtt`

### golang & rtlamr

Install Go programming language & set gopath

`sudo apt-get install golang`

https://github.com/golang/go/wiki/SettingGOPATH

If only running go to get rtlamr, just set environment temporarily with the following command

`export GOPATH=$HOME/go`


Install rtlamr https://github.com/bemasher/rtlamr

`go get github.com/bemasher/rtlamr`

To make things convenient, I'm copying rtlamr to /usr/local/bin

`sudo cp ~/go/bin/rtlamr /usr/local/bin/rtlamr`

## Install

### Clone Repo
Clone repo into opt

`cd /opt`

`sudo git clone https://github.com/jherby2k/amridm2mqtt.git amrscm2mqtt`

### Configure

Copy template to settings.py

`cd /opt/amrscm2mqtt`

`sudo cp settings_template.py settings.py`

Edit file and replace with appropriate values for your configuration

`sudo nano /opt/amrscm2mqtt/settings.py`

### Install Service and Start

Copy amrscm2mqtt service configuration into systemd config

`sudo cp /opt/amrscm2mqtt/amrscm2mqtt.systemd.service /etc/systemd/system/amrscm2mqtt.service`

Refresh systemd configuration

`sudo systemctl daemon-reload`

Start amridm2mqtt service

`sudo service amrscm2mqtt start`

Set amridm2mqtt to run on startup

`sudo systemctl enable amrscm2mqtt.service`

### Configure Home Assistant

To use these values in Home Assistant,

```yaml
mqtt:
  sensor:
    - name: Gas Meter
      state_topic: meters/12345678/reading
      availability_topic: meters/availability
      unit_of_measurement: m³
      device_class: gas
      state_class: total
      value_template: '{{ (value | int(0) * 0.0283) | round(2) }}'  # Optional, converts readings in cubic feet to m³
```

## Testing

Assuming you're using mosquitto as the server, and your meter's sends SCM messages with id 12345678, you can watch for events using the command:

`mosquitto_sub -t "meters/12345678/reading"`

Or if you've password protected mosquitto

`mosquitto_sub -t "meters/12345678/reading" -u <user_name> -P <password>`

If all else fails, you can listen to the base topic to see what's actually getting posted

`mosquitto_sub -t "meters/#" -u <user_name> -P <password>`

