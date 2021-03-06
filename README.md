# tensorrt

SSD
------------

1. Install requirements (pycuda, etc.) and build TensorRT engines from the trained SSD models.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/ssd
   $ ./install.sh
   $ ./build_engines.sh
   ```

2. Run the 'trt_ssd.py' demo program.  The demo supports 4 models: 'ssd_mobilenet_v1_coco', 'ssd_mobilenet_v1_egohands', 'ssd_mobilenet_v2_coco', or 'ssd_mobilenet_v2_egohands'.  For example, I tested the 'ssd_mobilenet_v1_coco' model with the 'huskies' picture.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos
   $ python3 trt_ssd.py --model ssd_mobilenet_v1_coco \
                        --image \
                        --filename ${HOME}/project/tf_trt_models/examples/detection/data/huskies.jpg
   ```

   Here's the result (JetPack-4.2.2, i.e. TensorRT 5).  Frame rate was good (over 20 FPS).

   ![Huskies detected](https://raw.githubusercontent.com/jkjung-avt/tensorrt_demos/master/doc/huskies.png)


   I also tested the 'ssd_mobilenet_v1_egohands' (hand detector) model with a video clip from YouTube, and got the following result.  Again, frame rate was pretty good.  But the detection didn't seem very accurate though :-(

   ```shell
   $ python3 trt_ssd.py --model ssd_mobilenet_v1_egohands \
                        --file \
                        --filename ${HOME}/Videos/Nonverbal_Communication.mp4
   ```

   (Click on the image below to see the whole video clip...)

   [![Hands detected](https://raw.githubusercontent.com/jkjung-avt/tensorrt_demos/master/doc/hands.png)](https://youtu.be/3ieN5BBdDF0)

3. The 'trt_ssd.py' demo program could also take various image inputs.  Refer to step 5 in Demo #1 again.

4. Referring to this comment, ['#TODO enable video pipeline'](https://github.com/AastaNV/TRT_object_detection/blob/master/main.py#L78), in the original TRT_object_detection code, I did implement an 'async' version of ssd detection code to do just that.  When I tested 'ssd_mobilenet_v1_coco' on the same huskies image with the async demo program, frame rate improved 3~4 FPS.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos
   $ python3 trt_ssd_async.py --model ssd_mobilenet_v1_coco \
                              --image \
                              --filename ${HOME}/project/tf_trt_models/examples/detection/data/huskies.jpg
   ```

5. To verify accuracy (mAP) of the optimized TensorRT engines and make sure they do not degrade too much (due to reduced floating-point precision of 'FP16') from the original TensorFlow frozen inference graphs, you could prepare validation data and run 'eval_ssd.py'.  Refer to [README_eval_ssd.md](README_eval_ssd.md) for details.

   | TensorRT engine         | mAP @<br>IoU=0.5:0.95 |  mAP @<br>IoU=0.5  |    FPS      |
   |:------------------------|:---------------------:|:------------------:|:-----------:|
   | mobilenet_v1 TF         |          0.232        |        0.351       |      --     |
   | mobilenet_v1 TRT (FP16) |          0.232        |        0.351       |     65.2    |
   | mobilenet_v2 TF         |          0.248        |        0.375       |      --     |
   | mobilenet_v2 TRT (FP16) |          0.248        |        0.375       |     61.4    |


<a name="YOLOv3"></a>
YOLOv3
---------------

Along the same line as Demo #3, this demo showcases how to convert a trained YOLOv3 model through ONNX to a TensorRT engine.  This demo also requires TensorRT 'Python API' and has been verified working against both TensorRT 5.x and 6.x.

Assuming this repository has been cloned at '${HOME}/project/tensorrt_demos', follow these steps:

1. Install 'pycuda' in case you have not done so in Demo #3.  Note that installation script resides in the 'ssd' folder.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/ssd
   $ ./install_pycuda.sh
   ```

2. Install version '1.4.1' (not the latest) of python3 'onnx' module.  Reference: [information provided by NVIDIA](https://devtalk.nvidia.com/default/topic/1052153/jetson-nano/tensorrt-backend-for-onnx-on-jetson-nano/post/5347666/#5347666).

   ```shell
   $ sudo pip3 install onnx==1.4.1
   ```

3. Download the trained YOLOv3 COCO models and convert the targeted model to ONNX and then to TensorRT engine.  This demo supports 5 models: 'yolov3-tiny-288', 'yolov3-tiny-416',  'yolov3-288', 'yolov3-416', and 'yolov3-608'.  **NOTE: I'm not sure whether my implementation of the 'yolov3-tiny-288' and 'yolov3-tiny-416' models is correct.  They are for reference only.**

   I use 'yolov3-416' as example below.

   ```shell
   $ cd ${HOME}/project/tensorrt_demos/yolov3_onnx
   $ ./download_yolov3.sh
   $ python3 yolov3_to_onnx.py --model yolov3-416
   $ python3 onnx_to_tensorrt.py --model yolov3-416
   ```

   The last step ('onnx_to_tensorrt.py') takes a little bit more than half an hour to complete on my Jetson Nano DevKit.  When that is done, the optimized TensorRT engine would be saved as 'yolov3-416.trt'.

4. Test the YOLOv3 TensorRT engine with the 'dog.jpg' image.

   ```shell
   $ wget https://raw.githubusercontent.com/pjreddie/darknet/master/data/dog.jpg -O ${HOME}/Pictures/dog.jpg
   $ python3 trt_yolov3.py --model yolov3-416
                           --image --filename ${HOME}/Pictures/dog.jpg
   ```

   This was tested against JetPack-4.3, i.e. TensorRT 6.

   ![YOLOv3-416 detection result on dog.jpg](https://raw.githubusercontent.com/jkjung-avt/tensorrt_demos/master/doc/dog_trt_yolov3.png)

5. The 'trt_yolov3.py' demo program could also take various image inputs.  Refer to step 5 in Demo #1 again.

6. I created 'eval_yolov3.py' for evaluating mAP of the optimized YOLOv3 engine.  It works the same way as 'eval_ssd.py'.  Refer to step #5 in Demo #3.

   ```shell
   $ python3 eval_yolov3.py --model yolov3-288
   $ python3 eval_yolov3.py --model yolov3-416
   $ python3 eval_yolov3.py --model yolov3-608
   ```

   I evaluated all of yolov3-tiny-288, yolov3-tiny-416, yolov3-288, yolov3-416 and yolov3-608 TensorRT engines with COCO 'val2017' data and got the following results.  The FPS (frames per second) numbers were measured using 'trt_yolov3.py' on my Jetson Nano DevKit with JetPack-4.3.

   | TensorRT engine        | mAP @<br>IoU=0.5:0.95 |  mAP @<br>IoU=0.5  | 
   |:-----------------------|:---------------------:|:------------------:|
   | yolov3-tiny-288 (FP16) |          0.077        |        0.158       |
   | yolov3-tiny-416 (FP16) |          0.096        |        0.202       |
   | yolov3-288 (FP16)      |          0.331        |        0.600       |
   | yolov3-416 (FP16)      |          0.373        |        0.664       |
   | yolov3-608 (FP16)      |          0.376        |        0.665       |
   | yolov3-608 (FP32)      |          0.376        |        0.665       |


Licenses
--------

I referenced source code of [NVIDIA/TensorRT](https://github.com/NVIDIA/TensorRT) samples to develop most of the demos in this repository.  Those NVIDIA samples are under [Apache License 2.0](https://github.com/NVIDIA/TensorRT/blob/master/LICENSE).
