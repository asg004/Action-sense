# Action Sense 

# Installation

First, Python >= 3.6

## Step 1 - Install Dependencies

Check this [installation guide](https://github.com/CV-ZMH/Installation-Notes-for-Deeplearning-Developement#install-tensorrt) for deep learning packages installation.

Here is required packages for this project and you need to install each of these.
1. Nvidia-driver 450
2. [Cuda-10.2](https://developer.nvidia.com/cuda-10.2-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal) and [Cudnn 8.0.5](https://developer.nvidia.com/rdp/cudnn-archive)
3. [Pytorch 1.7.1](https://pytorch.org/get-started/previous-versions/) and [Torchvision 0.8.2](https://pytorch.org/get-started/previous-versions/)
4. [TensorRT 7.1.3](https://docs.nvidia.com/deeplearning/tensorrt/archives/tensorrt-723/install-guide/index.html)
5. [ONNX 1.9.0](https://pypi.org/project/onnx/)

## Step 2 - Install [torch2trt](https://github.com/NVIDIA-AI-IOT/torch2trt)
```bash
git clone https://github.com/NVIDIA-AI-IOT/torch2trt
cd torch2trt
sudo python3 setup.py install --plugins
```
## Step 3 - Install trt_pose

```bash
git clone https://github.com/NVIDIA-AI-IOT/trt_pose
cd trt_pose
sudo python setup.py install
```
Other python packages are in [`requirements.txt`](requirements.txt).

Run below command to install them.
```bash
pip install -r requirements.txt
```
---

# Run Quick Demo

## Step 1 - Download the Pretrained Models
Action Classifier Pretrained models are already uploaded in the path `weights/classifier/dnn`.
- Download the pretrained weight files to run the demo.

| Model Type | Name | Trained Dataset |  Weight |
|---|---|---|---|
| Pose Estimation | trtpose | COCO       |[densenet121](https://drive.google.com/file/d/1De2VNUArwYbP_aP6wYgDJfPInG0sRYca/view?usp=sharing) |
|||||
| Tracking        | deepsort reid| Market1501 | [wide_resnet](https://drive.google.com/file/d/1xw7Sv4KhrzXMQVVQ6Pc9QeH4QDpxBfv_/view?usp=sharing)|
| Tracking        | deepsort reid| Market1501 | [siamese_net](https://drive.google.com/file/d/11OmfZqnnG4UBOzr05LEKvKmTDF5MOf2H/view?usp=sharing)|
| Tracking        | deepsort reid| Mars | [wide_resnet](https://drive.google.com/file/d/1lRvkNsJrR4qj50JHsKStEbaSuEXU3u1-/view?usp=sharing)|
| Tracking        | deepsort reid| Mars | [siamese_net](https://drive.google.com/file/d/1eyj5zKoLjnHqfSIz2eJjXq0r9Sw7k0R0/view?usp=sharing)|


- Then put them to these folder
    - *deepsort* weight to `weights/tracker/deepsort/`
    - *trt_pose* weight to `weights/pose_estimation/trtpose`.

## Step 2 - TensorRT Conversion (Optional)

If you don't have installed tensorrt on your system, just skip this step. You just need to set pytorch model weights of the corresponding [model path](configs/infer_trtpose_deepsort_dnn.yaml) in this config file.

Convert **trtpose model**
```bash
# check the I/O weight file in configs/trtpose.yaml
cd export_models
python convert_trtpose.py --config ../configs/infer_trtpose_deepsort_dnn.yaml
```
:bangbang:  Original **densenet121_trtpose** model is trained with **256** input size. So, if you want to convert tensorrt model with bigger input size (like 512), you need to change [size](https://github.com/CV-ZMH/human_activity_recognition/blob/ad2f8adfbd30e1ae1ea3b964a2f144ce757d944a/configs/infer_trtpose_deepsort_dnn.yaml#L6) parameter in [`configs/infer_trtpose_deepsort_dnn.yaml`](configs/infer_trtpose_deepsort_dnn.yaml) file.

Convert **Deepsort reid model**, pytorch >> onnx >> tensorRT
```basg
cd export_models
#1. torch to onnx
python convert_reid2onnx.py \
--model_path <your reid model path> \
--reid_name <siamesenet/wideresnet> \
--dataset_name <market1501/mars> \
--check

#2. onnx to tensorRT
python convert_reid2trt.py \
--onnx_path <your onnx model path> \
--mode fp16 \
--max_batch 100

#3. check your tensorrt converted model with pytorch model
python test_trt_inference.py \
--trt_model_path <your tensorrt model path> \
--torch_model_path <your pytorch model path> \
--reid_name <siamesenet/wideresnet> \
--dataset_name <market1501/mars> \
```

## Step 3 - Run Demo.py

Arguments list of `Demo.py`
- **task** [pose, track, action] : Inference mode for testing _Pose Estimation_, _Tracking_ or _Action Recognition_.
- **config** : inference config file path. (default=../configs/inference_config.yaml)
- **source** : video file path to predict action or track. If not provided, it will use *webcam* source as default.
- **save_folder** : save the result video folder path. Output filename format is composed of "_{source video name/webcam}{pose network name}{deepsort}{reid network name}{action classifier name}.avi_". If  not provided, it will not save the result video.
- **draw_kp_numbers** : flag to draw keypoint numbers of each person for visualization.
- **debug_track** : flag to debug tracking for tracker's bbox state and current detected bbox of tracker's inner process with bboxes visualization.

:bangbang:	Before running the `demo.py`, you need to change some parameters in
 [confiigs/infer_trtpose_deepsort_dnn.yaml](configs/infer_trtpose_deepsort_dnn.yaml) file.

Examples:
-  To use different reid network, [`reid_name`](https://github.com/CV-ZMH/human_activity_recognition/blob/ad2f8adfbd30e1ae1ea3b964a2f144ce757d944a/configs/infer_trtpose_deepsort_dnn.yaml#L37) and it's [`model_path`](https://github.com/CV-ZMH/human_activity_recognition/blob/ad2f8adfbd30e1ae1ea3b964a2f144ce757d944a/configs/infer_trtpose_deepsort_dnn.yaml#L38) in [`TRACKER`](https://github.com/CV-ZMH/human-action-recognition/blob/6bdf3a541eb6adc618c67e4e6df7d6fdaddffb1b/configs/infer_trtpose_deepsort_dnn.yaml#L20) node.
- Set also [`model_path`](https://github.com/CV-ZMH/human-action-recognition/blob/6bdf3a541eb6adc618c67e4e6df7d6fdaddffb1b/configs/infer_trtpose_deepsort_dnn.yaml#L13) of trtpose  in [`POSE`](https://github.com/CV-ZMH/human-action-recognition/blob/6bdf3a541eb6adc618c67e4e6df7d6fdaddffb1b/configs/infer_trtpose_deepsort_dnn.yaml#L4) node.
- You can also tune other parameters of [`TRACKER`](https://github.com/CV-ZMH/human-action-recognition/blob/6bdf3a541eb6adc618c67e4e6df7d6fdaddffb1b/configs/infer_trtpose_deepsort_dnn.yaml#L20) node and [`POSE`](https://github.com/CV-ZMH/human-action-recognition/blob/6bdf3a541eb6adc618c67e4e6df7d6fdaddffb1b/configs/infer_trtpose_deepsort_dnn.yaml#L4) node for better tracking and action recognition result.

Then, Run **action recogniiton**.
```bash
cd src
# for video, use --source flag to your video path
python demo.py --task action --source ../test_data/fun_theory.mp4 --save_folder ../output --debug_track
# for webcam, no need to provid --source flag
python demo.py --task action --save_path ../output --debug_track
```

Run **pose tracking**.
```bash
# for video, use --src flag to your video path
python demo.py --task track --source ../test_data/fun_theory.mp4 --save_path ../output
# for webcam, no need to provid --source flag
python demo.py --task track --save_path ../output
```

Run **pose estimation** only.
```bash
# for video, use --src flag to your video path
python demo.py --task pose --source ../test_data/fun_theory.mp4 --save_path ../output
# for webcam, no need to provid --source flag
python demo.py --task pose --save_path ../output
```
---

# Training

## Train Action Classifier Model
- For this step, it's almost same preparations as original repo to train action classifier. You can directly reference of dataset preparation, feature extraction information and training information from the [original repo](https://github.com/felixchenfy/Realtime-Action-Recognition).
- Then, Download the sample training dataset from the [original repo](https://drive.google.com/open?id=1V8rQ5QR5q5zn1NHJhhf-6xIeDdXVtYs9)

- Modify the [`data_root`](https://github.com/CV-ZMH/human_activity_recognition/blob/ad2f8adfbd30e1ae1ea3b964a2f144ce757d944a/configs/train_action_recogn_pipeline.yaml#L5) and [`extract_path`](https://github.com/CV-ZMH/human_activity_recognition/blob/ad2f8adfbd30e1ae1ea3b964a2f144ce757d944a/configs/train_action_recogn_pipeline.yaml#L6) with your IO dataset path in [configs/train_action_recogn_pipeline.yaml](configs/train_action_recogn_pipeline.yaml).

- Depend on your custom action training, you have to change the parameters in  [configs/train_action_recogn_pipeline.yaml](configs/train_action_recogn_pipeline.yaml).

- Then run below command to train action classifier step by step.
```bash
cd src && bash ./train_trtpose_dnn_action.sh
```

## Train reID Model for DeepSort Tracking
To train different reid network for cosine metric learning used in deepsort:
- Download the reid dataset [Mars](https://www.kaggle.com/twoboysandhats/mars-motion-analysis-and-reidentification-set)
- Prepare `mars` dataset with this command. This will split train/val from mars bbox-train folder and calculate mean & std over the train set. Use this mean & std for dataset normalization.
```bash
cd src && python prepare_mars.py --root <your dataset root> --train_percent 0.8 --bs 256
```
- Modify the [`tune_params`](https://github.com/CV-ZMH/human_activity_recognition/blob/ad2f8adfbd30e1ae1ea3b964a2f144ce757d944a/configs/train_reid.yaml#L26) for multiple runs to find hyper parameter search as your need.

- Then run below to train reid network.
```bash
cd src && python train_reid.py --config ../configs/train_reid.yaml
```
---

# TODO  
- [x] Add different reid network used in DeepSort
- [x] Add tensorrt for reid model
- [ ] Add more pose estimation models
- [ ] Add more tracking methods
- [ ] Add more action recognition models
