* Auto: 김준호
* Date: 2022-02-18

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/deep-vision-header.jpg" width="100%">

# Deploying Deep Learning
NVIDIA의 **[Jetson Nano/TX1/TX2/Xavier NX/AGX Xavier](http://www.nvidia.com/object/embedded-systems.html)** 를 사용한 inference(추론) 와 realtime(실시간) [DNN vision](#api-reference) library를 위한 guide에 오신걸 환영합니다.

해당 repo는 NVIDIA **[TensorRT](https://developer.nvidia.com/tensorrt)** 를 사용합니다. 이는 graph optimizations, kernel fusion, FP16/INT8 precision 을 적용함으로써 Jetson 플랫폼에서 인공신경망을 더 빠르게, 더 적은 에너지로 사용하기 위해서 입니다.

비전의 기초, [`imageNet`](docs/imagenet-console-2.md) for image recognition(이미지 분류), [`detectNet`](docs/detectnet-console-2.md) for object detection(객체 검출), [`segNet`](docs/segnet-console-2.md) for semantic segmentation, and [`poseNet`](docs/posenet.md) for pose estimation, 각 모델들은 shared [`tensorNet`](c/tensorNet.h) object 에서 상속되어 사용됩니다. 예제로 live camera로부터 streaming 받는 것과 이미지를 처리하는 것이 제공됩니다. 다음 **[API Reference](#api-reference)** 에서 C++, Python에 대한 자세한 자료를 찾아보실 수 있습니다. 

<img src="https://github.com/dusty-nv/jetson-inference/raw/dev/docs/images/deep-vision-primitives.jpg">

[Hello AI World](#hello-ai-world) 에서는 jetson platform에서 inference(추론)과 transfer learning을 하는 튜토리얼을 제공합니다. 여기서 직접 datasets을 수집하고 인공신경망을 직접 학습시켜볼 수 있습니다. 이 튜토리얼은 image classification(이미지 분류), object detection(객체 검출), sematic segmentation, pose estimation, mono depth를 다룹니다.

### 목차

* [Hello AI World](#hello-ai-world)
* [Video Walkthroughs](#video-walkthroughs)
* [API Reference](#api-reference)
* [Code Examples](#code-examples)
* [Pre-Trained Models](#pre-trained-models)
* [System Requirements](#recommended-system-requirements)
* [Change Log](CHANGELOG.md)

> &gt; &nbsp; JetPack 4.6 이 지원됩니다. 문서를 확인하세요. [updated containers](docs/aux-docker.md). <br/>
> &gt; &nbsp; [Pose Estimation](docs/posenet.md) 와 [Mono Depth](docs/depthnet.md) 튜토리얼을 해보세요! <br/>

## Hello AI World

Hello AI Wolrd는 jetson 보드에서 수행될 수 있으며, 이는 TensorRT를 통한 inference(추론)과 Pytorch를 이용한 transfer learning을 포함합니다. inference 부분은 Python과 C++로 직접 image classfication(이미지 분류)와 object detectino(객체 검출) application 코드를 작성하는 것으로 이루어져있고 약 2시간 정도가 소요됩니다. 반면, tranfer learning은 밤새 수행시켜주시는 것이 좋습니다.

#### 시스템 설정

* [Jetson 에 JetPack 설정하기](docs/jetpack-setup-2.md)
* [Docker Container 실행시키기](docs/aux-docker.md)
* [소스코드에서부터 프로젝트 설계하기](docs/building-repo-2.md)

#### Inference(추론)

* [ImageNet 데이터셋으로 이미지 분류하기](docs/imagenet-console-2.md)
	* [Jetson에서 ImageNet 프로그램 사용하기](docs/imagenet-console-2.md)
	* [직접 Image 인식 프로그램 코딩하기 (Python)](docs/imagenet-example-python-2.md)
	* [직접 Image 인식 프로그램 코딩하기 (C++)](docs/imagenet-example-2.md)
	* [라이브 카메라로 하는 이미지 인식 데모 수행하기](docs/imagenet-camera-2.md)
* [DetectNet으로 객체 위치 찾기](docs/detectnet-console-2.md)
	* [이미지로부터 객체 검출하기](docs/detectnet-console-2.md#detecting-objects-from-the-command-line)
	* [라이브 카메라로 하는 객체 인식 데모 수행하기](docs/detectnet-camera-2.md)
	* [직접 객체 검출 프로그램 코딩하기](docs/detectnet-example-2.md)
* [SegNet으로 하는 Semantic Segmentation](docs/segnet-console-2.md)
	* [Command Line으로 이미지들 Segmentation하기](docs/segnet-console-2.md#segmenting-images-from-the-command-line)
	* [라이브 카메라로 Segmentation 데모 수행하기](docs/segnet-camera-2.md)
* [PoseNet으로 하는 Pose Estimation](docs/posenet.md)
* [DepthNet으로 하는 Monocular Depth](docs/depthnet.md)

#### Training(훈련)

* [PyTorch로 하는 Transfer Learning](docs/pytorch-transfer-learning.md)
* Classification(분류)/Recognition(인식) (ResNet-18)
	* [Cat/Dog Dataset으로 다시 훈련하기](docs/pytorch-cat-dog.md)
	* [PlantCLEF Dataset으로 다시 훈련하기](docs/pytorch-plants.md)
	* [Classification(이미지 분류) Datasets 직접 수집하기](docs/pytorch-collect.md)
* Object Detection (SSD-Mobilenet)
	* [SSD-Mobilenet 다시 훈련하기](docs/pytorch-ssd.md)
	* [Detection Datasets 직접 수집하기](docs/pytorch-collect-detection.md)

#### Appendix

* [Camera Streaming and Multimedia](docs/aux-streaming.md)
* [Image Manipulation with CUDA](docs/aux-image.md)
* [Deep Learning Nodes for ROS/ROS2](https://github.com/dusty-nv/ros_deep_learning)

## Video 목록

아래 영상들은 [Jetson AI Certification](https://developer.nvidia.com/embedded/learn/jetson-ai-certification-programs) 코스에서 제공된 것입니다.

| Description                                                                                                                                                                                                                                                                                                        | Video                                                                                                                                                                                                                                                 |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| <a href="https://www.youtube.com/watch?v=QXIwdsyK7Rw&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=9" target="_blank">**Hello AI World Setup**</a><br/>Jetson Nano에서 Hello AI World 도커 컨테이너를 다운로드 받고 run 합니다. 카메라를 테스트 합니다. 그리고 RTP를 통해 네트워크에 어떻게 스트리밍 되는지 확인합니다.                         | <a href="https://www.youtube.com/watch?v=QXIwdsyK7Rw&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=9" target="_blank"><img src=https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/thumbnail_setup.jpg width="750"></a>               |
| <a href="https://www.youtube.com/watch?v=QatH8iF0Efk&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=10" target="_blank">**Image Classification Inference**</a><br/>딥러닝 기술과 Jetson Nano를 통해 수행하는 이미지 분류 프로그램을 코딩합니다. 그리고 라이브 카메라에서 실시간으로 이미지를 분류하는 것을 확인합니다. | <a href="https://www.youtube.com/watch?v=QatH8iF0Efk&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=10" target="_blank"><img src=https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/thumbnail_imagenet.jpg width="750"></a>           |
| <a href="https://www.youtube.com/watch?v=sN6aT9TpltU&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=11" target="_blank">**Training Image Classification Models**</a><br/>Jetson Nano에서 어떻게 Pytorch로 이미지 분류 모델을 훈련하는지 배웁니다. 그리고 커스텀 모델을 생성하기 위한 datasets을 수집하는 방법을 배웁니다.     | <a href="https://www.youtube.com/watch?v=sN6aT9TpltU&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=11" target="_blank"><img src=https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/thumbnail_imagenet_training.jpg width="750"></a>  |
| <a href="https://www.youtube.com/watch?v=obt60r8ZeB0&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=12" target="_blank">**Object Detection Inference**</a><br/>딥러닝 기술과 Jetson Nano를 통해 수행하는 객체 검출 프로그램을 코딩합니다. 그리고 라이브 카메라에서 실시간으로 객체를 검출하는 것을 확인합니다.              | <a href="https://www.youtube.com/watch?v=obt60r8ZeB0&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=12" target="_blank"><img src=https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/thumbnail_detectnet.jpg width="750"></a>          |
| <a href="https://www.youtube.com/watch?v=2XMkPW_sIGg&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=13" target="_blank">**Training Object Detection Models**</a><br/>Jetson Nano에서 어떻게 Pytorch로 객체 검출 모델을 훈련하는지 배웁니다. 그리고 커스텀 모델을 생성하기 위한 datasets을 수집하는 방법을 배웁니다.                  | <a href="https://www.youtube.com/watch?v=2XMkPW_sIGg&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=13" target="_blank"><img src=https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/thumbnail_detectnet_training.jpg width="750"></a> |
| <a href="https://www.youtube.com/watch?v=AQhkMLaB_fY&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=14" target="_blank">**Semantic Segmentation**</a><br/>Jetson Nano에서 Semantic segmentation network를 경험해보고 이를 라이브 카메라로 잘 수행되는지 확인합니다.                                 | <a href="https://www.youtube.com/watch?v=AQhkMLaB_fY&list=PL5B692fm6--uQRRDTPsJDp4o0xbzkoyf8&index=14" target="_blank"><img src=https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/thumbnail_segnet.jpg width="750"></a>             |

## API Reference

아래 두 문서 링크는 [C++](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/index.html) and [Python](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/python/jetson.html) 해당 repo에서 사용하는 API 라이브러리입니다.

#### jetson-inference

|                   | [C++](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/group__deepVision.html) | [Python](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/python/jetson.inference.html) |
|-------------------|--------------|--------------|
| Image Recognition | [`imageNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/classimageNet.html) | [`imageNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/python/jetson.inference.html#imageNet) |
| Object Detection  | [`detectNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/classdetectNet.html) | [`detectNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/python/jetson.inference.html#detectNet)
| Segmentation      | [`segNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/classsegNet.html) | [`segNet`](https://rawgit.com/dusty-nv/jetson-inference/pytorch/docs/html/python/jetson.inference.html#segNet) |
| Pose Estimation   | [`poseNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/classposeNet.html) | [`poseNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/python/jetson.inference.html#poseNet) |
| Monocular Depth   | [`depthNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/classdepthNet.html) | [`depthNet`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/python/jetson.inference.html#depthNet) |

#### jetson-utils

* [C++](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/group__util.html)
* [Python](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/python/jetson.utils.html)

위 라이브러리들은 `libjetson-inference` and `libjetson-utils`를 링크함으로써 다른 프로젝트에서도 사용할 수 있습니다.

## 코드 예제

Introductory code walkthroughs of using the library are covered during these steps of the Hello AI World tutorial:

* [Coding Your Own Image Recognition Program (Python)](docs/imagenet-example-python-2.md)
* [Coding Your Own Image Recognition Program (C++)](docs/imagenet-example-2.md)

Additional C++ and Python samples for running the networks on static images and live camera streams can be found here:

|                   | C++             | Python             |
|-------------------|---------------------|---------------------|
| &nbsp;&nbsp;&nbsp;Image Recognition | [`imagenet.cpp`](examples/imagenet/imagenet.cpp) | [`imagenet.py`](python/examples/imagenet.py) |
| &nbsp;&nbsp;&nbsp;Object Detection  | [`detectnet.cpp`](examples/detectnet/detectnet.cpp) | [`detectnet.py`](python/examples/detectnet.py) |
| &nbsp;&nbsp;&nbsp;Segmentation      | [`segnet.cpp`](examples/segnet/segnet.cpp) | [`segnet.py`](python/examples/segnet.py) |
| &nbsp;&nbsp;&nbsp;Pose Estimation   | [`posenet.cpp`](examples/posenet/posenet.cpp) | [`posenet.py`](python/examples/posenet.py) |
| &nbsp;&nbsp;&nbsp;Monocular Depth   | [`depthnet.cpp`](examples/depthnet/segnet.cpp) | [`depthnet.py`](python/examples/depthnet.py) |

> **note**:  for working with numpy arrays, see [Converting to Numpy Arrays](docs/aux-image.md#converting-to-numpy-arrays) and [Converting from Numpy Arrays](docs/aux-image.md#converting-from-numpy-arrays)

These examples will automatically be compiled while [Building the Project from Source](docs/building-repo-2.md), and are able to run the pre-trained models listed below in addition to custom models provided by the user.  Launch each example with `--help` for usage info.

## Pre-Trained Models

이 프로젝트는 여러 pre-trained models이 함께 제공되며 해당 툴을 사용하여 다운로드 받을 수 있습니다. [**Model Downloader**](docs/building-repo-2.md#downloading-models)

#### Image Recognition

| Network       | CLI argument   | NetworkType enum |
| --------------|----------------|------------------|
| AlexNet       | `alexnet`      | `ALEXNET`        |
| GoogleNet     | `googlenet`    | `GOOGLENET`      |
| GoogleNet-12  | `googlenet-12` | `GOOGLENET_12`   |
| ResNet-18     | `resnet-18`    | `RESNET_18`      |
| ResNet-50     | `resnet-50`    | `RESNET_50`      |
| ResNet-101    | `resnet-101`   | `RESNET_101`     |
| ResNet-152    | `resnet-152`   | `RESNET_152`     |
| VGG-16        | `vgg-16`       | `VGG-16`         |
| VGG-19        | `vgg-19`       | `VGG-19`         |
| Inception-v4  | `inception-v4` | `INCEPTION_V4`   |

#### Object Detection

| Network                 | CLI argument       | NetworkType enum   | Object classes       |
| ------------------------|--------------------|--------------------|----------------------|
| SSD-Mobilenet-v1        | `ssd-mobilenet-v1` | `SSD_MOBILENET_V1` | 91 ([COCO classes](data/networks/ssd_coco_labels.txt)) |
| SSD-Mobilenet-v2        | `ssd-mobilenet-v2` | `SSD_MOBILENET_V2` | 91 ([COCO classes](data/networks/ssd_coco_labels.txt)) |
| SSD-Inception-v2        | `ssd-inception-v2` | `SSD_INCEPTION_V2` | 91 ([COCO classes](data/networks/ssd_coco_labels.txt)) |
| DetectNet-COCO-Dog      | `coco-dog`         | `COCO_DOG`         | dogs                 |
| DetectNet-COCO-Bottle   | `coco-bottle`      | `COCO_BOTTLE`      | bottles              |
| DetectNet-COCO-Chair    | `coco-chair`       | `COCO_CHAIR`       | chairs               |
| DetectNet-COCO-Airplane | `coco-airplane`    | `COCO_AIRPLANE`    | airplanes            |
| ped-100                 | `pednet`           | `PEDNET`           | pedestrians          |
| multiped-500            | `multiped`         | `PEDNET_MULTI`     | pedestrians, luggage |
| facenet-120             | `facenet`          | `FACENET`          | faces                |

#### Semantic Segmentation

| Dataset      | Resolution | CLI Argument | Accuracy | Jetson Nano | Jetson Xavier |
|:------------:|:----------:|--------------|:--------:|:-----------:|:-------------:|
| [Cityscapes](https://www.cityscapes-dataset.com/) | 512x256 | `fcn-resnet18-cityscapes-512x256` | 83.3% | 48 FPS | 480 FPS |
| [Cityscapes](https://www.cityscapes-dataset.com/) | 1024x512 | `fcn-resnet18-cityscapes-1024x512` | 87.3% | 12 FPS | 175 FPS |
| [Cityscapes](https://www.cityscapes-dataset.com/) | 2048x1024 | `fcn-resnet18-cityscapes-2048x1024` | 89.6% | 3 FPS | 47 FPS |
| [DeepScene](http://deepscene.cs.uni-freiburg.de/) | 576x320 | `fcn-resnet18-deepscene-576x320` | 96.4% | 26 FPS | 360 FPS |
| [DeepScene](http://deepscene.cs.uni-freiburg.de/) | 864x480 | `fcn-resnet18-deepscene-864x480` | 96.9% | 14 FPS | 190 FPS |
| [Multi-Human](https://lv-mhp.github.io/) | 512x320 | `fcn-resnet18-mhp-512x320` | 86.5% | 34 FPS | 370 FPS |
| [Multi-Human](https://lv-mhp.github.io/) | 640x360 | `fcn-resnet18-mhp-512x320` | 87.1% | 23 FPS | 325 FPS |
| [Pascal VOC](http://host.robots.ox.ac.uk/pascal/VOC/) | 320x320 | `fcn-resnet18-voc-320x320` | 85.9% | 45 FPS | 508 FPS |
| [Pascal VOC](http://host.robots.ox.ac.uk/pascal/VOC/) | 512x320 | `fcn-resnet18-voc-512x320` | 88.5% | 34 FPS | 375 FPS |
| [SUN RGB-D](http://rgbd.cs.princeton.edu/) | 512x400 | `fcn-resnet18-sun-512x400` | 64.3% | 28 FPS | 340 FPS |
| [SUN RGB-D](http://rgbd.cs.princeton.edu/) | 640x512 | `fcn-resnet18-sun-640x512` | 65.1% | 17 FPS | 224 FPS |

* If the resolution is omitted from the CLI argument, the lowest resolution model is loaded
* Accuracy indicates the pixel classification accuracy across the model's validation dataset
* Performance is measured for GPU FP16 mode with JetPack 4.2.1, `nvpmodel 0` (MAX-N)

<details>
<summary>Legacy Segmentation Models</summary>

| Network                 | CLI Argument                    | NetworkType enum                | Classes |
| ------------------------|---------------------------------|---------------------------------|---------|
| Cityscapes (2048x2048)  | `fcn-alexnet-cityscapes-hd`     | `FCN_ALEXNET_CITYSCAPES_HD`     |    21   |
| Cityscapes (1024x1024)  | `fcn-alexnet-cityscapes-sd`     | `FCN_ALEXNET_CITYSCAPES_SD`     |    21   |
| Pascal VOC (500x356)    | `fcn-alexnet-pascal-voc`        | `FCN_ALEXNET_PASCAL_VOC`        |    21   |
| Synthia (CVPR16)        | `fcn-alexnet-synthia-cvpr`      | `FCN_ALEXNET_SYNTHIA_CVPR`      |    14   |
| Synthia (Summer-HD)     | `fcn-alexnet-synthia-summer-hd` | `FCN_ALEXNET_SYNTHIA_SUMMER_HD` |    14   |
| Synthia (Summer-SD)     | `fcn-alexnet-synthia-summer-sd` | `FCN_ALEXNET_SYNTHIA_SUMMER_SD` |    14   |
| Aerial-FPV (1280x720)   | `fcn-alexnet-aerial-fpv-720p`   | `FCN_ALEXNET_AERIAL_FPV_720p`   |     2   |

</details>

#### Pose Estimation

| Model                   | CLI argument       | NetworkType enum   | Keypoints |
| ------------------------|--------------------|--------------------|-----------|
| Pose-ResNet18-Body      | `resnet18-body`    | `RESNET18_BODY`    | 18        |
| Pose-ResNet18-Hand      | `resnet18-hand`    | `RESNET18_HAND`    | 21        |
| Pose-DenseNet121-Body   | `densenet121-body` | `DENSENET121_BODY` | 18        |

## 권장 사양

* Jetson Nano Developer Kit with JetPack 4.2 or newer (Ubuntu 18.04 aarch64).  
* Jetson Nano 2GB Developer Kit with JetPack 4.4.1 or newer (Ubuntu 18.04 aarch64).
* Jetson Xavier NX Developer Kit with JetPack 4.4 or newer (Ubuntu 18.04 aarch64).  
* Jetson AGX Xavier Developer Kit with JetPack 4.0 or newer (Ubuntu 18.04 aarch64).  
* Jetson TX2 Developer Kit with JetPack 3.0 or newer (Ubuntu 16.04 aarch64).  
* Jetson TX1 Developer Kit with JetPack 2.3 or newer (Ubuntu 16.04 aarch64).  

The [Transfer Learning with PyTorch](#training) section of the tutorial speaks from the perspective of running PyTorch onboard Jetson for training DNNs, however the same PyTorch code can be used on a PC, server, or cloud instance with an NVIDIA discrete GPU for faster training.


## 추가 

In this area, links and resources for deep learning are listed:

* [ros_deep_learning](http://www.github.com/dusty-nv/ros_deep_learning) - TensorRT inference ROS nodes
* [NVIDIA AI IoT](https://github.com/NVIDIA-AI-IOT) - NVIDIA Jetson GitHub repositories
* [Jetson eLinux Wiki](https://www.eLinux.org/Jetson) - Jetson eLinux Wiki


## Two Days to a Demo (DIGITS)

> **note:** the DIGITS/Caffe tutorial from below is deprecated.  It's recommended to follow the [Transfer Learning with PyTorch](#training) tutorial from Hello AI World.
 
<details>
<summary>Expand this section to see original DIGITS tutorial (deprecated)</summary>
<br/>
The DIGITS tutorial includes training DNN's in the cloud or PC, and inference on the Jetson with TensorRT, and can take roughly two days or more depending on system setup, downloading the datasets, and the training speed of your GPU.

* [DIGITS Workflow](docs/digits-workflow.md) 
* [DIGITS System Setup](docs/digits-setup.md)
* [Setting up Jetson with JetPack](docs/jetpack-setup.md)
* [Building the Project from Source](docs/building-repo.md)
* [Classifying Images with ImageNet](docs/imagenet-console.md)
	* [Using the Console Program on Jetson](docs/imagenet-console.md#using-the-console-program-on-jetson)
	* [Coding Your Own Image Recognition Program](docs/imagenet-example.md)
	* [Running the Live Camera Recognition Demo](docs/imagenet-camera.md)
	* [Re-Training the Network with DIGITS](docs/imagenet-training.md)
	* [Downloading Image Recognition Dataset](docs/imagenet-training.md#downloading-image-recognition-dataset)
	* [Customizing the Object Classes](docs/imagenet-training.md#customizing-the-object-classes)
	* [Importing Classification Dataset into DIGITS](docs/imagenet-training.md#importing-classification-dataset-into-digits)
	* [Creating Image Classification Model with DIGITS](docs/imagenet-training.md#creating-image-classification-model-with-digits)
	* [Testing Classification Model in DIGITS](docs/imagenet-training.md#testing-classification-model-in-digits)
	* [Downloading Model Snapshot to Jetson](docs/imagenet-snapshot.md)
	* [Loading Custom Models on Jetson](docs/imagenet-custom.md)
* [Locating Objects with DetectNet](docs/detectnet-training.md)
	* [Detection Data Formatting in DIGITS](docs/detectnet-training.md#detection-data-formatting-in-digits)
	* [Downloading the Detection Dataset](docs/detectnet-training.md#downloading-the-detection-dataset)
	* [Importing the Detection Dataset into DIGITS](docs/detectnet-training.md#importing-the-detection-dataset-into-digits)
	* [Creating DetectNet Model with DIGITS](docs/detectnet-training.md#creating-detectnet-model-with-digits)
	* [Testing DetectNet Model Inference in DIGITS](docs/detectnet-training.md#testing-detectnet-model-inference-in-digits)
	* [Downloading the Detection Model to Jetson](docs/detectnet-snapshot.md)
	* [DetectNet Patches for TensorRT](docs/detectnet-snapshot.md#detectnet-patches-for-tensorrt)
	* [Detecting Objects from the Command Line](docs/detectnet-console.md)
	* [Multi-class Object Detection Models](docs/detectnet-console.md#multi-class-object-detection-models)
	* [Running the Live Camera Detection Demo on Jetson](docs/detectnet-camera.md)
* [Semantic Segmentation with SegNet](docs/segnet-dataset.md)
	* [Downloading Aerial Drone Dataset](docs/segnet-dataset.md#downloading-aerial-drone-dataset)
	* [Importing the Aerial Dataset into DIGITS](docs/segnet-dataset.md#importing-the-aerial-dataset-into-digits)
	* [Generating Pretrained FCN-Alexnet](docs/segnet-pretrained.md)
	* [Training FCN-Alexnet with DIGITS](docs/segnet-training.md)
	* [Testing Inference Model in DIGITS](docs/segnet-training.md#testing-inference-model-in-digits)
	* [FCN-Alexnet Patches for TensorRT](docs/segnet-patches.md)
	* [Running Segmentation Models on Jetson](docs/segnet-console.md)

</details>

##
<p align="center"><sup>© 2016-2019 NVIDIA | </sup><a href="#deploying-deep-learning"><sup>Table of Contents</sup></a></p>

