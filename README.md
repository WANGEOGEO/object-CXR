<!--img src="logo.png" width="100%"/-->

- [Background](#background)
- [Data](#data)
- [Annotation](#annotation)
- [Download](#download)
- [Evaluation](#evaluation)
- [Baseline](#baseline)
- [Submission](#submission)
- [Timeline](#timeline)
- [Leaderboard](#leaderboard)
- [Prize](#prize)
- [Organizers](#organizers)
- [FAQ](#faq)



## Background
Analyzing chest X-rays is a common clinical approach for diagnosing pulmonary and heart diseases. However, foreign objects are occasionally presented on chest X-ray images, especially in rural and remote locations where standard filming guidances are not strictly followed. Foreign objects on chest X-rays may obscure pathological finds, thus increasing false negative diagnosis. They may also confuse junior radiologists from real pathological findings, e.g. buttons are visually similar to nodules on chest X-ray, thus increasing false positive diagnosis. Based on statistics from our (JF Healthcare) telemedicine platform, nearly one third of the uploaded chest X-rays from township hospitals do not qualify for diagnosis, and interference from foreign objects is the largest contributing factor. Before we develop our own detection system, our radiologists have to manually check the quality of the chest X-ray and promote re-filming if foreign objects within the lung field are presented. An algorithm based foreign object detection system could automatically promote re-filming, thus significantly reduce the cost and save radiologists' time for more diagnosis, which has been noticed before (L Hogeweg, 2013, Medical Physics).

We provide a large dataset of chest X-rays with strong annotations of foreign objects, and the competition for automatic detection of foreign objects. Specifically, 5000 frontal chest X-ray images with foreign objects presented and 5000 frontal chest X-ray images without foreign objects are provided. All the chest X-ray images were filmed in township hospitals in China and collected through our telemedicine platform. Foreign objects within the lung field of each chest X-ray are annotated with bounding boxes, ellipses or masks depending on the shape of the objects.

Detecting foreign objects is particularly challenging for deep learning (DL) based systems, as specific types of objects presented in the test set may be rarely or never seen in the training set, thus posing a few-shot/zero-shot learning problem. We hope this open dataset and challenge could both help the development of automatic foreign objects detection system, and promote the general research of object detection on chest X-rays, as large scale chest X-ray datasets with strong annotations are limited to the best of our knowledge. 

object-CXR challenge is going to be hosted at [MIDL 2020](https://2020.midl.io/).

## Data
5000 frontal chest X-ray images with foreign objects presented and 5000 frontal chest X-ray images without foreign objects were filmed and collected from about 300 township hosiptials in China. 12 medically-trained radiologists with 1 to 3 years of experience annotated all the images. Each annotator manually annotates the potential foreign objects on a given chest X-ray presented within the lung field. Foreign objects were annotated with bounding boxes, bounding ellipses or masks depending on the shape of the objects. Support devices were excluded from annotation. A typical frontal chest X-ray with foreign objects annotated looks like this:
![annotation](annotation.png)
We randomly split the 10000 images into training, validation and test dataset:

**training** 4000 chest X-rays with foreign objects presented; 4000 chest X-rays without foreign objects. 

**validation** 500 chest X-rays with foreign objects presented; 500 chest X-rays without foreign objects. 

**test** 500 chest X-rays with foreign objects presented; 500 chest X-rays without foreign objects. 


## Annotation

We provide object-level annotations for each image, which indicate the rough location of each foreign object using a closed shape.

Annotations are provided in csv files and a csv example is shown below.

```csv
image_path,annotation
/path/#####.jpg,ANNO_TYPE_IDX x1 y1 x2 y2;ANNO_TYPE_IDX x1 y1 x2 y2 ... xn yn;...
/path/#####.jpg,
/path/#####.jpg,ANNO_TYPE_IDX x1 y1 x2 y2
...
```

Three type of shapes are used namely rectangle, ellipse and polygon. We use `0`, `1` and `2` as `ANNO_TYPE_IDX` respectively.

- For rectangle and ellipse annotations, we provide the bounding box (upper left and lower right) coordinates in the format `x1 y1 x2 y2` where `x1` < `x2` and `y1` < `y2`.

- For polygon annotations, we provide a sequence of coordinates in the format `x1 y1 x2 y2 ... xn yn`.

> ### Note:
> Our annotations use a Cartesian pixel coordinate system, with the origin (0,0) in the upper left corner. The x coordinate extends from left to right; the y coordinate extends downward.

## Download
The training and validation dataset can be accessed here at [Google Drive](https://drive.google.com/drive/folders/1SubfNALJn6aO56lUYeJsVpFLZuXurlBC) and [Baidu Pan](https://pan.baidu.com/s/1eVL5FLphy0dZXk7xxI6GEw)(access code : fpwq). The imaging data has been
anonymousized and free to download for scientific research and non-commercial usage. 

The usage of this data is under CC-BY-NC-4.0 license.


## Evaluation
We use two metrics to evaluate the classification and localization performance of foreign objects detection on chest X-rays: Area Under Curve (AUC) and  Free-response Receiver Operating Characteristic (FROC).

### Classification
For the test dataset, each algorithm is required to generate a `prediction_classification.csv` file in the format below:
```
image_path,prediction
/path/#####.jpg,0.90
/path/#####.jpg,0.85
/path/#####.jpg,0.15
...
```
where each line corresponds to the prediciton result of one image. The first column is the image path, the second column is the predicted probability, ranging from 0 to 1, indicating whether this image has foreign objects or not.

We use AUC to evaluate the algorithm performance of classifying whether each given chest X-ray has foreign objects presented or not. AUC is commonly used to evaluate binary classification in medical imaging challenges. For example, the [CheXpert](https://stanfordmlgroup.github.io/competitions/chexpert/) competition for chest X-ray classification, and the classification task within [CAMELYON16](https://camelyon16.grand-challenge.org/Evaluation/) challenge for whole slide imaging classification. We believe AUC is adequate enough to measure the performance of the classification task of our challenge, especially given our positive and negative data is balanced.

### Localization
For the test dataset, each algorithm is required to generate a `prediction_localization.csv` file in the format below:
```
image_path,prediction
/path/#####.jpg,0.90 1000 500;0.80 200 400
/path/#####.jpg,
/path/#####.jpg,0.75 300 600;0.50 400 200;0.15 1000 200
...
```
where each line corresponds to the prediciton result of one image. The first column is the image path, the second column
is space seperated 3-element tuple of predicted foreign object coordinates with its probability in the format of (probability x y), where x and y are the width and height coordinates of the predicted foreign object. It is allowed to have zero predicted 3-element tuple for certain images, if there are no foreign objects presented. But please note the `,` after the first column even if the prediction is empty.

We use FROC to evaluate the algorithm performance of localizing foreign obects on each given chest X-ray. Because our object annotations are provided in different format, i.e. boxes, ellipses or masks depending on radiologists' annotation habits, it's not suitable to use other common metric, such as mean average precision (mAP) in natural object detection with pure bounding box annotations. FROC is more suitable in this case, since only localization coordinates are required for evaluation. FROC has been used as the metric for measuring medical imaging localization, for example, the lesion localization task within [CAMELYON16](https://camelyon16.grand-challenge.org/Evaluation/) challenge, and tuberculosis localization in (EJ Hwang, 2019, Clinical Infectious Disease).

FROC is computed as follow. A foreign object is counted as detected as long as one predicted cooridinate lies within its annotation. The sensitivity is the number of detected foreign objects dividide by the number of total foreign objects. A predicted coordinate is false positive, if it lies outside any foreign object annotation. When the numbers of false positive coordinates per image are 0.125, 0.25, 0.5, 1, 2, 4, 8, FROC is the average sensitivty of these different versions of predictions. 



[froc.py](https://github.com/jfhealthcare/object-CXR/tree/master/froc.py) provides the details of how FROC is computed.


## Baseline

We provide a baseline result in this [Jupyter Notebook](https://github.com/jfhealthcare/object-CXR/tree/master/baseline/baseline.ipynb).

## Submission
We host the online submission system at [codalab](https://worksheets.codalab.org/worksheets/0xcd2fb3db8ae74d03b53ad4c5bf81ebe2)

## Timeline
The competition lasts from ~~Feb/15/2020-Jun/30/2020 23:59 PDT~~ ended. The [MIDL 2020](https://2020.midl.io/challenges.html) objec-CXR workshop will be held during July/9 9:00-13:00 UTC-4. The zoom link for the virtual workshop will be provided shortly.

## Leaderboard
Team will be ranked primarily by AUC and then by FROC if there is a tie.

| Rank    | Date |  Model  | AUC| FROC|
| ------- | -----| --------| ---| ----|
| 1       |Jun 30, 2020 | XSD-v3 (ensemble) *TU Berlin*| 0.963 |0.851|
| 2       |Jun 28, 2020 | FrNet-v3 (ensemble) *TES_Vision* | 0.961 |0.785|
| 3       |Jun 30, 2020 | SSCNet (ensemble) *ShenzhenUniversity* | 0.960 |0.810|
| 4       |Jun 29, 2020 | XSD-v2 (single model) *TU Berlin*| 0.959 |0.825|
| 5       |Jun 30, 2020 | faster-rcnn-v3 (ensemble) *XVision-Romania*| 0.957 |0.822|
| 6       |Jun 29, 2020 | faster-rcnn-v2 (ensemble) *XVision-Romania*| 0.955 |0.822|
| 7       |Jun 7, 2020 | faster-rcnn (single model) *XVision*| 0.953 |0.839|
| 8       |Jun 29, 2020 | frankNet-v10.b (ensemble) *individual*| 0.953 |0.837|
| 9       |Jun 26, 2020 | frankNet-v9 (ensemble) *individual*| 0.953 |0.735|
| 10     |Jun 25, 2020 | SCNet (ensemble) *ShenzhenUniversity*| 0.952 |0.808|
| 11    |Jun 25, 2020 | FasterRCNN (single model) *individual*| 0.951 |0.852|
| 12    |Jun 30, 2020 | FasterRCNN (single model) *individual*| 0.951 |0.845|
| 13    |Jun 26, 2020 | XSD (single model) *TU Berlin*| 0.951 |0.788|
| 14    |Jun 23, 2020 | FasterRCNN (single model) *individual*| 0.951 |0.092|
| 15    |Jun 19, 2020 | frankNet-v6 (ensemble) *individual*| 0.948 |0.735|
| 16    |Jun 18, 2020 | frankNet-v5 (ensemble) *individual*| 0.946 |0.735|
| 17    |Jun 26, 2020 | MNet (single model) *individual*| 0.945 |0.812|
| 18    |Jun 23, 2020 | TimNetV2 (single model) *individual* | 0.938 |0.052|
| 19    |Jun 24, 2020 | TimNetV3 (single model) *individual* | 0.937 |0.811|
| 20    |Jun 30, 2020 | R_34_1x_DA (single model) *Istanbul Technical University*| 0.937 |0.007|
| 21    |May 27, 2020 | [frankNet-v2](https://github.com/frank-qcd-qk/object-cxr-2020) (single model)  *individual*| 0.935 |0.735|
| 22    |Jun 30, 2020 | efficient_rcnn-v4 (ensemble) *NWPU* | 0.934 |0.823|
| 23    |Jun 18, 2020 | FrNet-v2 (single model) *TES_Vision* | 0.934 |0.819|
| 24    |Jun 30, 2020 | FRCNN_R152_FPN_2x_DA (single model) *Istanbul Technical University*| 0.933 |0.780|
| 25    |Jun 5, 2020 | GxpNet-single (single model)  *Neusoft*| 0.932 |0.816|
| 26    |Jun 28, 2020 | efficient_rcnn-v3 (single model) *NWPU* | 0.930 |0.825|
| 27    |Jun 18, 2020 | TimNetV1 (single model) *individual* | 0.930 |0.810|
| 28    |Jun 25, 2020 | efficient_rcnn-v2 (single model) *NWPU* | 0.929 |0.823|
| 29    |Jun 23, 2020 | efficient_rcnn (single model) *NWPU* | 0.924 |0.812|
| 30    |Jun 10, 2020 | faster_rcnn (single model) *TES_Vision* | 0.924 |0.802|
| 31    |Mar 18, 2020 | baseline_faster-rcnn (single model) *individual participant* | 0.923 |0.800|
| 32    |Feb 21, 2020 | [JF Healthcare baseline](https://github.com/jfhealthcare/object-CXR#baseline) (single model)  *JF Healthcare*| 0.921 |0.803|
| 33    |Jun 30, 2020 | fasterRcnn (single model)  *North South University*| 0.921 |0.751|
| 34    |May 27, 2020 | [frankNet](https://github.com/frank-qcd-qk/object-cxr-2020) (single model)  *individual*| 0.918 |0.696|
| 35    |Jun 15, 2020 | EfficientNet (single model)  *SITP*| 0.916 |0.726|
| 36    |Jun 13, 2020 | FrNet (single model) *TES_Vision* | 0.908 |0.759|
| 37    |Jun 30, 2020 | FasterRCNN (single model) *ITU Vision Lab* | 0.908 |0.773|
| 38    |Jun 18, 2020 | FasterRCNN (single model)  *SITP*| 0.904 |0.729|
| 39    |Jun 30, 2020 | YOLO V3 (single model)  *individual*| 0.903 |0.674|
| 40    |Jun 23, 2020 | YOLO V3 (single model)  *USST*| 0.897 |0.759|


## Prize
The team wins the first place will be awarded an **NVIDIA TITAN RTX™**

## Organizers
[JF Healthcare](http://www.jfhealthcare.com/) is the primary organizer of this challenge.

<img src="jf.png" width="50%" align="middle"/>

[NVIDIA Inception Program](https://www.nvidia.com/en-us/deep-learning-ai/startups/) is the co-organizer of this challenge.

<img src="nvidia.png" width="50%" align="middle"/>

Startup companies that sign up will have an opportunity to join the NVIDIA Inception Program; providing access to financing services; NVIDIA Deep Learning Institute (DLI) training courses; GPU cloud resources and technical guidance; and a host of other services. Developers can access the link below for free training resources

https://www.nvidia.cn/developer/online-training

University challengers can be invited to participate in the University Ambassador Program of NVIDIA Deep Learning Institute (DLI). Learn more about NVIDIA Deep Learning Institute ( DLI ) and its University Ambassador Program, please visit

https://www.nvidia.cn/deep-learning-ai/education/?activetab=certification-tabs-3


## FAQ
For any questions, please contact yil8@uci.edu
