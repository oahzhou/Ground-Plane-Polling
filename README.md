# Ground Plane Polling for 6DoF Pose Estimation of Objects on the Road

Keras implementation for training and testing the models described in [Ground Plane Polling for 6DoF Pose Estimation of Objects on the Road](http://cvrr.ucsd.edu/publications/2018/GPP.pdf).
This repository was created by modifying the pre-existing Keras RetinaNet implementation found [here](https://github.com/fizyr/keras-retinanet). The original README from Keras Retinanet has been reproduced here with relevant modifications.

## Installation
1) Clone this repository.
2) Ensure numpy is installed using `pip install numpy --user`
3) In the repository, execute `pip install . --user`.
   Note that due to inconsistencies with how `tensorflow` should be installed,
   this package does not define a dependency on `tensorflow` as it will try to install that (which at least on Arch Linux results in an incorrect installation).
   Please make sure `tensorflow` is installed as per your systems requirements. The code has been tested with Keras 2.2.0 and Tensorflow 1.9.0.

## Dataset preparation
### 1. Download the KITTI object detection dataset and organize the data in directories as below:
```plain
└── KITTI_DATASET_ROOT
    |── raw
    |   ├── data_object_calib
    |   |   ├── training
    |   |   └── testing
    |   ├── data_object_image_2
    |   |   ├── training
    |   |   └── testing
    |   └── data_object_label_2
    |       └── training
    └── road_planes_database.mat
```
In the above example, the `road_planes_database.mat` file is an (N x 4) array of ground planes for the KITTI dataset. Different versions of this file can be found [here](https://github.com/arangesh/Ground-Plane-Polling/tree/master/road_planes_database).

Ensure that the filename is `road_planes_database.mat` irrespective of which ground plane database you end up using.

### 2. Create modified labels for training using the MATLAB script found [here](https://github.com/arangesh/Ground-Plane-Polling/blob/master/label_prep/create_mod_labels.m).

### 3. Create train-val split using [this Python script](https://github.com/arangesh/Ground-Plane-Polling/blob/master/prepare_kitti_data.py) as follows:
```shell
python3 prepare_kitti_data.py --output-dir=KITTI_DATASET_ROOT
```

## Training
Ground Plane Polling (GPP) models can be trained using [this](https://github.com/arangesh/Ground-Plane-Polling/blob/master/keras_retinanet_3D/bin/train.py) script.
Note that the train script uses relative imports since it is inside the `keras_retinanet-3D` package.
If you want to adjust the script for your own use outside of this repository, you will need to switch it to use absolute imports.

If you installed `keras-retinanet-3D` correctly, the train script will be installed as `retinanet-3D-train`.
However, if you make local modifications to the `keras-retinanet-3D` repository, you should run the script directly from the repository.
That will ensure that your local changes will be used by the train script.

The default backbone is `resnet50`. You can change this using the `--backbone=xxx` argument in the running script. For example:
```shell
python3 keras_retinanet_3D/bin/train.py --backbone=resnet50 --batch-size=2 --random-transform kitti /path/to/kitti/dataset/root
```

Trained models can't be used directly for inference. To convert a trained model to an inference model, do:
```shell
python3 keras_retinanet_3D/bin/convert_model.py --backbone=resnet50 /path/to/training/model /path/to/save/inference/model
```

## Testing
An example of using the GPP network for inference can be seen in [this script](https://github.com/arangesh/Ground-Plane-Polling/blob/master/keras_retinanet_3D/bin/run_network.py).

This script can be used to generate results on the KITTI test set as follows:
```shell
python3 keras_retinanet_3D/bin/run_network.py /path/to/inference/model /path/to/test/images /path/to/test/calibration/files /path/to/road_planes_database.mat /path/to/output/directory --save-images --backbone=resnet50
```
In the above example, the `road_planes_database.mat` file is an (N x 4) array of ground planes for the KITTI dataset. Different versions of this file can be found [here](https://github.com/arangesh/Ground-Plane-Polling/tree/master/road_planes_database).

For inference on your own dataset, replace the calibration files and `road_planes_database.mat` with relevant files of your own.

You can download our inference models trained on KITTI using [this link](https://drive.google.com/file/d/10ZIrDwySKl0jcFDYnEVo_s--yh88WVB8/view?usp=sharing).

## Debugging
Creating your own dataset does not always work out of the box. There is a [`debug.py`](https://github.com/arangesh/Ground-Plane-Polling/blob/master/keras_retinanet_3D/bin/debug.py) tool to help find the most common mistakes.
