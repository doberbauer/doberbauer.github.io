Title: Toy Camera
Date: 2023-06-24 00:00:00
Category: Exploration


### What is Toy Camera?

Toy Camera is a Raspberry Pi Zero W / OpenCV / Python project. It's a low resolution camera that can take pictures with a handful of filters. It's front-facing so you can take selfies and whatnot.
 
### Why did you make it?

- I wanted to
- I had the parts
- It was a fun Python exercise
- My Pi Zero W wasn't being used


### What parts did you use?

- Raspberry Pi Zero W
- 32 GB microSD
- Adafruit 1.8in TFT
- Raspberry Pi Camera module (or third-party raspberry pi compatible camera)
- Camera cable
- Case

(Future)
- Battery
- battery charging circuit 


### How do I install this?

I'm using Raspberry Pi OS Buster Lite (no desktop). I haven't tried other OS versions. Installing an OS is beyond the scope of this article.

Let's start with updating apt, installing pip, git and PIL.  
``` Terminal
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install python3-pip
sudo apt-get install git
sudo apt-get install python3-pil
```

``` Terminal
sudo pip3 install --upgrade setuptools
cd ~
sudo pip3 install --upgrade adafruit-python-shell
wget https://raw.githubusercontent.com/adafruit/Raspberry-Pi-Installer-Scripts/master/raspi-blinka.py
sudo python3 raspi-blinka.py

pip3 install adafruit-circuitpython-rgb-display
pip3 install --upgrade --force-reinstall spidev
sudo apt-get install fonts-dejavu

sudo pip3 install opencv-python --no-cache-dir

# If pip doesn't work
sudo apt install cmake build-essential pkg-config git
sudo apt install libjpeg-dev libtiff-dev libjasper-dev libpng-dev libwebp-dev libopenexr-dev
sudo apt install libavcodec-dev libavformat-dev libswscale-dev libv4l-dev libxvidcore-dev libx264-dev libdc1394-22-dev libgstreamer-plugins-base1.0-dev libgstreamer1.0-dev
sudo apt-get install python3-numpy
sudo apt install libgtk-3-dev libqtgui4 libqtwebkit4 libqt4-test python3-pyqt5
sudo apt install libatlas-base-dev liblapacke-dev gfortran
sudo apt install libhdf5-dev libhdf5-103
sudo apt install python3-dev python3-pip python3-numpy
git clone https://github.com/opencv/opencv.git
git clone https://github.com/opencv/opencv_contrib.git
```

### How does it work? 

**Overview**

The Adafruit TFT for Raspberry Pi has two buttons. I've set it up so the left button takes a picture and the right button changes mode. Pressing both buttons shuts the Pi off.

