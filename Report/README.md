# Fruit freshness identification

Huanchao Hong, https://github.com/ChaceHH-H/0018-deep_learning  
https://studio.edgeimpulse.com/studio/363592

## Introduction
The purpose of this project is to develop a system based on Arduino Nano 33 BLE Sense and ov7670 camera that can identify and classify the freshness of fruits in real time and determine whether the fruits have become moldy or rotten.  
The original intention of this project was to solve a problem that I personally encounter often: buying fruits and then forgetting to eat them, causing them to end up rotting and being wasted. So I hope to build on existing object recognition and image processing technology by using Arduino Nano 33 BLE Sense and ov7670 camera. Accurate assessment of fruit status is achieved by training a deep learning model to identify different stages of fruit rot. Using Arduino and ov7670 camera as the basis makes the entire system low-cost and easy to implement, providing the possibility for a wide range of applications.  
The construction of this system was inspired by two Hackster projects. The first project "Image Recognition with TinyML, Nano 33 BLE, and ov7670" shows how to use Nano 33 BLE with the ov7670 camera to achieve TinyML-based image recognition. This provided the infrastructure and technical demonstration for my project. The second project "ov7670 Camera and Image Sensor with Nano 33 BLE" details how to set up and use Nano 33 BLE with ov7670 camera, which helps me better understand the integration and configuration of hardware components.
![微信图片_20240424170818](https://github.com/ChaceHH-H/0018-deep_learning/assets/146041784/bbb01da9-1364-465f-be8d-3ba8c0d93bd2)

## Research Question
The problem I wanted to solve was the common wastage of fruit in households, specifically fruit rot due to forgetting to eat it.

## Application Overview
When building a fruit freshness recognition system based on Arduino Nano 33 BLE Sense and ov7670 camera module, I divided the project's architecture into several key software and data processing modules. These modules work together to implement a complete process from image collection to processing and analysis, and can ultimately determine the freshness of the fruit.  
Image capture module: The ov7670 camera module first takes a picture of the object to be recognized, and then the camera module transmits the original image data to the Arduino Nano 33 BLE Sense through the parallel interface.  
Image preprocessing module: Perform preliminary processing on the captured raw image data, including converting the file category of the image, reducing the resolution of the image to adapt to the processing capabilities, or applying some basic image enhancement techniques to improve image quality.  
Feature extraction module: Extract key features that are helpful in identifying the freshness of fruits. This step is a key data processing link in the entire system to better distinguish the visual differences between fresh fruits and expired fruits.  
Machine learning model: Using the processed images or extracted features, run the pre-trained machine learning model for freshness classification. This model is based on a large amount of annotated training data and can identify and learn the characteristic patterns of fruits with different freshness.  
Result output and feedback: According to the output of the machine learning model, the result is output on the serial port, and the LED is lit according to whether it is fresh.

## Data
In my project, I used a dataset provided by Mendeley Data, which contains Fresh and Rotten images of eight kinds of fruits.Initially I used all fruits and pictures, after the first few trainings I chose to keep only five fruits (apples, bananas, grapes, oranges, strawberries) because the pictures of Jujube, Pomegranate and Guava were very visually Similarly, the model cannot distinguish fruits no matter how many times it is trained, so I delete them.After I switched to training the model on edge impulse, the model provided by edge impulse could not accurately distinguish oranges, apples and strawberries. In order to improve the overall performance of the model, I decided to delete the pictures of oranges and strawberries from the data set, leaving only apples, bananas and Grape. In addition, there were some too similar pictures of fresh and rotten grapes, so I also deleted some of them.  
![屏幕截图 2024-04-24 170218](https://github.com/ChaceHH-H/0018-deep_learning/assets/146041784/5163505f-4c0f-4947-a406-e7307c156b29)


## Model
The final model architecture used in my project was MobileNetV1 96x96 0.20 provided by edge impulse. After 50 rounds of training, I obtained 86.1% ACCURACY and 34% loss. This decision was made after trying several different model architectures.  
Initially, I built a convolutional neural network using TensorFlow Keras on Colab, which included multiple layers of convolutions. After multiple training and adjustments, the model achieved an accuracy of up to 97%, showing very good performance. But after testing the deployment, I found that the RAM and LASH usage required by the tensorflow-trained model far exceeded the limit of the Arduino Nano 33 BLE. Even if I reduced the number of convolutional layers, quantized the model, and converted to TensorFlow Lite, the model still remained There are 687KB and it cannot be deployed on Arduino Nano 33 BLE.  
So I switched to edge impulse, and after testing multiple versions of MobileNetV1 and MobileNetV2, I chose the MobileNetV1 0.20 version. This model is not only suitable for running on the Arduino Nano 33 BLE with limited computing resources, but also maintains a high recognition accuracy.  
![屏幕截图 2024-04-24 100248](https://github.com/ChaceHH-H/0018-deep_learning/assets/146041784/71f7b761-f188-466a-8717-070ebc929583)

## Experiments
After I switched to edge impulse to train the model, I would initially use edge impulse's Test data to classify all to use the model test output, but I found that the results of this test would be very different from the performance of the actual deployment to the device. Because Test data uses data sets for testing, it usually achieves high accuracy. However, after being deployed to the device, due to issues such as camera shooting quality, the actual accuracy will be much lower than the results of Test data.Therefore, in future experiments, I will not use Test data. Instead, I will use live classification to connect to the device after each training, and use the device's camera to capture images for recognition testing. However, live classification only obtains images through the device camera, and The model is not actually run on the device. When I used the MobileNetV1 0.25 architecture to train a model with an accuracy of 94% and performed well in the live classification test, the model failed to run on the device after it was built.  
In addition, in the early iterations of the model, I only used pictures and data sets obtained from the Internet for training. These images were many different and different from the images taken by my device camera, as well as the fruits I tested, so before the model was deployed to After the device, the actual accuracy was much lower than the training results, so I tried to replace the data set with images I took using the device camera, but the image quality of the device camera was too low to obtain effective features.  
I tried two different ways to deploy the model to the device. One is to use edge impulse binary. The advantage of this is that the device camera can be obtained through the serial port and displayed on the edge impulse debug mode page, but edge impulse binary cannot make any modifications to the code. No other hardware can be added.The other is to build the arduino library and use the arduino ide to add the zip package. This way you can modify the code to add hardware, but you can't view the acquisition of the camera. I tried streaming to processing to display the camera image, but it failed and I only got the wrong distortion. image. So in the end I chose to use edge impulse binary to run impulse in debug mode after each training to obtain camera vision to test the accuracy of the model. After ensuring the feasibility of the model, I then built the arduino library and added the zip package with arduino ide. led and code.  
![屏幕录制 2024-04-24 072450 mp4_20240424_171328883](https://github.com/ChaceHH-H/0018-deep_learning/assets/146041784/1285f542-56f0-40ff-99a2-69d74f6339cc)

## Results and Observations
Overall, the project has achieved my goal and can distinguish the freshness of fruits. Although the photo quality of the OV7670 and OV7675 I used is low, since the recognition image of the project is not very complicated and only needs to distinguish 6 categories of images, the Arduino Nano 33 BLE Sense can still complete the task, especially for Apple The recognition response and accuracy are pretty good for grapes, but the accuracy for bananas will be lower.But the project is not perfect at present. The main reason is that the hardware used in the project, Arduino Nano 33 BLE Sense and ov7670 camera module, has insufficient performance. Nano 33 only has 256 kB Ram and 1024 kB Rom, so it is difficult to use TensorFlow training. model, and edge impulse's model architecture can only run on v1 0.2 and 0.1, so it is impossible to train an efficient model. In addition, the camera's photo quality is too low, resulting in a much lower accuracy after model deployment. So if I had more time, I would choose a higher performance platform and camera, such as a Raspberry Pi to deploy larger models. I will change the architecture of the model to TensorFlow Keras, which I gave up before because of its size.

## Bibliography
1. Kpower (2023) Image recognition with TinyML, nano 33 ble and OV7670, Hackster.io. Available at: https://www.hackster.io/umpheki/image-recognition-with-tinyml-nano-33-ble-and-ov7670-084b2a .
2. Kpower (2023b) OV7670 camera and image sensor with Nano 33 Ble, Hackster.io. Available at: https://www.hackster.io/umpheki/ov7670-camera-and-image-sensor-with-nano-33-ble-497c5f .
3. Sultana, N., Jahan, M. and Uddin, M.S. (2022) ‘An extensive dataset for successful recognition of fresh and rotten fruits’, Data in Brief, 44, p. 108552. doi:10.1016/j.dib.2022.108552. 


## Declaration of Authorship

I, AUTHORS Huanchao Hong, confirm that the work presented in this assessment is my own. Where information has been derived from other sources, I confirm that this has been indicated in the work.


**

2024/4/24

Words=1496
