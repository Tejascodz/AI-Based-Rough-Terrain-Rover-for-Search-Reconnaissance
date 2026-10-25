# AI-Based Rough Terrain Rover for Search and Reconnaissance

A semi-autonomous six-wheel rover for uneven, hazardous terrain. It combines a 3D-printed rocker-bogie
chassis, LiDAR terrain mapping, AI terrain classification and an environmental sensing payload, all monitored
and driven from a live web dashboard over Wi-Fi.

<p align="center">
  <img src="docs/images/rover.jpg" alt="The assembled rover: aluminium frame, six wheels on 3D-printed rocker-bogie arms, electronics bay, camera and LiDAR at the front" width="420">
</p>

Final-year engineering group project (team of 4), 2025–26.

## Highlights

* **Rocker-bogie mobility.** Six independently driven wheels on a custom 3D-printed rocker-bogie suspension
  mounted on a lightweight aluminium frame, with four servos for steering. All wheels stay in contact with the
  ground over rocks and slopes.
* **AI terrain classification.** Pixel-wise segmentation of camera images into soil, bedrock, sand and big rock,
  based on an AI4Mars-style segmentation model with CLAHE contrast enhancement, colour-ratio filters, Sobel
  gradients and LBP texture features. **87.5% overall confidence** within a 30 m range.
* **LiDAR terrain mapping.** A TF-Luna time-of-flight LiDAR builds 3D point clouds and radial depth profiles of
  the surroundings, showing obstacles, slopes and depressions.
* **Environmental payload.** Temperature, humidity, barometric pressure, gas, IMU and GPS, streamed live to the
  operator.
* **Web dashboard.** Manual drive controls with emergency stop, live telemetry (speed, steering, LiDAR distance,
  pitch, roll, signal strength), a 3D suspension posture view and a live LiDAR radar map.
* **Semi-autonomous operation.** Operator control over Wi-Fi combined with LiDAR-assisted obstacle awareness.

## System architecture

```mermaid
flowchart TB
    subgraph Payload["Environmental payload"]
        BARO[Barometric sensor]
        HT[Humidity and temperature]
        GAS[Gas sensor]
        GPS[GPS]
    end
    BARO & HT & GAS & GPS --> ESP8266[ESP8266<br/>sensor hub]
    ESP8266 -- "telemetry over Wi-Fi" --> LAPTOP[Operator laptop<br/>web dashboard]
    LAPTOP -- "drive commands" --> ESP32[ESP32<br/>Wi-Fi control link]
    ESP32 --> MEGA[Arduino Mega<br/>motion control]
    LIDAR[TF-Luna LiDAR] --> MEGA
    MEGA --> MD1[Motor driver 1] --> M12[Motors 1-2]
    MEGA --> MD2[Motor driver 2] --> M34[Motors 3-4]
    MEGA --> MD3[Motor driver 3] --> M56[Motors 5-6]
    MEGA --> SERVO[4 steering servos]
    MEGA <--> JETSON[Jetson Nano<br/>onboard AI, expansion]
    CAM[Camera] --> LAPTOP
```

| Subsystem | Role |
|---|---|
| **Sensing** | An ESP8266 reads the barometric, temperature/humidity, gas and GPS sensors and streams the data to the operator laptop. |
| **Control** | An Arduino Mega runs motion control: it reads the LiDAR, drives six DC motors through three motor drivers and positions four steering servos. |
| **Communication** | An ESP32 carries drive commands from the dashboard to the Arduino Mega over Wi-Fi; the ESP8266 carries telemetry the other way. |
| **Perception** | Camera frames are segmented into terrain classes; LiDAR scans give obstacle distance and surface geometry. |
| **Compute** | An NVIDIA Jetson Nano is integrated for onboard AI and future autonomy. |

## Hardware

| Component | Purpose |
|---|---|
| Custom 3D-printed rocker-bogie chassis on an aluminium frame | Six-wheel suspension for rough terrain |
| 6 × high-torque DC gear motors, 3 × motor drivers | Drive |
| 4 × servo motors | Steering |
| Arduino Mega | Motion control and LiDAR processing |
| ESP32, ESP8266 | Wi-Fi control link, sensor hub |
| NVIDIA Jetson Nano | Onboard AI compute |
| TF-Luna LiDAR | Distance and terrain mapping |
| USB camera | Imaging for terrain classification |
| IMU, GPS | Orientation and localisation |
| Gas, barometric, temperature and humidity sensors | Environmental monitoring |
| Lithium battery pack, buck converters | Power and voltage regulation |

