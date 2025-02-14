# Whole Application Acceleration

## Table of Contents
- [Directory structure introduction](#Directory-structure-introduction)
- [Background](#Background)
- [Model Performance](#Model-Performance)

## Directory structure introduction

Whole Application Acceleration (WAA) examples demonstrate how Xilinx® [Vitis Vision library](https://github.com/Xilinx/Vitis_Libraries/tree/master/vision) functions can be integrated with deep neural network (DNN) accelerator to achieve complete application acceleration. This application focuses on accelerating the pre/post-processing involved in inference of detection & classification networks.


```
Whole-Application-Acceleration
├── README.md
├── plugins
│   ├── nms                                 # Post-processing Accelerator source files
│   ├── blobfromimage                       # Pre-processing Accelerator source files
│   ├── of_tvl1                             # TVL1 optical flow Accelerator source files
│   ├── jpeg_decoder                        # JPEG decoder Accelerator xo file
│   ├── cost_volume                         # cost_volume Accelerator xo file
│   └── include                             # lib files from HW accelerator
└── apps
    ├── adas_detection                      # Adas detection application
    │      ├── README.md
    │      ├── app_build.sh
    │      ├── app_test.sh
    │      ├── src
    │      └── build_flow                   # Hardware build using source files
    │          ├── DPUCZDX8G_zcu102
    │          ├── DPUCAHX8H_u50_u280
    │          ├── DPUCADF8H_u200
    │          └── DPUCVDX8G_vck190
    ├── psmnet                              # PSMNet application
    │      ├── build_flow                   # Hardware build using source files
    │      │      └── DPUCVDX8G_vck190
    │      ├── build.sh
    │      ├── cost_volume.cpp
    │      ├── cost_volume.hpp
    │      ├── cpu_op.cpp
    │      ├── cpu_op.hpp
    │      ├── demo_psmnet.cpp
    │      ├── dpu_resize.cpp
    │      ├── dpu_resize.hpp
    │      ├── dpu_sfm.cpp
    │      ├── dpu_sfm.hpp
    │      ├── Makefile
    │      ├── my_xrt_bo.cpp
    │      ├── my_xrt_bo.hpp
    │      ├── performance.sh
    │      ├── psmnet_benchmark.hpp
    │      ├── psmnet.cpp
    │      ├── psmnet.hpp
    │      ├── README.md
    │      ├── test_cost_volume.cpp
    │      ├── test_dpu_task.cpp
    │      ├── test_performance_psmnet.cpp
    │      ├── test_resize_xrt.cpp
    │      ├── test_xrt_bo_alloc.cpp
    │      ├── vai_aie_task_handler.hpp
    │      └── vai_graph.hpp
    ├── resnet50                            # resnet50 application
    │      ├── README.md
    │      ├── app_build.sh
    │      ├── app_test.sh
    │      ├── src
    │      └── build_flow                   # Hardware build using source files
    │          ├── DPUCZDX8G_zcu102
    │          ├── DPUCAHX8H_u50_u280
    │          ├── DPUCADF8H_u200
    │          └── DPUCVDX8G_vck190
    ├── resnet50_jpeg                       # resnet50_jpeg application
    │      ├── README.md
    │      ├── app_build.sh
    │      ├── app_test.sh
    │      ├── src
    │      ├── libs                         #libs for jpeg-decoder
    │      └── build_flow                   # Hardware build using source files
    │          └── DPUCZDX8G_zcu102
    ├ fall_detection
    ├ ssd_mobilenet
    └ segmentation
```

## Background

Input images are preprocessed  before being fed for inference of different deep neural networks. The pre-processing steps vary from network to network. For example, for classification networks like Resnet-50 the input image is resized to 224 x 224 size and then channel-wise mean subtraction is performed before feeding the data to the DNN accelerator. For detection networks like YOLO v3 the input image is resized to 256 x 512 size using letterbox before feeding the data to the DNN accelerator. Similarly, some networks like SSD-Mobilenet also require the inference output to be processed to filter out unncessary data. Non Maximum Supression (NMS) is an example of post-processing function.


[Vitis Vision library](https://github.com/Xilinx/Vitis_Libraries/tree/master/vision) provides functions optimized for FPGA devices that are drop-in replacements for standard OpenCV library functions. These application demonstrates how Vitis Vision library functions can be used to accelerate the complete application.


## Model Performance
Refer below table for examples with demonstration on complete acceleration of various applications where pre/post-processing of several networks are accelerated on different target platforms.


| No. | Application                                           | Backbone Network | Accelerated Part(s)        | H/W Accelerated Functions                     | DPU Supported (% Improvement Over Non-WAA App) |
|-----|-------------------------------------------------------|------------------|----------------------------|-----------------------------------------------|--------------------------------------------------------------------------|
| 1   | [adas_detection](./apps/adas_detection/README.md)     | Yolo v3          | Pre-process                | resize, letter box, scale , DPU               | ZCU102  (44.53%) , ALVEO-U50 (32.75%) , ALVEO-U200 (23.62%) , ALVEO-U280 (26.49%), VCK190 (368.32%)                  |
| 2   | [fall_detection](./apps/fall_detection/README.md)     | VGG16            | Pre-process                | Lucas-Kanade Dense Non-Pyramidal Optical Flow | ALVEO-U200 (27.01 %)                                                        |
| 3   | [resnet50](./apps/resnet50/README.md)                 | resnet-50        | Pre-process                | resize, mean subtraction, scale, DPU          |  ZCU102  (49.95 %) , ALVEO-U50 (24.85 %) , ALVEO-U200 (25.13 %) , ALVEO-U280 (27.22 %), VCK190 (330 %)                   |
| 4   | [resnet50_jpeg](./apps/resnet50_jpeg/README.md)                 | resnet-50        | Pre-process                | JPEG decoder, resize, mean subtraction, scale, DPU          |  ZCU102  (*109.04%)                    |
| 5   | [ssd_mobilenet](./apps/ssd_mobilenet/README.md)       | mobilenet        | pre-process & post-process | resize, BGR2RGB, scale, sort, NMS, DPU        | ALVEO-U280 (74.23 %)                                                         |
| 6   | [segmentation](./apps/segmentation/README.md)         | fcn8               | pre-process & post-process                      | resize, normalization, argmax                                         | VCK190 (59.5 %)                                                 |
| 7   | [PSMNET](./apps/psmnet/README.md)              | PSMnet           | Intra-process              | Cost Volume Generation            | VCK190 (99.46 %)                                                |


`*` Includes imread/jpeg-decoder latency in both WAA and non WAA app

`**` Impact shown is on pre/post processing acceleration

:pushpin: **Note:** Non-WAA applications use equivalent OpenCV functions. As OpenCV's LK OF is sparse, equivalent non-WAA application uses OpenCV's Farneback OF algorithm with 10 threads.
