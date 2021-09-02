## SynchroSem

## Introduction
We present SynchroSem: a loosely coupled multi-modal semantic SLAM approach for synchronized Lidar-Visual-Inertial (LVI) systems. Our system integrates pose measurements from two different tightly coupled odometry systems, Visual-Inertial (VI) and Lidar-Inertial (LI). These are loosely coupled by means of a pose graph representation that enables a robust late-fusion of pose measurements, being resilient to geometric and visual degeneracy. The developed multi-modal semantic feature extraction allows the global optimization system to reduce long term operation errors via loop closure, achieving State-of-the-Art performance on public LVI datasets.
Our method has also been deployed on a custom robotics platform. We developed a simple and reproducible hardware synchronization system using commercial-off-the-shelf hardware, and it is made publicly available at https://halops.github.io/SynchroSem/.

### Videos
**[TODO: ADD EMBEDDED VIDEOS]**

## Hardware setup
The hardware platform for this project consists of a Clearpath Jackal unmanned ground vehicle (UGV), coupled with the Scaled Robotics sensor rig: a platform containing 2x Velodyne VLP-16 Lidars, 1x Realsense D435i stereo-inertial camera, 1x Microstrain 3DM-GX-45 high rate IMU, as well as 3x fisheye rolling shutter cameras with a 360º field of view (FOV). Moreover, the Jackal UGV has been equipped with an Intel NUC 10i7FNH, a powerful on-board computer with an Intel i7-1071OU processor.
**[TODO: ADD JACKAL PICTURE]**

## Sensor synchronization
### Theoretical basis
The hardware synchronization implemented in this work uses a **Raspberry PI 4B** as a central device, which is able to synchronize its internal clock to an external master device that is used as a reference. This reference is then transmitted to the other relevant sensors in the system, as shown in the picture below. The procedure for synchronization is as follows: **[TODO: INSERT PICTURE]**
1. The **reference** external clock is provided by a **Garmin GPS 18x LVC**, which provides a Pulse-Per-Second (**PPS**) signal as well as a serial interface for sending **NMEA** messages, which contain position and timing information. These two signals are interfaced via a **custom PCB** to both Velodyne sensors, as well as to the Raspberry PI. The Velodyne sensors contain specific firmware to handle PPS+NMEA synchronization signals, while on the Raspberry PI end we process the PPS signal using the Raspberry PI's GPIO pins and parse the NMEA strings through serial communication by using off the shelf tools (GPSd + Chrony).
2. Once the Raspberry PI has its own internal clock synchronized to the external master reference, it now transmits this timing information to the main computer, the Intel NUC. This synchronization is also handled using Chrony, making use of the NTP protocol through an Ethernet sub-network to periodically synchronize the NUC's clock to the reference. 
3. The Raspberry PI also generates the trigger signals for the Realsense VIO camera, issuing pulses at a user-defined frequency using the GPIO pins. Moreover, the Raspberry PI transmits the firing time through the Ethernet sub-network using ROS messages, for the NUC to be able to match the sensor data to its corresponding firing time.

### Hardware implementation
For the hardware implementation, we need to physically interface all the connections shown in the block diagram shown above by means of a main connection PCB and a series of different cables and connectors. These are detailed below.
#### Main connection PCB
![PCB schematic](/docs/assets/pcb_schematic.png)
#### GPS IN --> Garmin
#### GPS OUT --> Velodyne
#### TRIGGER OUT --> Realsense
#### RPI - Host PC

### Software configuration

## Code
The code will be open-sourced soon™


