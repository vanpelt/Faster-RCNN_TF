# Faster-RCNN for Cloud ML

This is a fork of [@smallcorgi](smallcorgi/Faster-RCNN_TF)'s experimental Tensorflow implementation of Faster RCNN made to run on [Google Cloud Machine Learning](https://cloud.google.com/products/machine-learning).  For details about R-CNN please refer to the paper [Faster R-CNN: Towards Real-Time Object Detection with Region Proposal Networks](http://arxiv.org/pdf/1506.01497v3.pdf) by Shaoqing Ren, Kaiming He, Ross Girshick, Jian Sun.

## Installation

### Cloud

Create a Google Cloud Project and enable Cloud ML as described [here](https://cloud.google.com/ml/docs/how-tos/getting-set-up)

### Local

This code *should* support running without a GPU / CUDA, but if compilation blows up I would not be suprised.  Ideally you'll test your code locally on a machine with CUDA.  In which case, be sure to set the CUDAHOME env variable, it's usually `/usr/local/cuda`.  It's also probably a good idea to use `virtualenv`.  Once you've activated your virtualenv, it should be as easy as:

```bash
git clone https://github.com/vanpelt/Faster-RCNN_CloudML
export CUDAHOME=/usr/local/cuda
pip install tensorflow
cd lib
python setup.py build_ext --inplace
```

## Hardware Requirements

For training the end-to-end version of Faster R-CNN with VGG16, 3G of GPU memory is sufficient (using CUDNN)

## Demo

We modified this library to support data generated by the CrowdFlower [annotation tool](http://annotation.crowdflower.com?shapes=true).  We asked the crowd to draw boxes around monkey's.  There is a `crowdflower` module in the datasets directory which converts the data to imdb format.  To use you'll need to copy our [dataset](https://storage.googleapis.com/crowdflower-machine-vision/monkeys.tar) to your bucket as described below.

We'll use the `cloud-ml.sh` script to submit jobs.  Before we can submit jobs we have to setup our google cloud storage bucket.  All jobs are run within a namespace, we'll call ours `demo`.  The following commands upload data to the google cloud storage bucket created for our machine learning project described above.

```bash
export MY_PROJECT_NAME=funky_monkey
export NAME=demo
gsutils cp config/cfg.yml.sample gs://$MY_PROJECT_NAME-ml/$NAME/cfg.yml
gsutils cp config/class_names.json.sample gs://$MY_PROJECT_NAME-ml/$NAME/class_names.json
gsutils -m cp gs://crowdflower-machine-vision/monkey/* gs://$MY_PROJECT_NAME-ml/$NAME/
gsutils -m cp gs://crowdflower-machine-vision/VGG_imagenet.npy gs://$MY_PROJECT_NAME-ml/$NAME/
```

Generally it's a good idea to test things locally because CloudML takes 5-10 minutes to startup.  To test training locally run:

```bash
./cloud-ml -l demo
```

If that looks good, let's send it to the :cloud:.

```bash
./cloud-ml demo
```

If you're feeling ambitious, you can run other datasets defined in the datasets directory.  For instance, here's how to run `pascal_voc`

```bash
export NAME=demo_voc
gsutils cp config/voc_cfg.yml.sample gs://$MY_PROJECT_NAME-ml/$NAME/cfg.yml
gsutils -m cp gs://crowdflower-machine-vision/VOC2007 gs://$MY_PROJECT_NAME-ml/$NAME/
gsutils -m cp gs://crowdflower-machine-vision/VGG_imagenet.npy gs://$MY_PROJECT_NAME-ml/$NAME/
./cloud-ml -v demo_voc
```

## References

[Faster-RCNN_TF](smallcorgi/Faster-RCNN_TF)

[Faster R-CNN caffe version](https://github.com/rbgirshick/py-faster-rcnn)

[A tensorflow implementation of SubCNN (working progress)](https://github.com/yuxng/SubCNN_TF)