# YOLOV4-tiny-vehicle-detection-on-Jetson-Nano
 Train and setup YOLOv4-tiny model on Jetson Nano to detect Vietnamese Vehicle
## 1. Train model
Open file **yolov4-tiny-training-Vehicle-dectection.ipynb** and then do step by step in file to train Custom YOLOv4-tiny for Vehicle Detection
## 2. Setup on Jetson Nano B01
Since Jetson Nano have a fixed flash (~16 GB), it's not able to install or store any more data. I need to Flash the OS on Jetson Nano to external USB (recommend at least 64GB) and change default boot to USB.
About how to to Flash and Boot from USB, check guidline in .pdf
### Install docker on Jetson Nano
I recommend to use docker instead of install packages directly on Jetson. Because of the following reasons:
* Ubuntu on Jetson Nano is not an official version released by Ubuntu, it is customize for Jetson Nano only by NVIDIA. When you try to install python package or Ubuntu package on Jetson, you will face with many compatible errors during installing process (some packages cannot find compatible version to download).
* Version update: Package version is a flustrated problem for python and Ubuntu software. When you install a package/software, it automaticall installs dependency packages. It may cause conflict with other packages dependency.
* Docker for Jetson Nano is customised by NVIDIA official. It's highly compatible with Jetson Nano and easy to set up. It reduces your headache during setup process
