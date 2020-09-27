# How-to Bluetooth a2dp sink on raspberry 3 raspian buster with alsa and pin paring

Based on information from [1], [2] and [3]

`$ sudo apt-get install bluez-tools bluealsa`

`$ sudo vim /lib/systemd/system/bluealsa.service`

and change

> ExecStart=/usr/bin/bluealsa -p a2dp-sink

`$ sudo vim /lib/systemd/system/bluetooth.service`

> ExecStart=/usr/lib/bluetooth/bluetoothd **--noplugin=sap**

Create /etc/bluetooth/pin.conf **WITH** a trailing line

`$ echo -e "* 1234\n" | sudo tee /etc/bluetooth/pin.conf`

Add a bluetooth agent as service
`$ vim /etc/systemd/system/bt-agent.service`
`$ sudo chown root:root /etc/bluetooth/pin.conf`
`$ sudo chmod 600 /etc/bluetooth/pin.conf`
   
>   [Unit]
>   Description=Bluetooth Auth Agent
>   After=bluetooth.service
>   PartOf=bluetooth.service
>
>   [Service]
>   Type=simple
>   ExecStart=/usr/bin/bt-agent -c NoInputNoOutput -p /etc/bluetooth/pin.conf
>   ExecStartPost=/bin/sleep 1
>   ExecStartPost=/bin/hciconfig hci0 sspmode 0
>
>   [Install]
>   WantedBy=bluetooth.target


create service for playback

`$ vim /etc/systemd/system/a2dp-playback.service`

>   [Unit]
>   Description=A2DP Playback
>   After=bluealsa.service syslog.service
>   Requires=bluealsa.service
>
>   [Service]
>   ExecStartPre=/bin/sleep 3
>   ExecStart=/usr/bin/bluealsa-aplay --profile-a2dp 00:00:00:00:00:00
>   StandardOutput=syslog
>   StandardError=syslog
>   SyslogIdentifier=A2DP-Playback
>   User=pi
>
>   [Install]
>   WantedBy=multi-user.target

start at boot time:
```
   $ sudo systemctl daemon-reload
   $ sudo systemctl enable bt-agent a2dp-playback
   $ sudo systemctl restart bt-agent a2dp-playback bluealsa
   $ sudo systemctl status bt-agent.service a2dp-playback bluealsa
```
[1] https://medium.com/@mathieu.requillart/my-ultimate-guide-to-the-raspberry-pi-audio-server-i-wanted-bluetooth-64c347ee0d22
[2] https://gist.github.com/mill1000/74c7473ee3b4a5b13f6325e9994ff84c
[3] https://sigmdel.ca/michel/ha/rpi/bluetooth_n_buster_01_en.html