<p align="center">
  <img src="docs/images/chassis_top_view.jpg" alt="Top view of the bare chassis: aluminium frame with the rocker-bogie arms and six wheels" width="320">
</p>

## Results

### Terrain classification

Segmentation of Mars-like terrain imagery into four classes. Colour-based enhancement filters corrected
the confusion between soil and bright sand.

| Terrain type | Share of pixels | Pixel count | Confidence |
|---|---|---|---|
| Soil | 1.29% | 11,849 | 65.5% |
| Bedrock | 51.13% | 468,731 | 84.6% |
| Sand | 32.95% | 302,120 | 94.7% |
| Big rock | 14.63% | 134,100 | 83.6% |
| Beyond 30 m (masked) | 30.04% | 393,600 | – |

**Overall confidence within 30 m: 87.5%.** Sand and bedrock are classified most reliably because of their
distinct colour and texture; large rocks are detected with high confidence, which matters for safe paths.

<p align="center">
  <img src="docs/images/terrain_classification_1.jpg" alt="Terrain classification result: original image, 30 m range mask, enhanced semantic segmentation and confidence map" width="49%">
  <img src="docs/images/terrain_classification_2.jpg" alt="Second terrain classification result with rocks in the foreground" width="49%">
</p>

### LiDAR terrain mapping

The TF-Luna scans produce 3D point clouds (90 and 360 points) that show elevation changes, nearby obstacles
and their approximate distance from the rover.

<p align="center">
  <img src="docs/images/lidar_point_cloud_90.jpg" alt="3D LiDAR point cloud with 90 points" width="45%">
  <img src="docs/images/lidar_point_cloud_360.jpg" alt="3D LiDAR point cloud with 360 points around the rover" width="45%">
</p>

### Mobility

In physical tests the rover crossed loose sand, uneven ground, stones and gravel, small rocks, and inclines
and declines. The rocker-bogie suspension kept all wheels on the ground, absorbed vibration and kept the
sensors steady, so readings stayed usable on rough surfaces.

### Dashboard and telemetry

Environmental data plotted live while driving: temperature, humidity, gas level, pressure and GPS position.

<p align="center">
  <img src="docs/images/dashboard_environment.jpg" alt="Environmental dashboard with live temperature, humidity, gas, pressure and GPS plots" width="90%">
</p>

Drive controls with live telemetry and a 3D view of the rover's posture from IMU pitch and roll:

<p align="center">
  <img src="docs/images/dashboard_control.jpg" alt="Control dashboard: operating mode, speed, steering, LiDAR distance, pitch and roll, direction controls with emergency stop, and a 3D suspension view" width="90%">
</p>

Live LiDAR radar map with an emergency stop for all systems:

<p align="center">
  <img src="docs/images/dashboard_lidar.jpg" alt="Live LiDAR mapping panel with radar view and emergency stop" width="90%">
</p>

## Current status and limitations

* The rover is **semi-autonomous**: driving is operator-controlled over Wi-Fi, with LiDAR-assisted obstacle
  awareness. Full autonomous navigation is future work.
* The Jetson Nano is integrated, but its role in this build is minimal because of hardware issues; motion
  control runs on the Arduino Mega.
* The terrain classification results above are on Mars-like terrain imagery.
* The dashboard can also run on simulated data for testing without the rover.

## Future work

* Move terrain classification onto the Jetson Nano for fully onboard perception.
* Autonomous path planning from combined LiDAR geometry and terrain classes.
* SLAM for mapping and localisation in GPS-denied areas.
* Solar charging for longer missions.

## Applications

Search and rescue, disaster-area assessment, hazardous site inspection and planetary surface simulation.

## Recognition

* Rank 11 of 1,054 teams across SAARC, Vishwakarma Awards 2025 (Smart Mobility)
* Presented at CICCS-25

## Author

**Tejas S** · [GitHub](https://github.com/Tejascodz) · [LinkedIn](https://www.linkedin.com/in/tejas-s-57298325a/)

Built as a final-year group project by a team of four.
