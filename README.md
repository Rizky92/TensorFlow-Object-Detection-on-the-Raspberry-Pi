# Tutorial to set up TensorFlow Object Detection API on the Raspberry Pi

TO DO before this guide is complete:
1. Upload Object_detection_picamera.py
2. Add pictures to guide
3. Add link to video version of this guide

These should all be finished by June 18th!

## Introduction
This guide provides step-by-step instructions for how to set up TensorFlow’s Object Detection API on the Raspberry Pi. By following the steps in this guide, you will be able to use your Raspberry Pi to perform object detection on live video feeds from a Picamera or USB webcam. Combine this guide with my <link> tutorial on how to train your own neural network to identify specific objects</link>, and you use your Pi for unique detection applications such as:

* Letting you know when your cat wants to be let inside or outside :smiley_cat:
* Telling you if there are any parking spaces available in front of your apartment building
* [Beehive bee counter](http://matpalm.com/blog/counting_bees/)
* [Counting cards at the blackjack table?](https://hackaday.io/project/27639-rainman-20-blackjack-robot)
* And anything else you can think of!

*Picture of kitty cat detector coming soon*

I will also post a YouTube video that walks through this guide step-by-step. Please check back later for a link to the video.

The guide walks through the following steps:
1. [Update the Raspberry Pi](https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi#1-update-the-raspberry-pi)
2. [Install TensorFlow](https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi#2-install-tensorflow)
3. [Install OpenCV](https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi#3-install-opencv)
4. [Compile and install Protobuf](https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi#4-compile-and-install-protobuf)
5. [Set up TensorFlow directory structure and the PYTHONPATH variable](https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi#5-set-up-tensorflow-directory-structure-and-pythonpath-variable)
6. [Detect objects!](https://github.com/EdjeElectronics/TensorFlow-Object-Detection-on-the-Raspberry-Pi#6-detect-objects)

The repository also includes the Object_detection_picamera.py script, which is a Python script that loads an object detection model in TensorFlow and uses it to detect objects in a Picamera video feed. The guide was written for TensorFlow v1.8.0 on a Raspberry Pi Model 3B running Raspbian Stretch v9. It will likely work for newer versions of TensorFlow.

## Steps
### 1. Update the Raspberry Pi
First, the Raspberry Pi needs to be fully updated. Open a terminal and issue:
```
sudo apt-get update
sudo apt-get dist-upgrade
```
Depending on how long it’s been since you’ve updated your Pi, the upgrade could take anywhere between a minute and an hour.
*Picture coming soon!*

### 2. Install TensorFlow
Next, we’ll install TensorFlow. In the /home/pi directory, create a folder called ‘tf’, which will be used to hold all the installation files for TensorFlow and Protobuf, and cd into it:
```
mkdir tf
cd tf
```
A pre-built, Rapsberry Pi-compatible wheel file for installing the latest version of TensorFlow is available in the [“TensorFlow for ARM” GitHub repository](https://github.com/lhelontra/tensorflow-on-arm/releases). GitHub user lhelontra updates the repository with pre-compiled installation packages each time a new TensorFlow is released. Thanks lhelontra!  Download the wheel file by issuing:
```
wget https://github.com/lhelontra/tensorflow-on-arm/releases/download/v1.8.0/tensorflow-1.8.0-cp35-none-linux_armv7l.whl
```
At the time this tutorial was written, the most recent version of TensorFlow was version 1.8.0. If a more recent version is available on the repository, you can download it rather than version 1.8.0.

Alternatively, if the owner of the GitHub repository stops releasing new builds, or if you want some experience compiling Python packages from source code, you can check out my video guide: [How to Install TensorFlow on the Raspberry Pi](https://youtu.be/WqCnW_2XDw8), which shows you how to build and install TensorFlow from source on the Raspberry Pi.

*Picture link to video guide coming soon!*

Now that we’ve got the file, install TensorFlow by issuing:
```
sudo pip3 install /home/pi/tensorflow-1.8.0-cp35-none-linux_armv7l.whl
```
TensorFlow also needs the LibAtlas package. Install it by issuing:
```
sudo apt-get install libatlas-base-dev
```
While we’re at it, let’s install other dependencies that will be used by the TensorFlow Object Detection API. These are listed on the [installation instructions](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/installation.md) in TensorFlow’s Object Detection GitHub repository. Issue:
```
sudo pip3 install pillow lxml jupyter matplotlib cython
sudo apt-get install python_tk
```
Alright, that’s everything we need for TensorFlow! Next up: OpenCV.

### 3. Install OpenCV
TensorFlow’s object detection examples typically use matplotlib to display images, but I prefer to use OpenCV because it’s easier to work with and less error prone. The object detection scripts in this guide’s GitHub repository use OpenCV. So, we need to install OpenCV.

To get OpenCV working on the Raspberry Pi, there’s quite a few dependencies that need to be installed through apt-get. If any of the following commands don’t work, issue “sudo apt-get update” and then try again. Issue:
```
sudo apt-get install libjpeg-dev libtiff5-dev libjasper-dev libpng12-dev
sudo apt-get install libavcodev-dev libavformat-dev libswscale-dev libv4l-dev
sudo apt-get install libxvidcore-dev libx264-dev
sudo apt-get install qt4-dev-tools
```
Now that we’ve got all those installed, we can install OpenCV. Issue:
```
pip3 install opencv-python
```
Alright, now OpenCV is installed!

### 4. Compile and Install Protobuf
Okay, here comes the hard part. The TensorFlow object detection API uses Protobuf, a package that implements Google’s Protocol Buffer data format. Unfortunately, there’s currently no easy way to install Protobuf on the Raspberry Pi. We have to compile it from source ourselves and then install it. Fortunately, a [guide](http://osdevlab.blogspot.com/2016/03/how-to-install-google-protocol-buffers.html) has already been written on how to compile and install Protobuf on the Pi. Thanks OSDevLab for writing the guide!

First, get the packages needed to compile Protobuf from source. Issue:
```
sudo apt-get install autoconf automake libtool curl
```
Then download the protobuf release from its GitHub repository by issuing:
```
wget https://github.com/google/protobuf/releases/download/v3.5.1/protobuf-all-3.5.1.tar.gz
```
If a more recent version of protobuf is available, download that instead. Unpack the file and cd into the folder:
```
tar -zxvf protobuf-all-3.5.1.tar.gz
cd protobuf-all-3.5.1.tar.gz
```
Configure the build by issuing the following command (it takes about 2 minutes):
```
./configure
```
Build the package by issuing:
```
make
```
The build process took 61 minutes on my Raspberry Pi. When it’s finished, issue:
```
make check 
```
This process takes even longer, clocking in at 107 minutes on my Pi. According to other guides I’ve seen, this command may exit out with errors, but Protobuf will still work. If you see errors, you can ignore them for now. Now that it’s built, install it by issuing:
```
sudo make install
```
Then move into the python directory and export the library path:
```
cd python
export LD_LIBRARY_PATH=../src/.libs
```
Next, issue:
```
python3 setup.py build --cpp_implementation 
python3 setup.py test --cpp_implementation
sudo python3 setup.py install --cpp_implementation
```
Then issue the following path commands:
```
export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION=cpp
export PROTOCOL_BUFFERS_PYTHON_IMPLEMENTATION_VERSION=3
```
Finally, issue:
```
sudo ldconfig
```
That’s it! Now Protobuf is installed on the Pi. Very it’s installed correctly by issuing the command below and making sure there are no errors reported.
```
protoc
```
*Picture of appropriate response to 'protoc' command coming soon!*

For some reason, the Raspberry Pi needs to be restarted after this process, or TensorFlow will not work. Go ahead and reboot the Pi by issuing:
```
sudo reboot now
```

### 5. Set up TensorFlow Directory Structure and PYTHONPATH Variable
Now that we’ve installed all the packages, we need to set up the TensorFlow directory. Move back to your home directory, then make a directory called “tensorflow1”, and cd into it.
```
mkdir tensorflow1
cd tensorflow1
```
Download the tensorflow repository from GitHub and checkout version 1.8.0 (or any other desired version) by issuing:
```
git clone --recurse_submodules https://github.com/tensorflow/models.git
git checkout v1.8.0
```
Next, we need to modify the PYTHONPATH environment variable to point at some directories inside the TensorFlow repository we just downloaded. We want PYTHONPATH to be set every time we open a terminal, so we have to modify the .bashrc file. Open it by issuing:
```
sudo nano ~/.bashrc
```
Move to the end of the file, and on the last line, add:
```
export PYTHONPATH=$PYTHONPATH:/home/pi/tensorflow1/models/research:/home/pi/tensorflow1/models/research/slim
```
*Picture of modified .bashrc file coming soon!*

Then, save and exit the file. This makes it so the “export PYTHONPATH” command is called every time you open a new terminal, so the PYTHONPATH variable will always be set appropriately.

Close and then re-open the terminal, and issue:
```
cd /home/pi/tensorflow1/models/research/object_detection
```
Now, we’ll download the SSD_Lite model from the [TensorFlow detection model zoo](https://github.com/tensorflow/models/blob/master/research/object_detection/g3doc/detection_model_zoo.md). The model zoo is Google’s collection of pre-trained object detection models that have various levels of speed and accuracy. The Raspberry Pi has a weak processor, so we need to use a model that takes less processing power. Though the model will run faster, it comes at a tradeoff of having lower accuracy. For this tutorial, we’ll use SSDLite-MobileNet, which is the fastest model available. 

Google is continuously releasing models with improved speed and performance, so check back at the model zoo often to see if there are any better models.

Download the SSDLite-MobileNet model and unpack it by issuing:
```
wget http://download.tensorflow.org/models/object_detection/ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
tar -xzvf ssdlite_mobilenet_v2_coco_2018_05_09.tar.gz
```
Now the model is in the object_detection directory and ready to be used.

### 6. Detect Objects!
Okay, now everything is set up for performing object detection on the Pi! The Python script in this repository, Object_detection_picamera.py, detects objects in live feeds from a Picamera or USB webcam. Basically, the script sets paths to the model and label map, loads the model into memory, initializes the Picamera, and then begins performing object detection on each video frame from the Picamera. 

If you’re using a Picamera, make sure it is enabled in the Raspberry Pi configuration menu.
*Picture of picamera enabled coming soon*
Download the Object_detection_picamera.py file into the object_detection directory by issuing:
```
wget (link to the file in this repository, which will be uploaded by 6/12/18)
```
Run the script by issuing: 
```
python3 Object_detection_picamera.py 
```
The script defaults to using an attached Picamera. If you have a USB webcam instead, add --camera=USB to the end of the command:
```
python3 Object_detection_picamera.py --camera=USB
```
Once the script initializes (which can take up to 30 seconds), you will see a window showing a live view from your camera. Common objects inside the view will be identified and have a rectangle drawn around them. 

*Picture of object detector in action coming soon!*

With the SSDLite model, the Raspberry Pi 3 performs fairly well, achieving a frame rate higher than 1FPS. This is fast enough for most real-time object detection applications.

You can also use a model you trained yourself by adding the frozen inference graph into the object_detection directory and changing the model path in the script. You can test this out using my playing card detector model located at this dropbox link (link will be added by 6/15/18). Once you’ve downloaded and extracted the model, or if you have your own model, place the model folder into the object_detection directory. Place the label_map.pbtxt file into the object_detection/data directory.

*Picture of directory setup coming soon*

Then, open the Object_detection_picamera.py script in a text editor. Go to the line where MODEL_NAME is set and change the string to match the name of the new model folder. Then, on the line where PATH_TO_LABELS is set, change the name of the labelmap file to match the new label map.

*Picture of modified script coming soon*

Now, when you run the script, it will use your model rather than the SSDLite_MobileNet model. If you’re using my model, it will detect and identify any playing cards dealt in front of the camera.

*Picture of detected playing cards coming soon*

Thanks for following through this guide, I hope you found it useful. Good luck with your object detection applications on the Raspberry Pi!
