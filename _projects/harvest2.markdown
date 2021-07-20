---
layout: page
title: Cropland Mask
description: Different methods for cropland segmentation from satellite images.
img: /assets/img/timelapse.gif
importance: 2
category: Nasa Harvest
---

<h5 id="skills-used-">Skills used:</h5>
<ol>
<li>Implementing deep learning models using pytorch</li>
<li>Preprocessing Senitnel2 and Planet images</li>
<li>Rasterizing geo-referenced datasets using geopandas and other libraries</li>
</ol>

<a href="https://sentinel.esa.int/web/sentinel/missions/sentinel-2">Sentinel-2</a> satellites have a resolution of 10m and collects data in 13 bands. Using the available Sentinel2 data for parts of Kenya and Uganda and the field boundary data, I created a dataset that could used to train a classifier to predict cropland and non-cropland.
<div class="row justify-content-sm-center">
    <div class="row">
        <div class="col-sm mt-3 mt-md-0">
            <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/timelapse.gif' | relative_url }}" alt="" title="example image"/>
        </div>
    </div>
</div>
<div class="caption">
    Timelapse of Sentinel2 images from a region in Keyna.
</div>


Cloudy pixels in images had to be filtered out using outlier detection and cloud masks to create a composite image which was the 'average' image over the entire year.

<div class="row justify-content-sm-center">
    <div class="row">
        <div class="col-sm mt-3 mt-md-0">
            <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/composite.png' | relative_url }}" alt="" title="example image"/>
        </div>
    </div>
</div>
<div class="caption">
    Composite Image of a Sentinel2 tile.
</div>

Using datasets of the roads and buildings in Kenya, and the <a href="https://registry.mlhub.earth/10.34911/rdnt.u41j87/">Plantvillage</a> dataset, I created a crop/non-crop dataset that could be used for deep learning. However, since the data is sparse as shown in the image below.

<div class="row justify-content-sm-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/zoomed_in_sat.png' | relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/zoomed_in_raster.png' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
    Left: Sentinel2 composite. Right: Corresponding labels (Non crop in blue, crop in yellow, no data points in purple).
</div>


<h5 id="nonspatial">Non Spatial methods</h5>
First, I tried using commonly used remote sensing techniques which do not use spatial information. The methods that I used were XGboost, random forest, and a multi layer perceptron. Although some of the segmentations had high accuracy, it was not the best crop/non crop mask since the predicted pixels were often random and not clustered well in fields.

<div class="row justify-content-sm-center">
    <div class="row">
        <div class="col-sm mt-3 mt-md-0">
            <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/non-spatial.png' | relative_url }}" alt="" title="example image"/>
        </div>
    </div>
</div>
<div class="caption">
    Comparison of different output maps from different methods.
</div>


<h5 id="patchconv">Patch Convolution</h5>
To incorporate spatial information into the predictions, I designed a method that took a 5x5 square around a pixel and applied a simple convolutional neural network to predict if the pixel was crop or non crop. This improved the predictions since more context around the pixel was used instead of just the single pixel.

<div class="row justify-content-sm-center">
    <div class="row">
        <div class="col-sm mt-3 mt-md-0">
            <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/patchconv-data.png' | relative_url }}" alt="" title="example image"/>
        </div>
    </div>
</div>
<div class="caption">
    Example input for patch convolution.
</div>

{% highlight python linenos %}
class patchconv(nn.Module):
    def __init__(self):
        super().__init__()
        self.conv1 = nn.Conv2d(kernel_size = 3, in_channels = 8, out_channels = 64, padding = 1)
        self.conv2 = nn.Conv2d(kernel_size = 3, in_channels = 64, out_channels = 64, padding = 1)
        self.mxpl = nn.MaxPool2d(kernel_size = 2)
        self.fc1 = nn.Linear(256,1)

    def forward(self, x):
        x = self.conv1(x)
        x = F.relu(x)
        x = self.conv2(x)
        x = F.relu(x)
        x = self.mxpl(x)
        x = x.reshape(x.size(0),-1)
        x = self.fc1(x)
        #x = torch.sigmoid(x)
        return x.squeeze()
{% endhighlight %}
<div class="caption">
    Patch convolutional model in pytorch.
</div>



<h5 id="Cotraining Model">Cotraining Model</h5>
To incorporate more spatial information when creating predictions, I decided to use a higher resolution satellite image as well. <a href="https://www.planet.com/products/planet-imagery/">Planet Labs</a> images have 3 meter resolution which is much higher than the 10 meter resolution of the Sentinel2 satellite.

<div class="row justify-content-sm-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/sentinel.png' | relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/planet.png' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
    Left: Sentinel2 imagery (10 m). Right: Planet Labs imagery (3 m).
</div>

I designed a <a href="https://en.wikipedia.org/wiki/Co-training#:~:text=Co%2Dtraining%20is%20a%20machine,and%20Tom%20Mitchell%20in%201998.">co-training</a> model that used high spatial resolution Planet Labs imagery and high temporal resolution Sentinel2 imagery to perform the segmentation. Areas that did have labels were trained using a supervised framework, while areas without labels were trained using an unsupervised framework with both methods sharing weights. I also tried predicting field boundaries simultaneously, which was much more challenging.

<div class="row justify-content-sm-center">
    <div class="row">
        <div class="col-sm mt-3 mt-md-0">
            <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/cotraining-diagram.png' | relative_url }}" alt="" title="example image"/>
        </div>
    </div>
</div>
<div class="caption">
    Co-training model diagram.
</div>


<div class="row justify-content-sm-center">
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/sat.png' | relative_url }}" alt="" title="example image"/>
    </div>
    <div class="col">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/output.png' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
    Comparision of a input and output for co-training model. Yellow is predicted cropland and blue is predicted non crop area.
</div>

<h5 id="dlh">Deep Lab Head Model</h5>
I also tried another simpler model based off the <a href="https://arxiv.org/abs/1706.05587">DeepLabv3</a> model using only Planet Labs imagery. Specifially, I used the DeepLabv3 Head which consists of multiple convolution operations each with a different dilation. This was useful since aggregating information at different spatial scales helped improve the accuracy of the segmentation.  

<div class="row justify-content-sm-center">
    <div class="row">
        <div class="col-sm mt-3 mt-md-0">
            <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/tiff.png' | relative_url }}" alt="" title="example image"/>
        </div>
    </div>
</div>
<div class="caption">
    Predicted crop mask for a full Planet Labs tile. The larger 8000 x 8000 Planet Labs tile was broken into smaller patches of 260 x 260 pixels as input to the model, but were overlapped at every 100 pixels in order to reduce artifacts in the output segmentation. Additional work is needed to refine predictions near the coasts.
</div>
