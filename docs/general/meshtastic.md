### Meshtastic

## Using serial for web client

Error observed in browser console when connecting over serial interface:

    Uncaught (in promise) NetworkError: Failed to execute 'open' on 'SerialPort': Failed to open serial port.

System logs:

```
chromium.desktop[117807]: [117807:117807:0203/185442.188970:ERROR:components/device_event_log/device_event_log_impl.cc:198] [18:54:42.188] Serial: serial_io_handler.cc:147 Failed to open serial port: FILE_ERROR_ACCESS_DENIED
chromium.desktop[117807]: [117807:117846:0203/185445.197355:ERROR:google_apis/gcm/engine/registration_request.cc:291] Registration response error message: DEPRECATED_ENDPOINT
```

The user probably needs to be member of dialout group to access serial port

	sudo usermod -aG dialout $USER

For this purpose, using chromium:

    sudo flatpak install org.chromium.Chromium

Set the override:

    flatpak override --user --filesystem=/run/udev:ro org.chromium.Chromium

Confirm change:

	flatpak info --show-permissions org.chromium.Chromium

Close and re-open the browser. 

https://client.meshtastic.org/

## Using a local TUI

Download and install the deb file:

    https://github.com/PeterGrace/meshtui/releases

Find the serial port:

    ls -la /dev/ttyACM0

Launch the program:

    meshtui -s /dev/ttyACM0

## Meshtastic Cli

Install the tools

	pipx install meshtastic

Use the tool

	meshtastic -s /dev/ttyACM0 <commands>

Get node list:

	meshtastic -s /dev/ttyACM0 --nodes | less -S

Send a traceroute:

	meshtastic -s /dev/ttyACM0 --traceroute xxxxx

Traceroute output will look like this:

```
Connected to radio
Sending traceroute request to 9213c6c5 on channelIndex:0 (this could take a while)
Route traced towards destination:
!b762276c --> !9213c6c5 (9.5dB)
Route traced back to us:
!9213c6c5 --> !b762276c (5.75dB)
```

Send a message:

	meshtastic -s /dev/ttyACM0 --dest xxxxx --sendtext "ping"

Message send output wil look lie this:

```
Connected to radio
Sending text message ping to 9213c6c5 on channelIndex:0 
```

