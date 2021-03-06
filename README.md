
# Revision History

- V 1.3.1 2018/4/27

1. 一个label只允许一个tag
2. 把最后一个tag操作作为默认值，给新建的label

- V 1.3.0 2018/4/26

1. 支持内嵌视频浏览

- V 1.2.1 2018/4/17

1. 禁用打开文件目录
2. 默认framerate改成6
3. ffmpeg质量改成8

- V 1.2.0 2018/4/13

1. 用ffmpeg把视频自动转换成ogg格式用于标注
2. 把ffmpeg打包到发布包中, 把x64和ia32的ffmpeg都加到包里了。没搞清楚打包的时候怎么选择一个。

- V 1.1.2 2018/4/10

1. 标记为empty frame的帧导出成negative。没有任何标注的帧不导出。

- V 1.1.1 2018/4/9

1. bugfix: 切换Frame之后，标注的编号变成一样的，无法删除

- V 1.1.0 2018/4/4

1.  forked from Microsoft/VoTT, 根据业务需求做修改适应影像标注需求

2. 初始化过程 - 
    1. 把标题改成"US Video ..."; 2
    2. 配置对话框汉化
    3. 配置对话框增加默认tag BP
    4. 配置对话框默认帧速率5
    5. 配置对话框增加配置Enable auto tracking, 默认值false
    6. 
3. 修改emptyframe造成无法前后翻页的bug

4. 改用electron-builder打包



# VoTT: Visual Object Tagging Tool

This tool provides end to end support for generating datasets and validating object detection models from video and image assets.

### End to End Object Detection Pipeline:
![Pipeline: tag video, export tags to CNTK, train model, run model on a new video, validate model suggestions and fix errors, return to export tags](media/detectioninabox.jpg)

The tool supports the following **features**:

