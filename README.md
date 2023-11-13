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

Log on Jetson Nano over SSH or execute Jetson Terminal
Download and install docker from NVIDIA official github: https://github.com/dusty-nv/jetson-containers.git
#### Set docker default runtime
You need to set Docker's default-runtime to nvidia, so that the NVCC compiler and GPU are available during docker build operations. Add **"default-runtime": "nvidia" to your /etc/docker/daemon.json** configuration file before attempting to build the containers:
```
{
    "runtimes": {
        "nvidia": {
            "path": "nvidia-container-runtime",
            "runtimeArgs": []
        }
    },

    "default-runtime": "nvidia"
}
```
Then restart the Docker service, or reboot your system before proceeding:
```
sudo systemctl restart docker
```
You can then confirm the changes by looking under docker info
```
sudo docker info | grep 'Docker Root Dir'
Docker Root Dir: /mnt/docker
...
Default Runtime: nvidia
```
#### Add Swap
If you're building containers or working with large models, it's advisable to mount SWAP (typically correlated with the amount of memory in the board). Do following steps:
```
cd ${HOME}
git clone https://github.com/JetsonHacksNano/installSwapfile.git
cd installSwapfile
./installSwapfile.sh
```
After that, check swap by 
```
free -m
```
#### Clone Docker Container Repo
```
sudo apt-get update && sudo apt-get install git python3-pip
git clone --depth=1 https://github.com/dusty-nv/jetson-containers
cd jetson-containers
pip3 install -r requirements.txt
```
#### Run container
I use lt4-ml container for this project since it has already contain: pytorch, tensorflow, pycuda and everything else I need to develop a deep learning application
```
mkdir ${HOME}/project
./run.sh -v ${HOME}/project:${HOME}/project dustynv/l4t-ml:r32.7.1
```
Choose to use version r32.7.1 since I'm using Jetpack 4.6
#### Setup custom YOLOv4-tiny TensorRT on Jetson Nano
In l4t-ml container environment, setup YOLOv4-tiny TensorRT.
**Note**: fiores is my user in Jetson Nano
```
cd /home/fiores/project
git clone https://github.com/jkjung-avt/tensorrt_demos.git
cd tensorrt_demos
```
Install a "yolo_layer" plugin to speed up inference time of the yolov3/yolov4 models.
```
cd /home/fiores/project/tensorrt_demos/plugins
make
```
Download the pre-trained customed YOLOv4-tiny weights, and config file. Rename these 2 files as same name (Ex yolov4-tiny-custom.weights, yolov4-tiny-custom.cfg) Then convert to ONNX and then TensorRT
```
cd /home/fiores/project/tensorrt_demos/yolo
python3 yolo_to_onnx.py -m yolov4-tiny-custom
python3 onnx_to_tensorrt.py -m yolov4-tiny-custom
```
For additional python package that required by application. Install by pip3 command: 
```
pip3 install matplotlib
```
Test the TensortRT yolov4-tiny engine with image
```
cd /home/fiores/project/tensorrt_demos/
python3 trt_yolo.py --image yolo/vehicle1.jpg \
                      -m yolov4-tiny-custom
```
