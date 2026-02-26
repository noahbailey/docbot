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

### meshtui

Download and install the deb file:

    https://github.com/PeterGrace/meshtui/releases

Find the serial port:

    ls -la /dev/ttyACM0

Launch the program:

    meshtui -s /dev/ttyACM0

### Contact

Install from pypi:

    pipx install contact

Launch the program:

    contact -s /dev/ttyACM0

https://github.com/pdxlocations/contact?tab=readme-ov-file#commands

## Meshmonitor web client

Make sure docker is installed first. 

    sudo apt install docker.io docker-compose

the `/opt/meshmonitor/compose.yml` file:

```yaml
services:
  serial-bridge:
    image: ghcr.io/yeraze/meshtastic-serial-bridge:latest
    container_name: meshtastic-serial-bridge
    devices:
      - /dev/ttyACM0:/dev/ttyACM0
    ports:
      - "4403:4403"
    restart: unless-stopped
    environment:
      - SERIAL_DEVICE=/dev/ttyACM0
      - BAUD_RATE=115200
      - TCP_PORT=4403

  meshmonitor:
    image: ghcr.io/yeraze/meshmonitor:latest
    container_name: meshmonitor
    ports:
      - "8080:3001"
    volumes:
      - meshmonitor-data:/data
    environment:
      - MESHTASTIC_NODE_IP=serial-bridge
      - ALLOWED_ORIGINS=http://meshy.local:8080
    restart: unless-stopped
    depends_on:
      - serial-bridge

volumes:
  meshmonitor-data:
```

Notes:
* On first run, password has to be changed from `changeme` to something else like `meshtastic`. 
