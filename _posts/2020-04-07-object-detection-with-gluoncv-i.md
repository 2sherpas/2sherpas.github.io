---
layout: post-sidebar
date: 2020-04-07
title: "Object detection with GluonCV - I"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 10
feature_image: feature-gluoncv-i
show_related_posts: true
square_related: recommend-coding
---

<br>
After reading [David's post]({% post_url 2020-03-30-Jupyter Notebook %}) about Jupyter I had an idea about starting to build something with GluonCV using Python and Jupyter.

When I work at home, my children constantly jump, play and ask constantly "Can I eat this, daddy?". Most of the time are sweets and I suggest them to replace it for fruit.

So I though, let's do a program that detect if the food is a fruit and tell them that is healthy!

This is a series of articles, the first is just a high-level introduction about GluonCV, in the [second post]({% post_url 2020-04-08-object-detection-with-gluoncv-ii %}) we will run object detection using a static image and in the [last one]({% post_url 2020-04-09-object-detection-with-gluoncv-iii %}), we will evolve the code of the previous article to build object detection in real time and if a fruit is detected, the computer will speak and tell us that this food is healthy.  

Let's start.

## What is GluonCV
<br>
GluonCV is part of the Apache MXNet deep learning framework under the Apache Software Foundation (ASF). GluonCV is an open-source and easy to use computer vision toolkit. It provides models for image classification, object deduction, semantic segmentation, instance segmentation, and pose estimation.

What is a model? A model is just a function that will take pictures and provide a prediction, so depending on the goal that we want to achieve we will need different models to provide precision predictions.

Does this means that, for example, to classify the objects in an image we will use always the same model? The answer is no, because each model has a prediction precision and computational / memory consumption. So there are models that are very accurate but require a lot of time and computational resources to make a prediction; however there are others models that are lighter and faster, but their precision slightly worse. So every time you choose a pre-trained model, keep those two factors in mind: accuracy and performance. You can see a table below with the different models for image classification, with a relation of the precision and speed (the size of the bubble is the memory consumed by each model ):

<br>
{:refdef: style="text-align: center;"}
![graph_model_comparative]({{site.url}}/{{site.baseurl}}img/post-assets/gluoncv_1.jpg)
{: refdef}
<br>


## Detecting objects in our image

As we mention earlier, there are models for image classification and also for object detection. What is the difference between those? Image classification allows us to classify an image, for example, if we have an image with a ball, with an image classification model we could classify the image in a folder with ball images. However, what if the image contains a ball and a goalkeeper? That's when object detection becomes useful:


<br>
{:refdef: style="text-align: center;"}
![me_example_object_detection]({{site.url}}/{{site.baseurl}}img/post-assets/gluoncv_2.png)
{: refdef}
<br>


For experiment we don't want to classify images, our images could contain more than one object, we want to detect those objects in the image and we will draw a box around those objects enclosing the object.

## Models
<br>
Why use a pre-trained model instead of creating your own model? Because building and training your own model takes time and not always you get the expected results.

All the high-performance image classification models are neural networks so we will review this in the next series.

<br>
{:refdef: style="text-align: center;"}
![training_model]({{site.url}}/{{site.baseurl}}img/post-assets/gluoncv_3.jpeg)
{: refdef}
<br>

Just an example, the pre-trained models, Pascal VOC model has a data set of 22,000 images and 50,000 labeled objects and COCO (Common Objects in Context) has 135.000 images and 890.000 labeled objects. 


<br>
See you in the [next article]({% post_url 2020-04-08-object-detection-with-gluoncv-ii %})! 