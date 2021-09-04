## Introduction
We present SynchroSem: a loosely coupled multi-modal semantic SLAM approach for synchronized Lidar-Visual-Inertial (LVI) systems. Our system integrates pose measurements from two different tightly coupled odometry systems, Visual-Inertial (VI) and Lidar-Inertial (LI). These are loosely coupled by means of a pose graph representation that enables a robust late-fusion of pose measurements, being resilient to geometric and visual degeneracy. The developed multi-modal semantic feature extraction allows the global optimization system to reduce long term operation errors via loop closure, achieving State-of-the-Art performance on public LVI datasets.
Our method has also been deployed on a custom robotics platform. We developed a simple and reproducible hardware synchronization system using commercial-off-the-shelf hardware, and it is made publicly available at https://halops.github.io/SynchroSem/.

### Videos
**[TODO: ADD EMBEDDED VIDEOS]**

## Hardware setup
The hardware platform for this project consists of a Clearpath Jackal unmanned ground vehicle (UGV), coupled with the Scaled Robotics sensor rig: a platform containing 2x Velodyne VLP-16 Lidars, 1x Realsense D435i stereo-inertial camera, 1x Microstrain 3DM-GX-45 high rate IMU, as well as 3x fisheye rolling shutter cameras with a 360º field of view (FOV). Moreover, the Jackal UGV has been equipped with an Intel NUC 10i7FNH, a powerful on-board computer with an Intel i7-1071OU processor.\
![Jackal picture](/docs/assets/sr_jackal.png)

## Sensor synchronization
### Theoretical basis
The hardware synchronization implemented in this work uses a **Raspberry PI 4B** as a central device, which is able to synchronize its internal clock to an external master device that is used as a reference. This reference is then transmitted to the other relevant sensors in the system, as shown in the picture below. The procedure for synchronization is as follows:
![Time sync diagram](/docs/assets/time_sync_diagram.png)
1. The **reference** external clock is provided by a **Garmin GPS 18x LVC**, which provides a Pulse-Per-Second (**PPS**) signal as well as a serial interface for sending **NMEA** messages, which contain position and timing information. These two signals are interfaced via a **custom PCB** to both Velodyne sensors, as well as to the Raspberry PI. The Velodyne sensors contain specific firmware to handle PPS+NMEA synchronization signals, while on the Raspberry PI end we process the PPS signal using the Raspberry PI's GPIO pins and parse the NMEA strings through serial communication by using off the shelf tools (GPSd + Chrony).
2. Once the Raspberry PI has its own internal clock synchronized to the external master reference, it now transmits this timing information to the main computer, the Intel NUC. This synchronization is also handled using Chrony, making use of the NTP protocol through an Ethernet sub-network to periodically synchronize the NUC's clock to the reference. 
3. The Raspberry PI also generates the trigger signals for the Realsense VIO camera, issuing pulses at a user-defined frequency using the GPIO pins. Moreover, the Raspberry PI transmits the firing time through the Ethernet sub-network using ROS messages, for the NUC to be able to match the sensor data to its corresponding firing time.

### Hardware implementation
For the hardware implementation, we need to physically interface all the connections shown in the block diagram shown above by means of a main connection PCB and a series of different cables and connectors. These are detailed below.
#### Main connection PCB
![PCB schematic](/docs/assets/pcb_schematic.png) 

<!--
#### GPS IN - Garmin
#### GPS OUT - Velodyne
#### TRIGGER OUT - Realsense
#### RPI - Host PC --> 


### Software configuration
In this section we will configure the Raspberry PI software to properly synchronize its internal clock to the Garmin GPS reference.
#### Configuring GPSD
GPSD is the software used for communication with the GPS itself. Not only it will read the GPIO pin #4 (Where the PPS signal is interfaced), but will also use the TX and RX lines of the Raspberry to decode the NMEA strings that indicate the UTC time.\
Activate hardware serial: 
```
sudo raspi-config
``` 
Install the following dependencies: 
```
sudo sh -c "apt-get update ; sudo DEBIAN_FRONTEND=noninteractive apt-get upgrade -y -o Dpkg::Options::="--force-confdef" -o Dpkg::Options::="--force-confold"" 
sudo sh -c " apt-get install usbmount eject gpsd gpsd-clients python-gps pps-tools -y" 
```

Configure GPSD daemon: 
```
sudo sed -i 's/USBAUTO="true"/USBAUTO="false"/g' /etc/default/gpsd
sudo sed -i 's:DEVICES="":DEVICES="/dev/ttyAMA0 /dev/pps0":g' /etc/default/gpsd
sudo sed -i 's:GPSD_OPTIONS="":GPSD_OPTIONS="-n":g' /etc/default/gpsd
systemctl enable gpsd
echo dtoverlay=pps-gpio,gpiopin=4 >> /boot/config.txt
echo pps-gpio >> /etc/modules
sudo su
sed -i '$ s/exit 0/gpspipe -r -n 1 \&/g' /etc/rc.local ; echo exit 0 >> /etc/rc.local
exit
``` 
#### Configuring chrony
Chrony will be used to synchronize the reference that is decoded by GPSD with the internal Raspberry PI clock. Moreover, Chrony will be used by the Host PC to probe the time reference from the RPI.
Installing chrony:
```
sudo apt-get install chrony
```
Change the configuration file /etc/chrony/chrony.conf to:
```
driftfile /var/lib/chrony/drift
allow
refclock PPS /dev/pps0 lock NMEA
refclock SHM 0 offset 0.5 delay 0.2 refid NMEA noselect
makestep 1 -1
```
The configuration can be tested with the synchronization script provided:
```
python chrony_setup.py
```
NOTE: If the clock has a high initial drift, chrony will slowly slew to the reference, but it may take a long while. To force chrony to update the clock instantly, use. This is done automatically in the chrony_setup.py script.
```
sudo chronyc -a makestep
```
#### Configuring the Realsense Trigger
The realsense camera is triggered externally using the Raspberry PI GPIO pins. A ROS node controls the UTC timestamps at which the images are triggered, and transmitted to the host computer using a ```sensor_msgs::TimeReference``` message. This can be found in this GitHub repository.

## Code
The code will be open-sourced soon


