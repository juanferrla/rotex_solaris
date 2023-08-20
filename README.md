# Monitoring Solar Heating System (Solaris RPS3) with ESP32, ESPHome and Home Assistant

The objetive of this POC is to monitor the old Rotex Solaris Heating System in Home Assistant, in real time, throught serial communication with an ESP32 board.

<img width="701" alt="image" src="https://github.com/juanferrla/rotex_solaris/assets/353433/052c74ea-6a6c-4518-9b65-d93741395988">


# Material

* Jack 3.5mm male connector (5€) to connect with RSP3 módule of Rotex Solaris
* ESP32 module (9€) to comunicate Rotex Solaris with Home Assistant
* Two resistences to shift voltage from 5v (Rotex Solaris) to 3.3v (ESP32)

# Connection & Configuration

## Jack Stereo 3.5mm and ESP32

  |Solaris RPS3|Description   |ESP32  |
  |------------|--------------|-------|
  |TIP         | RPS3 TX      |GPIO16 |
  |RING        | RPS3 RX      |Not neccesary|
  |SLEEVE      | GND          |GND    |

  Rotex Solaris generates 5v and we need to transform this 5V into 3.3v to avoid damaging the ESP32 board

  <img width="473" alt="lower shifter voltage" src="https://github.com/juanferrla/rotex_solaris/assets/353433/45eadc20-91be-41d7-a1a2-cab54835e3ec">

## Rotex Configuration

It is necesary activate Serial Comunication in RPS3 module. You must enter the code of technical user, in my case 0110 (I think that's default technical password). The following parameters are which i have setted, you can choose other in function of serial comunitations baudrate or cicle. 

  |Solaris RPS3|Description   |
  |------------|--------------|
  |cicle /s    | 15s          |
  |expedient   | AD-232       |      
  |baudrate    | 19200        |
  |direction   | 255          |

# Coding

Has been used ESPHome that is a system to control the ESP8266/ESP32 boards and control them throught Home Automation systems, in this case Home Assistant (HA).

The Rotex Solaris data will be sent to HA by WiFI in local network by two ways; direct API integration with HA and MQTT.
The implementation is simple, Rotex Solaris send every configurated time the data all together sepatared with semicolon. We need to specify UART port and parse the data using a simple Custom UART Component of ESPHome with and export the data parsed with a lambda function to propagate the data to HA. 

In HA has been defined a picture element mapping in the image the metrics captured. The picture was made with draw.io.

  |Metric|Reference   |Type     |
  |------------|--------------|-----|
  |Ha Manual to Automatic return (I/min)    | 1         |float|
  |BK Burner Contact Enable  | 2      |      bool|
  |P1 Circulation Pump Rate   | 3        |float|
  |P2 Booster Pump Enabled  | 4          |bool|
  |TK Collector Temp (°C)  | 5          |float|
  |TR Return Temp (°C)  | 6         |float|
  |TS Storage Temp (°C) | 7          |float|
  |TV Flow Temp (°C) | 8         |float|
  |V Flow Rate (I/min) | 9          |float|
  




# Home Assistant

## Metrics

```
    - sensor:
      - name: "Manual to Automatic return (Ha)"
        unit_of_measurement: "l/min"
        icon: "mdi:water-percent"
        state: > 
          {% set Ha = states('sensor.solaris_ha') | float %}
          {{ (Ha) | round(1, default=0) }}
    - sensor:
      - name: "Burner Contact Enable (BK)"
        icon: "mdi:engine"
        state: > 
          {% set Bk = states('sensor.solaris_bk') | int %}
          {{ (Bk) }}
    - sensor:
      - name: "Circulation Pump Rate (P1)"
        unit_of_measurement: "%"
        icon: "mdi:lightning-bolt"
        state: > 
          {% set P1 = states('sensor.solaris_p1') | float %}
          {{ (P1) }}
    - sensor:
      - name: "Booster Pump Enabled (P2)"
        icon: "mdi:lightning-bolt"
        state: > 
          {% set P2 = states('sensor.solaris_p2') | float %}
          {{ (P2) }}
    - sensor:
      - name: "Return Temp (Tr)"
        unit_of_measurement: "°C"
        device_class: "temperature"
        state: > 
          {% set Tr = states('sensor.solaris_tr') | float %}
          {{ (Tr) | round(1, default=0) }}
    - sensor:
      - name: "Storage Temp (Ts)"
        unit_of_measurement: "°C"
        device_class: "temperature"
        state: > 
          {% set Ts = states('sensor.solaris_ts') | float %}
          {{ (Ts) | round(1, default=0) }}
    - sensor:
      - name: "Flow Temp (Tv)"
        unit_of_measurement: "°C"
        device_class: "temperature"
        state: > 
          {% set Tv = states('sensor.solaris_tv') | float %}
          {{ (Tv) | round(1, default=0) }}
    - sensor:
      - name: "Flow Rate (V)"
        icon: "mdi:engine"
        unit_of_measurement: "l/min"
        state: > 
          {% set Fv = states('sensor.solaris_v') | float %}
          {{ (Fv) | round(2, default=0) }}
```

## Presentation
Using the draw.io image, it was created a picture card with state labels to paint the metrics as it's shown in the following:
<p align="center">
<img width="442" align-center src="https://github.com/juanferrla/rotex_solaris/assets/353433/ae67915f-6e29-415d-802f-8d2d3015a380">
<br />

<img width="442" src="https://github.com/juanferrla/rotex_solaris/assets/353433/af69bd3a-6ad6-4aff-8cfa-5c479b75ea8b">
</p>

```
type: picture-elements
elements:
  - type: state-label
    width: 50%
    height: 50%
    entity: sensor.storage_temp_ts
    style:
      top: 58.6%
      left: 72%
      font-weight: bold
  - type: state-label
    width: 50%
    height: 50%
    entity: sensor.collector_temp_tk
    style:
      top: 4.6%
      left: 59%
      font-weight: bold
  - type: state-label
    width: 50%
    height: 50%
    entity: sensor.flow_temp_tv
    style:
      top: 30%
      left: 75%
      font-weight: bold
  - type: state-label
    width: 50%
    height: 50%
    entity: sensor.return_temp_tr
    style:
      top: 83.5%
      left: 50%
      font-weight: bold
  - type: state-label
    width: 50%
    height: 50%
    entity: sensor.circulation_pump_rate_p1
    style:
      top: 56%
      left: 30%
      font-weight: bold
image: local/rotex.png
```

