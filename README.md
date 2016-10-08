# SSD: Single Shot MultiBox Object Detector

SSD is an unified framework for object detection with a single network.

You can use the code to train/evaluate/test for object detection task.

There is a pure cpp test end with zero dependence: <a href="https://github.com/zhreshold/mxnet-ssd.cpp/" target="_blank">mxnet-ssd.cpp</a>

### Disclaimer
This is a re-implementation of original SSD which is based on caffe. The official
repository is available [here](https://github.com/weiliu89/caffe/tree/ssd).
The arXiv paper is available [here](http://arxiv.org/abs/1512.02325).

This example is intended for reproducing the nice detector while fully utilize the
remarkable traits of MXNet.
* The model is fully compatible with caffe version.
* Model converter from caffe is available, I'll release it once I can convert any symbol other than VGG16.
* The result is almost identical to the original version. However, due to different non-maximum suppression Implementation, the results might differ slightly.

### Demo results
![demo1](https://cloud.githubusercontent.com/assets/3307514/19171057/8e1a0cc4-8be0-11e6-9d8f-088c25353b40.png)
![demo2](https://cloud.githubusercontent.com/assets/3307514/19171063/91ec2792-8be0-11e6-983c-773bd6868fa8.png)
![demo3](https://cloud.githubusercontent.com/assets/3307514/19171086/a9346842-8be0-11e6-8011-c17716b22ad3.png)


### Getting started
* You will need python modules: `easydict`, `cv2`, `matplotlib` and `numpy`.
You can install them via pip or package manegers, such as `apt-get`:
```
sudo apt-get install python-opencv python-matplotlib python-numpy
sudo pip install easydict
```
* Clone this repo:
```
# if you don't have git, install it via apt or homebrew/yum based on your system
sudo apt-get install git
# cd where you would like to clone this repo
cd ~
git clone --recursive https://github.com/zhreshold/mxnet-ssd.git
# make sure you clone this with --recursive
# if not done correctly or you are using downloaded repo, pull them all via:
# git submodule update --recursive --init
cd mxnet-ssd/mxnet
```
* Build MXNet: Follow the official instructions <a href="http://mxnet.readthedocs.io/en/latest/how_to/build.html/" target="_blank">here</a>.
Remember to enable CUDA if you want to be able to train, since CPU training is
insanely slow. Using CUDNN is optional, it's not fully tested but should be fine.

### Try the demo
* Download the pretrained model: [`ssd_300_voc_0712.zip`](https://dl.dropboxusercontent.com/u/39265872/ssd_300_voc0712.zip), and extract to `model/` directory. (This model is converted from VGG_VOC0712_SSD_300x300_iter_60000.caffemodel provided by paper author).
* Run
```
# cd /path/to/mxnet-ssd
python demo.py
# play with examples:
python demo.py --epoch 0 --images ./data/demo/dog.jpg --thresh 0.5
```
* Check `python demo.py --help` for more options.

### Train the model
This example only covers training on Pascal VOC dataset. Other datasets should
be easily supported by adding subclass derived from class `Imdb` in `dataset/imdb.py`.
See example of `dataset/pascal_voc.py` for details.
* Download the converted pretrained `vgg16_reduced` model [here](https://dl.dropboxusercontent.com/u/39265872/vgg16_reduced.zip), unzip `.param` and `.json` files
into `model/` directory by default.
* Download the PASCAL VOC dataset, skip this step if you already have one.
```
cd /path/to/where_you_store_datasets/
wget http://host.robots.ox.ac.uk/pascal/VOC/voc2012/VOCtrainval_11-May-2012.tar
wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtrainval_06-Nov-2007.tar
wget http://host.robots.ox.ac.uk/pascal/VOC/voc2007/VOCtest_06-Nov-2007.tar
# Extract the data.
tar -xvf VOCtrainval_11-May-2012.tar
tar -xvf VOCtrainval_06-Nov-2007.tar
tar -xvf VOCtest_06-Nov-2007.tar
```
* We are goint to use `trainval` set in VOC2007/2012 as a common strategy.
The suggested directory structure is to store `VOC2007` and `VOC2012` directories
in the same `VOCdevkit` folder.
* Then link `VOCdevkit` folder to `data/VOCdevkit` by default:
```
ln -s /path/to/VOCdevkit /path/to/this_example/data/VOCdevkit
```
Use hard link instead of copy could save us a bit disk space.
* Start training:
```
python train.py
```
* By default, this example will use `batch-size=32` and `learning_rate=0.002`.
You might need to change the parameters a bit if you have different configurations.
Check `python train.py --help` for more training options. For example, if you have 4 GPUs, use:
```
# note that a perfect training parameter set is yet to be discovered for multi-gpu
python train.py --gpus 0,1,2,3 --batch-size 128 --lr 0.0005
```
* Memory usage: MXNet is very memory efficient, training on `VGG16_reduced` model with `batch-size` 32 takes around 4684MB without CUDNN.

### Evalute trained model
Again, currently we only support evaluation on PASCAL VOC
Use:
```
# cd /path/to/mxnet-ssd
python evaluate.py --gpus 0,1 --batch-size 128 --epoch 0
```
