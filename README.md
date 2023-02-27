# SATEL CA-6 to MQTT

Firmware for an ESP controller to monitor a Satel CA-6 P alarm control panel
Tested with an M5-Stack ATOM

## Background:

The [Satel CA-6 P](https://www.satel.pl/en/products/intruder-alarms/ca-series/ca-6/pcb/ca-6-p/) alarm control panel is a quite old intruder alarm system with limited communication capabilities. It was built for a different era, when telephone line communication was high tech. Unlike the CA-10 it hasn't even got event log printing capabilities through its RS232 port that has been used by another [project](https://github.com/voyo/satel2mqtt).
But fortunately it communicates with its keypad in a really simple way. The keypad shows the current state of connected zones by 8 LEDs, controlled by two HCF4021 (8-Stage Static Shift Register). Ref.: https://blog.uaid.net.ua/satel-ca-6-kled/

The keypad has a 4 wire connection to the control panel:

  **+KPD**: +12V ~ +16V DC    
   **COM**: Common Ground     
   **CLK**: Clock Signal     
  **DATA**: Data Signal  
  
## Hardware:

Due to the 12V setup of the alarm control panel, the CLK and DATA lines also have 12V signal. This 12V signal first have to be shifted to 3.3V to use it with an ESP. 
For this a [Pololu 2595](https://www.pololu.com/product/2595) can be used, because it can accept as high as 18 V on the higher-voltage side.

The **+KPD** has to be connected to its HV input, **CLK** and **DATA** to any of the inputs on the HV side of the module (H1-H4).

On the low voltage side of the module LV should be connected to the 3.3V output of the ESP module and the respective L1-L4 should be connected to two GPIO pins of the ESP. Use pins which are not strapping pins, so they will not be pulled down or up during a reset.

In the code, PIN 33 and 19 has been used with a M5-Stack ATOM (ESP Pico).

COM should be connected to the GND of the ESP.

## Data Signal:

The alarm control panel communicates with the keypad by a clock synced data signal. The clock has a frequency of 2kHz, and runs for 16 periods together with the data signal, giving 32bits of data. This 32bits is decoded by the ESP and transmitted to MQTT broker as an array of zeros and ones.

Bit  1: ?  
Bit  2: ?  
Bit  3: ?  
Bit  4: ?  
Bit  5: ?  
Bit  6: ?  
Bit  7: ?  
Bit  8: ?  
Bit  9: ?  
Bit 10: ?  
Bit 11: ?  
Bit 12: 1 when Zone 7 LED is active  
Bit 13: ?  
Bit 14: 1 when Zone 6 LED is active  
Bit 15: ?  
Bit 16: 1 when Zone 5 LED is active  
Bit 17: ?  
Bit 18: ?  
Bit 19: ?  
Bit 20: 1 when Trouble LED is active  
Bit 21: 1 when Zone 7 LED is active  
Bit 22: 1 when Telephone LED is active  
Bit 23: ?  
Bit 24: 1 when Power LED is active  
Bit 25: ?  
Bit 26: 1 when Zone 3 LED is active  
Bit 27: ?  
Bit 28: 1 when Zone 4 LED is active  
Bit 29: ?  
Bit 30: 1 when Zone 2 LED is active  
Bit 31: ?  
Bit 32: 1 when Zone 1 LED is active  

Unfortunately, I couldn't identify Zone 8 as I haven't got any sensor connected to that one. Also, there are 4 partition LEDs as well that hasn't been idetentified yet.

## Integrating into Home Assistant

Home Assistant Manually configured MQTT Binary Sensors:

```
mqtt:
  binary_sensor:
    # Zone 1
    - name: SATEL Zone 1
      state_topic: "satel/alarm/raw_data"
      value_template: >
        {% if value_json[31] | int == 1 %}
        ON
        {% elif value_json[31] | int == 0 %}
        OFF
        {% endif %}
      device_class: door
      
     # Zone 2
    - name: SATEL Zone 2
      state_topic: "satel/alarm/raw_data"
      value_template: >
        {% if value_json[29] | int == 1 %}
        ON
        {% elif value_json[29] | int == 0 %}
        OFF
        {% endif %}
      device_class: motion
      
     # Zone 3
    - name: SATEL Zone 3
      state_topic: "satel/alarm/raw_data"
      value_template: >
        {% if value_json[25] | int == 1 %}
        ON
        {% elif value_json[25] | int == 0 %}
        OFF
        {% endif %}
      device_class: motion
      
     # Zone 4
    - name: SATEL Zone 4
      state_topic: "satel/alarm/raw_data"
      value_template: >
        {% if value_json[27] | int == 1 %}
        ON
        {% elif value_json[27] | int == 0 %}
        OFF
        {% endif %}
      device_class: motion
      
    # Zone 5  
    - name: SATEL Zone 5
      state_topic: "satel/alarm/raw_data"
      value_template: >
        {% if value_json[15] | int == 1 %}
        ON
        {% elif value_json[15] | int == 0 %}
        OFF
        {% endif %}
      device_class: motion
      
    # Zone 6
    - name: SATEL Zone 6
      state_topic: "satel/alarm/raw_data"
      value_template: >
        {% if value_json[13] | int == 1 %}
        ON
        {% elif value_json[13] | int == 0 %}
        OFF
        {% endif %}
      device_class: motion
      
    # Zone 7
    - name: SATEL Zone 7
      state_topic: "satel/alarm/raw_data"
      value_template: >
        {% if value_json[20] | int == 1 %}
        ON
        {% elif value_json[20] | int == 0 %}
        OFF
        {% endif %}
      device_class: door
```
