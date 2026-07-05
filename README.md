# NTRIP service setup for OxTS RT3000v3

How to setup an NTRIP service to feed RTCM messages into a OxTS GPS/IMU unit [RT3000v3](https://www.oxts.com/solutions/inertial-navigation-solutions/navigation-hardware/rt3000-v3/) via serial line connection (on a Rasberry Pi or similar).

---

[ **Preliminary Step:** Ensure you have a correctly wired serial cable for the OxTS [RT3000v3](https://www.oxts.com/solutions/inertial-navigation-solutions/navigation-hardware/rt3000-v3/) - because it _not_ a standard RS-232 cable from our experience (see - https://support.oxts.com/hc/en-us/article_attachments/360007523840) and has an integrated +V supply carried over the serial line from the OxTS unit also (!). This is to power an attached radio modem configuration (we presume) but will not play well with a standard _"off the shelf"_ RS-232 / serial-to-USB setup to/from a PC or SBC from our experience. ]

---

**Step 0:** install OS (Rasberry Pi OS or Ubuntu depending on hardware) + add user ``capture`` (or alternative) as default

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
git clone git@github.com:tobybreckon/oxts-ntrip-service.git
cd oxts-ntrip-service
sudo mkdir -p /etc/ntripclient
sudo cp ntripclient.env /etc/ntripclient/
sudo chmod 600 /etc/ntripclient/ntripclient.env
sudo chown root:root /etc/ntripclient/ntripclient.env
```

[ **Step 3a :** setup config ] 

```
< edit file /etc/ntripclient/ntripclient.env to the parameters for your ntrip service >
```

Defaults are set for public [Durham University - Department of Computer Science](https://www.durham.ac.uk/computer.science/) RTK NTRIP base station with a NTRIP caster hosted via [rtk2go.com:DURHAM_UNI_CS_UK](http://rtk2go.com:2101/SNIP::MOUNTPT?baseName=DURHAM_UNI_CS_UK).

**Step 4:** setup daemon service to run as user capture (or your alternative used earlier)
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

This is not needed but useful if you want _"always on"_ on screen monitoring and will display startup/connectivity errors from the service and the feedback messages from the OxTS unit.

Edit file ``ntripclient-log.desktop`` to ``gnome-terminal`` from ``lxterminal`` if OS is Ubuntu 

```
mkdir -p ~/.config/autostart
cp ntripclient-log.desktop ~/.config/autostart/
```

---

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

Tested on Raspberry OS (64-bit, Debian Trixie) and Ubuntu 24.04 LTS.

---

**References:**

This was used to support the following research work* :

[* _and in fact only works thanks in no small part to the sheer perseverance and patience of lead/co-author [Wenke (Tom) E](https://github.com/Tom-E-Durham) with a whole bunch of unforseen hardware challenges!_ ]

1. **[Dur360BEV: A Real-world 360-degree Single Camera Dataset and Benchmark for Bird-Eye View Mapping in Autonomous Driving](https://github.com/Tom-E-Durham/Dur360BEV)** (W. E, C. Yuan, Y. Sun, Y.F.A. Gaus, A. Atapour-Abarghouei, T.P. Breckon), In Proc. Int. Conf. on Robotics and Automation, IEEE, pp. 3737-3744, 2025.

2. **[DurTOMD: A Trail-based Off-road Multimodal Dataset for Traversable Pathway Segmentation under Challenging Illumination Conditions](https://github.com/yyyxs1125/TOMD)** (Y. Sun, L. Li, W. E, A. Atapour-Abarghouei, T.P. Breckon), In Proc. Int. Joint Conf. on Neural Networks, IEEE, pp. 1-8, 2025.

2. **[KD360-VoxelBEV: LiDAR and 360-degree Camera Cross Modality Knowledge Distillation for Bird’s-Eye-View Segmentation](https://github.com/Tom-E-Durham/KD360-VoxelBEV)** (W. E, Y. Sun, J. Liu, H.P.H. Shum, A. Atapour-Abarghouei, T.P. Breckon), In Proc. Winter Applications of Computer Vision, CVF/IEEE, pp. 3483-3493, 2026.

3. **[Mind2Drive: Predicting Driver Intentions from EEG in Real-world On-Road Driving](https://github.com/galosaimi/Mind2Drive)** (G. Alosaimi, H. Alhamdan, W. E, S. Katsigiannis, A. Atapour-Abarghouei, T.P. Breckon), In Proc. Int. Joint Conf. on Neural Networks, IEEE, 2026.

Please reference the above if you are using this in your own work (where reference #1 is considered the primary reference).

---

If you find any bugs or corrections to the above raise an issue (or much better still submit a git pull request with a fix) - toby.breckon@durham.ac.uk

_"may the source be with you"_ - anon.