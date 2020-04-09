---
layout: post-sidebar
date: 2020-04-09
title: "Image recognition with GluonCV - III"
categories: coding
author_name : Jaime Salcedo
author_url : /author/jaime
author_avatar: jaime
show_avatar : true
read_time : 15
feature_image: feature-gluoncv-iii
show_related_posts: true
square_related: recommend-coding
youtubeId: FE_TKchtK8g
---

<br>
In the [previous article]({% post_url 2020-04-08-image-recognition-with-gluoncv-ii %}) of this series, I did a small experiment with GluonCV using yolo network to detect objects into a photo.

That was fun but not very useful since it just labels our objects, I will introduce a little more logic into our program to use the webcam to capture images and then the program will analyze the objects in the image so the computer can tell to my children whether that food is healthy or not.


<br>
{:refdef: style="text-align: center;"}
![training_model]({{site.url}}/{{site.baseurl}}img/post-assets/gluoncv_iii_0.jpg)
{: refdef}
<br>



## Capturing video

In the first article of this series, I made a short introduction into neuronal networks and models, and I talked about three factors: the accuracy of the model, the images per second that the model was able to process and the memory consumed during that process.

Based on the above factors, yolo is an intermediate option. The performance is not in our critical path right now and switching between models is just a couple of lines of code. I will continue with the yolo model to be able to re-use code of the article II as a base:

Just like in the previous post, we are going to use yolo net and the `yolo3_darknet53_coco` model:

{% highlight python %}
net = gcv.model_zoo.get_model('yolo3_darknet53_coco', pretrained=True)
{% endhighlight %}

It is not the fastest, and probably if you choose any of the SSD models it can run more frames per second, but for our Poc that model will be enough.

My two-years-old daughter still can't read, so the computer will tell her if the food is healthy or not.
I'm going to use `pyttsx3` for that:

{% highlight python %}
engine = pyttsx3.init();
{% endhighlight %}

The next step is to capture images with the webcam, I will use `opencv` library for that task:

{% highlight python %}
cap = cv2.VideoCapture(0)
ret, frame = cap.read()
{% endhighlight %}

Everything ready!

# Identifying objects

Once I identify the probabilities of the object, I need to identify its label; In this case, I need to identify if my daughter shows a banana to the cam. The object `net.classes` contains all the labels that the network can detect, so with the index I can get the label and check if is banana or not.

If the object is a banana then the `engine.say` transforms the text into speech and plays it through the speakers:

{% highlight python %}
for i in idx:
    image_index = class_indices[0,i].astype('int').asscalar()
    if net.classes[image_index] == 'banana':
        engine.say('banana, that is healthy')

{% endhighlight %}

The last step is to display the image with the bounding boxes around the objects:

{% highlight python %}
scale = 1.0 * frame.shape[0] / scaled_frame.shape[0]
img = gcv.utils.viz.cv_plot_bbox(frame.asnumpy(), bounding_boxes,probabilities, class_indices, class_names=net.classes, scale=scale)
gcv.utils.viz.cv_plot_image(img)
{% endhighlight %}

I will repeat the same operation in an infinite loop until I finish the program.

# The final code

It is time to put everything together and run our program:

{% highlight python %}
import argparse
import time

import gluoncv as gcv
from gluoncv.utils import try_import_cv2
cv2 = try_import_cv2()
import mxnet as mx
import pyttsx3;

net = gcv.model_zoo.get_model('yolo3_darknet53_coco', pretrained=True)
engine = pyttsx3.init();

cap = cv2.VideoCapture(0)
time.sleep(1) 

engine.startLoop(False)

while True:

    ret, frame = cap.read()
    frame = mx.nd.array(cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)).astype('uint8')
    rgb_nd, scaled_frame = gcv.data.transforms.presets.yolo.transform_test(frame, short=512)

    # Run the prediction
    prediction = net(rgb_nd)
    class_indices, probabilities, bounding_boxes = [item[0] for item in prediction]

    idx = probabilities.topk(k=1)[0].astype('int').asnumpy().tolist()
    
    for i in idx:
        image_index = class_indices[0,i].astype('int').asscalar()
        print('With prob = %.5f, it contains %s' % (
        probabilities[0,i].asscalar(), net.classes[image_index]))
        if net.classes[image_index] == 'banana':
            engine.say('banana, that is healthy')
        if net.classes[image_index] == 'apple':
            engine.say('Apple? If course you can eat that')
        
    scale = 1.0 * frame.shape[0] / scaled_frame.shape[0]
    img = gcv.utils.viz.cv_plot_bbox(frame.asnumpy(), bounding_boxes,probabilities, class_indices, class_names=net.classes, scale=scale)
    
    gcv.utils.viz.cv_plot_image(img)
    
    engine.iterate()
    cv2.waitKey(1)
{% endhighlight %}


I hope you enjoyed this experiment. If you run this code locally, you will find that sometimes the program does not speak when it detects the banana, that is because it is taking the person in the topk; You can also ignore the person object or scan all the objects in the image looking for the banana and not only the top ones; That is up to you, but I didn't want to make the code too complex and difficult to follow.

You can also try playing with different models like `yolo3_mobilenet1.0_voc` a little less precise but a little faster.

By the way, my testing engineer approves the beta version:

<br>
{:refdef: style="text-align: center;"}
![training_model]({{site.url}}/{{site.baseurl}}img/post-assets/gluoncv_iii_1.png)
{: refdef}
<br>


She really enjoyed it!

{% include youtubePlayer.html id=page.youtubeId %}
<br>
Thanks for reading