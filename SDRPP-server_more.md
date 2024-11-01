## 1. Install Required Dependencies
Install all necessary libraries and tools required to build and run SDR++ and RTL-SDR.

```
sudo apt update
sudo apt install -y cmake build-essential libfftw3-dev libusb-1.0-0-dev \
libasound2-dev libpulse-dev libglfw3-dev libportaudio2 librtlsdr-dev \
libvolk2-dev git dos2unix
```
## 2. Clone and Build SDR++ from Source
```
Clone the repository and build SDR++:<br> 
git clone https://github.com/AlexandreRouma/SDRPlusPlus.git<br>
cd SDRPlusPlus<br>
mkdir build<br>
cd build<br>
cmake ..<br>
make -j$(nproc)<br>
sudo make install<br>
sudo ldconfig
```
Verify the installation:
```
sdrpp --version
```

## 3. Configuration File Setup

Create the SDR++ config file in the SDR++ user’s home directory:<br>
```
mkdir -p /home/sdrpp/.config/sdrpp
nano /home/sdrpp/.config/sdrpp/config.json
Copy and paste the following configuration into config.json:
```

```
{
    "bandColors": {
        "amateur": "#FF0000FF",
        "aviation": "#00FF00FF",
        "broadcast": "#0000FFFF",
        "marine": "#00FFFFFF",
        "military": "#FFFF00FF"
    },
    "bandPlan": "General",
    "bandPlanEnabled": true,
    "bandPlanPos": 0,
    "centerTuning": false,
    "colorMap": "Classic",
    "decimationPower": 0,
    "fastFFT": false,
    "fftHeight": 300,
    "fftHold": false,
    "fftHoldSpeed": 60,
    "fftRate": 20,
    "fftSize": 65536,
    "fftSmoothing": false,
    "fftSmoothingSpeed": 100,
    "fftWindow": 2,
    "frequency": 100000000.0,
    "fullWaterfallUpdate": false,
    "fullscreen": false,
    "invertIQ": false,
    "iqCorrection": false,
    "lockMenuOrder": false,
    "max": 0.0,
    "maximized": false,
    "menuElements": [
        {
            "name": "Source",
            "open": true
        },
        {
            "name": "Radio",
            "open": true
        },
        {
            "name": "Recorder",
            "open": true
        },
        {
            "name": "Frequency Manager",
            "open": true
        },
        {
            "name": "VFO Color",
            "open": true
        },
        null,
        {
            "name": "Band Plan",
            "open": true
        },
        {
            "name": "Display",
            "open": true
        }
    ],
    "menuWidth": 300,
    "min": -120.0,
    "moduleInstances": {
        "RTL-SDR Source": {
            "enabled": true,
            "module": "rtl_sdr_source"
        },
        "RTL-TCP Source": {
            "enabled": true,
            "module": "rtl_tcp_source"
        },
        "Radio": {
            "enabled": true,
            "module": "radio"
        },
        "Recorder": {
            "enabled": true,
            "module": "recorder"
        }
    },
    "modulesDirectory": "/usr/lib/sdrpp/plugins",
    "offset": 0.0,
    "offsetMode": 0,
    "resourcesDirectory": "/usr/share/sdrpp",
    "showMenu": true,
    "showWaterfall": true,
    "snrSmoothing": false,
    "snrSmoothingSpeed": 20,
    "source": "",
    "streams": {
        "Radio": {
            "muted": false,
            "sink": "Audio",
            "volume": 1.0
        }
    },
    "theme": "Dark",
    "uiScale": 1.0,
    "vfoColors": {
        "Radio": "#FFFFFF"
    },
    "vfoOffsets": {},
    "windowSize": {
        "h": 720,
        "w": 1280
    },
    "networkDiscovery": {
        "enabled": true,
        "broadcastAddress": "255.255.255.255",
        "port": 5259
    }
}
```

## 4. Disable Unused Plugins

Disable all plugins except rtl_sdr_source.so and rtl_tcp_source.so by renaming them with a .disabled extension.<br>
```
cd /usr/lib/sdrpp/plugins
```
```
sudo mv airspyhf_source.so airspyhf_source.so.disabled
sudo mv airspy_source.so airspy_source.so.disabled
sudo mv audio_sink.so audio_sink.so.disabled
sudo mv audio_source.so audio_source.so.disabled
sudo mv bladerf_source.so bladerf_source.so.disabled
sudo mv file_source.so file_source.so.disabled
sudo mv fobossdr_source.so fobossdr_source.so.disabled
sudo mv frequency_manager.so frequency_manager.so.disabled
sudo mv hackrf_source.so hackrf_source.so.disabled
sudo mv hermes_source.so hermes_source.so.disabled
sudo mv limesdr_source.so limesdr_source.so.disabled
sudo mv network_sink.so network_sink.so.disabled
sudo mv network_source.so network_source.so.disabled
sudo mv perseus_source.so perseus_source.so.disabled
sudo mv plutosdr_source.so plutosdr_source.so.disabled
sudo mv rfnm_source.so rfnm_source.so.disabled
sudo mv rfspace_source.so rfspace_source.so.disabled
sudo mv rigctl_server.so rigctl_server.so.disabled
sudo mv sdrpp_server_source.so sdrpp_server_source.so.disabled
sudo mv sdrplay_source.so sdrplay_source.so.disabled
sudo mv spectran_http_source.so spectran_http_source.so.disabled
sudo mv spyserver_source.so spyserver_source.so.disabled
```
## 5. Configure SDR++ as a Systemd Service

Create a Service File:<br>
```
sudo nano /etc/systemd/system/sdrpp.service
```
Add the Service Configuration:<br>
```
[Unit]
Description=SDR++ Server Mode
After=network.target

[Service]
Type=simple
User=sdrpp
Group=sdrpp
ExecStart=/usr/bin/sdrpp --server
Restart=always
Environment="HOME=/home/sdrpp"
Environment="SDL_AUDIODRIVER=dummy"
Environment="AUDIODEV=alsa"

[Install]
WantedBy=multi-user.target
Enable and Start the Service:
```
```
sudo systemctl daemon-reload
sudo systemctl enable sdrpp.service
sudo systemctl start sdrpp.service
```
### Check the Service Status:
```
sudo systemctl status sdrpp.service
```

## 6. Troubleshoot Common Issues
PulseAudio Connection Refused: Ensure SDL_AUDIODRIVER is set to dummy to avoid audio requirements on the server.<br>
Buffer Underflows: Adjust the sample rate or bandwidth settings in the client if you see buffer underflows.<br>

## 7. Verify and Test
Connect with SDR++ Client: Open SDR++ on a client machine, connect to the server IP (e.g., 192.168.1.148) on port 5259.<br>

### Monitor Logs: Keep an eye on logs for errors or unexpected issues:

sudo journalctl -u sdrpp.service -f<br>

This setup should enable SDR++ to run effectively as a server with only the RTL-SDR and RTL-TCP plugins active. Let me know if any further adjustments are needed.
