---
layout: post-sidebar
date: 2020-04-08
title: "Object detection with GluonCV - II"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 12
feature_image: feature-gluoncv-ii
show_related_posts: true
square_related: recommend-coding
---

<br>
In the [first article]({% post_url 2020-04-07-object-detection-with-gluoncv-i %}) of this series, we did a short introduction on GluonCV, MXNet and pre-trained models; today we are going to start with the code.

We are going to analyze a static image, and the code produced in this article will be the basis for the [third article]({% post_url 2020-04-09-object-detection-with-gluoncv-iii %}) of this series because at the end of the day, a video is a collection of images.


<br>
{:refdef: style="text-align: center;"}
![training_model]({{site.url}}/{{site.baseurl}}img/post-assets/gluoncv_ii_0.jpg)
{: refdef}
<br>



## Setting up the environment
<br>
Before you start coding, install Jupyter Notebook following [David's article]({% post_url 2020-03-30-Jupyter Notebook %})
After that,open the terminal, and install MXNet by running the following command:

> pip install mxnet

There are different optimized versions of MXNet for CPUs and GPUs but we will use the standard one.
The next step is to install GlounCV:

> pip install gluoncv

And we are ready to go!

## Starting with the code
<br>
Open your Jupyter notebook and start importing the libraries

{% highlight python %}
import mxnet as mx
import gluoncv as gcv
import matplotlib.pyplot as plt
{% endhighlight %}

In this step we will already have all the components that we are going to need, so let's load our image
{% highlight python %}
image_file_path = 'my_image.jpg'
image = mx.image.imread(image_file_path)
{% endhighlight %}


Our image is loaded, let's see if our program can detect the healthy foods in the image:
<br>
{:refdef: style="text-align: center;"}
![me_example_object_detection]({{site.url}}/{{site.baseurl}}img/post-assets/gluoncv_ii_1.png)
{: refdef}
<br>



At this point, since our image has been uploaded by MXNet library we can see the shape of our image:
{% highlight python %}
print('Shape:' , image.shape)
## This will print => shape: (720, 1080, 3)
{% endhighlight %}
The output of this instruction is the shape of our image, in this case a 1080px x 720px image with 3 channels (RGB).

That is great, but out image is not ready yet to be processed, each model needs the image to be loaded with a certain characteristics, such as a specific size etc...so to prepare our image, we need to transform it; Hopefully MXNet includes utilities to make life easier for us:

{% highlight python %}
image, trf_image = gcv.data.transforms.presets.yolo.transform_test(image)
{% endhighlight %}

In this case we are transforming the image by applying the yolo presets, but we could use other presets like ssd.

The next step is to load the model.

For this example I'm going to use the pre-trained `yolo3_darknet53_coco` which has relatively low memory consumption:

{% highlight python %}
nwk = gcv.model_zoo.get_model('yolo3_darknet53_coco', pretrained=True)
{% endhighlight %}

We are ready to run our prediction using our network.

{% highlight python %}
prediction = nwk(image)
{% endhighlight %}

The prediction result contains a set of arrays; The first one contains class indexes, the second one contains the probabilities and the last contains the box coordinates. Since the index does not give us any meaningful information, we have to extract the name from the network.classes.

The function `plot_bbox` within the gluoncv utilities will responsible for rendering the image, with the boxes and labels:

{% highlight python %}
class_index, probability, box = [item[0] for item in prediction]
gcv.utils.viz.plot_bbox(trf_image,
                       box,
                       probability,
                       class_index,
                       class_name = network.classes)
{% endhighlight %}



<br>
{:refdef: style="text-align: center;"}
![me_example_object_detection]({{site.url}}/{{site.baseurl}}img/post-assets/gluoncv_ii_2.png)
{: refdef}
<br>

You can find the code [here](https://gist.github.com/jsalcedo1987/34916a705026c5dd79ae881a6810f6b3)
See you in the [next article]({% post_url 2020-04-09-object-detection-with-gluoncv-iii %})! 
