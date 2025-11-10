# 4. Jetson Development Tutorial

<img class="common_img" src="../_static/media/chapter_1/section_4/media/image3.png" style="width:300px" />

## 4.1 Getting Started

### 4.1.1 Wiring Instruction

This section uses DuPont wires to connect fan modules. For wiring instructions, refer to the figure below:

<img class="common_img" src="../_static/media/chapter_1/section_4/media/image4.png" style="width:500px" />

> [!NOTE]
>
> **Before powering on, ensure that no metal objects are touching the controller. Otherwise, the exposed pins at the bottom of the board may cause a short circuit and damage the controller.**

<table class="docutils-nobg" border="1">
<colgroup>
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
<col style="width: 20%" />
</colgroup>
<tbody>
<tr>
<td colspan="5" style="text-align: center;">
<p><strong>Connection diagram</strong></p>
</td>
</tr>
<tr>
<td style="text-align: center;">
<p><strong>Item</strong></p>
</td>
<td style="text-align: center;">
<p><strong>Pin</strong></p>
</td>
<td style="text-align: center;">
<p><strong>Connection method</strong></p>
</td>
<td style="text-align: center;">
<p><strong>Item</strong></p>
</td>
<td style="text-align: center;">
<p><strong>Pin</strong></p>
</td>
</tr>
<tr>
<td style="text-align: center;">
<p>Jetson Nano</p>
</td>
<td style="text-align: center;">
<p><strong>SCL (pin 5)</strong></p>
</td>
<td style="text-align: center;">
<p>Dupont wire</p>
</td>
<td style="text-align: center;">
<p>Color Sensor</p>
</td>
<td style="text-align: center;">
<p><strong>SCL</strong></p>
</td>
</tr>
<tr>
<td style="text-align: center;">
<p>Jetson Nano</p>
</td>
<td style="text-align: center;">
<p><strong>SDA (pin 3)</strong></p>
</td>
<td style="text-align: center;">
<p>Dupont wire</p>
</td>
<td style="text-align: center;">
<p>Color Sensor</p>
</td>
<td style="text-align: center;">
<p><strong>SDA</strong></p>
</td>
</tr>
<tr>
<td style="text-align: center;">
<p>Jetson Nano</p>
</td>
<td style="text-align: center;">
<p><strong>GND</strong></p>
</td>
<td style="text-align: center;">
<p>Dupont wire</p>
</td>
<td style="text-align: center;">
<p>Color Sensor</p>
</td>
<td style="text-align: center;">
<p><strong>GND</strong></p>
</td>
</tr>
<tr>
<td style="text-align: center;">
<p>Jetson Nano</p>
</td>
<td style="text-align: center;">
<p><strong>5V</strong></p>
</td>
<td style="text-align: center;">
<p>Dupont wire</p>
</td>
<td style="text-align: center;">
<p>Color Sensor</p>
</td>
<td style="text-align: center;">
<p><strong>5V</strong></p>
</td>
</tr>
</tbody>
</table>

### 4.1.2 Environment Configuration

Install NoMachine on your computer. The software package is located under "**[Appendix-> Remote Desktop Connection Tool](https://drive.google.com/drive/folders/1E7j2At6guFY7DP1m_pJMMDWaqk78y1Uq?usp=sharing)**". For the detailed operations of NoMachine, please refer to the the same directory.

Drag the program into the Jetson Nano system image. For demonstration purposes, the files are placed on the Desktop in this example.

## 4.2 Test Case

Program to display the color sensor detection results in the Jetson terminal window.

### 4.2.1 Program Download

1)  Open the terminal and enter the following command to navigate to the program directory:

"**cd Desktop/**", and press Enter

```py
cd Desktop/
```

2. Run the program by entering:

   ```py
   python3 ColorSensorDemo.py
   ```

<img class="common_img" src="../_static/media/chapter_1/section_4/media/image6.png" style="width:500px" />

### 4.2.2 Project Outcome

Place the object in the color sensor's recognition area, the sensor then determines the closest match among red, green, and blue, and prints the result to the terminal.

<img class="common_img" src="../_static/media/chapter_1/section_4/media/image7.png" style="width:500px" />

### 4.2.3 Program Brief Analysis

- **Import Libraries**

```py
import os
import sys
import time
import signal
import smbus
import Jetson.GPIO as GPIO
from apds9960.const import *
from apds9960 import APDS9960


if sys.version_info.major == 2:
    print('Please run this program with python3!')
    sys.exit(0)
```

Import the library files required by the program, including the system file os, time processing function time, GPIO function, and **APDS9960** library files used by the color sensor. Meanwhile, the program will detect the currently used Python version and will issue a prompt if it is not Python 3.

- **Initialization Sequence**

Then set the RGB calibration value of the color sensor and initialize the color sensor through the APDS library, so that the color sensor can set the initial value of color detection.

Set the variable start to True for the color sensor to start detecting, and set the shutdown function. When the terminal receives the shutdown signal, it will set the variable start to False to shut down the detection.

```py
# Calibration values (校准值)
R_W = 2600
G_W = 4400
B_W = 6400
R_B = 120
G_B = 180
B_B = 260

# Color sensor initialization (颜色传感器初始化)
bus = smbus.SMBus(1)
apds = APDS9960(bus)
apds.enableLightSensor()
detect_color = None

start = True
# Pre-processing before closing (关闭前处理)
def Stop(signum, frame):
    global start

    start = False
    print('关闭程序')

signal.signal(signal.SIGINT, Stop)
```

- **Main Function**

In the while loop, use the functions `apds.readRedLight`, `apds.readGreenLight`, and `apds.readBlueLight` to read the channel values of the RGB primary colors.

Next, add the calibration data and use conditional statements to determine which RGB component has the highest proportion, thereby identifying the target color. For example, if the R component is greater than the maximum of the G and B components by more than 40 units, the target is identified as red.

```py
while start:

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
