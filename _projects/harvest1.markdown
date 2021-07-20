---
layout: page
title: Cropland Classification
description: Exploration of different methods of predicting croptype from satellite time series data.
img: /assets/img/timeseries.png
importance: 3
category: Nasa Harvest
---
<h5 id="skills-used-">Skills used:</h5>
<ol>
<li>Comparing deep learning and machine learning models using pytorch and sklearn</li>
<li>Multi label classification from time series data</li>
<li>Data resampling and class balancing</li>
</ol>

<p>I compared the performance of Random Forest, XGBoost, and a Multi Layer Perceptron on the dataset provided by <a href="https://zindi.africa/competitions/iclr-workshop-challenge-2-radiant-earth-computer-vision-for-crop-recognition">Zindi</a> for their cropland detection challenge.</p> The data consisted of time series data collected from <a href="https://sentinel.esa.int/web/sentinel/missions/sentinel-2">Sentinel-2</a> satellites.


<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        <img class="img-fluid rounded z-depth-1" src="{{ '/assets/img/intercrop.jpeg' | relative_url }}" alt="" title="example image"/>
    </div>
</div>
<div class="caption">
    An example of an intercropped field.
</div>


<p><a href="https://en.wikipedia.org/wiki/Intercropping">Intercropping</a> is common in Kenya, which made detection challenging. In my analysis, I saw that intercropped crops would be classified as one of the other and not as the separate category. For example, a field of intercropped Maize and Cassava would be incorrectly classified as only Maize very often since Maize was the larger crop in the field. I decided to try a new approach with <a href="https://en.wikipedia.org/wiki/Multi-label_classification">multi label</a> classification. </p>

For example: Intercropped Maize and Soybean became [1 1 0] = [Maize Cassava Bean] instead of it's own seperate label.

I also calculated <a href="https://eos.com/blog/6-spectral-indexes-on-top-of-ndvi-to-make-your-vegetation-analysis-complete/#:~:text=GCI,a%20measurement%20of%20plant%20health.">important bands</a> for remote sensing such as NDVI, GCVI, EVI, and LSWI and used grid search to find the best parameters.  

{% highlight python linenos %}
class Net_normal(nn.Module):
    def __init__(self,shape):
        super(Net_normal, self).__init__()
        self.fc1 = nn.Linear(shape[0],shape[1])
        self.fc1_bn = nn.BatchNorm1d(shape[1])
        self.fc2 = nn.Linear(shape[1],shape[2])
        self.fc2_bn = nn.BatchNorm1d(shape[2])
        self.fc3 = nn.Linear(shape[2],shape[3])
        self.fc3_bn = nn.BatchNorm1d(shape[3])
        self.fc4 = nn.Linear(shape[3],shape[4])
        self.dropout = nn.Dropout(p=.4)

    def forward(self, x):
        x = self.fc1(x)
        x = F.relu(x)
        x = self.fc1_bn(x)
        x = self.dropout(x)
        x = self.fc2(x)
        x = F.relu(x)
        x = self.fc2_bn(x)
        x = self.dropout(x)
        x = self.fc3(x)
        x = F.relu(x)
        #x = self.fc3_bn(F.tanh(x))
        #x = self.dropout(x)
        x = self.fc4(x)

        return x
{% endhighlight %}
<div class="caption">
    A fully connected feed forward neural network was used as the deep learning model.
</div>


I found that the multi-label classification with binary logits loss had higher accuracy when evaluating on the validation set as compared to a normal multi class classification. This might suggest that classifying intercropped fields as a multiple fields could be more succesful than classifying them as separate classes.
