* Auth: 김준호
* Data: 2022-03-01

<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/deep-vision-header.jpg" width="100%">
<p align="right"><sup><a href="aux-streaming.md">Back</a> | <a href="https://github.com/dusty-nv/ros_deep_learning">Next</a> | </sup><a href="../README.md#hello-ai-world"><sup>Contents</sup></a>
<br/>
<sup>Appendix</sup></p>  

# CUDA로 이미지 다루기

해당 페이지는 CUDA와 함께 구현된 jetson-util의 여러가지의 이미지 포맷, 변환, pre/post-processing(전/후처리) 함수를 다룹니다.

**이미지 매니지먼트**
* [이미지 포맷(formats)](#image-formats)
* [이미지 할당(allocation)](#image-allocation)
* [이미지 복사(copy)](#copying-images)
* [Python에서의 이미지 캡슐](#image-capsules-in-python)
	* [Python에서 이미지 데이터에 접근(access)](#accessing-image-data-in-python)
	* [Numpy 배열로 변환 (to numpy)](#converting-to-numpy-arrays)
	* [Numpy 배열을 변환 (from numpy)](#converting-from-numpy-arrays)

**CUDA 루틴**
* [컬러 변환](#color-conversion)
* [크기 변환](#resizing)
* [이미지 자르기(Cropping)](#cropping)
* [이미지 정규화(Normalization)](#normalization)
* [오버레이(Overlay)](#overlay)
* [도형 그리기(Drawing Shapes)](#drawing-shapes)

이와 같은 함수를 사용하는 예제를 위해 아래 psuedocode 외에 추가적으로 [`cuda-examples.py`](https://github.com/dusty-nv/jetson-utils/tree/master/python/examples/cuda-examples.py) 를 살펴보세요. 이 페이지를 살펴보기 전에 비디오 캡쳐, 출력, 이미지 불러오기, 저장 등에 대한 정보를 이전 페이지인 [Camera Streaming and Multimedia](aux-streaming.md) 를 살펴보고 오시기를 권장드립니다. 

## 이미지 포맷(formats)

[video streaming](aux-streaming#source-code) API나 딥러닝 객체(가령, [`imageNet`](c/imageNet.h), [`detectNet`](c/detectNet.h), and [`segNet`](c/segNet.h))들은 RGB/RGBA 이미지의 포맷을 입력으로 받기를 기대하지만, 이미지 획득(sensor acquisition)이나 저수준I/O 를 위한 다양한 다른 포맷들이 정의돼있습니다.:

|                 | 포맷 스트링 | [`imageFormat` enum](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/group__imageFormat.html#ga931c48e08f361637d093355d64583406)   | 데이터 타입 | 비트 수 |
|-----------------|---------------|--------------------|-----------|-----------|
| **RGB/RGBA**    | `rgb8`        | `IMAGE_RGB8`       | `uchar3`  | 24        |
|                 | `rgba8`       | `IMAGE_RGBA8`      | `uchar4`  | 32        |
|                 | `rgb32f`      | `IMAGE_RGB32F`     | `float3`  | 96        |
|                 | `rgba32f`     | `IMAGE_RGBA32F`    | `float4`  | 128       |
| **BGR/BGRA**    | `bgr8`        | `IMAGE_BGR8`       | `uchar3`  | 24        |
|                 | `bgra8`       | `IMAGE_BGRA8`      | `uchar4`  | 32        |
|                 | `bgr32f`      | `IMAGE_BGR32F`     | `float3`  | 96        |
|                 | `bgra32f`     | `IMAGE_BGRA32F`    | `float4`  | 128       |
| **YUV (4:2:2)** | `yuyv`        | `IMAGE_YUYV`       | `uint8`   | 16        |
|                 | `yuy2`        | `IMAGE_YUY2`       | `uint8`   | 16        |
|                 | `yvyu`        | `IMAGE_YVYU`       | `uint8`   | 16        |
|                 | `uyvy`        | `IMAGE_UYVY`       | `uint8`   | 16        |
| **YUV (4:2:0)** | `i420`        | `IMAGE_I420`       | `uint8`   | 12        |
|                 | `yv12`        | `IMAGE_YV12`       | `uint8`   | 12        |
|                 | `nv12`        | `IMAGE_NV12`       | `uint8`   | 12        |
| **Bayer**       | `bayer-bggr`  | `IMAGE_BAYER_BGGR` | `uint8`   | 8         |
|                 | `bayer-gbrg`  | `IMAGE_BAYER_GBRG` | `uint8`   | 8         |
|                 | `bayer-grbg`  | `IMAGE_BAYER_GRBG` | `uint8`   | 8         |
|                 | `bayer-rggb`  | `IMAGE_BAYER_RGGB` | `uint8`   | 8         |
| **Grayscale**   | `gray8`       | `IMAGE_GRAY8`      | `uint8`   | 8         |
|                 | `gray32f`     | `IMAGE_GRAY32F`    | `float`   | 32        |
* 비트 수는 각 픽셀에 효과적인 비트 수를 나타냅니다.
* YUV 포맷의 구체적은 스펙은 다음 페이지를 참고하세요. [fourcc.org](http://fourcc.org/yuv.php)

> **note:** C++에서 RGB/RGBA는 꼭 `uchar3`/`uchar4`/`float3`/`float4` 벡터 타입을 사용해야하는 유일한 포맷입니다. 나열된 타입들이 사용되면 주어진 이미지는 RGB/RGBA로 간주되기 때문입니다.

이미지의 데이터 포맷과 색공간(colorspace)를 변환하고 싶다면, 아래 [Color Conversion](#color-conversion) 섹션을 확인하세요.

## 이미지 할당 (Allocation)
중간 과정, 결과 이미지를 저장하기 위해 (i.e. 이미지를 처리할 때 사용할 working 메모리) 빈 GPU 메모리를 할당하기 위해서는 C++이나 Python 으로 작성된 [`cudaAllocMapped()`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaMappedMemory.h) 중 하나의 함수를 사용하세요. [`videoSource`](aux-streaming#source-code)의 입력 스트립은 자동으로 사용할 메모리를 GPU에 할당해놓습니다. 그리고 해당 메모리에 남아있는 이미지를 그대로 반환합니다. 따라서 이를 위해 메모리를 따로 할당할 필요가 없습니다.

[`cudaAllocMapped()`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaMappedMemory.h) 에 의해 할당된 메모리는 shared CPU/GPU 메모리 공간에 할당됩니다.
따라서 CPU, GPU 사이에 메모리를 복사하지 않더라도 CPU, GPU 어디에서든 해당 이미지에 접근할 수 있습니다. (이를 그래서 ZeroCopy 메모리 라고도 합니다.)

반면 동기화(Synchronization)이 필요합니다. GPU 연산을 한 이후, CPU에서 해당 이미지에 접근하고 싶다면 먼저 [`cudaDeviceSynchronize()`](https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__DEVICE.html#group__CUDART__DEVICE_1g10e20b05a95f638a4071a655503df25d) 함수를 호출해야합니다. C++에서 메모리를 free 하고 싶다면, [`cudaFreeHost()`](https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__MEMORY.html#group__CUDART__MEMORY_1g71c078689c17627566b2a91989184969) 함수를 사용하세요. Python에서는 메모리가 자동으로 가비지 컬렉터에 의해 반환되겠지만 명시적으로 메모리를 반환하려면 `del` 을 사용하세요.

아래 psuedocode는 ZeroCopy 메모리를 할당/동기화/해제 하는 예제입니다.

#### Python
```python
import jetson.utils

# allocate a 1920x1080 image in rgb8 format
img = jetson.utils.cudaAllocMapped(width=1920, height=1080, format='rgb8')

# do some processing on the GPU here
...

# wait for the GPU to finish processing
jetson.utils.cudaDeviceSynchronize()

# Python will automatically free the memory, but you can explicitly do it with 'del'
del img
```

#### C++
```cpp
#include <jetson-utils/cudaMappedMemory.h>

void* img = NULL;

// allocate a 1920x1080 image in rgb8 format
if( !cudaAllocMapped(&img, 1920, 1080, IMAGE_RGB8) )
	return false;	// memory error

// do some processing on the GPU here 
...

// wait for the GPU to finish processing
CUDA(cudaDeviceSynchronize());

// release the memory
CUDA(cudaFreeHost(img));
```

C++에서는 포인터의 타입을 `uchar3/uchar4/float3/float4` 와 같이 선언한다면 명시적인 [`imageFormat`](#image-formats) 을 생략할 수 있습니다. 아래 코드는 위의 할당 예제와 정확히 일치하는 코드 예제입니다.:

```cpp
uchar3* img = NULL;	// can be uchar3 (rgb8), uchar4 (rgba8), float3 (rgb32f), float4 (rgba32f)

if( !cudaAllocMapped(&img, 1920, 1080) )
	return false;	
```

> **note:** 만약 위와 같은 벡터 타입을 사용한다면, 이러한 이미지들은 이들의 맞는 RGB/RGBA colorspace 로 가정됩니다. 그래서 만약 `uchar3/uchar4/float3/float4` BGR/BGRA 데이터를 포함하는 이미지를 나타내는 경우, 적절한 [image format](#image-formats)을 명시적으로 사용하지 않는다면 일부 처리 기능에 의해 RGB/RGBA로 입력될 수 있습니다.

## 이미지 복사(copy)

`cudaMemcpy()` 는 같은 dimension과 포맷을 갖는 이미지 사이의 메모리를 복사할 때 이용될 수 있습니다. [`cudaMemcpy()`](https://docs.nvidia.com/cuda/cuda-runtime-api/group__CUDART__MEMORY.html#group__CUDART__MEMORY_1gc263dbe6574220cc776b45438fc351e8) 는 C++로 제공되는 표준 CUDA 함수이고, Python에서는 jetson.utils 라이브러리에서 같은 버전의 함수를 제공합니다. 

#### Python
```python
import jetson.utils

# load an image and allocate memory to copy it to
img_a = jetson.utils.loadImage("my_image.jpg")
img_b = jetson.utils.cudaAllocMapped(width=img_a.width, height=img_a.height, format=img_a.format)

# copy the image (dst, src)
jetson.utils.cudaMemcpy(img_b, img_a)

# or you can use this shortcut, which will make a duplicate
img_c = jetson.utils.cudaMemcpy(img_a)
```

#### C++
```cpp
#include <jetson-utils/cudaMappedMemory.h>
#include <jetson-utils/imageIO.h>

uchar3* img_a = NULL;
uchar3* img_b = NULL;

int width = 0;
int height = 0;

// load example image
if( !loadImage("my_image.jpg", &img_a, &width, &height) )
	return false;	// loading error
	
// allocate memory to copy it to
if( !cudaAllocMapped(&img_b, width, height) )
	return false;  // memory error
	
// copy the image (dst, src)
if( CUDA_FAILED(cudaMemcpy(img_b, img_a, width * height * sizeof(uchar3), cudaMemcpyDeviceToDevice)) )
	return false;  // memcpy error
```

## Python에서의 이미지 캡슐(Image Capsules)

Python에서 이미지를 할당하거나 [`videoSource.Capture()`](aux-streaming#source-code)로 비디오 피드에서 이미지를 캡쳐해올 때, 이는 self-contained memory capsule 객체 (of type `<jetson.utils.cudaImage>`)를 반환합니다. self-contained memory capsule은 메모리를 복사할 필요가 없이 전달될 수 있습니다. 

```python
<jetson.utils.cudaImage>
  .ptr      # memory address (not typically used)
  .size     # size in bytes
  .shape    # (height,width,channels) tuple
  .width    # width in pixels
  .height   # height in pixels
  .channels # number of color channels
  .format   # format string
  .mapped   # true if ZeroCopy
```

따라서 `img.width` 나 `img.height`를 사용하여 이미지의 properties에 접근할 수 있습니다.

### Python에서의 이미지 데이터에 접근하기

CUDA 이미지들은 subscriptable 합니다. 이는 CPU에서 직접 pixel 데이터에 직접 접근하기 위해 인덱싱 할 수 있다는 의미입니다.:

```python
for y in range(img.height):
	for x in range(img.width):
		pixel = img[y,x]    # returns a tuple, i.e. (r,g,b) for RGB formats or (r,g,b,a) for RGBA formats
		img[y,x] = pixel    # set a pixel from a tuple (tuple length must match the number of channels)
```

> **note:** Python subscripting operator는 오직 이미지가 mapped ZeroCopy memory에 할당되었을 때만 이용가능 합니다 (i.e. by [`cudaAllocMapped()`](#image-allocation)). 그렇지 않으면, CPU에서는 접근할 수 없으며, exception이 throw됩니다.

이미지에 접근하기 위해 사용되는 인덱싱 튜플은 다음과 같은 폼을 따를 수 있습니다.:

* `img[y,x]` - note the ordering of the `(y,x)` tuple, same as numpy
* `img[y,x,channel]` - only access a particular channel (i.e. 0 for red, 1 for green, 2 for blue, 3 for alpha)
* `img[y*img.width+x]` - flat 1D index, access all channels in that pixel

이미지 subscripting이 지원되더라도 큰 이미지에서 각각의 pixel에 접근하는 것은 python에서 권장되지 않습니다. 이는 어플리케이션을 굉장히 느려지게 만들기 때문입니다. GPU를 사용할 수 없다고 가정한다면 Numpy를 사용하는데 더 나은 대안입니다.

### Numpy Array로 변환

먼저 `jetson.utils.cudaToNumpy()`를 호출함으로써 `cudaImage` 메모리 캡슐에 Numpy로 접근할 수 있습니다. underlying 메모리는 복사되지 않고 Numpy에서 이에 직접 접근합니다. 따라서 Numpy에서 해당 데이터를 in-place(직접)로 바꾼다면, `cudaImage`에서도 바뀌게 됩니다.

`cudaToNumpy()`를 사용한 예제를 원한다면, jetson-utils의 샘플 [`cuda-to-numpy.py`](https://github.com/dusty-nv/jetson-utils/blob/master/python/examples/cuda-to-numpy.py)을 보시길 바랍니다.

OpenCV는 BGR colorspace를 원하기 때문에 만약 OpenCV를 사용하려면 `cv2.cvtColor()`함수를 `cv2.COLOR_RGB2BGR` 옵션과 함께 사용하여 OpenCV 함수를 적용하기 전에 colorspace를 바꾸어줘야합니다.

### Numpy array로 부터 변환하기

Numpy의 ndarray로된 이미지가 있다고 해봅시다. 아마 OpenCV로 부터 주어졌겠죠? Numpy Array로는 오직 CPU에서 밖에 접근할 수 없습니다. 여기서 `jetson.utils.cudaFromNumpy()`를 사용하여 GPU에 이를 복사할 수 있습니다. (into shared CPU/GPU ZeroCopy memory).  

`cudaFromNumpy()`를 사용한 예제를 원한다면, jetson-utils의 예제인 [`cuda-from-numpy.py`](https://github.com/dusty-nv/jetson-utils/blob/master/python/examples/cuda-from-numpy.py)를 살펴보세요.


## 색 변환(Color Conversion)

[`cudaConvertColor()`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaColorspace.h) 함수를 이미지의 포맷과 colorspace를 변환하기 위해 GPU를 사용합니다. 예로, RGB에서 BGR로 변환할 수도 있고(반대도 마찬가지) YUV에서 RGB로, RGB에서 Gray scale로도 바꿀 수 있습니다. 또한 데이터 타입과 채널의 수도 바꿀 수 있습니다. (e.g. RGB8 -> RGBA32F). 변환할 수 있는 더 많은 예를 원한다면 위 [Image Formats](#image-formats) 섹션을 확인하세요.

[`cudaConvertColor()`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaColorspace.h)는 다음과 같은 지원하지 않는 변환과 한계가 있습니다.:

* YUV 포맷은 BGR/BGRA 포맷과 Grayscale을 지원하지 않습니다. (RGB/RGBA만 가능)
* YUV NV12, YUYV, YVYU, UYVY는 오직 RGB/RGBA로만 변환 가능합니다. (반대는 아님)
* Bayer 포맷은 오직 RGB8 (`uchar3`) 와 RGBA8 (`uchar4`) 로만 변환 가능합니다.

다음 Python/C++ psuedocode는 RGB8로 이미지를 불러오고 이를 RGBA32F로 변환합니다. (이는 보여지기 위한 것이므로, 원래는 RGBA32F 포맷으로 바로 이미지를 불러올 수 있습니다.) 더 많은 예제를 위해서는 [`cuda-examples.py`](https://github.com/dusty-nv/jetson-utils/tree/master/python/examples/cuda-examples.py) 를 확인하세요.

#### Python

```python
import jetson.utils

# load the input image (default format is rgb8)
imgInput = jetson.utils.loadImage('my_image.jpg', format='rgb8') # default format is 'rgb8', but can also be 'rgba8', 'rgb32f', 'rgba32f'

# allocate the output as rgba32f, with the same width/height as the input
imgOutput = jetson.utils.cudaAllocMapped(width=imgInput.width, height=imgInput.height, format='rgba32f')

# convert from rgb8 to rgba32f (the formats used for the conversion are taken from the image capsules)
jetson.utils.cudaConvertColor(imgInput, imgOutput)
```

#### C++

```c++
#include <jetson-utils/cudaColorspace.h>
#include <jetson-utils/cudaMappedMemory.h>
#include <jetson-utils/imageIO.h>

uchar3* imgInput = NULL;   // input is rgb8 (uchar3)
float4* imgOutput = NULL;  // output is rgba32f (float4)

int width = 0;
int height = 0;

// load the image as rgb8 (uchar3)
if( !loadImage("my_image.jpg", &imgInput, &width, &height) )
	return false;

// allocate the output as rgba32f (float4), with the same width/height
if( !cudaAllocMapped(&imgOutput, width, height) )
	return false;

// convert from rgb8 to rgba32f
if( CUDA_FAILED(cudaConvertColor(imgInput, IMAGE_RGB8, imgOutput, IMAGE_RGBA32F, width, height)) )
	return false;	// an error or unsupported conversion occurred
```

## 리사이즈(Resizing)

[`cudaResize()`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaResize.h) 함수는 (다운샘플링이던 업샘플링이던)다른 크기로 이미지의 크기를 변환하기 위해 GPU를 사용합니다. 다음 Python/C++ psuedocode는 이미지를 불러오고 이를 특정한 비율로 resize합니다. 더 많은 예제를 위해선 [`cuda-examples.py`](https://github.com/dusty-nv/jetson-utils/tree/master/python/examples/cuda-examples.py)를 보세요.

#### Python

```python
import jetson.utils

# load the input image
imgInput = jetson.utils.loadImage('my_image.jpg')

# allocate the output, with half the size of the input
imgOutput = jetson.utils.cudaAllocMapped(width=imgInput.width * 0.5, 
                                         height=imgInput.height * 0.5, 
                                         format=imgInput.format)

# rescale the image (the dimensions are taken from the image capsules)
jetson.utils.cudaResize(imgInput, imgOutput)
```

#### C++

```c++
#include <jetson-utils/cudaResize.h>
#include <jetson-utils/cudaMappedMemory.h>
#include <jetson-utils/imageIO.h>

// load the input image
uchar3* imgInput = NULL;

int inputWidth = 0;
int inputHeight = 0;

if( !loadImage("my_image.jpg", &imgInput, &inputWidth, &inputHeight) )
	return false;

// allocate the output image, with half the size of the input
uchar3* imgOutput = NULL;

int outputWidth = inputWidth * 0.5f;
int outputHeight = inputHeight * 0.5f;

if( !cudaAllocMapped(&imgOutput, outputWidth, outputHeight) )
	return false;

// rescale the image
if( CUDA_FAILED(cudaResize(imgInput, inputWidth, inputHeight, imgOutput, outputWidth, outputHeight)) )
	return false;
```

## 이미지 자르기(Cropping)

[`cudaCrop()`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaCrop.h) 함수는 GPU를 사용하여 이미지를 특별한 region of interest(ROI)로 자릅니다. 다음 Python/C++ psudocode는 이미지를 불러오고, 중심을 기준으로 이미지를 반으로 자릅니다. 더 많은 예제를 위해서 다음을 보세요. [`cuda-examples.py`](https://github.com/dusty-nv/jetson-utils/tree/master/python/examples/cuda-examples.py).


여기서 ROI rectangles 은 `(left, top, right, bottom)` 좌표들로 제공됩니다.

#### Python

```python
import jetson.utils

# load the input image
imgInput = jetson.utils.loadImage('my_image.jpg')

# determine the amount of border pixels (cropping around the center by half)
crop_factor = 0.5
crop_border = ((1.0 - crop_factor) * 0.5 * imgInput.width,
               (1.0 - crop_factor) * 0.5 * imgInput.height)

# compute the ROI as (left, top, right, bottom)
crop_roi = (crop_border[0], crop_border[1], imgInput.width - crop_border[0], imgInput.height - crop_border[1])

# allocate the output image, with the cropped size
imgOutput = jetson.utils.cudaAllocMapped(width=imgInput.width * crop_factor,
                                         height=imgInput.height * crop_factor,
                                         format=imgInput.format)

# crop the image to the ROI
jetson.utils.cudaCrop(imgInput, imgOutput, crop_roi)
```

#### C++

```c++
#include <jetson-utils/cudaCrop.h>
#include <jetson-utils/cudaMappedMemory.h>
#include <jetson-utils/imageIO.h>

// load the input image
uchar3* imgInput = NULL;

int inputWidth = 0;
int inputHeight = 0;

if( !loadImage("my_image.jpg", &imgInput, &inputWidth, &inputHeight) )
	return false;

// determine the amount of border pixels (cropping around the center by half)
const float crop_factor = 0.5
const int2  crop_border = make_int2((1.0f - crop_factor) * 0.5f * inputWidth,
                                    (1.0f - crop_factor) * 0.5f * inputHeight);

// compute the ROI as (left, top, right, bottom)
const int4 crop_roi = make_int4(crop_border.x, crop_border.y, inputWidth - crop_border.x, inputHeight - crop_border.y);

// allocate the output image, with half the size of the input
uchar3* imgOutput = NULL;

if( !cudaAllocMapped(&imgOutput, inputWidth * crop_factor, inputHeight * cropFactor) )
	return false;

// crop the image
if( CUDA_FAILED(cudaCrop(imgInput, imgOutput, crop_roi, inputWidth, inputHeight)) )
	return false;
```

## 정규화(Normalization)

[`cudaNormalize()`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaNormalize.h) 함수는 GPU를 사용하여 이미지의 픽셀값들의 범위를 바꿉니다. 예를 들어 `[0,255]` 사이의 픽셀값들을 가지고 있던 이미지를 `[0,1]` 의 픽셀값을 갖도록 변환합니다. 또 다른 자주 사용하는 범위로써 `[-1,1]`가 있습니다.

> **note:** jetson-inference, jetson-utils의 모든 함수들은 이미지의 pixel값이 `[0,255]` 사이에 있기를 기대합니다. 그래서 보통은 사용할 일이 없지만 다른 소스나 결과로 사용할 때 사용 가능합니다.  

아래 Python/C++ pseudocode는 이미지를 불러오고, 이를 `[0,255]` 에서 `[0,1]`로 정규화(Normalization)하는 예제입니다.

#### Python

```python
import jetson.utils

# load the input image (its pixels will be in the range of 0-255)
imgInput = jetson.utils.loadImage('my_image.jpg')

# allocate the output image, with the same dimensions as input
imgOutput = jetson.utils.cudaAllocMapped(width=imgInput.width, height=imgInput.height, format=imgInput.format)

# normalize the image from [0,255] to [0,1]
jetson.utils.cudaNormalize(imgInput, (0,255), imgOutput, (0,1))
```

#### C++

```c++
#include <jetson-utils/cudaNormalize.h>
#include <jetson-utils/cudaMappedMemory.h>
#include <jetson-utils/imageIO.h>

uchar3* imgInput = NULL;
uchar3* imgOutput = NULL;

int width = 0;
int height = 0;

// load the input image (its pixels will be in the range of 0-255)
if( !loadImage("my_image.jpg", &imgInput, &width, &height) )
	return false;

// allocate the output image, with the same dimensions as input
if( !cudaAllocMapped(&imgOutput, width, height) )
	return false;

// normalize the image from [0,255] to [0,1]
CUDA(cudaNormalize(imgInput, make_float2(0,255),
                   imgOutput, make_float2(0,1),
                   width, height));
```

## 오버레이(Overlay)

[`cudaOverlay()`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaOverlay.h) 함수는 GPU를 사용하여 출력 이미지 위에 특정한 위치에 입력 이미지를 합성합니다. 오버레이 연산들은 보통 여러 장의 이미지를 하나로 합칠 때 연속적으로 호출됩니다.

아래 Python/C++ pseudocode는 이미지를 불러오고 출력 이미지를 나란히 합성합니다.

#### Python

```python
import jetson.utils

# load the input images
imgInputA = jetson.utils.loadImage('my_image_a.jpg')
imgInputB = jetson.utils.loadImage('my_image_b.jpg')

# allocate the output image, with dimensions to fit both inputs side-by-side
imgOutput = jetson.utils.cudaAllocMapped(width=imgInputA.width + imgInputB.width, 
                                         height=max(imgInputA.height, imgInputB.height),
                                         format=imgInputA.format)

# compost the two images (the last two arguments are x,y coordinates in the output image)
jetson.utils.cudaOverlay(imgInputA, imgOutput, 0, 0)
jetson.utils.cudaOverlay(imgInputB, imgOutput, imgInputA.width, 0)
```

#### C++

```c++
#include <jetson-utils/cudaOverlay.h>
#include <jetson-utils/cudaMappedMemory.h>
#include <jetson-utils/imageIO.h>

#include <algorithm>  // for std::max()

uchar3* imgInputA = NULL;
uchar3* imgInputB = NULL;
uchar3* imgOutput = NULL;

int2 dimsA = make_int2(0,0);
int2 dimsB = make_int2(0,0);

// load the input images
if( !loadImage("my_image_a.jpg", &imgInputA, &dimsA.x, &dimsA.y) )
	return false;

if( !loadImage("my_image_b.jpg", &imgInputB, &dimsB.x, &dimsB.y) )
	return false;

// allocate the output image, with dimensions to fit both inputs side-by-side
const int2 dimsOutput = make_int2(dimsA.x + dimsB.x, std::max(dimsA.y, dimsB.y));

if( !cudaAllocMapped(&imgOutput, dimsOutput.x, dimsOutput.y) )
	return false;

// compost the two images (the last two arguments are x,y coordinates in the output image)
CUDA(cudaOverlay(imgInputA, dimsA, imgOutput, dimsOutput, 0, 0));
CUDA(cudaOverlay(imgInputB, dimsB, imgOutput, dimsOutput, dimsA.x, 0));
```

## 도형 그리기(Drawing Shapes)

[`cudaDraw.h`](https://github.com/dusty-nv/jetson-utils/tree/master/cuda/cudaDraw.h) 에 기본 도형들, 예를 들어 원, 선, 네모들을 그리기 위한 함수들이 정의돼있습니다. 

아래 Python/C++ 예제 pseudocode들은 위 함수들을 이용한 것입니다. 다른 예제를 더 보고 싶다면 [`cuda-examples.py`](https://github.com/dusty-nv/jetson-utils/tree/master/python/examples/cuda-examples.py) 이를 확인하세요. 

#### Python

``` python
# load the input image
input = jetson.utils.loadImage("my_image.jpg")

# cudaDrawCircle(input, (cx,cy), radius, (r,g,b,a), output=None)
jetson.utils.cudaDrawCircle(input, (50,50), 25, (0,255,127,200))

# cudaDrawRect(input, (left,top,right,bottom), (r,g,b,a), output=None)
jetson.utils.cudaDrawRect(input, (200,25,350,250), (255,127,0,200))

# cudaDrawLine(input, (x1,y1), (x2,y2), (r,g,b,a), line_width, output=None)
jetson.utils.cudaDrawLine(input, (25,150), (325,15), (255,0,200,200), 10)
```

> **note:** if the optional `output` image isn't specified, the operation will be performed in-place on the `input` image.

#### C++

``` cpp
#include <jetson-utils/cudaDraw.h>
#include <jetson-utils/imageIO.h>

uchar3* img = NULL;
int width = 0;
int height = 0;

// load example image
if( !loadImage("my_image.jpg", &img, &width, &height) )
	return false;	// loading error
	
// see cudaDraw.h for definitions
CUDA(cudaDrawCircle(img, width, height, 50, 50, 25, make_float4(0,255,127,200)));
CUDA(cudaDrawRect(img, width, height, 200, 25, 350, 250, make_float4(255,127,0,200)));
CUDA(cudaDrawLine(img, width, height, 25, 150, 325, 15, make_float4(255,0,200,200), 10));
```

##
<p align="right">Next | <b><a href="https://github.com/dusty-nv/ros_deep_learning">Deep Learning Nodes for ROS/ROS2</a></b>
<br/>
Back | <b><a href="aux-streaming.md">Camera Streaming and Multimedia</a></p>
<p align="center"><sup>© 2016-2020 NVIDIA | </sup><a href="../README.md#hello-ai-world"><sup>Table of Contents</sup></a></p>