TFT Display Definitions
Code adapted from ladyada of Adafruit Industries.
``` Python
# SPDX-FileCopyrightText: 2021 ladyada for Adafruit Industries
# SPDX-License-Identifier: MIT
import sys
sys.path.append('/home/pi/.local/lib/python3/site-packages')
import digitalio
import board
from adafruit_rgb_display import st7789
from PIL import Image, ImageDraw, ImageFont
#from adafruit_rgb_display.rgb import color565
import time
import cv2
import numpy as np
import os

''' Display definitions '''
# Configuration for CS and DC pins for Raspberry Pi
cs_pin = digitalio.DigitalInOut(board.CE0)
dc_pin = digitalio.DigitalInOut(board.D25)
reset_pin = None
BAUDRATE = 64000000  # The pi can be very fast!
# Create the ST7789 display:
display = st7789.ST7789(
    board.SPI(),
    cs=cs_pin,
    dc=dc_pin,
    rst=reset_pin,
    baudrate=BAUDRATE,
    width=240,
    height=240,
    x_offset=0,
    y_offset=80,
)
backlight = digitalio.DigitalInOut(board.D22)
backlight.switch_to_output()
backlight.value = True
buttonA = digitalio.DigitalInOut(board.D23)
buttonB = digitalio.DigitalInOut(board.D24)
buttonA.switch_to_input()
buttonB.switch_to_input()

''' Image Definitions '''

x = 0
y = 0
height = display.width  # swap height/width to make it landscape!
width = display.height
image = Image.new("RGB", (width, height))
rotation = 270
text_rotation = 90
text_x = 0
text_y = 200
draw = ImageDraw.Draw(image)
draw.rectangle((0, 0, width, height), outline=0, fill=(0, 0, 0)) #Clear screen
display.image(image, rotation)
font = ImageFont.truetype("/usr/share/fonts/truetype/dejavu/DejaVuSans.ttf", 24)

''' Camera definitions '''

cap = cv2.VideoCapture(0)
dim = (240,180)
cap_w = 640
cap_h = 480
cap.set(cv2.CAP_PROP_FRAME_WIDTH, cap_w)
cap.set(cv2.CAP_PROP_FRAME_HEIGHT, cap_h)

''' Mode definitions '''

current_mode = 0
modes = ["Default","Pixel","BW","Negative","Sepia","Sketch","Water color","Detect Face","Psychedelic"] #Mode str len <= x chars
global face_cascade
face_cascade = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')
''' Cycle Mode Function '''

def cycle_mode(_current_mode,_modes):
    if _current_mode == len(_modes)-1:
        _current_mode = 0  #reset it to zero
    else:
        _current_mode += 1
    return _current_mode

def process_frame(frame,current_mode,modes):
    mode = modes[current_mode]
    if mode == modes[0]: #Default
        pass
    elif mode == modes[1]: #Pixel
        frame = mode_pixel(frame)
    elif mode == modes[2]: #BW
        frame = mode_bw(frame)
    elif mode == modes[3]: #Negative
        frame = mode_negative(frame)
    elif mode == modes[4]: #Sepia
        frame = mode_sepia(frame)
    elif mode == modes[5]: #Sketch
        frame = mode_sketch(frame)
    elif mode == modes[6]: #Water color
        frame = mode_wc(frame)
    elif mode == modes[7]: #Detect Face
        frame = mode_face(frame)
    elif mode == modes[8]: #Psychedelic
        frame = mode_psych(frame)
    else:
        pass
    return frame
''' --- MODES --- '''

''' Pixel Mode '''
def mode_pixel(frame):
    fh, fw, _ = frame.shape
    h, w = (16, 16)
    temp = cv2.resize(frame, (w, h), interpolation=cv2.INTER_LINEAR)
    return cv2.resize(temp, (fw, fh), interpolation=cv2.INTER_NEAREST)
''' BW Mode '''
def mode_bw(frame):
    frame = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    return frame

''' Negative Mode '''
def mode_negative(frame):
    frame = cv2.bitwise_not(frame)
    return frame

''' Sepia Mode '''
def mode_sepia(frame):
    frame = np.array(frame, dtype=np.float64)
    frame = cv2.transform(frame, np.matrix([[0.272, 0.534, 0.131],
                                    [0.349, 0.686, 0.168],
                                    [0.393, 0.769, 0.189]])) # multipying image with special sepia matrix
    frame[np.where(frame > 255)] = 255 # normalizing values greater than 255 to 255
    frame = np.array(frame, dtype=np.uint8) # converting back to int
    return frame

''' Sketch Mode '''
def mode_sketch(frame):
    _, frame = cv2.pencilSketch(frame, sigma_s=60, sigma_r=0.07, shade_factor=0.05)
    return frame

''' Water Color Mode '''
def mode_wc(frame):
    frame = cv2.stylization(frame, sigma_s=60, sigma_r=0.07)
    return frame

''' Face Detection Mode '''
def mode_face(frame):
    gray = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    faces = face_cascade.detectMultiScale(gray, scaleFactor=1.05, minNeighbors=5, minSize=(30, 30))
    print(len(faces))
    for (x,y,w,h) in faces:
        cv2.rectangle(frame,(x,y),(x+w,y+h),color=(255,255,0),thickness=2)
    return frame
''' Psychedelic Color Mode '''
def mode_psych(frame):
    img_hsv = cv2.cvtColor(frame,cv2.COLOR_BGR2HSV)
    n_shift = 180 // 2
    img_hsv[:,:,0] = (img_hsv[:,:,0].astype(int) + n_shift) % 181
    frame = cv2.cvtColor(img_hsv, cv2.COLOR_HSV2BGR)
    return frame

''' Main loop '''

while True:
    if buttonA.value and buttonB.value: # none pressed
        # Viewfinder
        
        _, frame = cap.read()
        frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        frame = cv2.resize(frame,dim)
        frame = Image.fromarray(frame)
        frame = frame.transpose(Image.FLIP_LEFT_RIGHT)
        display.image(frame,rotation)
        
    if buttonB.value and not buttonA.value:  # just button A pressed
        #Take picture and display result for 5 seconds
        
        _, frame = cap.read()
        draw.rectangle((0, 0, width, height), outline=0, fill=0)
        draw.text((text_x, text_y), "Processing...", font=font, fill="#FFFFFF")
        display.image(image, text_rotation)
        frame = process_frame(frame,current_mode,modes)
        timestr = time.strftime("%Y-%m-%d--%H-%M-%S")
        filename = "pics/" + modes[current_mode] + " " + timestr + ".jpg"
        out = cv2.imwrite(filename, frame)
        
        draw.rectangle((0, 0, width, height), outline=0, fill=0)
        draw.text((text_x, text_y), "Done!", font=font, fill="#FFFFFF")
        display.image(image, text_rotation)
        frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
        frame = cv2.resize(frame,dim)
        frame = Image.fromarray(frame)
        display.image(frame, rotation) #Display it
        time.sleep(5)   # Wait for a few seconds (ms?) 
        draw.rectangle((0, 0, width, height), outline=0, fill=0) #Clear screen
        draw.text((text_x,text_y), modes[current_mode], font=font, fill="#FFFFFF") 
        display.image(image, text_rotation) #Display the current mode
        
    if buttonA.value and not buttonB.value:  # just button B pressed
        #Change Mode
        
        draw.rectangle((0, 0, width, height), outline=0, fill=0)
        current_mode = cycle_mode(current_mode,modes)
        draw.text((text_x, text_y), modes[current_mode], font=font, fill="#FFFFFF")
        display.image(image, text_rotation)
        time.sleep(0.5)
        
    if not buttonA.value and not buttonB.value:  #both pressed
        os.system('sudo showdown now')
        break
cap.release()
draw.rectangle((0, 0, width, height), outline=0, fill=(0, 0, 0)) #Clear screen
display.image(image, rotation)
backlight.value = False
quit()
```

## Run at Boot

I used cron to schedule toycam at boot.
"crontab -e" is the command to bring up the cron file to edit
`@reboot python3 /home/pi/toycam.py &`

## Improvements

- 3D printed case
- Battery
- Sleep mode?
- More filter modes
- On/off switch 
- MeanShift segmentation mode




