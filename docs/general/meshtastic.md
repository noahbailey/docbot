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