- The ability to tag and annotate Image Directories or Stand alone videos.
- Computer-assisted tagging and tracking of objects in videos using the [Camshift tracking algorithm](http://opencv.jp/opencv-1.0.0_org/docs/papers/camshift.pdf).
- Exporting tags and assets to CNTK , Tensorflow (PascalVOC) or YOLO format for training an object detection model.
- Running and validating a trained CNTK object detection model on new videos to generate stronger models.

## Table of Contents

 - [Installation](#installation)
 - [Tagging a Video](#tagging-a-video)
 - [Tagging an Image Directory](#tagging-an-image-directory)
 - [Reviewing and Improving an Object Detection Model](#reviewing-and-improving-an-object-detection-model)
 - [Upcoming Features](#upcoming-features)
 - [How to Contribute](#how-to-contribute)

---
## Installation

### Installing the Visual Object Tagging Tool

 1. Download and extract the app [release package](https://github.com/CatalystCode/CNTK-Object-Detection-Video-Tagging-Tool/releases)

 2. Run the app by launching the "VOTT" executable which will be located inside the unzipped folder.

 ### Installing CNTK with the FRCNN Prerequisites for Reviewing Model

*Please note that installation of **CNTK and FASTER-RCNN dependencies** are **optional for tagging** and are **only required for CNTK model review and training**.*

1. Install [CNTK](https://github.com/Microsoft/CNTK/wiki/Setup-CNTK-on-your-machine) (*Note: currently the tool only supports the full installation method (non pip) of CNTK*).

2. Follow the setup instructions of the [CNTK Faster-RCNN tutorial](https://docs.microsoft.com/en-us/cognitive-toolkit/object-detection-using-faster-r-cnn#setup) (*Note: Faster-RCNN currently only supports Linux python version 3.4 and not 3.5*).

3. Configure `CNTK-Config.json` (which resides in the '\resources\app' directory of the tagging tool) with the following properties to enable the model review feature:

```json
{
    "cntkPath" : "{CNTK Path default is c:/local/cntk}",
}
```
## Tagging a Video

 1. Select the option to tag a video

    ![](media/video-option.jpg)

 2. Load an MP4 video file either by dragging it into the app or clicking on and selecting it.

    ![](media/2_load.jpg)

 3. Configure the tagging job and specify the settings in the screenshot below:

    ![](media/3_Job_Configuration.jpg)

    **Frame Extraction Rate**: number of frames to tag per second of video<br>

    **Tagging Region Type**:  type of bounding box for tagging regions<br>
      - *Rectangle*: tag bounding boxes of any dimension
      - *Square*: tag bounding boxes of auto-fixed dimensions

    **Suggested Region Method**: how to suggest regions for next frame<br>
     - *Tracking*: Use camshift to track tagged regions in next frame
     - *Copy Last Frame*: Copy all regions to the next frame.

    **Enable Scene Change Detection**: Detect scene changes to prevent false positives when tracking. (Note this option is slightly slower)

    **Labels**: labels of the tagged regions (e.g. `Cat`, `Dog`, `Horse`, `Person`)<br>

 4. Tag the video frame by frame
 
    ![](media/4_Tagging_Job.jpg)

    **Tagging**: click and drag a bounding box around the desired area, then move or resize the region until it fits the object
     - Selected regions appear as red ![red](https://placehold.it/15/f03c15/000000?text=+) and unselected regions will appear as blue ![#1589F0](https://placehold.it/15/1589F0/000000?text=+).
     - Assign a tag to a region by clicking on it and selecting the desired tag from the labeling toolbar at the bottom of the tagging control
     - Click the ![cleartags](media/cleartags.png) button to clear all tags on a given frame

    **Navigation**: users can navigate between video frames by using the ![prev-nxt](media/prev-next.png) buttons, the left/right arrow keys, or the video skip bar
     - Tags are auto-saved each time a frame is changed

    **Tracking**: new regions are tracked by default until a given scene changes.
     - Since the [camshift algorithm](http://opencv.jp/opencv-1.0.0_org/docs/papers/camshift.pdf) has some known limitations, you can disable tracking for certain sets of frames. To toggle tracking *on* and *off* use the file menu setting, or the keyboard shortcut Ctrl/Cmd + T.


 5. Export video Tags using the Object Detection Menu or Ctrl/Cmd + E

    ![]( media/5_Export.jpg)
    
    *Note on exporting: the tool reserves a random 20% sample of the tagged frames as a test set.*
 
    Specify the following export configuration settings:
    
    ![]( media/5a_Export.jpg)
    
    - **Export Format**: What framework to export to defaults to *CNTK*<br>
    - **Export Frames Until**: how far into the video the export operation will proceed<br>
      - *Last Tagged Region*: exports frames up until the last frame containing tags
      - *Last Visited Frame*: exports frames up until the last frame that the user explicitly visited
      - *Last Frame*: exports all video frames<br>
    - **Output directory**: directory path for exporting training data<br>
    
---

## Tagging an Image Directory

 1. Select the option to tag an image directory

    ![](media/image-option.jpg)

 2. Load an image directory by selecting it.

    ![](media/2_load.jpg)

 3. Configure the tagging job and specify the settings in the screenshot below:

    ![](media/3_image_Job_Configuration.jpg)

    **Frame Extraction Rate**: number of frames to tag per second of video<br>

    **Tagging Region Type**:  type of bounding box for tagging regions<br>
      - *Rectangle*: tag bounding boxes of any dimension
      - *Square*: tag bounding boxes of auto-fixed dimensions

    **Labels**: labels of the tagged regions (e.g. `Cat`, `Dog`, `Horse`, `Person`)<br>

 4. Tag each Image
 
    ![](media/4_image_Tagging_Job.jpg)

    **Tagging**: click and drag a bounding box around the desired area, then move or resize the region until it fits the object
     - Selected regions appear as red ![red](https://placehold.it/15/f03c15/000000?text=+) and unselected regions will appear as blue ![#1589F0](https://placehold.it/15/1589F0/000000?text=+).
     - Assign a tag to a region by clicking on it and selecting the desired tag from the labeling toolbar at the bottom of the tagging control
     - Click the ![cleartags](media/cleartags.png) button to clear all tags on a given frame

    **Navigation**: users can navigate between video frames by using the ![prev-nxt](media/prev-next.png) buttons, the left/right arrow keys, or the video skip bar
     - Tags are auto-saved each time a frame is changed

 5. Export Image directory tags Tags using the Object Detection Menu or Ctrl/Cmd + E

    ![]( media/5_image_Export.jpg)
    
    *Note on exporting: the tool reserves a random 20% sample of the tagged frames as a test set.*
 
    Specify the following export configuration settings:
    
    ![]( media/5a_Export.jpg)
    
    - **Export Format**: What framework to export to defaults to *CNTK*<br>
    - **Export Frames Until**: how far into the video the export operation will proceed<br>
      - *Last Tagged Region*: exports frames up until the last frame containing tags
      - *Last Visited Frame*: exports frames up until the last frame that the user explicitly visited
      - *Last Frame*: exports all video frames<br>
    - **Output directory**: directory path for exporting training data<br>
    
---
## Reviewing and Improving an Object Detection Model

 1. Train model with [Object Detection using FasterRCNN](https://docs.microsoft.com/en-us/cognitive-toolkit/object-detection-using-faster-r-cnn#run-faster-r-cnn-on-your-own-data)<br> 
 2. Since CNTK does not embed the names of the classes in the model, on default, the module returns non descriptive names for the classes, e.g. "class_1", "class_2". To resolve this copy the class_map.txt file generated by the FASTER-RCNN tutorial to the same directory that contains your model.   

 3. Load a new asset that the model has not been trained on
 4. Configure a new or load a previous tagging job 
 4. Apply model to new asset using Ctrl/Cmd + R
 5. Specify a model path and temporary output directory<br> 
    ![](media/6_Review.jpg)
 6. When the model finishes processing, validate tags, re-export and retrain it
 7. Repeat step 1 on new assets until the model performance is satisfactory
 
 
## Supporting additonal object detection Export and Review formats.

In the latest release we provide support for [Export and Review formats](https://github.com/CatalystCode/VOTT/tree/master/src/lib/detection_algorithms). To add a new object detection format, copy the interface folder and use the Yolo and CNTK implementations as reference. 

## Upcoming Features 

- Tagging project management
- Altenative Tracking algorithms such as optical flow.
- Classification Labeling Support
- Segmentation Annoatation support.

-----------

## How to Contribute

You are welcome to send us any bugs you may find, suggestions, or any other comments.

Before sending anything, please go over the repository issues list, just to make sure that it isn't already there.

You are more than welcome to fork this repository and send us a pull request if you feel that what you've done should be included.
