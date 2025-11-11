# 3. Raspberry Pi Development Tutorial

<img class="common_img" src="../_static/media/chapter_1/section_3/media/image3.png" style="width:300px" />

## 3.1 Getting Started

### 3.1.1 Wiring Instruction

This section provides an example of connecting the color sensor module with Dupont wires. Connect the module to the following pins: 5V, GND, SDA (GPIO24), and SCL (GPIO22). The wiring method is illustrated in the figure below.

<img class="common_img" src="../_static/media/chapter_1/section_3/media/image4.png" style="width:500px" />

> [!NOTE]
>
> * **When using Hiwonder's lithium battery, connect the battery cable with the red wire to the positive (+) terminal and the black wire to the negative (–) terminal of the DC port.**
>
> * **If the battery is not connected to the cables, do not connect the cable ends directly together. Doing so may cause a short circuit and damage the system.**

### 3.1.2 Environment Configuration

Install NoMachine on your computer. The software package is located under "**[Appendix-> Remote Desktop Connection Tool](https://drive.google.com/drive/folders/1E7j2At6guFY7DP1m_pJMMDWaqk78y1Uq?usp=sharing)**". For the detailed operations of NoMachine, please refer to the same directory.

Drag the program and SDK library files into the Raspberry Pi system image. For demonstration purposes, the files are placed on the Desktop in this example.

> [!NOTE]
>
> **Make sure the library files are placed in the same directory as the program.**

Open the terminal and enter the command to change to the program directory: 

```bash
sudo chmod a+x Sensor_Demo/
```

## 3.2 Test Case

In this case, the RGB sensor is used to detect the corresponding color, and the detected color name is displayed in the terminal window.

### 3.2.1 Program Download

1. Open the terminal and enter the command to navigate to the program directory, enter: **cd Desktop/Sensor_Demo/**, then press Enter.

```bash
cd Desktop/Sensor_Demo/
```

2. To run this example program, enter: 

```bash
python3 ColorSensorDemo.py
```

### 3.2.2 Project Outcome

Aim the color sensor at objects in red, green, and blue respectively. The sensor detects each color and displays the corresponding color name on the monitor.

### 3.2.3 Program Brief Analysis

- **Import Libraries**

```py
#!/usr/bin/env python3
import os
import sys
import time
import signal
import smbus
import RPi.GPIO as GPIO
#import Board
from apds9960.const import *
from apds9960 import APDS9960
```

- **Check the running Python version**

```py
if sys.version_info.major == 2:
    print('Please run this program with python3!')
    sys.exit(0)
```

Check if the running Python version is 3.0 or higher.

If yes, the program runs normally; if not, it prints a message via the `print()` function and exits.

- **Define Color Calibration Values**

Describe the saturation of the red, green, and blue colors.

```py
# Calibration values (校准值)
R_W = 2600
G_W = 4400
B_W = 6400
R_B = 120
G_B = 180
B_B = 260
```

Initialize the color sensor.

```py
bus = smbus.SMBus(1)
apds = APDS9960(bus)
apds.enableLightSensor()
detect_color = None
```

Initialize communication with the APDS9960 color sensor and enable the color sensor function.

Detect interrupt signal

```py
def Stop(signum, frame):
    global start

    start = False
    print('Closing...')
```

When "**Ctrl+C**" is pressed to interrupt the program, it prints "**Shutting down...**" and sets the global variable start to False.

Color Detection

```py
while True:

    # Read the three color channel values (读取三个颜色通道值)
    red = apds.readRedLight()
    green = apds.readGreenLight()
    blue = apds.readBlueLight()
    
    # Add calibration (加入校准)
    r = abs(int((red - R_B)*255/(R_W - R_B)))
    g = abs(int((green - G_B)*255/(G_W - G_B)))
    b = abs(int((blue - B_B)*255/(B_W - B_B)))
    
    # Determine color (判别颜色)
    if r - max(g, b) > 40:
        detect_color = 'red'
    elif g - max(r, b) > 40:
        detect_color = 'green'
    elif b - max(r, g) > 40:
        detect_color = 'blue'
    else:
        detect_color = None
    print(detect_color)
    
    time.sleep(0.2)
```

1. Use the method of **APDS9960** library to read the red, green, and blue channel values.

   ```py
       # Read the three color channel values (读取三个颜色通道值)
       red = apds.readRedLight()
       green = apds.readGreenLight()
       blue = apds.readBlueLight()
   ```

2. Calibrate and adjust the range of the red, green, and blue color channel values.

   ```py
       # Add calibration (加入校准)
       r = abs(int((red - R_B)*255/(R_W - R_B)))
       g = abs(int((green - G_B)*255/(G_W - G_B)))
       b = abs(int((blue - B_B)*255/(B_W - B_B)))
   ```

3. Determine the dominant color in the current environment based on the values of the three color channels.

   ```py
       # Determine color (判别颜色)
       if r - max(g, b) > 40:
           detect_color = 'red'
       elif g - max(r, b) > 40:
           detect_color = 'green'
       elif b - max(r, g) > 40:
           detect_color = 'blue'
   ```

4. Print the detected color or None. So you can identify the dominant color in the current environment.

   ```py
       else:
           detect_color = None
       print(detect_color)
   ```

5. Pause the code for 0.2 seconds before repeating, in order to control the detection frequency.

   ```py
       time.sleep(0.2)
   ```

   
