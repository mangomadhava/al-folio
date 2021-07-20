---
layout: page
title: Street2Sat
description: Machine learning pipeline for creating ground truth geo-referenced labeled datasets from street level images.
img: /assets/img/predictions-s2s.png
importance: 1
category: Nasa Harvest
---

PAPER ACCEPTED TO ICML CLIMATE CHANGE <a href="https://www.climatechange.ai/papers/icml2021/74">WORKSHOP</a>

Code available <a href="https://github.com/nasaharvest/street2sat_website/tree/ICML_paper_code"> here. </a>

(Still in development, our team is working on deploying to google cloud and on a web app)

<h5 id="skills-used-">Skills used:</h5>

<li>Deploying deep learning models on a web app using Heroku and Flask</li>
<li>Training Yolo object detection on hand labeled images</li>
<li>MongoDB for data storage</li>

<p></p>
<p>
Ground-truth labels on crop type and other variables are critically needed to develop machine learning methods that use satellite observations to combat climate change and food insecurity. These labels difficult and costly to obtain over large areas, particularly in Sub-Saharan Africa where they are most scarce. I wrote code for Street2Sat, a new framework for obtaining large data sets of geo-referenced crop type labels obtained from vehicle mounted cameras that can be extended to other applications.
</p>

The Street2Sat pipeline has 5 steps:
<li>Data Collection drives</li>
<li>Image Preprocessing</li>
<li>Crop Type Prediction</li>
<li>Distance Estimation</li>
<li>GPS Coordinate Correction</li>

<p></p>
<h5 id="Data Collection">Data Collection</h5>
Data was collected in Western Kenya using car mounted GoPro Cameras. This study will be extended to 5 more countries in the near future.

<div class="row justify-content-sm-center">
    <div class="col-sm-8 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/GP__0633.JPG' | relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col-sm-4 mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/gopro.jpeg' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
    Left: Example Image from Data Collection. Right: GoPro 360 camera that was used.
</div>

<h5 id="Image Processing">Image Processing</h5>

An iterated approach on <a href="https://en.wikipedia.org/wiki/Otsu%27s_method">Otsu's Method</a> for thresholding was used to straighten images before they were used to train the Yolo object detection. Otsu's method was run on the whole image, halves of the image, thirds of the image, and fourths of the image, and all the predicted rotation angles were averaged. In addition, the image was blurred using a gaussian filter before running on Otsu's method in order to reduce incorrect rotations resulting from small objects.
<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/line.png' | relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/falsecolor.png' | relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col-sm mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/straight.png' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
Left is original image with predicted rotation angle. Middle is blurred image used in Otsu's method. Right is the rotated image.
</div>


<h5 id="pred">Crop Type Prediction</h5>

The <a href="https://github.com/ultralytics/yolov5">Yolo Object Detection</a> framework is used to train and predict where crops were located in the image. The images were labeled for crop type by Nasa Harvest teams with agricultural expert guidance. The trained Yolo model is hosted on Heroku using Flask and MongoDB. The crop type prediction can be run on the web app.

<div class="row justify-content-sm-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/GP__0633.JPG' | relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/predictions-s2s.png' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
    Left: Original Image. Right: Predicted crop type.
</div>


<h5 id="pred">Distance Estimation</h5>

The heights of the predicted bounding boxes are used to predict the distance from the camera to the crop. We calculated the distance to each bounding box and then calculated the average depth for all of the boxes of the same crop type class to get a single depth for the field. We used the following equation to predict distance:

$$
\begin{equation}
    d = \frac{(l_{focal} * h_{crop} * h_{image})}{(h_{bbox} * h_{sensor})}
\end{equation}
$$


The focal length, $$l_{focal}$$, was obtained from the EXIF image metadata; the crop height, $$h_{crop}$$, is from known heights of crops; $$h_{bbox}$$ is the height of the bounding box in pixels; $$h_{image}$$ is the height of the image in pixels; and the GoPro sensor height $$h_{sensor}$$ is 4.55 mm.

<h5 id="gps">GPS Coordinate Estimation</h5>

After obtaining the average distance to the crop in the image, the bearing of the camera had to be calculated. To find the bearing, a velocity vector is calculated between the current image and the closest other image in time that is uploaded. We assume that the camera is pointed 90 degrees orthogonal to the drive direction and relocate the point the average distance in that direction.


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/pointss2s.png' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
    Predicted points in red. Original points in grey.
</div>
