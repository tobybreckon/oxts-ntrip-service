# NTRIP service setup for Twizy Vehicle

How to setup the ntrip service to feed RTCM messages to the OxTS unit on the Twizy Vehicle

**Step 0:** install pi + add user ``capture`` as default

**Step 1:** Install Dependencies

```
sudo apt update && sudo apt upgrade -y
sudo apt install -y git cmake build-essential
````

**Step 2:** clone and build ntripclient

```
cd /opt
sudo git clone https://github.com/nunojpg/ntripclient.git
cd ntripclient
sudo make
```

**Step 3:** setup config

```
cd /tmp/
git clone git@github.com:tobybreckon/twizy-ntrip-service.git
cd twizy-ntrip-service
sudo mkdir -p /etc/ntripclient
sudo cp ntripclient.env /etc/ntripclient/
sudo chmod 600 /etc/ntripclient/ntripclient.env
sudo chown root:root /etc/ntripclient/ntripclient.env
```

** step 4:** setup daemon service to run as user pi

```
sudo cp ntripclient.service /etc/systemd/system/ntripclient.service
sudo usermod -aG dialout capture
```

**Step 5:** enable and start the service

```
# Reload systemd to pick up the new service file
sudo systemctl daemon-reload

# Enable the service to start automatically at boot
sudo systemctl enable ntripclient.service

# Start the service immediately
sudo systemctl start ntripclient.service
```

**Step 7:** check status

```
# Check the current status
sudo systemctl status ntripclient.service

# Watch live logs
sudo journalctl -u ntripclient.service -f

# See recent logs
sudo journalctl -u ntripclient.service --since "10 minutes ago"
```

**Step 8:** setup autostart of terminal

```
mkdir -p ~/.config/autostart
cp ntripclient-log.desktop ~/.config/autostart/
```

**Extra - service management commands:**

```
# Stop the service
sudo systemctl stop ntripclient.service

# Restart the service (e.g. after config changes)
sudo systemctl restart ntripclient.service

# Disable autostart at boot
sudo systemctl disable ntripclient.service

# Check if enabled at boot
sudo systemctl is-enabled ntripclient.service
```