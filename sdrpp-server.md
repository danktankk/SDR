# Full Setup for SDR++ in Headless Server Mode in a VM (No Audio Config Required)

This guide will walk you through setting up SDR++ in server mode on a Linux system, without requiring audio configuration. The steps are designed to be minimal and secure, using a dedicated user for running the SDR++ service.

## Prerequisites

- A Linux system with `apt` package manager (e.g., Ubuntu, Debian)
- Access to the terminal with sudo privileges

## Step 1: Install Dependencies (Minimal)

Update your system and install basic tools. Note that audio-related dependencies like `alsa-utils` are generally not required for this setup, but you can include them if necessary.

\```bash
sudo apt update
sudo apt install -y alsa-utils  # Optional, only if absolutely needed as audio will not be coming from the VM or metal.
\```

## Step 2: Create a Dedicated SDR++ User and add necessary groups

For security reasons, it's recommended to run the SDR++ server under a dedicated user account with limited privileges.

\```bash
sudo useradd -r -s /bin/false sdrpp
sudo mkdir /home/sdrpp
sudo chown -R sdrpp:sdrpp /home/sdrpp
sudo usermod -aG plugdev,dialout sdrpp
\```

## Step 3: Install SDR++

Download the SDR++ `.deb` package that matches your system architecture (e.g., `sdrpp_ubuntu_noble_amd64.deb`), and then install it.

\```bash
sudo dpkg -i sdrpp_ubuntu_noble_amd64.deb
sudo apt-get install -f  # Resolves missing dependencies, if any
\```

## Step 4: Set Up the SDR++ Systemd Service

To ensure SDR++ starts automatically in server mode on system boot, create a systemd service file.

1. Create the service file:

\```bash
sudo nano /etc/systemd/system/sdrpp-server.service
\```

2. Add the following configuration to the file.  Dont worry about  the port as it defaults to 5259 on the sdrpp client.

\```ini
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

[Install]
WantedBy=multi-user.target
\```

3. Save and close the file (`CTRL + o`, then `CTRL + x`).

## Step 5: Enable and Start the SDR++ Service

Make sure the systemd configuration is reloaded, enable the service to start on boot, and then start it immediately.

\```bash
sudo systemctl daemon-reload
sudo systemctl enable sdrpp-server.service
sudo systemctl start sdrpp-server.service
\```

Verify that the service is running correctly:

\```bash
sudo systemctl status sdrpp-server.service
\```

You should see an output indicating that the service is active and running.

## Conclusion

You have now successfully set up SDR++ to run in server mode, starting automatically on system boot. This setup is minimal and does not require audio configuration. Enjoy using your SDR++ server!

## Optional: Configure ALSA to Use a Dummy Device
There is a slim chance you may need to still get some sort of audio taking with your sdrpp server.
Suppress ALSA-related errors by using a dummy device:

### Load the Dummy Driver:

\```bash
sudo modprobe snd-dummy
\```

### Create or Edit the ALSA Configuration File:

\```bash
sudo nano /etc/asound.conf
\```

Add these lines to configure ALSA to use the dummy device:

\```ini
pcm.!default {
    type hw
    card 0
}
ctl.!default {
    type hw
    card 0
}
\```


