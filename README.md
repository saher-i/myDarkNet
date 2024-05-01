This repository hosts the modifications made by Saher Iqbal (si2443) to the code originally sourced from AlexeyAB's Darknet GitHub repository (https://github.com/AlexeyAB/darknet). This was done as a part of the final project for her ESP course. 

The project is based on YOLOv4 instead of YOLOv3 because of the constraints in running YOLOv3, which was according to me because YOLOv3 has been outdated now and uses a different darknet. YOLOv3 uses Darknet53 backbone whereas YOLOv4 architecture uses CSPdarknet53 as a backbone.

The following steps were taken to run the code and get the results:

Step 1:

```
git clone https://github.com/AlexeyAB/darknet
cd darknet
mkdir build_release
cd build_release
cmake ..
cmake --build . --target install --parallel 8
```

Step 2:

    --> Installing requirements necessary for running the code on a GCP instance.
    --> A few modifications were made in the code base.

Step 3:

Profilling was done in terms of measuring BFLOPS (billion floating-point operations per second) using the follwing command:
```
perf record -g ./darknet detector train /home/si2443/darknet/cfg/coco.data /home/si2443/darknet/cfg/yolov4.cfg yolov4.conv.137
```
Results obtained were as follows:

    --> Shortcut Layers - 0.001 BF
    --> Max Layer (5x5, 9x9, 13x13 kernels) - 0.005 to 0.031 BF
    --> 1x1 Convolution Layers (~0.095 to ~0.379 BF)
    --> 3x3 Convolution Layers (~1.703 BF)
    --> Higher Dimension Convolution Layers (~3.407 BF)

The above numbers represent approximate figures for each layer.

Total BFLOPS obtained were 128.459

Mini_batch = 8, batch = 64, time_steps = 1, train = 1: Indicates that the model was being trained with a mini-batch size of 8, a batch size of 64, and a single time step for recurrent connections (if any). The training flag is set to 1, meaning that the model is in training mode.

By running the command:
```
perf report
```
I could better visualise the profiling results.

Step 4:

As convolution operation was taking the most time, I wanted to dig deep into the convolution_layer.c code and added print statements to see which parts of the code was taking the most time. I started with the forward_convolution_layer function. 

In order to run my code and get the debug statements printed, I used the follwing command:
```
./darknet detector test /home/si2443/darknet/cfg/coco.data /home/si2443/darknet/cfg/yolov4.cfg /home/si2443/darknet/yolov4_weights/yolov4.weights -thresh 0.25

```
