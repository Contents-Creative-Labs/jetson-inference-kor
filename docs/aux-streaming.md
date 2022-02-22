translated by: 김준호<br>
Date: 2022-02-21<br>


<img src="https://github.com/dusty-nv/jetson-inference/raw/master/docs/images/deep-vision-header.jpg" width="100%">
<p align="right"><sup><a href="../README.md#appendix">Back</a> | <a href="aux-image.md">Next</a> | </sup><a href="../README.md#hello-ai-world"><sup>Contents</sup></a>
<br/>
<sup>Appendix</sup></p>  

# 카메라 스트리밍과 멀티미디어

해당 프로젝트는 아래 테이블과 같은 다양한 인터페이스와 프로토콜을 통해 비디오 피드와 이미지들을 스트리밍하는 것을 지원합니다.

* [MIPI CSI cameras](#mipi-csi-cameras)
* [V4L2 cameras](#v4l2-cameras)
* [RTP](#rtp) / [RTSP](#rtsp) 
* [Videos](#video-files) & [Images](#image-files)
* [Image sequences](#image-files)
* [OpenGL windows](#output-streams)

스트림들은 resource URI에 의해 구별되며 [`videoSource`](#source-code) 와 [`videoOutput`](#source-code) API들을 통해 접근할 수 있습니다. 아래 테이블에는 각 스트림의 타입에 대하여 지원되는 입출력 프로토콜들이 나와있습니다.

### Input(입력) 스트림

|                  | Protocol     | Resource URI              | Notes                                                    |
|------------------|--------------|---------------------------|----------------------------------------------------------|
| [MIPI CSI camera](#mipi-csi-cameras) | `csi://`     | `csi://0`                 | CSI 카메라 0 (`0`을 다른 카메라 번호로 대체하세요.)    |
| [V4L2 camera](#v4l2-cameras)     | `v4l2://`    | `v4l2:///dev/video0`      | V4L2 device 0 (`0`을 다른 카메라 번호로 대체하세요.)     |
| [RTP stream](#rtp)       | `rtp://`     | `rtp://@:1234`            | localhost, port 1234 (추가적인 configuration(구성) 필요) |
| [RTSP stream](#rtsp)      | `rtsp://`    | `rtsp://<remote-ip>:1234` | `<remote-ip>` 를 원격 host의 ip나 hostname으로 대체하세요. |
| [Video file](#video-files)       | `file://`    | `file://my_video.mp4`     | MP4, MKV, AVI, FLV 를 지원합니다. (아래 codecs를 확인하세요.)   |
| [Image file](#image-files)       | `file://`    | `file://my_image.jpg`     | JPG, PNG, TGA, BMP, GIF, 등을 지원합니다.           |
| [Image sequence](#image-files)   | `file://`    | `file://my_directory/`    | 알파벳 순으로 이미지를 탐색합니다.          |

* 지원하는 디코더 codecs(코덱):  H.264, H.265, VP8, VP9, MPEG-2, MPEG-4, MJPEG
* `file://`, `v4l2://`, 그리고 `csi://` 프로토콜의 prefix(접두사)들은 URI에서 생략할 수 있습니다.

### Output Streams

|                  | Protocol     | Resource URI              | Notes                                                    |
|------------------|--------------|---------------------------|----------------------------------------------------------|
| [RTP stream](#rtp)              | `rtp://`     | `rtp://<remote-ip>:1234`  | Replace `<remote-ip>` with remote host's IP or hostname  |
| [Video file](#video-files)       | `file://`    | `file://my_video.mp4`     | Supports saving MP4, MKV, AVI, FLV (see codecs below)    |
| [Image file](#image-files)       | `file://`    | `file://my_image.jpg`     | Supports saving JPG, PNG, TGA, BMP                       |
| [Image sequence](#image-files)   | `file://`    | `file://image_%i.jpg`     | `%i` is replaced by the image number in the sequence     |
| [OpenGL window](#output-streams)   | `display://` | `display://0`             | Creates GUI window on screen 0                           |

* 지원하는 인코더 (codecs)코덱:  H.264, H.265, VP8, VP9, MJPEG
* `file://` 프로토콜의 prefixes(접미사)는 URI에서 생략될 수 있습니다.
* By default(기본값으로), OpenGL 디스플레이 윈도우는 `--headless` 옵션을 명시하지 않으면 항상 생성됩니다.

## 커맨드 라인 인자

jetson-inference의 각 C++, Python 예제 프로그램들은 스트림 URI와 추가적인 옵션을 위해 같은 커맨드 라인 인자들의 셋을 받습니다. 따라서 해당 옵션들은 어떤한 예제들에서도 사용이 가능합니다. (e.g. [`imagenet`](../examples/imagenet/imagenet.cpp)/[`imagenet.py`](../examples/python/imagenet.py), [`detectnet`](../examples/detectnet/detectnet.cpp)/[`detectnet.py`](../examples/python/detectnet.py), [`segnet`](../examples/segnet/segnet.cpp)/[`segnet.py`](../examples/python/segnet.py), [`video-viewer`](https://github.com/dusty-nv/jetson-utils/tree/master/video/video-viewer/video-viewer.cpp)/[`video-viewer.py`](https://github.com/dusty-nv/jetson-utils/tree/master/python/examples/video-viewer.py), ect). 커맨드 라인 인자들은 보통 다음과 같은 폼을 갖습니다.:

```bash
$ imagenet [options] input_URI [output_URI]  # output URI is optional
```

다음은 input, output URI가 두 개의 positional arguments(위치 인자)로 주어진 것입니다.:

```bash
$ imagenet input.jpg output.jpg              # classify input.jpg, save as output.jpg
```

위에서 언급하였듯이, 같은 커맨드 라인 파싱을 사용한다면 jetson-inference의 어떤 예제라도 이와 같이 대체될 수 있습니다. 아래는 각 프로그램을 수행할 때 사용할 수 있는 추가 스트림 옵션들입니다.

#### Input(입력) 옵션

```
    input_URI            input(입력) 스트림의 리소스 URI (위 테이블을 확인하세요)
  --input-width=WIDTH    명시적으로 입력 스트림의 해상도를 요청 
  --input-height=HEIGHT  (해상도 선택은 옵션, RTP는 필요함)
  --input-codec=CODEC    RTP 아래 코덱중 하나를 사용하도록 설정해야합니다.:
                             * h264, h265
                             * vp8, vp9
                             * mpeg2, mpeg4
                             * mjpeg
  --input-flip=FLIP      input에 적용할 flip method (V4L2 제외):
                             * none (default)
                             * counterclockwise
                             * rotate-180
                             * clockwise
                             * horizontal
                             * vertical
                             * upper-right-diagonal
                             * upper-left-diagonal
  --input-loop=LOOP      파일에 기반한 입력일 때, 반복 수행할 횟수:
                             * -1 = 무한 반복
                             *  0 = 반복 안함 (기본값)
                             * >0 = 반복 횟수 설정
  --input-rtsp-latency=2000
                         들어오는 RTSP 스트림을 버퍼할 milliseconds. 
                             해당 옵션값을 0으로 주면 딜레이가 거의 발생하지 않지만,
			     네트워크 상태에 따라 끊김 현상이 발생할 수 있습니다.
```

#### Output(출력) 옵션

```
    output_URI           출력 스트림의 리소스 URI (위 테이블을 확인하세요.)
  --output-codec=CODEC   출력 스트림에 사용하고자 하는 코덱:
                            * h264 (default), h265
                            * vp8, vp9
                            * mpeg2, mpeg4
                            * mjpeg
  --bitrate=BITRATE      bps, 스트림의 VBR bitrate 
                         기본값은 4000000 (4 Mbps)
  --headless             기본 OpenGL GUI 윈도우를 생성하지 않음.
```

아래는 여러 종류의 스트림으로 `video-viewer` 명령어를 수행한 예제입니다. 인자들만 동일하다면 파싱 방법이 동일하기 때문에 `video-viewer` 대신에 다른 명령어들로 대체하여 수행 시켜볼 수도 있습니다. 해당 페이지의 [Source Code](#source-code) 섹션에서 자신의 어플리케이션에서 `videoSource` 와 `videoOutput` API를 어떻게 사용할지 보기 위해 `video-viewer` 소스 코드의 컨텐츠를 검색해볼 수도 있습니다.


## MIPI CSI 카메라

MIPI CSI 카메라는 작은 소형 센서 카메라입니다. 그리고 이는 jetson의 하드웨어의 CSI/ISP 인터페이스에 연결하여 사용할 수 있습니다. 지원되는 카메라는 아래와 같습니다.:

* [Raspberry Pi Camera Module v2](https://www.raspberrypi.org/products/camera-module-v2/) (IMX219) for Jetson Nano and Jetson Xavier NX
* OV5693 camera module from the Jetson TX1/TX2 devkits.  
* See the [Jetson Partner Supported Cameras](https://developer.nvidia.com/embedded/jetson-partner-supported-cameras) page for more sensors supported by the ecosystem.

아래는 MIPI CSI 카메라를 구동하는 예제입니다. 만약 여러 대의 카메라가 연결돼있다면 카메라의 번호 0 대신 다른 번호를 넣으면 됩니다.:

```bash
$ video-viewer csi://0                        # MIPI CSI camera 0 (substitue other camera numbers)
$ video-viewer csi://0 output.mp4             # save output stream to MP4 file (H.264 by default)
$ video-viewer csi://0 rtp://<remote-ip>:1234 # broadcast output stream over RTP to <remote-ip>
```

기본값으로, CSI 카메라는 1280x720 의 해상도로 생성됩니다. 다른 해상도를 사용하기 위해선 `--input-width` 와 `input-height` 옵션을 사용할 수 있습니다. 주의할 점은 직접 지정한 해상도가 카메라가 지원하는 포맷중 하나의 것과 일치하도록 값을 설정해야 한다는 것입니다. 

```bash
$ video-viewer --input-width=1920 --input-height=1080 csi://0
```

## V4L2(USB) 카메라

USB 웹캠은 대부분 주로 V4L2 장치 형태로 지원됩니다. 예로 Logitech [C270](https://www.logitech.com/en-us/product/hd-webcam-c270) or [C920](https://www.logitech.com/en-us/product/hd-pro-webcam-c920) 같은 카메라가 있습니다.

```bash
$ video-viewer v4l2:///dev/video0                 # /dev/video0 can be replaced with /dev/video1, ect.
$ video-viewer /dev/video0                        # dropping the v4l2:// protocol prefix is fine
$ video-viewer /dev/video0 output.mp4             # save output stream to MP4 file (H.264 by default)
$ video-viewer /dev/video0 rtp://<remote-ip>:1234 # broadcast output stream over RTP to <remote-ip>
```

> **주의:**  만얄 MIPI CSI 카메라가 연결돼있다면, 이는 `/dev/video0` 로 연결이 돼있습니다. 그럴 때 만약 USB 웹캠도 연결돼 있다면, 이는  `/dev/video1` 으로 나타납니다. 따라서 V4L2 카메라를 사용하기 위해선 위 예제 커맨드 라인을 `/dev/video1` 로 적절히 대체하여 수행해야 합니다. CSI 카메라를 V4L2로 사용하는 것은 해당 프로젝트에서 지원하지 않습니다. V4L2에서 ISP를 사용하지 않고 raw Bayer를 사용하기 때문입니다. (CSI 카메라를 사용하려면 [위](#mipi-csi-cameras)를 참고하세요. 

#### V4L2 포맷

기본적으로, V4L2 카메라는 지정된 해상도 중 (1280x720이 기본값) 가장 높은 FPS를 선택하여 이들의 카메라 포맷으로 생성됩니다. 가장 높은 FPS를 갖는 포맷은 H.264나 MJPEG으로 인코드 될 수 있습니다. 왜냐하면 USB 카메라는 보통 압축되지 않은 YUV/RGB 영상을 전송할 때 낮은 FPS로 전송되기 때문입니다. 이러한 경우에, 해당 코덱은 이를 감지하고, 높은 FPS를 얻기 위해 Jetson의 하드웨어 디코더를 이용하여 카메라 스트림을 자동으로 디코딩합니다.

만약 명시적으로 V4L2가 사용할 포맷을 선택하고 싶다면, `--input-width`, `--input-height`, 와 `--input-codec` 옵션을 사용하여 이와 같이 할 수 있습니다.
가능한 디코더 옵션은 다음과 같습니다. `--input-codec=h264, h265, vp8, vp9, mpeg2, mpeg4, mjpeg`

```bash
$ video-viewer --input-width=1920 --input-height=1080 --input-codec=h264 /dev/video0
```

V4L2 소스를 이용하여 jetson-inference 프로그램들 중 하나를 실행할 때 V4L2 카메라가 지원하는 다른 포맷드이 터미널에 로깅됩니다. 그렇지만 `v4l2-ctl` 명령어를 통해서고 지원하는 해당 리스트들을 터미널에 출력할 수 있습니다.:


```bash
$ sudo apt-get install v4l-utils
$ v4l2-ctl --device=/dev/video0 --list-formats-ext
```
  
## RTP

RTP 네트워크 스트림은 UDP/IP 상에서 특정 호스트나 멀티캐스트 그룹으로 브로드캐스트됩니다. RTP 스트림을 수신할 때 반드시 코덱을 명시적으로 알려줘야합니다 (`--input-codec`). 왜냐하면 RTP는 이를 자동으로 쿼리하지 못하기 때문입니다. 이는 다른 디바이스로부터 RTP를 입력으로 사용합니다.:

```bash
$ video-viewer --input-codec=h264 rtp://@:1234         # recieve on localhost port 1234
$ video-viewer --input-codec=h264 rtp://224.0.0.0:1234 # subscribe to multicast group
```

위 커맨드 라인들은 RTP를 입력 소스로 명시하고, 여기서 네트워크의 다른 원격 호스트가 Jetson으로 스트리밍합니다. 또한 Jetson으로 부터 RTP 스트림을 output(출력)할 수 있으며 네트워크 상의 다른 원격 호스트에게도 송신할 수 있습니다.

#### RTP 송신

RTP 출력 스트림을 송신하기 위해선 `output_URI`에 타겟 IP/포트 를 명시해야합니다. 만약 필요하다면 비트레이트나 출력 코덱을 명시할 수 있습니다. (비트레이트의 기본값은 `--bitrate=4000000` , 4Mbps) 이며, (코덱의 기본값은 `--output-codec=h264`)입니다. 이는 `h264, h265, vp8, vp9, mjpeg`들의 코덱으로도 사용할 수 있습니다.

```bash
$ video-viewer --bitrate=1000000 csi://0 rtp://<remote-ip>:1234         # transmit camera over RTP, encoded as H.264 @ 1Mbps 
$ video-viewer --output-codec=h265 my_video.mp4 rtp://<remote-ip>:1234  # transmit a video file over RTP, encoded as H.265
```

RTP를 출력하고 있다면 해당 스트림이 송신될 원격 호스트의 IP 주소나 호스트 이름을 명시적으로 설정해야합니다. (위에서는 `<remote-ip>`와 같이 사용합니다.) PC에서 RTP 스트림을 보기 위한 몇가지 포인트들을 살펴봅시다. 

#### 원격으로 RTP (view)보기

If your Jetson is transmitting RTP to another remote host (like a PC), here are some example commands that you can use to view the stream:

* Using GStreamer:
	* [Install GStreamer](https://gstreamer.freedesktop.org/documentation/installing/index.html) and run this pipeline (replace `port=1234` with the port you are using)
	
	```bash
	$ gst-launch-1.0 -v udpsrc port=1234 \
	caps = "application/x-rtp, media=(string)video, clock-rate=(int)90000, encoding-name=(string)H264, payload=(int)96" ! \
	rtph264depay ! decodebin ! videoconvert ! autovideosink
	```
	
* Using VLC Player:
	* Create a SDP file (.sdp) with the following contents (replace `1234` with the port you are using)
	
	```
     c=IN IP4 127.0.0.1
     m=video 1234 RTP/AVP 96
     a=rtpmap:96 H264/90000
	```
	
	* Open the stream in VLC by double-clicking the SDP file
	* You may want to reduce the `File caching` and `Network caching` settings in VLC as [shown here](https://www.howtogeek.com/howto/windows/fix-for-vlc-skipping-and-lagging-playing-high-def-video-files/)
	
* If your remote host is another Jetson:
	* Use the same `video-viewer` command as [above](#rtp) (replace `1234` with the port you are using)
	
	```bash
	$ video-viewer --input-codec=h264 rtp://@:1234
     ```

## RTSP

RTSP network streams are subscribed to from a remote host over UDP/IP.  Unlike RTP, RTSP can dynamically query the stream properties (like resolution and codec), so these options don't need to be explicitly provided.

```bash
$ video-viewer rtsp://<remote-ip>:1234 my_video.mp4      # subscribe to RTSP feed from <remote-ip>, port 1234 (and save it to file)
$ video-viewer rtsp://username:password@<remote-ip>:1234 # with authentication (replace username/password with credentials)
```

> **note:** RTSP is supported as an input only.  Outputting RTSP would require additional support in GStreamer for an RTSP server.

## Video Files

You can playback and record video files in MP4, MKV, AVI, and FLV formats.

```bash
# playback
$ video-viewer my_video.mp4                              # display the video file
$ video-viewer my_video.mp4 rtp://<remote-ip>:1234       # transmit the video over RTP

# recording
$ video-viewer csi://0 my_video.mp4                      # record CSI camera to video file
$ video-viewer /dev/video0 my_video.mp4                  # record V4L2 camera to video file
```

#### Codecs

When loading video files, the codec and resolution is automatically detected, so these don't need to be set.
When saving video files, the default codec is H.264, but this can be set with the `--output-codec` option.

```bash
$ video-viewer --output-codec=h265 input.mp4 output.mp4  # transcode video to H.265
```

The following codecs are supported:

* Decode - H.264, H.265, VP8, VP9, MPEG-2, MPEG-4, MJPEG
* Encode - H.264, H.265, VP8, VP9, MJPEG


#### Resizing Inputs

When loading video files, the resolution is automatically detected.  However, if you would like the input video to be re-scaled to a different resolution, you can specify the `--input-width` and `--input-height` options:

```bash
$ video-viewer --input-width=640 --input-height=480 my_video.mp4  # resize video to 640x480
```

#### Looping Inputs

By default, the video will terminate once the end of stream (EOS) is reached.  However, by specifying the `--loop` option, you can set the number of loops that you want the video to run for.  Possible options for `--loop` are:

* `-1` = loop forever
* &nbsp;` 0` = don't loop (default)
* `>0` = set number of loops

```bash
$ video-viewer --loop=10 my_video.mp4    # loop the video 10 times
$ video-viewer --loop=-1 my_video.mp4    # loop the video forever (until user quits)
```

## Image Files

You can load/save image files in the following formats:

* Load:  JPG, PNG, TGA, BMP, GIF, PSD, HDR, PIC, and PNM (PPM/PGM binary)
* Save:  JPG, PNG, TGA, BMP

```bash
$ video-viewer input.jpg output.jpg	# load/save an image
```

You can also loop images and image sequences - see the [Looping Inputs](#looping-inputs) section above.

#### Sequences

If the path is a directory or contains wildcards, all of the images will be loaded/saved sequentially (in alphanumeric order).

```bash
$ video-viewer input_dir/ output_dir/   # load all images from input_dir and save them to output_dir
$ video-viewer "*.jpg" output_%i.jpg    # load all jpg images and save them to output_0.jpg, output_1.jpg, ect
```

> **note:** when using wildcards, always enclose it in quotes (`"*.jpg"`). Otherwise, the OS will auto-expand the sequence and modify the order of arguments on the command-line, which may result in one of the input images being overwritten by the output.

When saving a sequence of images, if the path is just to a directory (`output_dir`), then the images will automatically be saved as JPG with the format `output_dir/%i.jpg`, using the image number as it's filename (`output_dir/0.jpg`, `output_dir/1.jpg`, ect).  

If you wish to specify the filename format, do so by using the printf-style `%i` in the path (`output_dir/image_%i.png`).  You can apply additional printf modifiers such as `%04i` to create filenames like `output_dir/image_0001.jpg`.


## Source Code

Streams are accessed using the [`videoSource`](https://github.com/dusty-nv/jetson-utils/tree/master/video/videoSource.h) and [`videoOutput`](https://github.com/dusty-nv/jetson-utils/tree/master/video/videoOutput.h) objects.  These have the ability to handle each of the types of streams from above through a unified set of APIs.  Images can be captured and output in the following data formats:  

| Format string | [`imageFormat` enum](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/group__imageFormat.html#ga931c48e08f361637d093355d64583406) | Data Type | Bit Depth |
|---------------|------------------|-----------|-----------|
| `rgb8`        | `IMAGE_RGB8`     | `uchar3`  | 24        |
| `rgba8`       | `IMAGE_RGBA8`    | `uchar4`  | 32        |
| `rgb32f`      | `IMAGE_RGB32F`   | `float3`  | 96        |
| `rgba32f`     | `IMAGE_RGBA32F`  | `float4`  | 128       |

* the Data Type and [`imageFormat`](https://rawgit.com/dusty-nv/jetson-inference/dev/docs/html/group__imageFormat.html#ga931c48e08f361637d093355d64583406) enum are C++ types
* in Python, the format string can be passed to `videoSource.Capture()` to request a specific format (the default is `rgb8`)
* in C++, the `videoSource::Capture()` template will infer the format from the data type of the output pointer

To convert images to/from different formats, see the [Image Manipulation with CUDA](aux-image.md) page for more info.

Below is the source code to `video-viewer.py` and `video-viewer.cpp`, slightly abbreviated to improve readability:

### Python
```python
import jetson.utils
import argparse
import sys

# parse command line
parser = argparse.ArgumentParser()
parser.add_argument("input_URI", type=str, help="URI of the input stream")
parser.add_argument("output_URI", type=str, default="", nargs='?', help="URI of the output stream")
opt = parser.parse_known_args()[0]

# create video sources & outputs
input = jetson.utils.videoSource(opt.input_URI, argv=sys.argv)
output = jetson.utils.videoOutput(opt.output_URI, argv=sys.argv)

# capture frames until user exits
while output.IsStreaming():
	image = input.Capture(format='rgb8')  // can also be format='rgba8', 'rgb32f', 'rgba32f'
	output.Render(image)
	output.SetStatus("Video Viewer | {:d}x{:d} | {:.1f} FPS".format(image.width, image.height, output.GetFrameRate()))
```

### C++
```c++
#include "videoSource.h"
#include "videoOutput.h"

int main( int argc, char** argv )
{
	// create input/output streams
	videoSource* inputStream = videoSource::Create(argc, argv, ARG_POSITION(0));
	videoOutput* outputStream = videoOutput::Create(argc, argv, ARG_POSITION(1));
	
	if( !inputStream )
		return 0;

	// capture/display loop
	while( true )
	{
		uchar3* nextFrame = NULL;  // can be uchar3, uchar4, float3, float4

		if( !inputStream->Capture(&nextFrame, 1000) )
			continue;

		if( outputStream != NULL )
		{
			outputStream->Render(nextFrame, inputStream->GetWidth(), inputStream->GetHeight());

			// update status bar
			char str[256];
			sprintf(str, "Video Viewer (%ux%u) | %.1f FPS", inputStream->GetWidth(), inputStream->GetHeight(), outputStream->GetFrameRate());
			outputStream->SetStatus(str);	

			// check if the user quit
			if( !outputStream->IsStreaming() )
				break;
		}

		if( !inputStream->IsStreaming() )
			break;
	}

	// destroy resources
	SAFE_DELETE(inputStream);
	SAFE_DELETE(outputStream);
}
```

<p align="right">Next | <b><a href="aux-image.md">Image Manipulation with CUDA</a></b>
<p align="center"><sup>© 2016-2020 NVIDIA | </sup><a href="../README.md#hello-ai-world"><sup>Table of Contents</sup></a></p>

