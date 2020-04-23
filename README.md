# urc-backend-pi
Manual for helping to run the urc-backend on Raspberry Pi with initial Zigbee2MQTT configuration and with adb support.

## 1. Upgrade, installing dependencies

``sudo apt update``

``sudo apt upgrade``

``sudo apt install python3 python-pip3 mosquitto mosquitto-clients npm git make g++ gcc adb``

``pip3 install flask flask-cors paho-mqtt python-dotenv requests adb-shell``

## 2. Mosquitto MQTT Broker

``sudo nano /etc/mosquitto/mosquitto.conf``

Delete the following line:

``include_dir /etc/mosquitto/conf.d``

Add the following instead:

```
allow_anonymous false
password_file /etc/mosquitto/pwfile
listener 1883
```

Type the following in terminal and don't forget to enter your desired username instead of the placeholder:

``sudo mosquitto_passwd -c /etc/mosquitto/pwfile <Username>``

Then type in your desired password. You will need these credentials later.

## 3. Zigbee2MQTT

``sudo curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -``

``sudo git clone https://github.com/Koenkk/zigbee2mqtt.git /opt/zigbee2mqtt``

``sudo chown -R pi:pi /opt/zigbee2mqtt``

``cd /opt/zigbee2mqtt``

``npm i``

``nano /opt/zigbee2mqtt/data/configuration.yaml``

Delete comment symbols in lines with username and password enter your user and password configured in step 2.

In the line with serial set: ``disable_led: true``

``cd /opt/zigbee2mqtt``

``npm start``

### Start Zigbee2MQTT on system startup

``sudo nano /etc/systemd/system/zigbee2mqtt.service``

Add the following lines:

```
[Unit]
Description=zigbee2mqtt
After=network.target

[Service]
ExecStart=/usr/bin/npm start
WorkingDirectory=/opt/zigbee2mqtt
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```

Start service and enable it:

``sudo systemctl start zigbee2mqtt``

``sudo systemctl enable zigbee2mqtt.service``

### Zigbee2MQTT Configuration

#### Connect Zigbee device

 1. Turn on lamp or other zigbee device
 2. Device should connect automatically (may have a look at the log)

You can change the friendly name of the device here for easier sending requets: ``nano /opt/zigbee2mqtt/data/configuration.yaml``

#### MQTT commands

Hint: Remember that you have to escape quotation marks when sending JSON with ``mosquitto_pub`` command.

Turn on: ``mosquitto_pub -u <Username> -P <Password> -t zigbee2mqtt/<Friendly-name>/set -m {\"state\":\"ON\"}``

Change brightness: ``mosquitto_pub -u <Username> -P <Password> -t zigbee2mqtt/<Friendly-name>/set -m {\"brightness\":100}``

## 4. urc-backend

``git clone https://github.com/christianfl/urc-backend.git ~``

### Start urc-backend on system startup

``sudo nano /etc/systemd/system/urc_backend.service``

Add the following lines:

```
[Unit]
Description=urc_backend
After=network.target

[Service]
ExecStart=python app.py
WorkingDirectory=/home/pi/urc-backend
StandardOutput=inherit
StandardError=inherit
Restart=always
User=pi

[Install]
WantedBy=multi-user.target
```

``sudo systemctl start urc_backend``

``sudo systemctl enable urc_backend.service``

## 5. Deactivate Raspberry Pi LEDs on system startup

``sudo nano /opt/stop_raspi_leds.sh``

Add the following lines:

```
sudo sh -c 'echo 0 > /sys/class/leds/led1/brightness'
sudo sh -c 'echo 0 > /sys/class/leds/led0/brightness'
```

Make script executable: ``sudo chmod +x /opt/stop_raspi_leds.sh``

``sudo nano /etc/systemd/system/ledoff.service``

Add the following lines:

```
[Unit]
Description=stopraspiled

[Service]
ExecStart=/usr/bin/sudo /opt/stop_raspi_leds.sh

[Install]
WantedBy=multi-user.target
```
Start service and enable it:

``sudo systemctl start ledoff``

``sudo systemctl enable ledoff.service``

``sudo reboot``

## 6. Establish adb connection

Establish a connection with the native adb implementation. The port is usually ``5555``:

``adb connect <IP-address>:<PORT>``

Then copy the adbkey from ``~/.android/adbkey`` to the urc-backend directory and don't forget to include the path into the device file of the adb accessible device.

## 7. Change hostname

``sudo hostname -b smarthome``

Change hostname manually here: ``sudo nano /etc/hostname``

Change line starting with 127.0.0.1 to smarthome: ``sudo nano /etc/hosts``
